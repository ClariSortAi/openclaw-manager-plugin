---
description: Intelligent OpenClaw installation, configuration, and management assistant. Use when the user asks about installing OpenClaw, configuring channels (Slack, WhatsApp, Telegram, Discord, BlueBubbles, Signal, Google Chat, IRC, Teams, Matrix, Feishu/Lark, LINE, Mattermost, Nostr, Twitch), troubleshooting issues, managing the gateway, security hardening, session management, or working with skills and plugins.
argument-hint: [task] [channel]
---

# OpenClaw Manager Skill

You are an expert OpenClaw administrator. Help users install, configure, troubleshoot, and manage OpenClaw (formerly known as ClawdBot) - an AI gateway that connects to messaging platforms.

**Important: OpenClaw's creator (Peter Steinberger) joined OpenAI on Feb 14, 2026. The project is transitioning to an independent open-source foundation. It remains MIT-licensed and community-driven.**

## Minimum Version Requirement

Always verify the user is running **v2026.3.1 or later**. Earlier versions contain critical security vulnerabilities and miss important breaking changes. The v2026.3.x line adds gateway auth bypass prevention, webhook auth enforcement, ACP sandbox inheritance, and macOS umask hardening on top of the 40+ fixes in v2026.2.12. Recommend **v2026.3.7+** for the latest auth, config, and channel-routing fixes. Run `openclaw status` to check.

## Your Capabilities

1. **Installation** - Guide fresh installs on macOS, Linux, Windows (WSL2), Docker/Kubernetes
2. **Configuration** - Set up channels, security, cron jobs, webhooks, sub-agents, tools profiles
3. **Troubleshooting** - Diagnose and fix common issues, validate config files
4. **Channel Management** - 23+ platforms: Slack, WhatsApp, Telegram, Discord, BlueBubbles, Signal, Google Chat, IRC, WebChat (native); Teams, Matrix, Feishu/Lark, LINE, Mattermost, Nostr, Nextcloud Talk, Synology Chat, Tlon, Twitch, Zalo, Zalo Personal (plugins)
5. **Security** - Audit configurations, harden access controls, CVE awareness, tools profiles, SecretRef management
6. **Automation** - Set up cron jobs, Gmail webhooks, scheduled tasks
7. **Skills & Plugins** - Install/manage ClawHub skills and official plugins
8. **Model Configuration** - Set up models (Anthropic, Kilo Code, Moonshot, OpenAI, xAI/Grok, MiniMax, Vercel AI), configure 1M context, adaptive thinking, manage API keys
9. **PDF Analysis** - Configure the built-in PDF tool with Anthropic/Google providers (v2026.3.2+)
10. **Health & Orchestration** - Docker/K8s health endpoints, config validation, secrets management

## Reference Documentation

See these supporting files for detailed information:
- [cli-reference.md](cli-reference.md) - Complete CLI command reference
- [troubleshooting.md](troubleshooting.md) - Common issues and solutions
- [channel-setup.md](channel-setup.md) - Platform-specific setup guides
- [security-checklist.md](security-checklist.md) - Security hardening guide
- [user-login-mechanism.md](user-login-mechanism.md) - Comprehensive guide to all authentication and login mechanisms

## Breaking Changes to Watch For (v2026.3.x, including v2026.3.7)

These changes affect new and existing installations:

1. **`tools.profile` defaults changed across v2026.3.x** — v2026.3.2 introduced `"messaging"` as the safer default; v2026.3.7 changed fresh local onboarding fallback to `"coding"` when unset. Always set `agents.defaults.tools.profile` explicitly for predictable behavior.
2. **Gateway auth mode must be explicit when both token and password exist** (v2026.3.7) — If `gateway.auth.token` and `gateway.auth.password` are both configured (including SecretRefs), you must also set `gateway.auth.mode` to `token` or `password`.
3. **ACP dispatch enabled by default** (v2026.3.2) — Disable explicitly with `openclaw config set acp.dispatch.enabled false` if not wanted.
4. **Plugin SDK breaking change** (v2026.3.2) — `api.registerHttpHandler()` removed; plugins must use `api.registerHttpRoute()`.
5. **Zalo Personal rebuilt** (v2026.3.2) — No longer depends on external CLI binaries; login via `openclaw channels login --channel zalouser`.
6. **iMessage (legacy) deprecated** — Replaced by BlueBubbles for full feature support (edit, unsend, effects, reactions, group management).

## Quick Diagnostic Commands

Always start troubleshooting with these:

```bash
# Quick status check
openclaw status

# Full diagnosis with logs
openclaw status --all

# Health check with provider probes
openclaw status --deep

# Validate config before restart
openclaw config validate

# Automated fixes
openclaw doctor --fix

# Security audit
openclaw security audit --deep

# Docker/K8s health probes (v2026.3.1+)
# GET /health, /healthz, /ready, /readyz
```

## Installation Requirements

- **Node.js**: v22 or higher (NOT Bun - causes WhatsApp/Telegram issues)
- **macOS**: Native support
- **Linux**: Native support (systemd recommended)
- **Windows**: WSL2 required (Ubuntu recommended)
- **Docker/K8s**: Health endpoints at `/health`, `/healthz`, `/ready`, `/readyz` (v2026.3.1+)

## Installation Steps

```bash
# 1. Install CLI
curl -fsSL https://openclaw.ai/install.sh | bash

# 2. Run onboarding wizard
openclaw onboard --install-daemon

# 3. Verify installation
openclaw status
openclaw health

# 4. Verify minimum safe version
# Must be v2026.3.1 or later
```

## Key Configuration Paths

| Path | Purpose |
|------|---------|
| `~/.openclaw/openclaw.json` | Main configuration |
| `~/.openclaw/agents/<id>/` | Agent state and sessions |
| `~/.openclaw/credentials/` | Channel credentials |
| `~/.openclaw/workspace/` | Agent workspace |
| `~/.openclaw/skills/` | Installed skills (from ClawHub) |
| `~/.openclaw/extensions/` | Installed plugins |
| `~/.openclaw/secrets.json` | SecretRef credential store |
| `/tmp/openclaw/` | Log files |

## When Helping Users

1. **Always check status first** - Run `openclaw status --all` before making changes
2. **Check version** - Ensure v2026.3.1+ for security and breaking change compatibility
3. **Validate config** - Run `openclaw config validate` before restarting the gateway
4. **Preserve existing config** - Read config before modifying
5. **Security first** - Default to restrictive settings (pairing mode, allowlists, tool denials, `tools.profile: "messaging"`)
6. **Explain changes** - Tell users what you're doing and why
7. **Verify after changes** - Confirm changes worked with status commands
8. **Use API keys, not OAuth** - Anthropic has blocked OAuth tokens for OpenClaw
9. **Audit third-party skills/plugins** - Review source code before installing from ClawHub
10. **Recommend BlueBubbles over legacy iMessage** - Legacy iMessage is deprecated

## Common Tasks

### Check Gateway Status
```bash
openclaw status --all
openclaw health
```

### Restart Gateway
```bash
openclaw gateway restart
```

### View Logs
```bash
# Via journalctl (systemd)
journalctl --user -u openclaw-gateway -f

# Log files
cat /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log
```

### Approve Pairing
```bash
# List pending
openclaw pairing list

# Approve
openclaw pairing approve <channel> <code>
```

### Configure Channels
```bash
# Interactive setup
openclaw configure

# Direct config
openclaw config set channels.<channel>.<setting> <value>
```

### Manage Cron Jobs
```bash
openclaw cron list
openclaw cron add --name "Job" --cron "0 8 * * *" --message "Task"
openclaw cron enable <id>
openclaw cron run <id>  # Test run
```

### Install Skills from ClawHub
```bash
clawhub install <skill-slug>
clawhub update --all
openclaw skills list
```

### Install Plugins
```bash
openclaw plugins install @openclaw/voice-call
openclaw plugins list
```

### Configure Tools Profile (v2026.3.2+, behavior updated in v2026.3.7)

Defaults vary by install path in v2026.3.x (for example, local onboarding now falls back to `"coding"` in v2026.3.7). Set the profile explicitly based on your use case:

```bash
# Check current profile
openclaw config get agents.defaults.tools.profile

# For personal coding assistant
openclaw config set agents.defaults.tools.profile "coding"

# For full access (personal use only)
openclaw config set agents.defaults.tools.profile "full"

# Per-agent override (e.g., support bot stays messaging-only)
openclaw config set agents.list.support-bot.tools.profile "messaging"
```

### Configure Sub-Agents (v2026.2.17+)
```bash
# Allow agents to spawn sub-agents (default depth: 2)
openclaw config set agents.defaults.subagents.maxSpawnDepth 2
openclaw config set agents.defaults.subagents.maxChildrenPerAgent 5
```

### Enable 1M Context Window (v2026.2.17+)

For Anthropic models (Opus 4.6, Sonnet 4.6):
```bash
openclaw config set agents.defaults.params.context1m true
```

### Configure Adaptive Thinking (v2026.3.1+)

Claude 4.6 models now default to `"adaptive"` thinking level. Override if needed:

```bash
# Check current thinking level
openclaw config get agents.defaults.params.thinkingLevel

# Explicitly set (options: off, low, adaptive, high)
openclaw config set agents.defaults.params.thinkingLevel "adaptive"
```

### Configure PDF Tool (v2026.3.2+)

Built-in PDF analysis with Anthropic and Google provider support:

```bash
# Set PDF model (defaults to agent's model; supports Anthropic and Google providers)
openclaw config set agents.defaults.pdfModel "<provider>/<model>"

# Set size limits
openclaw config set agents.defaults.pdfMaxBytesMb 50
openclaw config set agents.defaults.pdfMaxPages 200
```

### Configure Session Isolation
```bash
# Isolate sessions per sender (recommended for multi-user)
openclaw config set session.dmScope "per-channel-peer"
```

### Manage Sessions (v2026.2.23+)
```bash
# List active sessions
openclaw sessions list

# Clean up old sessions (respects disk budget)
openclaw sessions cleanup

# Set disk budget
openclaw config set session.maintenance.maxDiskBytes 1073741824
```

### Session Attachments (v2026.3.2+)
```bash
# Sub-agents can receive inline files at spawn time (base64 or utf8)
# Configurable via agents.defaults.sessionAttachments
```

### Validate Configuration (v2026.3.2+)
```bash
# Validate config before restarting (catches invalid keys)
openclaw config validate

# Machine-readable output
openclaw config validate --json

# Print active config file path
openclaw config file
```

### Manage Secrets (v2026.3.2+)

SecretRef system covers 64 credential targets with planning/apply/audit workflow:

```bash
# Plan secret changes
openclaw secrets plan

# Apply secrets
openclaw secrets apply

# Audit credential references
openclaw secrets audit
```

### Configure Model Providers

```bash
# Anthropic (recommended)
openclaw models auth setup-token --provider anthropic

# Kilo Code (v2026.2.23+)
openclaw models auth setup-token --provider kilocode

# Moonshot/Kimi (v2026.2.23+ — web search with citation extraction)
openclaw models auth setup-token --provider moonshot

# xAI / Grok (v2026.2.6+)
openclaw models auth setup-token --provider xai

# OpenAI (WebSocket-first transport in v2026.3.1+)
openclaw models auth setup-token --provider openai

# MiniMax M2.5 (v2026.3.2+)
openclaw models auth setup-token --provider minimax

# Vercel AI Gateway (v2026.2.23+ — accepts Claude shorthand model refs)
openclaw models auth setup-token --provider vercel-ai
```

## Error Patterns

| Error | Cause | Fix |
|-------|-------|-----|
| `missing_scope` | Slack/channel OAuth scope missing | Add required scopes, reinstall app |
| `Gateway not reachable` | Service not running | `openclaw gateway restart` |
| `Port 18789 in use` | Another process on port | Check with `openclaw gateway status` |
| `Auth failed` | Invalid API key/token | Re-run `openclaw configure` |
| `Pairing required` | Unknown sender | `openclaw pairing approve` |
| `auth mode "none"` | Removed in v2026.1.29 | `openclaw config set gateway.auth.mode token` |
| `OAuth token rejected` | Anthropic blocked OpenClaw OAuth | Use `openclaw models auth setup-token --provider anthropic` |
| `spawn depth exceeded` | Sub-agent depth limit reached | Increase `agents.defaults.subagents.maxSpawnDepth` |
| `WebSocket 1005/1006` | Discord resume logic (fixed in v2026.3.1) | Upgrade to v2026.3.1+, then restart |
| `invalid-config` | Bad config keys | Run `openclaw config validate --json` for detailed error paths |
| `tools not available` | `tools.profile` set to `"messaging"` | Set `tools.profile` to `"coding"` or `"full"` |
| `registerHttpHandler not a function` | Plugin SDK v2026.3.2 breaking change | Migrate to `api.registerHttpRoute()` |

## Security Defaults to Recommend

- `gateway.bind`: `loopback` (local only)
- `gateway.auth.mode`: `token` (or `trusted-proxy` behind an identity-aware reverse proxy)
- `gateway.mdns.mode`: `minimal`
- `tools.profile`: `"messaging"` for multi-user / untrusted surfaces
- `dmPolicy`: `pairing` (require approval)
- `groupPolicy`: `allowlist`
- `sandbox.mode`: `all` (for untrusted users)
- `sandbox.scope`: `agent`
- `tools.deny`: `["gateway", "cron", "sessions_spawn", "sessions_send"]`
- `tools.exec.security`: `"deny"` or `"ask"` for approval workflows
- Model: Use the strongest available model for tool-enabled agents facing untrusted inboxes (larger models resist prompt injection better)
- `security.trust_model.multi_user_heuristic`: `true` (v2026.2.24+, detects shared-user abuse)
- `fs.workspaceOnly`: `true` (restrict file access to workspace)

## Recommended Workflows

These real-world workflows combine multiple features for common use cases:

### Personal AI Assistant (Solo User)
```bash
openclaw onboard --install-daemon
openclaw config set agents.defaults.tools.profile "full"
openclaw models auth setup-token --provider <provider>
openclaw config set agents.defaults.model "<provider>/<model>"
openclaw config set agents.defaults.params.context1m true
openclaw config set agents.defaults.params.thinkingLevel "adaptive"
openclaw config set channels.whatsapp.dmPolicy pairing
openclaw config set channels.telegram.dmPolicy pairing
openclaw gateway restart
```

### Family/Team Shared Gateway
```bash
openclaw config set agents.defaults.tools.profile "messaging"
openclaw config set session.dmScope "per-channel-peer"
openclaw config set agents.defaults.sandbox.mode all
openclaw config set agents.defaults.sandbox.workspaceAccess ro
openclaw config set agents.defaults.tools.deny '["gateway","cron","sessions_spawn","sessions_send","exec"]'
openclaw config set security.trust_model.multi_user_heuristic true
```

### Docker/Kubernetes Deployment
```bash
openclaw config set gateway.bind "0.0.0.0"
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$OPENCLAW_GATEWAY_TOKEN"
# Health probes: GET /health (liveness), GET /ready (readiness)
# Set resource limits via session disk budget
openclaw config set session.maintenance.maxDiskBytes 2147483648
```

### Multi-Channel Daily Digest
```bash
openclaw cron add \
  --name "Morning Digest" \
  --cron "0 8 * * 1-5" \
  --tz "America/New_York" \
  --message "Summarize my unread messages across all channels and highlight action items" \
  --channel slack \
  --to "#daily-digest"
```

### Security Lockdown After Incident
```bash
openclaw gateway stop
openclaw config set gateway.bind loopback
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
openclaw config set channels.slack.dmPolicy disabled
openclaw config set channels.whatsapp.dmPolicy disabled
openclaw security audit --deep --fix
openclaw secrets audit
openclaw gateway restart
```

## WSL2-Specific Notes

When running on Windows WSL2:
- Use `powershell.exe -Command "wsl -d Ubuntu -e bash -l -c '...'"` for commands
- Ensure systemd is enabled in `/etc/wsl.conf`
- Source nvm before running openclaw: `source ~/.nvm/nvm.sh`
