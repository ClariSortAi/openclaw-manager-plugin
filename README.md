# OpenClaw Manager Plugin

A Claude Code plugin that provides intelligent OpenClaw installation, configuration, and management assistance.

## Features

- **Installation Guidance** - Step-by-step setup on macOS, Linux, or Windows (WSL2)
- **Channel Configuration** - Slack, WhatsApp, Telegram, Discord setup with OAuth/token help
- **Troubleshooting** - Diagnose and fix common gateway, auth, and channel issues
- **Security Hardening** - Audit configurations and apply best practices
- **Cron Job Management** - Schedule automated tasks and webhooks

## Installation

### From a Marketplace

If this plugin is available in a marketplace you've added:

```
/plugin install openclaw-manager@marketplace-name
```

### Direct Installation

```
/plugin install path/to/openclaw-manager-plugin
```

Or add the GitHub repository:

```
/plugin install github:yourusername/openclaw-manager-plugin
```

## Usage

Once installed, invoke the skill with:

```
/openclaw-manager
```

Or with specific tasks:

```
/openclaw-manager install on WSL2
/openclaw-manager setup Slack channel
/openclaw-manager troubleshoot gateway
/openclaw-manager security audit
/openclaw-manager configure cron job
```

## What's Included

| File | Purpose |
|------|---------|
| `skills/openclaw-manager/SKILL.md` | Main skill definition |
| `skills/openclaw-manager/cli-reference.md` | Complete CLI command reference |
| `skills/openclaw-manager/troubleshooting.md` | Common issues and solutions |
| `skills/openclaw-manager/channel-setup.md` | Platform setup guides |
| `skills/openclaw-manager/security-checklist.md` | Security hardening guide |

## Requirements

- Claude Code CLI
- For OpenClaw itself:
  - Node.js v22+ (NOT Bun)
  - macOS, Linux, or Windows WSL2

## License

MIT

## Contributing

Issues and PRs welcome at the repository.
