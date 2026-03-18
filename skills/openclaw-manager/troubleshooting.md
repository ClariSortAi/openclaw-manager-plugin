# OpenClaw Troubleshooting Guide

## Diagnostic Workflow

Always follow this order:

```bash
# 1. Quick status (check version is v2026.3.1+, recommend v2026.3.13+ / tag `v2026.3.13-1`)
openclaw status

# 2. Validate config (catches invalid keys — v2026.3.2+)
openclaw config validate

# 3. Full diagnosis
openclaw status --all

# 4. Deep health check
openclaw status --deep

# 5. Auto-fix
openclaw doctor --fix

# 6. Live logs
journalctl --user -u openclaw-gateway -f
```

## Critical: Version Check

Before troubleshooting anything else, verify you are on **v2026.3.1 or later** (recommend **v2026.3.13+**, released on GitHub as tag `v2026.3.13-1`):

```bash
openclaw status
```

If on an older version, upgrade immediately — the v2026.3.x line adds critical security hardening (gateway auth bypass prevention, webhook auth enforcement, ACP sandbox inheritance) on top of the 40+ fixes in v2026.2.12:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw config validate
openclaw gateway restart
```

If you need `openclaw backup` commands or Talk silence timeout tuning, upgrade to **v2026.3.8+**. For latest security hardening, gateway RPC probe controls, plugin collision safeguards, and pairing/webhook fixes, upgrade to **v2026.3.13+** (GitHub tag `v2026.3.13-1`).

## Common Issues

### Gateway Issues

#### Gateway Not Running
**Symptoms:** `openclaw status` shows gateway not reachable

**Fix:**
```bash
openclaw gateway restart
# Or manually:
openclaw gateway start
```

#### Gateway Reports Running But RPC Is Unavailable
**Symptoms:** Service appears active, but automation or CLI calls still fail/hang when they require RPC.

**Diagnose:**
```bash
# v2026.3.13+: fail hard if RPC is unavailable (scope-limited probe RPC is treated as degraded)
openclaw gateway status --require-rpc
```

**Fix:**
```bash
openclaw doctor --fix
openclaw gateway restart
openclaw gateway status --require-rpc
```

#### Port 18789 Already in Use
**Symptoms:** Gateway fails to start, port conflict

**Diagnose:**
```bash
openclaw gateway status
# Check what's using the port
lsof -i :18789
```

**Fix:**
```bash
# Stop existing process or use different port
openclaw config set gateway.port 18790
openclaw gateway restart
```

#### Gateway Mode Not Set
**Symptoms:** `Gateway start blocked`

**Fix:**
```bash
openclaw config set gateway.mode local
openclaw gateway restart
```

#### Gateway Refuses to Start After Upgrade (auth: "none" Removed)
**Symptoms:** Gateway exits immediately after upgrading to v2026.1.29+

**Cause:** `gateway.auth.mode` was set to `"none"`, which is permanently removed.

**Fix:**
```bash
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
openclaw gateway restart
```

### Authentication Issues

#### Startup/Pairing Fails After Upgrade (Explicit `gateway.auth.mode` Required)
**Symptoms:** Startup, pairing, or TUI auth errors after upgrading to v2026.3.7+

**Cause:** Both `gateway.auth.token` and `gateway.auth.password` are configured (including SecretRefs) but `gateway.auth.mode` is not explicitly set.

**Fix:**
```bash
# Pick one auth mode explicitly
openclaw config set gateway.auth.mode token
# or
openclaw config set gateway.auth.mode password

openclaw config validate
openclaw gateway restart
```

#### No API Key Found
**Symptoms:** Agent can't make requests, auth errors

**Fix:**
```bash
# Re-run auth setup
openclaw configure
# Or set directly
openclaw models auth setup-token --provider anthropic
```

#### Anthropic OAuth Token Rejected
**Symptoms:** `OAuth token rejected`, `unauthorized`, or auth failures when using Anthropic models

**Cause:** Anthropic has officially blocked OpenClaw from using Claude OAuth tokens. You must use direct API keys instead.

**Fix:**
```bash
# Switch to direct API key (get one from console.anthropic.com)
openclaw models auth setup-token --provider anthropic

# Verify it works
openclaw status --deep
```

**Note:** This is not a bug -- Anthropic blocked OAuth access for OpenClaw as a policy decision. The only supported method is direct API keys.

#### OAuth Token Refresh Failed (Non-Anthropic Providers)
**Symptoms:** Token expired errors for non-Anthropic providers

**Fix:**
```bash
# Re-authenticate with the provider
openclaw models auth setup-token --provider <provider-name>
# Or re-run configure
openclaw configure
```

### Channel Issues

#### Slack: missing_scope Error
**Symptoms:** Bot receives messages but can't reply

**Cause:** Missing OAuth scopes (often `im:write`)

**Fix:**
1. Go to api.slack.com/apps → Your app
2. OAuth & Permissions → Add missing scopes
3. Reinstall to Workspace
4. Update token: `openclaw config set channels.slack.botToken "xoxb-..."`
5. Restart: `openclaw gateway restart`

**Required Slack Scopes:**
- `chat:write`, `im:write`, `channels:history`, `channels:read`
- `groups:history`, `im:history`, `mpim:history`
- `users:read`, `app_mentions:read`
- `reactions:read`, `reactions:write`

#### WhatsApp: Not Linked
**Symptoms:** `channels status` shows `linked: false`

**Fix:**
```bash
openclaw channels login
# Scan QR in WhatsApp → Settings → Linked Devices
```

#### WhatsApp: Disconnected Loop
**Symptoms:** Keeps reconnecting, not stable

**Causes:**
- Using Bun instead of Node (Bun has issues)
- Network instability
- WhatsApp account issues

**Fix:**
```bash
# Ensure using Node, not Bun
which node
node --version  # Should be v22.16.0+

# Restart gateway
openclaw gateway restart

# If persistent, re-link
openclaw channels logout
openclaw channels login
```

#### Telegram: Bot Not Responding
**Symptoms:** Messages not processed

**Check:**
```bash
openclaw channels status
openclaw config get channels.telegram
```

**Fix:**
```bash
# Verify token is set
openclaw config set channels.telegram.botToken "123:abc..."
openclaw gateway restart
```

#### Telegram: Inbound Media Attachments Fail Intermittently
**Symptoms:** Telegram text messages work, but inbound media (images/files) intermittently fails to process or download.

**Cause:** Older builds may hit transport-policy and network-path edge cases during Telegram media retrieval. v2026.3.13 improves this path with tighter transport handling and IPv4 fallback retries.

**Fix:**
```bash
# Upgrade to the current stable line
curl -fsSL https://openclaw.ai/install.sh | bash

# Restart and retest media delivery
openclaw gateway restart
openclaw channels status
```

If failures persist behind a webhook endpoint, verify Telegram webhook secret configuration; v2026.3.13+ rejects invalid/missing secrets before request body parsing.

#### iMessage: Not Working (Migrate to BlueBubbles)
**Symptoms:** iMessage channel not receiving messages

**Important:** Legacy iMessage is deprecated. Migrate to BlueBubbles for full feature support.

**For BlueBubbles issues:**
```bash
# Check BlueBubbles server is running
openclaw channels status

# Verify server URL and password
openclaw config get channels.bluebubbles
```

**If still using legacy iMessage:**
- Not on macOS (iMessage is macOS only)
- Full Disk Access not granted
- Messages app not signed in

```bash
# Verify macOS
uname -s  # Must be "Darwin"

# Grant Full Disk Access
# System Settings → Privacy & Security → Full Disk Access → Add Terminal

# Enable and restart
openclaw config set channels.imessage.enabled true
openclaw gateway restart
```

#### Discord: WebSocket Disconnects (Fixed in v2026.3.1)
**Symptoms:** Bot goes offline for 30+ minutes, WebSocket error 1005 or 1006 in logs

**Cause:** Fixed in v2026.3.1 — Discord WebSocket resume logic used to fail on certain disconnects in v2026.2.24.

**Fix:**
```bash
# Upgrade to v2026.3.1+
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw gateway restart
```

If still on v2026.2.24, force restart: `openclaw gateway restart`

#### Teams: Plugin Not Working
**Symptoms:** Teams channel not available

**Cause:** Teams is a plugin-only channel since v2026.1.15

**Fix:**
```bash
# Install the Teams plugin
openclaw plugins install @openclaw/msteams

# Check plugin status
openclaw plugins info msteams
openclaw channels status
```

### Pairing Issues

#### Messages Not Triggering (Pairing Required)
**Symptoms:** New users can't interact with bot

**Check:**
```bash
openclaw pairing list
```

**Fix:**
```bash
# Approve pending requests
openclaw pairing approve <channel> <code>

# Or switch to allowlist mode
openclaw config set channels.<channel>.dmPolicy allowlist
openclaw config set channels.<channel>.allowFrom '["user1", "user2"]'
```

#### Pairing Code Not Arriving
**Symptoms:** User doesn't receive pairing code

**Causes:**
- Hit 3-request cap (pairing codes expire after 1 hour, max 3 pending)
- Channel not connected
- Code already consumed (v2026.3.13+ setup/bootstrap codes are single-use)

**Fix:**
```bash
# Check channel status
openclaw channels status

# Clear pending and retry
openclaw pairing list
```

### Plugin Issues

#### Plugin Not Loading
**Symptoms:** Installed plugin doesn't appear in `plugins list`

**Fix:**
```bash
# Check plugin health
openclaw plugins doctor

# Verify plugin is enabled
openclaw config get plugins.entries.<plugin-id>.enabled

# Enable if needed
openclaw plugins enable <plugin-id>
openclaw gateway restart
```

#### Plugin Conflict (Slot Error)
**Symptoms:** Error about conflicting plugins in same slot

**Cause:** Two plugins competing for an exclusive slot (e.g., both memory plugins)

**Fix:**
```bash
# Choose one plugin for the slot
openclaw config set plugins.slots.memory "memory-core"
# Or
openclaw config set plugins.slots.memory "memory-lancedb"
openclaw gateway restart
```

#### Plugin Channel/Binding Collision
**Symptoms:** Plugin install/startup fails with channel or binding collision errors.

**Cause:** v2026.3.13 fail-fast checks now reject channel and network binding conflicts immediately.

**Fix:**
```bash
# Inspect current plugin/channel state
openclaw plugins list
openclaw channels list

# Disable or remove conflicting plugin/channel before retrying
openclaw plugins disable <plugin-id>
# or
openclaw plugins remove <plugin-id>

openclaw gateway restart
```

### Skill Issues

#### ClawHub Skill Not Working
**Symptoms:** Installed ClawHub skill not available

**Fix:**
```bash
# Check skill requirements
openclaw skills check

# Verify skill is enabled
openclaw config get skills.entries.<skill-name>.enabled

# Check if skill requires specific binaries/env
openclaw skills info <skill-name>
```

### Cron Job Issues

#### Cron Job Not Running
**Symptoms:** Scheduled job doesn't execute at expected time

**Diagnose:**
```bash
# Check job status and next run time
openclaw cron status
openclaw cron list

# View run history for the job
openclaw cron runs
```

**Common causes:**
- Wrong timezone: verify with `openclaw cron list` -- use `--tz` flag when creating jobs
- Gateway not running: cron requires an active gateway
- Job disabled: `openclaw cron enable <id>`

**Fix:**
```bash
# Test job manually first
openclaw cron run <id>

# Fix timezone if needed
openclaw cron edit <id>
```

#### Cron Notifications Missing After Upgrade (v2026.3.11+)
**Symptoms:** Cron jobs execute, but expected notify/webhook fallback behavior no longer appears.

**Cause:** v2026.3.11 tightened isolated cron delivery semantics and requires migration of legacy notify/webhook metadata.

**Fix:**
```bash
# Run post-upgrade migration
openclaw doctor --fix

# Re-check cron definitions and run history
openclaw cron list
openclaw cron runs
```

#### Isolated Cron Jobs Stall or Hang (Fixed in v2026.3.13+ / tag `v2026.3.13-1`)
**Symptoms:** Isolated cron jobs occasionally stop progressing, especially when nested lane execution is involved.

**Cause:** Older v2026.3.x builds could deadlock in isolated cron nested-lane scheduling paths.

**Fix:**
```bash
# Upgrade to current stable line
curl -fsSL https://openclaw.ai/install.sh | bash

# Re-run migration and validate scheduler behavior
openclaw doctor --fix
openclaw cron runs
```

#### Cron Webhook SSRF (Security)
**Note:** CVE-2026-27488 (patched in v2026.2.19) allowed cron webhook targets to reach private/internal endpoints. Ensure you are on v2026.2.19+ if using cron webhooks.

### Sub-Agent Issues (v2026.2.17+)

#### Sub-Agent Spawn Fails
**Symptoms:** Agent cannot create sub-agents, "spawn depth exceeded" errors

**Diagnose:**
```bash
openclaw config get agents.defaults.subagents.maxSpawnDepth
openclaw config get agents.defaults.subagents.maxChildrenPerAgent
```

**Fix:**
```bash
# Increase spawn depth (default: 2)
openclaw config set agents.defaults.subagents.maxSpawnDepth 3

# Increase children per agent (default: 5)
openclaw config set agents.defaults.subagents.maxChildrenPerAgent 10

openclaw gateway restart
```

#### Sub-Agent Not Responding
**Symptoms:** Spawned sub-agent hangs or produces no output

**Common causes:**
- Model rate limits exceeded (sub-agents make additional API calls)
- Sandbox restrictions blocking sub-agent tools
- Insufficient context window for the task

**Fix:**
```bash
# Check agent status
openclaw agents list

# Review logs for sub-agent errors
openclaw logs | grep -i "sub-agent\|spawn"
```

### Session Management Issues (v2026.2.23+)

#### Disk Space Growing from Sessions
**Symptoms:** `~/.openclaw/agents/` directory consuming excessive disk

**Fix:**
```bash
# Clean up old sessions
openclaw sessions cleanup

# Set disk budget to auto-manage
openclaw config set session.maintenance.maxDiskBytes 1073741824
openclaw config set session.maintenance.highWaterBytes 858993459
```

### Tools Profile Issues (v2026.3.2+)

#### Agent Can't Run Commands / "Tools Not Available"
**Symptoms:** Agent refuses to execute shell commands, read files, or use browser tools

**Cause:** `tools.profile` is currently set to `"messaging"` (chat-only tools). In v2026.3.x, defaults can differ by onboarding path, so rely on the configured value, not assumptions.

**Fix:**
```bash
# Check current profile
openclaw config get agents.defaults.tools.profile

# Set to coding or full
openclaw config set agents.defaults.tools.profile "coding"
openclaw gateway restart
```

**Per-agent override** (keep restrictive default, open up for specific agents):
```bash
openclaw config set agents.list.my-dev-agent.tools.profile "full"
```

### Config Validation Issues (v2026.3.2+)

#### Gateway Refuses to Start with Invalid Config
**Symptoms:** Gateway exits with "invalid-config" error

**Diagnose:**
```bash
# Get detailed error paths
openclaw config validate --json
```

**Fix:** Review the invalid key paths in the output and correct them:
```bash
# Check what config file is active
openclaw config file

# Fix the invalid keys
openclaw config unset <invalid.path>
openclaw config validate
openclaw gateway restart
```

### Backup & Recovery Command Issues (v2026.3.8+)

#### `openclaw backup` Command Not Found
**Symptoms:** `unknown command "backup"` or similar CLI error

**Cause:** Running OpenClaw older than v2026.3.8.

**Fix:**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw status
openclaw backup create --only-config
```

### ACP Dispatch Issues (v2026.3.2+)

#### Unexpected ACP Behavior
**Symptoms:** Agent Communication Protocol dispatching messages unexpectedly

**Cause:** ACP dispatch is now enabled by default in v2026.3.2.

**Fix:**
```bash
# Disable ACP dispatch if not wanted
openclaw config set acp.dispatch.enabled false
openclaw gateway restart
```

### PDF Tool Issues (v2026.3.2+)

#### PDF Analysis Not Working
**Symptoms:** PDF tool returns errors or no results

**Diagnose:**
```bash
# Check PDF tool configuration
openclaw config get agents.defaults.pdfModel
openclaw config get agents.defaults.pdfMaxBytesMb
```

**Fix:**
```bash
# Ensure a supported model is configured (Anthropic or Google)
openclaw config set agents.defaults.pdfModel "anthropic/claude-opus-4-6"

# Increase size limits if needed
openclaw config set agents.defaults.pdfMaxBytesMb 50
openclaw config set agents.defaults.pdfMaxPages 200
```

### Plugin SDK Breaking Change (v2026.3.2)

#### Plugin Error: "registerHttpHandler is not a function"
**Symptoms:** Plugin crashes on startup after upgrading to v2026.3.2

**Cause:** `api.registerHttpHandler()` was removed. Plugins must use `api.registerHttpRoute()`.

**Fix:** Update the plugin code to use the new API:
```javascript
// Old (removed):
// api.registerHttpHandler('/webhook', handler)

// New (v2026.3.2+):
api.registerHttpRoute({ path: '/webhook', method: 'POST', handler })
```

### Zalo Personal Issues (v2026.3.2)

#### Zalo Personal Login Changed
**Symptoms:** Old Zalo Personal login method no longer works

**Cause:** Rebuilt in v2026.3.2 using native `zca-js` — no external CLI dependency.

**Fix:**
```bash
openclaw channels login --channel zalouser
```

### Service Issues (systemd)

#### Service Not Starting
**Check:**
```bash
systemctl --user status openclaw-gateway
journalctl --user -u openclaw-gateway -n 50
```

**Fix:**
```bash
# Reinstall service
openclaw onboard --install-daemon

# Or manually enable
systemctl --user enable openclaw-gateway
systemctl --user start openclaw-gateway
```

#### Service Crashes on Restart
**Check logs:**
```bash
journalctl --user -u openclaw-gateway -f
```

**Common causes:**
- Config syntax error: `openclaw config validate --json`
- Port conflict: Check other processes
- Missing dependencies: Reinstall
- `auth: "none"` in config after upgrade (see above)
- Invalid config keys: `openclaw config validate` for detailed error paths

### WSL2-Specific Issues

#### Commands Not Found
**Cause:** nvm/node not in path

**Fix:**
```bash
# Always source nvm first
source ~/.nvm/nvm.sh
openclaw status
```

#### systemd Not Available
**Check:**
```bash
ps -p 1 -o comm=  # Should show "systemd"
```

**Fix:** Enable systemd in WSL
```bash
sudo nano /etc/wsl.conf
# Add:
# [boot]
# systemd=true

# Then in PowerShell:
wsl --shutdown
wsl -d Ubuntu
```

## Log Locations

| Location | Content |
|----------|---------|
| `journalctl --user -u openclaw-gateway` | systemd service logs |
| `/tmp/openclaw/openclaw-YYYY-MM-DD.log` | Gateway log file |
| `~/.openclaw/agents/<id>/sessions/` | Session transcripts |

## Reset & Recovery

### Pre-Reset Backup (Recommended, v2026.3.8+)
```bash
# Create and verify backup before reset/reinstall
openclaw backup create
openclaw backup verify "<backup-file>"
```

### Soft Reset (Keep Credentials)
```bash
openclaw reset
openclaw onboard --install-daemon
```

### Hard Reset (Full Clean)
```bash
openclaw gateway stop
rm -rf ~/.openclaw
openclaw onboard --install-daemon
```

**Warning:** Hard reset loses all sessions, credentials, and pairings.

## Getting Help

```bash
# Command help
openclaw --help
openclaw <command> --help

# Documentation
https://docs.openclaw.ai

# Troubleshooting guide
https://docs.openclaw.ai/gateway/troubleshooting
```
