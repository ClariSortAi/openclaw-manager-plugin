# OpenClaw Manager Plugin for Claude Code

[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-6B5CE7?style=flat-square)](https://code.claude.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.1.0-blue?style=flat-square)](CHANGELOG.md)

> **If you can use Claude Code, you can use OpenClaw.** This plugin democratizes access to powerful AI-to-messaging integrations—no DevOps expertise required.

[OpenClaw](https://openclaw.ai) (formerly known as **ClawdBot**) is an AI gateway that connects Claude and other LLMs to messaging platforms like Slack, WhatsApp, Telegram, Discord, and more. This plugin turns Claude Code into your personal OpenClaw expert—handling installation, configuration, troubleshooting, and security hardening through natural conversation.

**This is a weekend project built to help more people access OpenClaw.** The setup process can be intimidating for newcomers, so we wrapped all that complexity into a conversational interface. Just tell Claude what you want, and it handles the rest.

## Why This Plugin?

Setting up OpenClaw (formerly ClawdBot) manually involves:
- Installing dependencies (Node.js v22+, systemd)
- Creating OAuth apps and managing tokens
- Configuring channel policies and access controls
- Setting up cron jobs and webhooks
- Hardening security settings
- Debugging connection issues

**That's a lot to ask of someone who just wants to chat with Claude from Slack.**

With this plugin, you just say what you want:

```
> /openclaw-manager install on WSL2
> /openclaw-manager setup Slack channel
> /openclaw-manager troubleshoot gateway not responding
> /openclaw-manager security audit
```

Claude handles the rest—running commands, checking configurations, explaining what's happening, and fixing issues.

## Quick Start

### Installation

**From GitHub (recommended):**
```bash
claude plugin install github:ClariSortAi/openclaw-manager-plugin
```

**From local directory:**
```bash
claude plugin install /path/to/openclaw-manager-plugin
```

**For development:**
```bash
claude --plugin-dir ./openclaw-manager-plugin
```

### Usage

Once installed, invoke the skill:

```bash
# Interactive mode - Claude guides you
/openclaw-manager

# Specific tasks
/openclaw-manager install on macOS
/openclaw-manager configure WhatsApp
/openclaw-manager troubleshoot connection issues
/openclaw-manager setup cron job for daily summary
/openclaw-manager security hardening
```

Or just describe what you need in natural language—Claude will automatically use this skill when you mention OpenClaw.

## What Can It Do?

| Task | Example |
|------|---------|
| **Fresh Install** | `Help me install OpenClaw on Ubuntu` |
| **Channel Setup** | `Configure Slack with Socket Mode` |
| **Troubleshooting** | `My WhatsApp keeps disconnecting` |
| **Security** | `Audit my OpenClaw security settings` |
| **Automation** | `Set up a daily standup reminder cron job` |
| **Configuration** | `Change my gateway to allowlist mode` |
| **Skills** | `Install a skill from ClawHub` |
| **Plugins** | `Set up Microsoft Teams via the msteams plugin` |
| **Sub-Agents** | `Configure nested sub-agent spawn depth` |
| **1M Context** | `Enable the 1M context window for Opus 4.6` |

### Supported Platforms

- **Slack** - Socket Mode with OAuth scopes, native text streaming
- **WhatsApp** - QR code linking, multi-account support
- **Telegram** - BotFather integration
- **Discord** - Bot token and privileged intents
- **iMessage** - macOS native integration
- **Microsoft Teams** - Plugin-based (`@openclaw/msteams`)
- **Matrix** - Plugin-based (`@openclaw/matrix`)
- **Nostr** - Plugin-based (`@openclaw/nostr`)
- **Zalo** - Plugin-based (`@openclaw/zalo`)

### Supported Operating Systems

- macOS (native)
- Linux (native, systemd)
- Windows (WSL2 required)

## Plugin Contents

```
openclaw-manager-plugin/
├── .claude-plugin/
│   ├── plugin.json              # Plugin manifest
│   └── marketplace.json         # Marketplace metadata
├── skills/
│   └── openclaw-manager/
│       ├── SKILL.md             # Main skill definition
│       ├── cli-reference.md     # Complete CLI command reference
│       ├── troubleshooting.md   # Common issues and solutions
│       ├── channel-setup.md     # Platform-specific setup guides
│       └── security-checklist.md # Security hardening guide
├── CLAUDE.md                    # Development conventions
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
└── LICENSE
```

## Requirements

- **Claude Code** v1.0.33 or later
- For OpenClaw itself:
  - Node.js v22+ (NOT Bun—causes WhatsApp/Telegram issues)
  - macOS, Linux, or Windows WSL2
  - **Minimum OpenClaw version: v2026.1.29** (critical security patches)

## How It Works

This plugin provides Claude with comprehensive OpenClaw knowledge through structured documentation:

1. **Skill Definition** (`SKILL.md`) - Core capabilities and decision-making guidance
2. **CLI Reference** - Every command, flag, and configuration path
3. **Troubleshooting Guide** - Diagnostic workflows and common fixes
4. **Channel Setup** - Platform-specific OAuth, tokens, and configurations
5. **Security Checklist** - Hardening recommendations, CVE awareness, and audit procedures

Claude reads these documents and uses them to guide you through any OpenClaw task, running the right commands and explaining each step.

## Examples

### Installing OpenClaw

```
You: Help me install OpenClaw on my Mac

Claude: I'll guide you through installing OpenClaw on macOS. Let me first check
if you have the prerequisites...

[Runs: node --version]

You have Node.js v22.1.0 - perfect. Now let's install the CLI...

[Runs: curl -fsSL https://openclaw.ai/install.sh | bash]

CLI installed. Running the onboarding wizard with daemon setup...

[Runs: openclaw onboard --install-daemon]
```

### Troubleshooting

```
You: My Slack bot isn't responding to messages

Claude: Let me diagnose your Slack connection. First, checking overall status...

[Runs: openclaw status --all]

I see the gateway is running but Slack shows "missing_scope" error. This means
your Slack app is missing required OAuth permissions. The missing scope appears
to be `im:write`.

Here's how to fix it:
1. Go to api.slack.com/apps → Your app
2. OAuth & Permissions → Add `im:write` scope
3. Reinstall to Workspace

Then we'll update your token...
```

## Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

- Report issues on GitHub
- PRs for new features, better docs, or bug fixes
- Share your use cases to improve the skill

## Disclaimer

**This is a community project built over a weekend to help democratize OpenClaw.**

- No warranty, no guarantees—use at your own risk
- Not affiliated with or endorsed by Anthropic, OpenAI, or the OpenClaw team/foundation
- This plugin provides guidance and runs commands on your behalf; always review what it's doing
- Security configurations are suggestions based on best practices; your threat model may differ
- If something breaks, you get to keep both pieces

That said, we've put care into making this useful and safe. The plugin follows OpenClaw's official documentation and recommends secure defaults. If you find issues, please [report them](https://github.com/ClariSortAi/openclaw-manager-plugin/issues).

## License

MIT - see [LICENSE](LICENSE)

## Links

- [OpenClaw Documentation](https://docs.openclaw.ai) (formerly ClawdBot)
- [ClawHub Skill Registry](https://clawhub.ai)
- [Claude Code Plugins](https://code.claude.com/docs/plugins)
- [Report Issues](https://github.com/ClariSortAi/openclaw-manager-plugin/issues)

---

**Made for the Claude Code community.** If you can use Claude Code, you can use OpenClaw. Give it a star if this saves you time!
