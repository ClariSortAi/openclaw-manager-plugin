# OpenClaw Security Checklist

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

### 3. DM Policy
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

### 4. Group Policy
```bash
# Secure: Allowlist only
openclaw config set channels.slack.groupPolicy allowlist

# Require mentions in groups
openclaw config set channels.slack.requireMention true
```

### 5. Sandbox Mode
```bash
# Enable sandboxing for all sessions
openclaw config set agents.defaults.sandbox.mode all

# Restrict workspace access
openclaw config set agents.defaults.sandbox.workspaceAccess none
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

## Security Hardening Checklist

### Network Security
- [ ] Gateway bound to loopback (127.0.0.1)
- [ ] Token authentication enabled
- [ ] Strong, random gateway token
- [ ] Tailscale used for remote access (not LAN binding)
- [ ] mDNS in minimal mode

### Access Control
- [ ] DM policy set to `pairing` or `allowlist`
- [ ] Group policy set to `allowlist`
- [ ] Mention required in groups
- [ ] Elevated tool access restricted
- [ ] Unknown senders cannot execute commands

### Sandboxing
- [ ] Sandbox mode enabled (`all` or `non-main`)
- [ ] Workspace access restricted (`none` or `ro`)
- [ ] Docker isolation configured (if applicable)

### Credentials
- [ ] File permissions set to 600/700
- [ ] No credentials in environment variables (use config)
- [ ] API keys stored via `openclaw configure`
- [ ] Tokens rotated periodically

### Logging & Monitoring
- [ ] Sensitive data redaction enabled
- [ ] Log retention policy defined
- [ ] Session transcripts secured

## Configuration Examples

### Minimal Secure Setup
```json
{
  "gateway": {
    "bind": "loopback",
    "auth": {
      "mode": "token"
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
      "sandbox": {
        "mode": "all",
        "workspaceAccess": "ro"
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
      "sandbox": {
        "mode": "all",
        "workspaceAccess": "none"
      },
      "tools": {
        "elevated": {
          "enabled": false
        }
      }
    }
  },
  "session": {
    "dmScope": "per-channel-peer"
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

### Attack Vectors
1. **Prompt Injection** - Malicious content in messages
2. **Social Engineering** - Tricking bot into harmful actions
3. **Credential Theft** - Accessing stored credentials
4. **Session Hijacking** - Cross-user context leakage

### Mitigations
- Strict access control (pairing/allowlist)
- Sandboxing for untrusted users
- Session isolation (per-peer scope)
- Disable elevated tools for groups
- Use modern, instruction-hardened models

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

# 2. Regenerate API keys via providers

# 3. Re-authenticate channels
openclaw channels logout
openclaw channels login
```

### Investigation
```bash
# Check logs
cat /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

# Review sessions
ls ~/.openclaw/agents/main/sessions/

# Full audit
openclaw security audit --deep
```

## Reporting Vulnerabilities

Email: security@openclaw.ai

Use responsible disclosure - avoid public discussion before fixes are available.
