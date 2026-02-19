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

### Slack Text Streaming

As of v2026.2.17, Slack supports native single-message text streaming. This is enabled by default -- the bot updates a single message in real-time rather than sending multiple messages.

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

## iMessage (macOS Only)

### Prerequisites
- macOS with iMessage configured
- Messages app signed into your Apple ID
- Full Disk Access granted to the OpenClaw process (or Terminal)

### Setup Steps

1. **Grant Full Disk Access**
   - System Settings → Privacy & Security → Full Disk Access
   - Add Terminal (or the app running OpenClaw)
   - This is required for OpenClaw to read the iMessage database

2. **Enable the Channel**
```bash
openclaw config set channels.imessage.enabled true
openclaw config set channels.imessage.dmPolicy pairing
openclaw gateway restart
```

3. **Test**
   - Send an iMessage to your own number or Apple ID from another device
   - Approve pairing: `openclaw pairing approve imessage <code>`

### Configuration
```json
{
  "channels": {
    "imessage": {
      "enabled": true,
      "dmPolicy": "pairing",
      "allowFrom": ["+15551234567", "user@icloud.com"]
    }
  }
}
```

### Limitations
- macOS only (iMessage is not available on Linux or WSL2)
- Requires Full Disk Access for the chat.db database
- Apple ID changes may require re-configuration
- Group chats have limited support

---

## Microsoft Teams (Plugin Required)

As of v2026.1.15, Microsoft Teams is a **plugin-only** channel via `@openclaw/msteams`.

### Setup Steps

1. **Install the Plugin**
```bash
openclaw plugins install @openclaw/msteams
```

2. **Configure**
   - Follow the plugin's setup wizard for Azure App Registration
   - Requires: Azure AD application, Bot Framework registration, Teams app manifest

3. **Verify**
```bash
openclaw plugins info msteams
openclaw channels status
```

### Notes
- Requires an Azure AD tenant with admin consent
- Bot Framework registration is separate from the Azure AD app
- Teams channels use the plugin system rather than native channel config

---

## Matrix (Plugin Required)

Matrix is supported via the `@openclaw/matrix` plugin.

### Setup Steps

1. **Install the Plugin**
```bash
openclaw plugins install @openclaw/matrix
```

2. **Configure with Your Matrix Homeserver**
```bash
openclaw config set plugins.entries.matrix.config.homeserver "https://matrix.example.com"
openclaw config set plugins.entries.matrix.config.userId "@bot:example.com"
openclaw config set plugins.entries.matrix.config.accessToken "syt_..."
openclaw gateway restart
```

3. **Verify**
```bash
openclaw plugins info matrix
openclaw channels status
```

---

## Nostr (Plugin Required)

Nostr is supported via the `@openclaw/nostr` plugin for decentralized messaging.

### Setup Steps

1. **Install the Plugin**
```bash
openclaw plugins install @openclaw/nostr
```

2. **Configure with your Nostr identity and relay preferences**
```bash
openclaw config set plugins.entries.nostr.enabled true
openclaw gateway restart
```

3. **Verify**
```bash
openclaw plugins info nostr
```

---

## Zalo (Plugin Required)

Zalo is supported via the `@openclaw/zalo` (Official Account) or `@openclaw/zalouser` (personal) plugins.

### Setup Steps

1. **Install the Plugin**
```bash
# For Zalo Official Account
openclaw plugins install @openclaw/zalo

# For personal Zalo account
openclaw plugins install @openclaw/zalouser
```

2. **Configure per the plugin's setup instructions**
```bash
openclaw plugins info zalo
openclaw gateway restart
```

---

## Gmail Webhooks

OpenClaw can receive Gmail messages via Google Pub/Sub webhooks, allowing it to process and respond to emails.

### Setup Steps

1. **Set Up Gmail Pub/Sub**
```bash
openclaw webhooks gmail setup
```
This walks through:
   - Creating a Google Cloud project (or selecting an existing one)
   - Enabling the Gmail API and Pub/Sub API
   - Creating a Pub/Sub topic and subscription
   - Granting Gmail publish permissions to the topic
   - Setting up a watch on your Gmail inbox

2. **Run the Webhook Listener**
```bash
openclaw webhooks gmail run
```

3. **Verify**
   - Send a test email to the configured Gmail address
   - Check logs: `openclaw logs`

### Notes
- Requires a Google Cloud project with billing enabled
- Gmail watch expires after 7 days and must be renewed (the listener handles renewal automatically)
- Supports filtering by label, sender, or subject in config

---

## Session Isolation Modes

When multiple users interact with OpenClaw, you can control how sessions are isolated:

| Mode | Description |
|------|-------------|
| `main` | All DMs share one session (default) |
| `per-channel-peer` | Each sender+channel pair gets isolated context |
| `per-account-channel-peer` | Collapses multi-channel users via identity links |

### Configure Session Isolation
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

## Access Control Policies

### DM Policies
| Policy | Behavior |
|--------|----------|
| `pairing` | Unknown senders get pairing code (default, 1-hour expiration, max 3 pending) |
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
