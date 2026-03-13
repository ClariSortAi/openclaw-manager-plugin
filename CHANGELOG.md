# Changelog

All notable changes to the OpenClaw Manager Plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- **User login mechanism guide**: Comprehensive documentation covering gateway authentication (token, password, trusted-proxy, Tailscale), channel login (WhatsApp QR, Zalo Personal), model provider authentication, pairing mechanisms, device management, and session isolation
- Documented v2026.3.7 auth-mode upgrade requirement: when both `gateway.auth.token` and `gateway.auth.password` are configured (including SecretRefs), `gateway.auth.mode` must be set explicitly.
- Added v2026.3.8 command coverage for `openclaw backup create` and `openclaw backup verify` in core skill, CLI reference, and troubleshooting recovery flows.
- Added v2026.3.8 advanced configuration coverage for `talk.silenceTimeoutMs`, `tools.web.search.brave.mode: "llm-context"`, and `openclaw acp --provenance`.

### Changed
- Updated `tools.profile` guidance across OpenClaw manager docs to reflect v2026.3.7 behavior: defaults can vary by onboarding path, so profile should be set explicitly.
- Added v2026.3.7 compatibility notes and recommendation language in the main skill/troubleshooting references while keeping minimum safe version at v2026.3.1.
- Updated recommended target version from v2026.3.7+ to v2026.3.8+ while preserving minimum safe baseline at v2026.3.1.
- Expanded security checklist with v2026.3.8 hardening notes (browser private-network redirect blocking and `system.run` approved script snapshot binding).

## [1.3.0] - 2026-03-03

### Added
- **12 new channel guides**: BlueBubbles, Signal, Google Chat, IRC, WebChat (native); LINE, Mattermost, Nextcloud Talk, Synology Chat, Tlon, Twitch (plugin); Zalo Personal rebuild documentation
- **Breaking changes section in SKILL.md**: Documents `tools.profile` default change, ACP dispatch enabled, Plugin SDK migration, Zalo Personal rebuild, iMessage deprecation
- **Tools profile documentation**: `tools.profile` setting (`"messaging"`, `"coding"`, `"full"`) as security boundary with per-agent overrides
- **Tool execution security**: `tools.exec.security` setting (`"deny"`, `"ask"`, `"allow"`) for approval workflows
- **Config validation CLI**: `openclaw config validate [--json]` and `openclaw config file` commands (v2026.3.2+)
- **Health endpoints**: Docker/K8s liveness (`/health`, `/healthz`) and readiness (`/ready`, `/readyz`) probes (v2026.3.1+)
- **PDF tool**: First-class PDF analysis with Anthropic/Google providers, configurable `pdfModel`, `pdfMaxBytesMb`, `pdfMaxPages` (v2026.3.2+)
- **SecretRef system**: `openclaw secrets plan/apply/audit` for managing 64 credential targets (v2026.3.2+)
- **Session attachments**: Inline file support for `sessions_spawn` with base64/utf8 encoding (v2026.3.2+)
- **Adaptive thinking**: Claude 4.6 defaults to `"adaptive"` thinking level (v2026.3.1+)
- **Telegram streaming**: Default `partial` mode with `sendMessageDraft` live preview (v2026.3.2+)
- **Telegram DM topics**: Per-DM topic configuration with topic-aware sessions (v2026.3.1+)
- **Feishu v2026.3.x improvements**: Reaction notifications, rich-text parsing, multi-account routing, `feishu_doc` tool, voice/TTS, webhook rate-limiting
- **New model providers**: xAI/Grok (v2026.2.6+), MiniMax M2.5 (v2026.3.2+), Vercel AI Gateway normalization (v2026.2.23+)
- **Ollama embeddings**: `memorySearch.provider = "ollama"` support (v2026.3.2+)
- **`OPENCLAW_SHELL` environment variable** (v2026.3.1+)
- **`diffs` plugin tool**: Read-only diff rendering (v2026.3.1+)
- **Filesystem restriction**: `fs.workspaceOnly` security setting
- **v2026.3.x security hardening table**: 13 new security items including gateway auth canonicalization, webhook auth enforcement, DNS pinning, ACP sandbox inheritance, macOS umask hardening
- **5 real-world workflow recipes**: Personal assistant, family/team gateway, Docker/K8s deployment, multi-channel daily digest, security lockdown after incident
- **New troubleshooting sections**: Tools profile issues, config validation, ACP dispatch, PDF tool, Plugin SDK breaking change, Zalo Personal login change
- **Interactive onboarding wizard** (`/onboarding`): 3-round interview flow that asks about goals, channels, environment, and security posture, then generates a personalized deployment journey with specific milestones and commands
- **7 journey templates**: Personal assistant, family/team, enterprise, developer tool, Docker/K8s, upgrader (v2026.2.x → v2026.3.x migration), and public chatbot/service
- **Feature-to-use-case matrix**: Maps tools profiles, security configs, channel complexity, automation features, and model recommendations to user goals

### Changed
- Updated minimum safe version from **v2026.2.12 to v2026.3.1** throughout all documentation
- Deprecated legacy iMessage channel — redirected to BlueBubbles with migration guidance
- Expanded official plugins table to include 10 native channels + 12 plugin channels + 4 utility plugins
- Updated security checklist from 11 to 15 critical settings (added tools profile, exec security, filesystem restriction, SecretRef)
- Updated hardening checklist with config validation, tools.profile, exec security, SecretRef audit
- Discord WebSocket 1005/1006 marked as fixed in v2026.3.1 (previously documented as known issue)
- Updated error patterns table with `invalid-config`, `tools not available`, `registerHttpHandler` errors
- Expanded model providers section with xAI, MiniMax, Vercel AI, and OpenAI WebSocket transport
- Updated plugin.json keywords and marketplace.json tags for new channels (BlueBubbles, Signal, Google Chat, IRC, LINE, Mattermost, Twitch, Docker, Kubernetes)

### Fixed
- README version badge now shows correct version
- README minimum version updated from v2026.1.29 to v2026.3.1

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
