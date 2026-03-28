# User Login & Authentication Mechanisms

This guide covers all authentication and login mechanisms in OpenClaw, from gateway access control to channel linking and model provider authentication.

## Overview

OpenClaw uses multiple authentication layers:

1. **Gateway Authentication** - Controls who can access the OpenClaw gateway API
2. **Channel Login** - Links messaging channels (WhatsApp, Zalo Personal, etc.) to your gateway
3. **Model Provider Authentication** - Authenticates with AI model providers (Anthropic, OpenAI, etc.)
4. **Pairing** - Approves individual users/senders to interact with the bot
5. **Device Management** - Controls which devices can access the gateway

---

## Gateway Authentication

The gateway is the control plane for OpenClaw. It requires authentication to prevent unauthorized access.

### Authentication Modes

#### Token Authentication (Recommended)

Token-based authentication using a secure random token:

```bash
# Generate and set a secure token
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
openclaw gateway restart
```

**Security:** Strong, cryptographically random tokens provide excellent security. Store tokens securely (environment variables, secrets manager).

#### Password Authentication

Password-based authentication (less secure than tokens):

```bash
openclaw config set gateway.auth.mode password
openclaw config set gateway.auth.password "your-secure-password"
openclaw gateway restart
```

**Security:** Use strong, unique passwords. Consider token auth instead for better security.

#### Trusted Proxy Authentication

For deployments behind an authenticated reverse proxy:

```bash
openclaw config set gateway.auth.mode trusted-proxy
# Requires an authenticated reverse proxy with TLS in front of the gateway
```

**Use case:** Enterprise deployments with identity-aware proxies (e.g., Tailscale, Cloudflare Access).

#### Tailscale Authentication

When using Tailscale for secure remote access:

```bash
openclaw config set gateway.auth.allowTailscale true
```

OpenClaw verifies identity headers (`tailscale-user-login`) by resolving the client IP through the local Tailscale daemon. This allows secure remote access without exposing the gateway to the public internet.

**Security:** Tailscale nodes are verified against the local daemon rather than trusting headers blindly.

### Explicit Auth Mode (v2026.3.7+)

**Important:** If both `gateway.auth.token` and `gateway.auth.password` are configured (including SecretRefs), you **must** explicitly set `gateway.auth.mode`:

```bash
# Pick one auth mode explicitly
openclaw config set gateway.auth.mode token
# or
openclaw config set gateway.auth.mode password
```

Without an explicit mode, the gateway will refuse to start or reject authentication requests.

### Breaking Change: `auth: "none"` Removed

As of v2026.1.29, `gateway.auth.mode: "none"` is **permanently removed**. All gateways require authentication. If you had `"none"` in your config, fix it:

```bash
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
openclaw gateway restart
```

### Environment Variables

You can set the gateway token via environment variable:

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token-here"
openclaw gateway start
```

This is useful for Docker/Kubernetes deployments where secrets are managed externally.

---

## Channel Login

Channel login links messaging platforms to your OpenClaw gateway. Different channels use different login methods.

### WhatsApp

WhatsApp uses QR code linking:

```bash
openclaw channels login
# Or for a specific account:
openclaw channels login --account secondary
```

`v2026.3.23+` note: if only one login-capable channel is configured, `openclaw channels login`/`logout` auto-selects it.
`v2026.3.24+` note: use `openclaw --container <name-or-id> channels login` (or set `OPENCLAW_CONTAINER`) when channel auth must run inside a running OpenClaw Docker/Podman container.

**Process:**
1. Run the command
2. A QR code appears in your terminal
3. Open WhatsApp on your phone → Settings → Linked Devices
4. Scan the QR code
5. The device links automatically

**Prerequisites:**
- Real mobile number (VoIP numbers are blocked)
- Separate phone recommended (not your primary device)

**Multi-Account:**
```bash
# Link first account (default)
openclaw channels login

# Link additional account
openclaw channels login --account secondary
openclaw config set channels.whatsapp.accounts.secondary.allowFrom '["+15559876543"]'
```

### Zalo Personal

Zalo Personal (rebuilt in v2026.3.2) uses native JavaScript integration:

```bash
openclaw channels login --channel zalouser
```

**Note:** The Zalo Personal plugin was rebuilt in v2026.3.2. It **no longer depends on external CLI binaries** and uses native `zca-js` integration.

### Other Channels

Most other channels use token-based configuration rather than interactive login:

- **Slack**: OAuth tokens (bot token + app token)
- **Telegram**: Bot token from @BotFather
- **Discord**: Bot token from Discord Developer Portal
- **BlueBubbles**: Server URL and password
- **Signal**: signal-cli registration/linking

See [channel-setup.md](channel-setup.md) for platform-specific setup instructions.

### Verifying Channel Login

Check channel connection status:

```bash
openclaw channels status
openclaw channels list
```

---

## Model Provider Authentication

OpenClaw needs to authenticate with AI model providers to make API calls.

### Anthropic (Claude)

**Important:** Anthropic has **blocked OAuth tokens** for OpenClaw. You must use direct API keys:

```bash
# This will NOT work:
openclaw models auth  # OAuth flow for Anthropic is blocked

# Use API keys directly instead:
openclaw models auth setup-token --provider anthropic
# Then paste your API key from console.anthropic.com
```

**Getting an API Key:**
1. Go to https://console.anthropic.com
2. Navigate to API Keys
3. Create a new key
4. Copy and paste when prompted by `openclaw models auth setup-token`

### Other Providers

```bash
# OpenAI
openclaw models auth setup-token --provider openai

# xAI / Grok (v2026.2.6+)
openclaw models auth setup-token --provider xai

# Kilo Code (v2026.2.23+)
openclaw models auth setup-token --provider kilocode

# Moonshot/Kimi (v2026.2.23+)
openclaw models auth setup-token --provider moonshot

# MiniMax M2.5 (v2026.3.2+)
openclaw models auth setup-token --provider minimax

# Vercel AI Gateway (v2026.2.23+)
openclaw models auth setup-token --provider vercel-ai
```

### Verifying Model Authentication

```bash
openclaw models list
openclaw status --deep  # Includes provider health checks
```

---

## Pairing

Pairing is the mechanism that approves individual users/senders to interact with the bot. This is separate from channel login.

### Pairing Policies

| Policy | Behavior |
|--------|----------|
| `pairing` | Unknown senders get a pairing code (default, 1-hour expiration, max 3 pending; setup/bootstrap codes are single-use in v2026.3.13+) |
| `allowlist` | Only listed senders allowed |
| `open` | Anyone can message (requires `"*"` in allowFrom) |
| `disabled` | DMs blocked |

### Approving Pairing Requests

When someone messages the bot for the first time (with `pairing` policy):

```bash
# List pending pairing requests
openclaw pairing list

# Approve a sender
openclaw pairing approve <channel> <code>
```

**Example:**
```bash
openclaw pairing approve slack ABC123
openclaw pairing approve whatsapp +15551234567
```

### Configuring Pairing

```bash
# Require pairing (default, secure)
openclaw config set channels.slack.dmPolicy pairing

# Allowlist only (more restrictive)
openclaw config set channels.slack.dmPolicy allowlist
openclaw config set channels.slack.allowFrom '["U12345678"]'

# Open access (NOT RECOMMENDED - exposes to spam/abuse)
openclaw config set channels.whatsapp.dmPolicy open
openclaw config set channels.whatsapp.allowFrom '["*"]'
```

---

## Device Management

Device management controls which devices can access the gateway.

### Listing Devices

```bash
openclaw devices list
```

Shows:
- Pending devices (awaiting approval)
- Paired devices (approved)
- Device IDs and metadata

### Approving Devices

```bash
openclaw devices approve <device-id>
```

### Rejecting Devices

```bash
openclaw devices reject <device-id>
```

### Revoking Device Access

```bash
openclaw devices revoke <device-id>
```

---

## Session Isolation & Identity

When multiple users interact with OpenClaw, you can control how sessions are isolated:

### Session Isolation Modes

| Mode | Description |
|------|-------------|
| `main` | All DMs share one session (default) |
| `per-channel-peer` | Each sender+channel pair gets isolated context |
| `per-account-channel-peer` | Collapses multi-channel users via identity links |

### Configuring Session Isolation

```bash
# Isolate each sender (recommended for multi-user setups)
openclaw config set session.dmScope "per-channel-peer"
```

### Identity Links

With `per-account-channel-peer`, you can link identities across channels so the same user on Slack and WhatsApp shares a session:

```json
{
  "session": {
    "dmScope": "per-account-channel-peer",
    "identityLinks": [
      {
        "slack": "U12345678",
        "whatsapp": "+15551234567"
      }
    ]
  }
}
```

---

## Troubleshooting Login Issues

### Gateway Authentication Failures

**Symptoms:** `Auth failed`, `unauthorized`, gateway refuses connections

**Solutions:**
1. Check auth mode is set:
   ```bash
   openclaw config get gateway.auth.mode
   ```

2. Verify token/password is configured:
   ```bash
   openclaw config get gateway.auth.token
   openclaw config get gateway.auth.password
   ```

3. If both token and password exist, set mode explicitly (v2026.3.7+):
   ```bash
   openclaw config set gateway.auth.mode token
   ```

4. Regenerate token if compromised:
   ```bash
   openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
   openclaw gateway restart
   ```

### Channel Login Failures

**WhatsApp QR Code Not Working:**
- Ensure you're using a real mobile number (not VoIP)
- Check WhatsApp is up to date
- Try unlinking and relinking: `openclaw channels logout` then `openclaw channels login`

**Zalo Personal Login Issues:**
- Ensure you're on v2026.3.2+ (rebuilt with native JS)
- Check plugin is installed: `openclaw plugins list`
- Verify channel status: `openclaw channels status`

### Model Provider Authentication Failures

**Anthropic OAuth Token Rejected:**
- **Cause:** Anthropic has blocked OAuth tokens for OpenClaw
- **Fix:** Use direct API keys:
  ```bash
  openclaw models auth setup-token --provider anthropic
  ```

**Other Provider Auth Failures:**
- Verify API key is correct
- Check provider status/outages
- Re-authenticate:
  ```bash
  openclaw models auth setup-token --provider <provider-name>
  ```

### Pairing Issues

**Pairing Code Not Working:**
- Check pairing policy: `openclaw config get channels.<channel>.dmPolicy`
- Verify code hasn't expired (1-hour expiration)
- Check max pending limit (3 pending requests)
- Generate a fresh code if a previous setup/bootstrap code was already consumed (single-use in v2026.3.13+)
- List pending: `openclaw pairing list`

---

## Security Best Practices

1. **Gateway Auth:**
   - Always use token authentication (not password)
   - Generate strong, random tokens: `openssl rand -hex 32`
   - Store tokens securely (env vars, secrets manager)
   - Use `loopback` binding unless remote access needed
   - Use Tailscale for secure remote access

2. **Channel Access:**
   - Use `pairing` policy (default) for DMs
   - Use `allowlist` for groups
   - Never use `open` policy without understanding risks
   - Regularly review paired users: `openclaw pairing list`

3. **Model Provider Auth:**
   - Use API keys, not OAuth (especially for Anthropic)
   - Rotate keys periodically
   - Use SecretRef system for credential management (v2026.3.2+)

4. **Device Management:**
   - Regularly review approved devices: `openclaw devices list`
   - Revoke access for unknown devices
   - Monitor for suspicious activity

5. **Session Isolation:**
   - Use `per-channel-peer` for multi-user setups
   - Configure identity links carefully
   - Review session activity: `openclaw sessions list`

---

## Related Commands

```bash
# Gateway authentication
openclaw config get gateway.auth.mode
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"

# Channel login
openclaw channels login
openclaw channels login --channel zalouser
openclaw channels status

# Model authentication
openclaw models auth setup-token --provider anthropic
openclaw models list

# Pairing
openclaw pairing list
openclaw pairing approve <channel> <code>

# Device management
openclaw devices list
openclaw devices approve <id>
openclaw devices revoke <id>

# Session management
openclaw sessions list
openclaw config get session.dmScope
```

---

## See Also

- [Channel Setup Guide](channel-setup.md) - Platform-specific channel configuration
- [Security Checklist](security-checklist.md) - Security hardening and authentication settings
- [CLI Reference](cli-reference.md) - Complete command reference
- [Troubleshooting Guide](troubleshooting.md) - Common issues and solutions
