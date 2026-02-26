# Changelog

All notable changes to the OpenClaw Manager Plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-02-26

### Added
- **9 new CVEs documented**: CVE-2026-26322 (SSRF, CVSS 7.6), CVE-2026-26319 (missing Telnyx auth), CVE-2026-26329 (path traversal), CVE-2026-27001 (prompt injection via workspace path), CVE-2026-27004 (session transcript leakage), CVE-2026-27484 (Discord auth bypass), CVE-2026-27485 (symlink file disclosure), CVE-2026-27488 (cron webhook SSRF), CVE-2026-27576 (ACP prompt-size bypass)
- **v2026.2.12 emergency patch documentation**: 40+ security fixes covering SSRF protection, path traversal prevention, prompt injection mitigation, session security hardening
- **ClawHavoc attack scale update**: 1,184+ malicious skills confirmed (up from 341), 2,419 total removed, VirusTotal integration now active on ClawHub
- **Feishu/Lark native channel**: Full setup guide for Chinese enterprise chat integration (v2026.2.2+)
- **Discord interactive UI**: Buttons, selects, and modals documentation (v2026.2.16+)
- **Discord WebSocket known issue**: v2026.2.24 1005/1006 disconnect workaround
- **Session management commands**: `openclaw sessions list`, `openclaw sessions cleanup` with disk-budget controls (v2026.2.23+)
- **Kilocode provider**: First-class model provider support (v2026.2.23+)
- **Moonshot/Kimi provider**: Web search with citation extraction (v2026.2.23+)
- **Multi-user trust heuristic**: `security.trust_model.multi_user_heuristic` configuration (v2026.2.24+)
- **HTTP security headers**: Strict-Transport-Security for direct HTTPS (v2026.2.23+)
- **`clawhub search` command**: Search ClawHub skill registry
- **`plugins remove` command**: Remove/uninstall plugins
- **Cron job troubleshooting**: Timezone issues, job not running, webhook SSRF awareness
- **Sub-agent troubleshooting**: Spawn failures, depth limits, unresponsive sub-agents
- **Session disk management troubleshooting**: Cleanup and disk budget configuration

### Changed
- Updated minimum safe version from v2026.1.29 to **v2026.2.12** throughout all documentation
- Expanded security checklist with v2026.2.12 breaking change (`/hooks/agent` sessionKey rejection)
- Updated threat model with SSRF and session leakage attack vectors
- Added Feishu/Lark to official plugins/channels table in CLI reference
- Expanded error patterns table with `spawn depth exceeded` and `WebSocket 1005/1006`
- Updated marketplace.json tags to include all supported channels (iMessage, Teams, Matrix, Feishu, Nostr, Zalo)
- Updated plugin.json keywords with Feishu, Lark, Nostr, Zalo

## [1.1.0] - 2026-02-19

### Added
- **Critical security documentation**: CVE-2026-25253 (one-click RCE, CVSS 8.8), CVE-2026-24763, CVE-2026-25157
- **Breaking change notice**: `auth: "none"` permanently removed in v2026.1.29
- **Anthropic OAuth block**: Documented that Anthropic blocked OAuth tokens for OpenClaw; users must use direct API keys
- **Government advisories**: Belgium CCB, Dutch DPA, and Meta ban warnings
- **Control plane tool denials**: Recommended deny list for production (`gateway`, `cron`, `sessions_spawn`, `sessions_send`)
- **iMessage channel setup**: Full setup guide for macOS native integration
- **Microsoft Teams**: Plugin-based channel via `@openclaw/msteams` (plugin-only since v2026.1.15)
- **Matrix, Nostr, Zalo**: Plugin-based channel documentation
- **Gmail webhook setup**: Complete Pub/Sub webhook configuration guide
- **Session isolation modes**: `main`, `per-channel-peer`, `per-account-channel-peer` with identity links
- **Nested sub-agents**: `maxSpawnDepth` and `maxChildrenPerAgent` configuration (v2026.2.17)
- **1M context window**: `params.context1m` for Anthropic models (v2026.2.17)
- **Sonnet 4.6 model support**: Model selection guidance recommending Opus 4.6 for tool safety
- **Tailscale authentication**: `gateway.auth.allowTailscale` configuration details
- **Sandbox scope options**: `agent`, `session`, `shared` scope documentation
- **Workspace access levels**: `none`, `ro`, `rw` with mount paths
- **mDNS discovery modes**: `minimal`, `off`, `full` with `OPENCLAW_DISABLE_BONJOUR`
- **ClawHub commands**: `clawhub install`, `clawhub update --all`, `clawhub sync --all`
- **Plugin management commands**: Full `openclaw plugins` CLI reference
- **Plugin slot system**: Exclusive plugin categories (e.g., memory slot)
- **Official plugins table**: Voice Call, Teams, Matrix, Nostr, Zalo, Memory
- **Skill/plugin validation checklist**: Audit guidance for third-party code
- **Bug report issue template**: `.github/ISSUE_TEMPLATE/bug_report.md`
- **CLAUDE.md**: Project development conventions for contributors
- **OpenClaw foundation transition note**: Steinberger → OpenAI, project moving to independent foundation

### Changed
- Expanded skill description to cover new channels (Teams, Matrix) and new features (skills, plugins)
- Updated minimum version requirement to v2026.1.29 throughout all documentation
- Enhanced security checklist with version/patch section, credential rotation cadence, and plugin validation
- Expanded threat model with malicious plugin risk and token leakage vector
- Updated configuration examples with model selection, tool denials, mDNS, and plugin allowlists
- Expanded error patterns table with `auth: "none"` and OAuth rejection entries
- Updated environment variables table with `OPENCLAW_GATEWAY_PORT` and `OPENCLAW_DISABLE_BONJOUR`
- Updated disclaimer to mention OpenAI and the OpenClaw foundation

### Fixed
- Corrected `SKILLS.md` → `SKILL.md` filename reference in README plugin contents tree
- Fixed changelog dates from `2025-` to `2026-`
- Fixed `How It Works` section referencing old `SKILLS.md` filename

## [1.0.1] - 2026-02-03

### Changed
- Added "formerly ClawdBot" naming context throughout documentation
- Added disclaimer section (no warranty, weekend project, use at your own risk)
- Updated tagline: "If you can use Claude Code, you can use OpenClaw"
- Added `clawdbot` keyword for discoverability

## [1.0.0] - 2026-02-03

### Added
- Initial release of OpenClaw Manager Plugin for Claude Code
- **Installation guidance** for macOS, Linux, and Windows WSL2
- **Channel configuration** support for:
  - Slack (Socket Mode with OAuth)
  - WhatsApp (QR code linking, multi-account)
  - Telegram (BotFather integration)
  - Discord (bot token setup)
- **Troubleshooting workflows** with diagnostic commands
- **Security hardening** checklist and audit guidance
- **Cron job management** for scheduled tasks
- Comprehensive CLI reference documentation
- Platform-specific setup guides

### Documentation
- SKILL.md with capabilities and quick reference
- cli-reference.md with full command documentation
- troubleshooting.md with diagnostic workflows
- channel-setup.md with OAuth/token setup guides
- security-checklist.md with hardening recommendations
