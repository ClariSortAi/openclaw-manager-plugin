# OpenClaw CLI Reference

## Core Commands

### Status & Health
```bash
openclaw status              # Quick status summary
openclaw status --all        # Full diagnosis with log tail
openclaw status --deep       # Health checks with provider probes
openclaw health              # Quick health check
openclaw doctor              # Diagnose issues
openclaw doctor --fix        # Auto-fix common problems
openclaw doctor --generate-gateway-token  # Generate a new gateway token
```

### Gateway Management
```bash
openclaw gateway             # Show gateway info
openclaw gateway start       # Start gateway
openclaw gateway stop        # Stop gateway
openclaw gateway restart     # Restart gateway
openclaw gateway status      # Detailed gateway status
```

### Configuration
```bash
openclaw configure           # Interactive configuration wizard
openclaw config get <path>   # Get config value
openclaw config set <path> <value>  # Set config value
openclaw config unset <path> # Remove config value
```

### Channel Management
```bash
openclaw channels list       # List configured channels
openclaw channels status     # Show channel connection status
openclaw channels login      # Link a channel (QR code for WhatsApp)
openclaw channels logout     # Unlink a channel
openclaw channels add        # Add channel account
openclaw channels remove     # Remove channel account
```

### Pairing & Access Control
```bash
openclaw pairing list        # List pending pairing requests
openclaw pairing approve <channel> <code>  # Approve sender
```

### Device Management
```bash
openclaw devices list        # List pending and paired devices
openclaw devices approve <id>  # Approve device
openclaw devices reject <id>   # Reject device
openclaw devices revoke <id>   # Revoke device access
```

### Cron Jobs
```bash
openclaw cron list           # List all cron jobs
openclaw cron status         # Scheduler status
openclaw cron add            # Add new job
openclaw cron rm <id>        # Remove job
openclaw cron enable <id>    # Enable job
openclaw cron disable <id>   # Disable job
openclaw cron run <id>       # Run job immediately (debug)
openclaw cron runs           # View run history
openclaw cron edit <id>      # Edit job settings
```

### Cron Add Options
```bash
openclaw cron add \
  --name "Job Name" \
  --cron "0 8 * * *" \        # Cron expression (or --every/--at)
  --tz "America/New_York" \   # Timezone
  --message "Task prompt" \   # What to do
  --channel slack \           # Delivery channel
  --to "#channel" \           # Destination
  --session isolated \        # Session scope
  --model openai-codex/gpt-5.2  # Model override
```

### Skills
```bash
openclaw skills list         # List available skills
openclaw skills info <name>  # Show skill details
openclaw skills check        # Check skill requirements
```

### ClawHub (Skill Registry)

ClawHub is the public skill registry at clawhub.ai with 3,200+ community skills (down from 10,700+ after the ClawHavoc cleanup):

```bash
clawhub search <query>         # Search for skills on ClawHub
clawhub install <skill-slug>   # Install a skill from ClawHub
clawhub update --all           # Update all installed ClawHub skills
clawhub sync --all             # Sync all skills with registry
```

Skills are installed to `~/.openclaw/skills/` and are immediately available. Always audit third-party skills before installation -- ClawHub now integrates with VirusTotal for automatic scanning.

### Plugins
```bash
openclaw plugins list          # List installed plugins
openclaw plugins info <id>     # Show plugin details
openclaw plugins install <spec>  # Install plugin (npm package or local path)
openclaw plugins install -l <path>  # Link local plugin for development
openclaw plugins update <id>   # Update a plugin
openclaw plugins update --all  # Update all plugins
openclaw plugins enable <id>   # Enable a plugin
openclaw plugins disable <id>  # Disable a plugin
openclaw plugins remove <id>   # Remove/uninstall a plugin
openclaw plugins doctor        # Check plugin health
```

Plugin install supports npm package specs (e.g., `@openclaw/voice-call`). Bundled plugins are disabled by default; installed plugins are enabled by default.

### Agents
```bash
openclaw agents list         # List configured agents
openclaw agents add          # Add new agent
openclaw agents delete <id>  # Delete agent
openclaw agents set-identity <id>  # Update agent identity
```

### Session Management (v2026.2.23+)
```bash
openclaw sessions list         # List active sessions
openclaw sessions cleanup      # Clean up old sessions (respects disk budget)
```

Session disk budget controls:
```bash
openclaw config set session.maintenance.maxDiskBytes 1073741824    # 1 GB max
openclaw config set session.maintenance.highWaterBytes 858993459   # Trigger cleanup at 800 MB
```

### Memory
```bash
openclaw memory status       # Memory index status
openclaw memory index        # Reindex memory files
openclaw memory search "query"  # Search memory (FTS fallback with query expansion)
```

### Security
```bash
openclaw security audit          # Basic security audit
openclaw security audit --deep   # Thorough security check
openclaw security audit --fix    # Auto-fix issues
openclaw security audit --json   # Machine-readable output
```

### Webhooks
```bash
openclaw webhooks gmail setup    # Set up Gmail Pub/Sub webhook
openclaw webhooks gmail run      # Run Gmail webhook listener
```

### Setup & Reset
```bash
openclaw setup               # Initialize config and workspace
openclaw onboard             # Full onboarding wizard
openclaw onboard --install-daemon  # With systemd service
openclaw reset               # Reset config/state (keeps CLI)
openclaw uninstall           # Full uninstall
```

### Other Commands
```bash
openclaw dashboard           # Open Control UI
openclaw logs                # View logs
openclaw message             # Send messages
openclaw models list         # List available models
openclaw models auth         # Configure model auth
openclaw models auth setup-token --provider anthropic  # Direct API key setup
```

## Configuration Paths

### Get/Set Examples
```bash
# Gateway settings
openclaw config get gateway.bind
openclaw config set gateway.bind loopback
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.allowTailscale true
openclaw config set gateway.mdns.mode minimal

# Channel settings
openclaw config get channels.slack
openclaw config set channels.slack.botToken "xoxb-..."
openclaw config set channels.whatsapp.dmPolicy pairing

# Agent settings
openclaw config get agents.defaults.model
openclaw config set agents.defaults.model "anthropic/claude-opus-4-6"
openclaw config set agents.defaults.sandbox.mode all
openclaw config set agents.defaults.sandbox.workspaceAccess none
openclaw config set agents.defaults.sandbox.scope agent

# Sub-agent settings (v2026.2.17+)
openclaw config set agents.defaults.subagents.maxSpawnDepth 2
openclaw config set agents.defaults.subagents.maxChildrenPerAgent 5

# 1M context window (Anthropic models, v2026.2.17+)
openclaw config set agents.defaults.params.context1m true

# Session isolation
openclaw config set session.dmScope "per-channel-peer"

# Per-agent params overrides (v2026.2.23+)
openclaw config set agents.defaults.params.cacheRetention true

# Control plane tool denials (production recommended)
openclaw config set agents.defaults.tools.deny '["gateway","cron","sessions_spawn","sessions_send"]'

# Session disk budget (v2026.2.23+)
openclaw config set session.maintenance.maxDiskBytes 1073741824
openclaw config set session.maintenance.highWaterBytes 858993459

# Multi-user trust heuristic (v2026.2.24+)
openclaw config set security.trust_model.multi_user_heuristic true

# HTTP security headers (v2026.2.23+)
openclaw config set gateway.security.hsts true

# Skill configuration
openclaw config set skills.entries.my-skill.enabled true
openclaw config set skills.entries.my-skill.apiKey "SECRET_VALUE"
openclaw config set skills.load.extraDirs '["path/to/skills"]'
openclaw config set skills.load.watch true

# Plugin configuration
openclaw config set plugins.enabled true
openclaw config set plugins.allow '["voice-call"]'
openclaw config set plugins.slots.memory "memory-core"
```

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `OPENCLAW_STATE_DIR` | Override state directory |
| `OPENCLAW_CONFIG_PATH` | Override config file path |
| `OPENCLAW_GATEWAY_TOKEN` | Gateway auth token |
| `OPENCLAW_GATEWAY_PORT` | Override gateway port (default: 18789) |
| `OPENCLAW_DISABLE_BONJOUR` | Set to `1` to disable mDNS discovery |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token |
| `SLACK_BOT_TOKEN` | Slack bot token |
| `SLACK_APP_TOKEN` | Slack app token |

## Official Plugins

| Plugin | Package | Description |
|--------|---------|-------------|
| Voice Call | `@openclaw/voice-call` | Twilio/log voice calling |
| Microsoft Teams | `@openclaw/msteams` | Teams channel (plugin-only since v2026.1.15) |
| Matrix | `@openclaw/matrix` | Matrix protocol channel |
| Nostr | `@openclaw/nostr` | Nostr decentralized messaging |
| Zalo | `@openclaw/zalo` | Zalo Official Account |
| Zalo User | `@openclaw/zalouser` | Zalo personal account |
| Memory (Core) | bundled | Long-term memory (default slot) |
| Feishu/Lark | native (v2026.2.2+) | Chinese enterprise chat (Feishu and Lark) |
| Memory (LanceDB) | bundled | Vector-based memory alternative |

Plugin slots allow exclusive categories (e.g., only one memory plugin active):
```bash
openclaw config set plugins.slots.memory "memory-core"
# or
openclaw config set plugins.slots.memory "memory-lancedb"
```
