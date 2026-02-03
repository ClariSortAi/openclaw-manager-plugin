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

### Agents
```bash
openclaw agents list         # List configured agents
openclaw agents add          # Add new agent
openclaw agents delete <id>  # Delete agent
openclaw agents set-identity <id>  # Update agent identity
```

### Memory
```bash
openclaw memory status       # Memory index status
openclaw memory index        # Reindex memory files
openclaw memory search "query"  # Search memory
```

### Security
```bash
openclaw security audit      # Basic security audit
openclaw security audit --deep  # Thorough security check
openclaw security audit --fix   # Auto-fix issues
```

### Webhooks
```bash
openclaw webhooks gmail setup    # Set up Gmail Pub/Sub
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
```

## Configuration Paths

### Get/Set Examples
```bash
# Gateway settings
openclaw config get gateway.bind
openclaw config set gateway.bind loopback
openclaw config set gateway.auth.mode token

# Channel settings
openclaw config get channels.slack
openclaw config set channels.slack.botToken "xoxb-..."
openclaw config set channels.whatsapp.dmPolicy pairing

# Agent settings
openclaw config get agents.defaults.model
openclaw config set agents.defaults.sandbox.mode all
```

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `OPENCLAW_STATE_DIR` | Override state directory |
| `OPENCLAW_CONFIG_PATH` | Override config file path |
| `OPENCLAW_GATEWAY_TOKEN` | Gateway auth token |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token |
| `SLACK_BOT_TOKEN` | Slack bot token |
| `SLACK_APP_TOKEN` | Slack app token |
