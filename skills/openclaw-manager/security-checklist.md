# OpenClaw Security Checklist

## Critical Version Requirement

**Minimum safe version: v2026.1.29**

Run `openclaw status` to check your version. If you are on anything older than v2026.1.29, upgrade immediately:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw gateway restart
```

### Known Critical Vulnerabilities

| CVE | Severity | Description | Fixed In |
|-----|----------|-------------|----------|
| CVE-2026-25253 | **Critical (CVSS 8.8)** | One-click RCE: visiting a malicious site or clicking a crafted link leaks the gateway auth token, giving attackers full gateway control | v2026.1.29 |
| CVE-2026-24763 | High | Command injection via crafted input | v2026.1.29 |
| CVE-2026-25157 | High | Command injection via crafted input | v2026.1.29 |

Additional vulnerabilities covering SSRF, missing authentication, and path traversal were also patched in v2026.1.29. A January 2026 audit identified 512 total vulnerabilities (8 critical).

**Government advisories:**
- Belgium's Centre for Cybersecurity issued an emergency advisory classifying CVE-2026-25253 as critical
- The Dutch Data Protection Authority warned about serious cybersecurity and privacy risks and estimated ~20% of available plugins may contain malware
- Meta banned OpenClaw over security concerns

### Breaking Change: `auth: "none"` Permanently Removed

As of v2026.1.29, `gateway.auth.mode: "none"` is no longer accepted. All gateways require authentication. If you had `"none"` in your config, the gateway will refuse to start. Fix:

```bash
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
openclaw gateway restart
```

### Anthropic OAuth Tokens Blocked

Anthropic has officially blocked OpenClaw from using Claude OAuth tokens. You must use direct API keys:

```bash
# This will NOT work:
openclaw models auth  # OAuth flow for Anthropic is blocked

# Use API keys directly instead:
openclaw models auth setup-token --provider anthropic
# Then paste your API key from console.anthropic.com
```

## Quick Audit

```bash
openclaw security audit --deep
```

## Critical Settings

### 1. Gateway Binding
```bash
# Check current binding
openclaw config get gateway.bind

# Secure: Local only (default)
openclaw config set gateway.bind loopback

# Less secure: LAN access
openclaw config set gateway.bind 0.0.0.0  # NOT recommended
```

**Recommendation:** Keep `loopback` unless you need remote access. Use Tailscale for secure remote access.

### 2. Authentication Mode
```bash
# Use token auth (secure)
openclaw config set gateway.auth.mode token

# Generate strong token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
```

**Alternative: Trusted reverse proxy**
```bash
openclaw config set gateway.auth.mode trusted-proxy
# Requires an authenticated reverse proxy with TLS in front of the gateway
```

### 3. Tailscale Authentication

When `gateway.auth.allowTailscale` is enabled, OpenClaw verifies identity headers (`tailscale-user-login`) by resolving the client IP through the local Tailscale daemon:

```bash
openclaw config set gateway.auth.allowTailscale true
```

This allows secure remote access without exposing the gateway to the public internet. Tailscale nodes are verified against the local daemon rather than trusting headers blindly.

### 4. DM Policy
```bash
# Check current policy
openclaw config get channels.<channel>.dmPolicy

# Secure: Require pairing (default)
openclaw config set channels.slack.dmPolicy pairing

# Alternative: Allowlist only
openclaw config set channels.slack.dmPolicy allowlist
openclaw config set channels.slack.allowFrom '["U12345678"]'
```

**Never use `open` policy without understanding the risks.**

### 5. Group Policy
```bash
# Secure: Allowlist only
openclaw config set channels.slack.groupPolicy allowlist

# Require mentions in groups
openclaw config set channels.slack.requireMention true
```

### 6. Sandbox Mode

OpenClaw supports three workspace access levels:

| Level | Mount | Description |
|-------|-------|-------------|
| `none` | Inaccessible | Agent cannot access workspace |
| `ro` | `/agent` (read-only) | Agent can read but not write |
| `rw` | `/workspace` (read/write) | Agent has full workspace access |

Sandbox scope options:

| Scope | Description |
|-------|-------------|
| `agent` | Per-agent container (recommended) |
| `session` | Per-session container |
| `shared` | Single container for all (NOT recommended) |

```bash
# Enable sandboxing for all sessions
openclaw config set agents.defaults.sandbox.mode all

# Restrict workspace access
openclaw config set agents.defaults.sandbox.workspaceAccess none

# Set sandbox scope
openclaw config set agents.defaults.sandbox.scope agent
```

### 7. Control Plane Tool Denials

In production, deny high-risk tools that allow agents to modify the gateway itself:

```json
{
  "agents": {
    "defaults": {
      "tools": {
        "deny": ["gateway", "cron", "sessions_spawn", "sessions_send"]
      }
    }
  }
}
```

Also restrict elevated tools (`exec`, `browser`, `web_fetch`, `web_search`) to trusted agents or use tool allowlists:

```bash
openclaw config set agents.defaults.tools.deny '["gateway","cron","sessions_spawn","sessions_send"]'
```

### 8. mDNS Discovery

OpenClaw can broadcast gateway presence via mDNS. Three modes:

| Mode | Description |
|------|-------------|
| `minimal` | Recommended. Omits `cliPath` and `sshPort` |
| `off` | Disabled entirely |
| `full` | Includes operational details (NOT recommended) |

```bash
# Set minimal mode (recommended)
openclaw config set gateway.mdns.mode minimal

# Or disable entirely
openclaw config set gateway.mdns.mode off

# Alternatively, disable via environment variable
export OPENCLAW_DISABLE_BONJOUR=1
```

### 9. Model Selection

Use modern, instruction-hardened models for bots with tool access:

- **Anthropic Opus 4.6**: Recommended for strong prompt injection resistance and tool safety
- **Anthropic Sonnet 4.6**: Good balance of capability and speed
- Avoid older or weaker models for tool-enabled agents facing untrusted inboxes

```bash
openclaw config set agents.defaults.model "anthropic/claude-opus-4-6"
```

## File Permissions

### Check Permissions
```bash
ls -la ~/.openclaw
```

### Fix Permissions
```bash
# Config directory: owner only
chmod 700 ~/.openclaw

# Config file: owner read/write only
chmod 600 ~/.openclaw/openclaw.json

# Credentials: owner only
chmod -R 600 ~/.openclaw/credentials/
```

Use full-disk encryption on the gateway host for an additional layer of protection.

## Security Hardening Checklist

### Version & Patches
- [ ] Running v2026.1.29 or later (CVE-2026-25253 patched)
- [ ] `auth: "none"` not present in config (permanently removed)
- [ ] Using direct API keys, not Anthropic OAuth tokens

### Network Security
- [ ] Gateway bound to loopback (127.0.0.1)
- [ ] Token authentication enabled
- [ ] Strong, random gateway token
- [ ] Tailscale used for remote access (not LAN binding)
- [ ] mDNS set to `minimal` or `off`

### Access Control
- [ ] DM policy set to `pairing` or `allowlist`
- [ ] Group policy set to `allowlist`
- [ ] Mention required in groups
- [ ] Elevated tool access restricted
- [ ] Unknown senders cannot execute commands
- [ ] Control plane tools denied (`gateway`, `cron`, `sessions_spawn`, `sessions_send`)

### Sandboxing
- [ ] Sandbox mode enabled (`all` or `non-main`)
- [ ] Workspace access restricted (`none` or `ro`)
- [ ] Sandbox scope set to `agent` (not `shared`)
- [ ] Docker isolation configured (if applicable)

### Credentials
- [ ] File permissions set to 600/700
- [ ] No credentials in environment variables (use config)
- [ ] API keys stored via `openclaw configure`
- [ ] Tokens rotated monthly (API keys, email credentials, messaging tokens)
- [ ] Consider using a secret manager instead of environment files

### Logging & Monitoring
- [ ] Sensitive data redaction enabled
- [ ] Custom redaction patterns added via `logging.redactPatterns` if needed
- [ ] Log retention policy defined
- [ ] Session transcripts secured

### Skill & Plugin Validation
- [ ] Every ClawHub skill audited before installation (review source, check author, verify VirusTotal)
- [ ] Only trusted plugins installed (they run in-process with full privileges)
- [ ] Plugin allowlist configured via `plugins.allow`

## Configuration Examples

### Minimal Secure Setup
```json
{
  "gateway": {
    "bind": "loopback",
    "auth": {
      "mode": "token"
    },
    "mdns": {
      "mode": "minimal"
    }
  },
  "channels": {
    "slack": {
      "dmPolicy": "pairing",
      "groupPolicy": "disabled"
    }
  },
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-4-6",
      "sandbox": {
        "mode": "all",
        "workspaceAccess": "ro",
        "scope": "agent"
      },
      "tools": {
        "deny": ["gateway", "cron", "sessions_spawn", "sessions_send"]
      }
    }
  }
}
```

### Multi-User Setup (More Restrictive)
```json
{
  "gateway": {
    "bind": "loopback",
    "auth": {
      "mode": "token"
    },
    "mdns": {
      "mode": "off"
    }
  },
  "channels": {
    "slack": {
      "dmPolicy": "allowlist",
      "allowFrom": ["U12345", "U67890"],
      "groupPolicy": "allowlist",
      "groups": ["C11111"],
      "requireMention": true
    }
  },
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-4-6",
      "sandbox": {
        "mode": "all",
        "workspaceAccess": "none",
        "scope": "agent"
      },
      "tools": {
        "deny": ["gateway", "cron", "sessions_spawn", "sessions_send"],
        "elevated": {
          "enabled": false
        }
      }
    }
  },
  "session": {
    "dmScope": "per-channel-peer"
  },
  "plugins": {
    "allow": ["voice-call"]
  }
}
```

## Threat Model

### What OpenClaw Can Do
- Execute shell commands
- Read/write files
- Access network
- Send messages on your behalf
- Browse the web
- Spawn sub-agents (configurable depth)

### Attack Vectors
1. **Prompt Injection** - Malicious content in messages
2. **Social Engineering** - Tricking bot into harmful actions
3. **Credential Theft** - Accessing stored credentials
4. **Session Hijacking** - Cross-user context leakage
5. **Malicious Plugins/Skills** - ~20% of third-party plugins may contain malware (Dutch DPA estimate)
6. **Token Leakage** - CVE-2026-25253 demonstrated gateway token exfiltration via crafted links

### Mitigations
- Keep OpenClaw updated to latest version (minimum v2026.1.29)
- Strict access control (pairing/allowlist)
- Sandboxing for untrusted users
- Session isolation (per-peer scope)
- Disable elevated tools for groups
- Use modern, instruction-hardened models (Opus 4.6 recommended)
- Deny control plane tools in production
- Audit all third-party skills and plugins before installation
- Rotate credentials monthly
- Run in Docker container for process isolation

## Incident Response

### Containment
```bash
# 1. Stop gateway immediately
openclaw gateway stop

# 2. Lock down binding
openclaw config set gateway.bind loopback

# 3. Disable risky channels
openclaw config set channels.slack.dmPolicy disabled
```

### Credential Rotation
```bash
# 1. Rotate gateway token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"

# 2. Rotate remote/provider tokens
# Regenerate API keys via provider dashboards (console.anthropic.com, etc.)

# 3. Re-authenticate channels
openclaw channels logout
openclaw channels login

# 4. Rotate messaging platform tokens (Slack, Telegram, Discord)
# Generate new tokens from each platform's developer portal
```

### Investigation
```bash
# Check gateway logs
cat /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

# Review sessions
ls ~/.openclaw/agents/main/sessions/

# Check config changes
cat ~/.openclaw/openclaw.json

# Full audit
openclaw security audit --deep

# Machine-readable audit output
openclaw security audit --deep --json
```

## Reporting Vulnerabilities

Email: security@openclaw.ai

Use responsible disclosure - avoid public discussion before fixes are available.
