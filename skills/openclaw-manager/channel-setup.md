# Channel Setup Guide

## Slack (Socket Mode)

### Prerequisites
- Slack workspace admin access
- Slack app with Socket Mode enabled

### Setup Steps

1. **Create Slack App**
   - Go to https://api.slack.com/apps
   - Click "Create New App" → "From an app manifest"
   - Select workspace, paste manifest (JSON)

2. **Manifest Template**
```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "AI assistant"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    }
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "im:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "mpim:history",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim"
      ]
    }
  }
}
```

3. **Get Tokens**
   - **Bot Token (xoxb-)**: OAuth & Permissions → Bot User OAuth Token
   - **App Token (xapp-)**: Basic Information → App-Level Tokens → Generate with `connections:write` scope

4. **Configure OpenClaw**
```bash
openclaw config set channels.slack.botToken "xoxb-..."
openclaw config set channels.slack.appToken "xapp-..."
openclaw gateway restart
```

5. **Test**
   - DM the bot in Slack
   - Approve pairing: `openclaw pairing approve slack <code>`

### Required Scopes
| Scope | Purpose |
|-------|---------|
| `chat:write` | Send messages |
| `im:write` | Send DMs |
| `channels:history` | Read channel messages |
| `im:history` | Read DM history |
| `users:read` | Get user info |
| `app_mentions:read` | Respond to @mentions |

---

## WhatsApp

### Prerequisites
- Real mobile number (VoIP numbers blocked)
- Separate phone recommended

### Setup Steps

1. **Configure**
```bash
openclaw config set channels.whatsapp.dmPolicy allowlist
openclaw config set channels.whatsapp.allowFrom '["+15551234567"]'
```

2. **Link Device**
```bash
openclaw channels login
# Scan QR: WhatsApp → Settings → Linked Devices
```

3. **Verify**
```bash
openclaw channels status
```

### Configuration Options
```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "pairing",
      "allowFrom": ["+15551234567"],
      "selfChatMode": false,
      "sendReadReceipts": true,
      "ackReaction": {
        "emoji": "👀",
        "direct": true
      }
    }
  }
}
```

### Self-Chat Mode (Personal Number)
If using your own WhatsApp number:
```bash
openclaw config set channels.whatsapp.selfChatMode true
```
Then message yourself to test.

### Multi-Account
```bash
openclaw channels login --account secondary
openclaw config set channels.whatsapp.accounts.secondary.allowFrom '["+15559876543"]'
```

---

## Telegram

### Prerequisites
- Telegram account
- Bot token from @BotFather

### Setup Steps

1. **Create Bot**
   - Message @BotFather on Telegram
   - `/newbot` → Follow prompts
   - Save the token

2. **Configure**
```bash
openclaw config set channels.telegram.botToken "123456:ABC-..."
openclaw config set channels.telegram.dmPolicy pairing
openclaw gateway restart
```

3. **Test**
   - Message your bot on Telegram
   - Approve pairing

### BotFather Commands
```
/setjoingroups - Allow/disallow group membership
/setprivacy - Set privacy mode (disable to see all group messages)
```

### Configuration
```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "123456:ABC-...",
      "dmPolicy": "pairing",
      "groupPolicy": "allowlist",
      "requireMention": true
    }
  }
}
```

---

## Discord

### Prerequisites
- Discord server admin access
- Discord application

### Setup Steps

1. **Create Application**
   - Go to https://discord.com/developers/applications
   - New Application → Name it
   - Bot → Add Bot → Copy Token

2. **Enable Intents**
   - Bot → Privileged Gateway Intents
   - Enable: Message Content Intent

3. **Invite Bot**
   - OAuth2 → URL Generator
   - Scopes: `bot`, `applications.commands`
   - Permissions: Send Messages, Read Message History
   - Copy URL, open in browser, add to server

4. **Configure**
```bash
openclaw config set channels.discord.token "MTIz..."
openclaw config set channels.discord.dmPolicy pairing
openclaw gateway restart
```

### Configuration
```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "MTIz...",
      "dmPolicy": "pairing",
      "guildAllowlist": ["guild-id"],
      "channelAllowlist": ["channel-id"]
    }
  }
}
```

---

## Access Control Policies

### DM Policies
| Policy | Behavior |
|--------|----------|
| `pairing` | Unknown senders get pairing code (default) |
| `allowlist` | Only listed senders allowed |
| `open` | Anyone can message (requires `"*"` in allowFrom) |
| `disabled` | DMs blocked |

### Group Policies
| Policy | Behavior |
|--------|----------|
| `allowlist` | Only listed groups (default) |
| `open` | Any group (requires mention) |
| `disabled` | Groups blocked |

### Example: Strict Access
```bash
openclaw config set channels.slack.dmPolicy allowlist
openclaw config set channels.slack.allowFrom '["U12345678"]'
openclaw config set channels.slack.groupPolicy disabled
```

### Example: Open Access (Dangerous)
```bash
openclaw config set channels.whatsapp.dmPolicy open
openclaw config set channels.whatsapp.allowFrom '["*"]'
# NOT RECOMMENDED - exposes to spam/abuse
```
