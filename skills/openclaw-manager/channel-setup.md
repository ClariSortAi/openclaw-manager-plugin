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

### Slack Interactive Reply Directives (v2026.3.13+)

OpenClaw can apply opt-in interactive reply directives in shared Slack delivery flows. If your automation or plugin emits directive metadata, keep both gateway and plugin components on v2026.3.13+ so responses render as intended.

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

### Telegram Streaming (v2026.3.2+)

Telegram now defaults to `partial` streaming mode — the bot updates a single message in real-time using `sendMessageDraft` for private preview. This gives users a "typing" experience as the response generates.

### Telegram DM Topics (v2026.3.1+)

Per-DM topic configuration with topic-aware sessions:
```bash
openclaw config set channels.telegram.dmTopics.enabled true
```
Each DM conversation can have its own topic context, with sessions scoped to the topic.

### Telegram Security/Reliability Notes (v2026.3.13+)

- Webhook secret validation now happens before body parsing, so invalid or missing secrets are rejected earlier.
- Inbound media download handling was hardened (transport-policy threading + IPv4 fallback retries) to reduce attachment fetch failures on mixed IPv4/IPv6 networks.

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

### Discord Interactive UI (v2026.2.16+)

Discord supports interactive UI components including buttons, selects, and modals. These are enabled by default when the bot has the `applications.commands` scope.

**Known Issue (v2026.2.24, fixed in v2026.3.1):** Discord WebSocket 1005/1006 disconnects could cause the bot to go offline for 30+ minutes. Fixed in v2026.3.1 with distinct sentinel IDs for wildcard component handlers. Upgrade to v2026.3.1+ to resolve.

---

## BlueBubbles (iMessage — Recommended)

BlueBubbles replaces legacy iMessage with full feature support including message editing, unsend, effects, reactions, and group management.

### Prerequisites
- macOS with iMessage configured
- BlueBubbles server running on the Mac

### Setup Steps

1. **Install BlueBubbles Server on macOS**
   - Download from [bluebubbles.app](https://bluebubbles.app)
   - Follow the server setup wizard
   - Note the server URL and password

2. **Configure OpenClaw**
```bash
openclaw config set channels.bluebubbles.serverUrl "http://localhost:1234"
openclaw config set channels.bluebubbles.password "your-server-password"
openclaw config set channels.bluebubbles.dmPolicy pairing
openclaw gateway restart
```

3. **Test**
   - Send an iMessage to your number from another device
   - Approve pairing: `openclaw pairing approve bluebubbles <code>`

### Configuration
```json
{
  "channels": {
    "bluebubbles": {
      "enabled": true,
      "serverUrl": "http://localhost:1234",
      "password": "...",
      "dmPolicy": "pairing",
      "allowFrom": ["+15551234567"]
    }
  }
}
```

### Advantages Over Legacy iMessage
- Full feature support: edit, unsend, tapback reactions, message effects
- Group chat management (create, rename, add/remove members)
- No Full Disk Access requirement (uses BlueBubbles API instead of chat.db)
- Better reliability and reconnection handling

---

## iMessage Legacy (Deprecated)

**This channel is deprecated. Use BlueBubbles instead for full feature support.**

Legacy iMessage is macOS-only and reads the chat.db database directly. It has limited group chat support and requires Full Disk Access.

### If You Must Use Legacy iMessage

1. **Grant Full Disk Access**
   - System Settings → Privacy & Security → Full Disk Access
   - Add Terminal (or the app running OpenClaw)

2. **Enable the Channel**
```bash
openclaw config set channels.imessage.enabled true
openclaw config set channels.imessage.dmPolicy pairing
openclaw gateway restart
```

### Limitations
- macOS only (not available on Linux or WSL2)
- Requires Full Disk Access for the chat.db database
- No edit/unsend/effects support
- Group chats have limited support
- **Will be removed in a future version**

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

### Zalo Personal (Rebuilt in v2026.3.2)

The Zalo Personal plugin was rebuilt in v2026.3.2 using native `zca-js` integration. It **no longer depends on external CLI binaries**. Login via:
```bash
openclaw channels login --channel zalouser
```

---

## Signal (Native)

Signal is supported natively via signal-cli for privacy-focused messaging.

### Prerequisites
- signal-cli installed and configured
- A dedicated Signal phone number

### Setup Steps

1. **Install signal-cli** (follow [signal-cli docs](https://github.com/AsamK/signal-cli))

2. **Register or link to a Signal account**
```bash
signal-cli -a +15551234567 register
# or link to existing account
signal-cli -a +15551234567 link
```

3. **Configure OpenClaw**
```bash
openclaw config set channels.signal.phoneNumber "+15551234567"
openclaw config set channels.signal.dmPolicy pairing
openclaw gateway restart
```

### Signal Groups Config Compatibility (v2026.3.13+)

If your `channels.signal` config includes group controls and older builds reject those keys during validation, upgrade to v2026.3.13+ where Signal channel schema coverage includes groups settings.

---

## Google Chat (Native)

Google Chat is supported natively via HTTP webhook integration.

### Setup Steps

1. **Create a Google Chat Bot** in the Google Cloud Console
2. **Enable the Chat API**
3. **Configure webhook URL** in OpenClaw
```bash
openclaw config set channels.googlechat.enabled true
openclaw config set channels.googlechat.dmPolicy pairing
openclaw gateway restart
```

---

## IRC (Native)

IRC is supported natively with pairing/allowlist access controls.

### Setup Steps

```bash
openclaw config set channels.irc.server "irc.libera.chat"
openclaw config set channels.irc.port 6697
openclaw config set channels.irc.nick "openclaw-bot"
openclaw config set channels.irc.dmPolicy pairing
openclaw gateway restart
```

---

## WebChat (Native)

WebChat is the built-in Gateway UI accessible over WebSocket at the control plane address.

### Access
```bash
openclaw dashboard  # Opens the Control UI in a browser
# WebChat is available at ws://127.0.0.1:18789
```

No additional configuration needed — it runs as part of the gateway.

---

## LINE (Plugin Required)

LINE is supported via the `@openclaw/line` plugin using the LINE Messaging API.

### Setup Steps

1. **Install the Plugin**
```bash
openclaw plugins install @openclaw/line
```

2. **Create a LINE Messaging API channel** in the LINE Developers Console

3. **Configure**
```bash
openclaw plugins info line
openclaw gateway restart
```

---

## Mattermost (Plugin Required)

Mattermost is supported via the `@openclaw/mattermost` plugin.

### Setup Steps

1. **Install the Plugin**
```bash
openclaw plugins install @openclaw/mattermost
```

2. **Configure with your Mattermost server**
```bash
openclaw config set plugins.entries.mattermost.config.serverUrl "https://mattermost.example.com"
openclaw config set plugins.entries.mattermost.config.token "..."
openclaw gateway restart
```

---

## Nextcloud Talk (Plugin Required)

```bash
openclaw plugins install @openclaw/nextcloud-talk
openclaw gateway restart
```

---

## Synology Chat (Plugin Required)

```bash
openclaw plugins install @openclaw/synology-chat
openclaw gateway restart
```

---

## Tlon (Plugin Required)

Tlon is supported via the `@openclaw/tlon` plugin for decentralized messaging.

```bash
openclaw plugins install @openclaw/tlon
openclaw gateway restart
```

---

## Twitch (Plugin Required)

Twitch chat integration via the `@openclaw/twitch` plugin.

```bash
openclaw plugins install @openclaw/twitch
openclaw gateway restart
```

---

## Feishu / Lark (v2026.2.2+)

Feishu (飞书) and Lark are natively supported as of v2026.2.2 -- the first Chinese enterprise chat integration.

### Prerequisites
- Feishu/Lark developer account
- Custom app created in the Feishu Open Platform

### Setup Steps

1. **Create a Custom App**
   - Go to the Feishu Open Platform developer console
   - Create a new custom app
   - Enable the Bot capability
   - Note the App ID and App Secret

2. **Configure OpenClaw**
```bash
openclaw config set channels.feishu.appId "cli_..."
openclaw config set channels.feishu.appSecret "..."
openclaw config set channels.feishu.dmPolicy pairing
openclaw gateway restart
```

3. **Verify**
```bash
openclaw channels status
```

### Configuration
```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_...",
      "appSecret": "...",
      "dmPolicy": "pairing"
    }
  }
}
```

### Notes
- Supports both Feishu (China) and Lark (international) via the same configuration
- Requires event subscription configuration in the Feishu developer console
- Group chat support follows the same `groupPolicy` pattern as other channels

### Feishu Improvements (v2026.3.x)
- Reaction notifications and typing backoff fixes (v2026.3.1)
- Rich-text parsing enhancements and media type corrections (v2026.3.1)
- Multi-account routing with mention validation (v2026.3.1)
- `feishu_doc` tool actions: table creation and file uploads (v2026.3.1)
- TTS/voice bubbles, Opus audio as `msg_type: "audio"` (v2026.3.1)
- Webhook ingress rate-limiting with stale-window pruning (v2026.3.1)
- Multi-app mention routing validation (v2026.3.2)

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
| `pairing` | Unknown senders get pairing code (default, 1-hour expiration, max 3 pending; setup/bootstrap codes are single-use in v2026.3.13+) |
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
