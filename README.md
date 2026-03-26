# OpenClaw Manager Plugin for Claude Code

[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-6B5CE7?style=flat-square)](https://code.claude.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.3.3-blue?style=flat-square)](CHANGELOG.md)

> **If you can use Claude Code, you can use OpenClaw.** This plugin democratizes access to powerful AI-to-messaging integrations—no DevOps expertise required.

<video src="https://github.com/user-attachments/assets/0a62a2dd-8f6c-4121-8e15-54fe8f65a61f" controls width="100%"></video>

[OpenClaw](https://openclaw.ai) (formerly known as **ClawdBot**) is an AI gateway that connects Claude and other LLMs to messaging platforms like Slack, WhatsApp, Telegram, Discord, and more. This plugin turns Claude Code into your personal OpenClaw expert—handling installation, configuration, troubleshooting, and security hardening through natural conversation.

**This is a weekend project built to help more people access OpenClaw.** The setup process can be intimidating for newcomers, so this wrapped all that complexity into a conversational interface. Just tell Claude what you want, and it handles the rest.

## Why This Plugin?

Setting up OpenClaw (formerly ClawdBot) manually involves:
- Installing dependencies (Node.js v22.14.0+; Node 24 recommended, systemd)
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

Or start with the **interactive onboarding wizard** — it interviews you about your goals, channels, environment, and security needs, then generates a personalized setup journey:

```
> /onboarding
> /onboarding set up a family gateway with WhatsApp and Telegram
```

Claude handles the rest—running commands, checking configurations, explaining what's happening, and fixing issues.

## Quick Start

### Installation

**From GitHub (recommended):**
```bash
# 1. Add the marketplace (one-time)
/plugin marketplace add ClariSortAi/openclaw-manager-plugin

# 2. Install the plugin
/plugin install openclaw-manager@ClariSortAi-openclaw-manager-plugin
```

**For development / local testing:**
```bash
claude --plugin-dir ./openclaw-manager-plugin
```

### Usage

Once installed, invoke the skills:

```bash
# Personalized onboarding — interviews you, then builds a custom plan
/onboarding

# Or jump straight to specific tasks
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
| **Docker/K8s** | `Set up health probes for Kubernetes` |
| **PDF Analysis** | `Configure PDF tool for document processing` |


### Supported Platforms (23+)

**Native (built-in):**
- **Slack** - Socket Mode, native text streaming
- **WhatsApp** - QR code linking, multi-account
- **Telegram** - BotFather, streaming, DM topics
- **Discord** - Bot token, interactive UI
- **BlueBubbles** - iMessage with full feature support (recommended)
- **Signal** - Privacy-focused via signal-cli
- **Google Chat** - HTTP webhook integration
- **IRC** - Classic server support
- **WebChat** - Built-in gateway UI

**Plugin-based (install separately):**
- **Feishu/Lark** - Chinese enterprise chat (native since v2026.2.2)
- **Microsoft Teams** - `@openclaw/msteams`
- **Matrix** - `@openclaw/matrix`
- **LINE** - `@openclaw/line`
- **Mattermost** - `@openclaw/mattermost`
- **Nostr** - `@openclaw/nostr`
- **Nextcloud Talk, Synology Chat, Tlon, Twitch** - Plugin ecosystem
- **Zalo / Zalo Personal** - `@openclaw/zalo`, `@openclaw/zalouser`

### Supported Operating Systems

- macOS (native)
- Linux (native, systemd)
- Windows (WSL2 required)
- Docker / Kubernetes (health endpoints for liveness/readiness probes)

## Plugin Contents

```
openclaw-manager-plugin/
├── .claude-plugin/
│   ├── plugin.json              # Plugin manifest
│   └── marketplace.json         # Marketplace metadata
├── skills/
│   ├── openclaw-manager/        # Core management skill
│   │   ├── SKILL.md             # Main skill definition
│   │   ├── cli-reference.md     # Complete CLI command reference
│   │   ├── troubleshooting.md   # Common issues and solutions
│   │   ├── channel-setup.md     # Platform-specific setup guides (23+)
│   │   └── security-checklist.md # Security hardening guide
│   └── onboarding/              # Interactive onboarding wizard
│       ├── SKILL.md             # Interview flow + journey synthesis
│       └── references/
│           ├── journey-templates.md  # 7 journey templates by use case
│           └── feature-matrix.md     # Feature-to-goal mapping tables
├── CLAUDE.md                    # Development conventions
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
└── LICENSE
```

## Requirements

- **Claude Code** latest 
- For OpenClaw itself:
  - Node.js v22.14.0+ (Node 24 recommended; NOT Bun—causes WhatsApp/Telegram issues)
  - macOS, Linux, or Windows WSL2
  - **Minimum OpenClaw version: v2026.3.1** (critical security hardening + breaking change support)
  - **Recommended OpenClaw version: v2026.3.24+** (latest stable as of this plugin release alignment)

## How It Works

This plugin provides Claude with comprehensive OpenClaw knowledge through two skills:

### Core Skill (`/openclaw-manager`)
1. **Skill Definition** (`SKILL.md`) - Core capabilities, workflows, and decision-making guidance
2. **CLI Reference** - Every command, flag, and configuration path
3. **Troubleshooting Guide** - Diagnostic workflows and common fixes
4. **Channel Setup** - Platform-specific setup for 23+ channels
5. **Security Checklist** - Hardening recommendations, CVE awareness, and audit procedures

### Onboarding Wizard (`/onboarding`)
1. **Interview Flow** - 3-round structured interview about goals, channels, environment, and security
2. **Journey Templates** - 7 pre-built deployment paths (personal, family, enterprise, developer, Docker/K8s, upgrader, public chatbot)
3. **Feature Matrix** - Maps user goals to specific configurations and commands

Claude reads these documents and uses them to guide you through any OpenClaw task. The onboarding wizard asks about your specific situation first, then generates a tailored plan instead of generic docs.

## Examples

### Installing OpenClaw

```
You: Help me install OpenClaw on my Mac

Claude: I'll guide you through installing OpenClaw on macOS. Let me first check
if you have the prerequisites...

[Runs: node --version]

You have Node.js v22.14.0 - perfect. Now let's install the CLI...

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

**This is a side quest project built over a weekend to help democratize OpenClaw.**

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
