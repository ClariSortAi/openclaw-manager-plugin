# OpenClaw Troubleshooting Guide

## Diagnostic Workflow

Always follow this order:

```bash
# 1. Quick status
openclaw status

# 2. Full diagnosis
openclaw status --all

# 3. Deep health check
openclaw status --deep

# 4. Auto-fix
openclaw doctor --fix

# 5. Live logs
journalctl --user -u openclaw-gateway -f
```

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

### Authentication Issues

#### No API Key Found
**Symptoms:** Agent can't make requests, auth errors

**Fix:**
```bash
# Re-run auth setup
openclaw configure
# Or set directly
openclaw models auth setup-token --provider anthropic
```

#### OAuth Token Refresh Failed
**Symptoms:** Token expired errors

**Fix:**
```bash
# Switch to setup token
openclaw models auth setup-token --provider anthropic
# Or re-authenticate
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
node --version  # Should be v22+

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
- Hit 3-request cap
- Channel not connected

**Fix:**
```bash
# Check channel status
openclaw channels status

# Clear pending and retry
openclaw pairing list
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
- Config syntax error: `openclaw doctor`
- Port conflict: Check other processes
- Missing dependencies: Reinstall

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
