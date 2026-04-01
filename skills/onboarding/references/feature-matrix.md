# Feature-to-Use-Case Matrix

Map user goals to specific OpenClaw features, configurations, and documentation sections.

## Use Case → Tools Profile

| Use Case | Tools Profile | Sandbox | Workspace Access |
|----------|--------------|---------|-----------------|
| Personal assistant | `full` | `none` | `rw` |
| Family/team | `messaging` | `all` | `ro` |
| Enterprise | `messaging` | `all` | `none` |
| Developer tool | `coding` | varies | `rw` |
| Public chatbot | `messaging` | `all` | `none` |

## Use Case → Security Configuration

| Use Case | DM Policy | Group Policy | Tool Denials | Session Isolation |
|----------|-----------|-------------|-------------|------------------|
| Personal | `pairing` | `disabled` | none | `main` |
| Family/team | `pairing` | `allowlist` | gateway, cron | `per-channel-peer` |
| Enterprise | `allowlist` | `allowlist` | gateway, cron, sessions_spawn, sessions_send, exec | `per-channel-peer` |
| Developer | `pairing` | `allowlist` | gateway | `main` or `per-channel-peer` |
| Public chatbot | `open` (with `requireMention: true`) | `allowlist` or `disabled` | all elevated + exec, browser, web_fetch | `per-channel-peer` |

**Warning:** `open` DM policy exposes the bot to anyone. Only use with full sandbox, messaging-only tools profile, and all elevated tool denials.

## Channel → Setup Complexity

Ranked by setup difficulty to help sequence channel connections:

| Difficulty | Channel | Type | Key Requirement |
|-----------|---------|------|----------------|
| Easy | WebChat | native | Nothing (built-in) |
| Easy | Telegram | native | BotFather token |
| Easy | IRC | native | Server address |
| Medium | WhatsApp | native | QR code from phone |
| Medium | Discord | native | Bot token + intents |
| Medium | Signal | native | signal-cli + phone number |
| Medium | Slack | native | App manifest + two tokens |
| Medium | BlueBubbles | native | BlueBubbles server on Mac |
| Medium | Google Chat | native | GCP project + Chat API |
| Medium | Feishu/Lark | native | Feishu Open Platform app |
| Hard | Microsoft Teams | plugin | Azure AD + Bot Framework |
| Hard | Gmail Webhooks | webhook | GCP project + Pub/Sub |
| Easy-Medium | Matrix, LINE, Mattermost, Nostr, Twitch, Zalo | plugin | Plugin install + provider tokens |

## Automation Goal → Feature Mapping

| Goal | Feature | Commands | Min Version |
|------|---------|----------|-------------|
| Scheduled messages | Cron jobs | `openclaw cron add` | v2026.1 |
| Email processing | Gmail webhooks | `openclaw webhooks gmail setup` | v2026.1 |
| Daily digest | Cron + multi-channel | `openclaw cron add --channel slack` | v2026.1 |
| CI/CD notifications | Cron + webhooks | `openclaw cron add` | v2026.1 |
| Delegated tasks | Sub-agents | `agents.defaults.subagents.maxSpawnDepth` | v2026.2.17 |
| Document processing | PDF tool | `agents.defaults.pdfModel` | v2026.3.2 |
| Code review | Diffs plugin | `openclaw plugins install @openclaw/diffs` | v2026.3.1 |
| Memory/search | Memory + Ollama | `memorySearch.provider = "ollama"` | v2026.3.2 |

## Environment → Installation Notes

| Environment | Install Method | Service Manager | Health Probes | Notes |
|-------------|---------------|----------------|---------------|-------|
| macOS | install.sh | LaunchAgent | n/a | umask 077 in v2026.3.1+ |
| Linux (systemd) | install.sh | systemd | n/a | `--install-daemon` flag |
| Windows WSL2 | install.sh in WSL | systemd (if enabled) | n/a | Source nvm first, check /etc/wsl.conf |
| Docker | npm install | Process supervisor | `/healthz`, `/readyz` | Bind `0.0.0.0`, use env vars for token |
| Kubernetes | Docker image | K8s orchestrator | `/healthz`, `/readyz` | PV for ~/.openclaw, Secrets for tokens |
| Cloud VM | install.sh + systemd | systemd | n/a | Tailscale recommended for remote access |

## Security Posture → Checklist Items

### Relaxed (Solo)
- [ ] Running v2026.3.1+
- [ ] Token auth enabled
- [ ] Gateway on loopback
- [ ] Pairing mode for DMs

### Moderate (Small Team)
All of Relaxed, plus:
- [ ] `per-channel-peer` session isolation
- [ ] `tools.profile: "messaging"` for non-admin agents
- [ ] Sandbox enabled
- [ ] Control plane tools denied
- [ ] Multi-user trust heuristic on

### Strict (Enterprise)
All of Moderate, plus:
- [ ] `tools.exec.security: "deny"`
- [ ] `fs.workspaceOnly: true`
- [ ] mDNS off
- [ ] HSTS enabled
- [ ] SecretRef for all credentials
- [ ] Allowlist (not pairing) for DMs
- [ ] Plugin allowlist configured
- [ ] Weekly security audit cron
- [ ] Log redaction patterns configured
- [ ] Credential rotation schedule defined
- [ ] Incident response runbook documented

## Model Recommendations by Use Case

OpenClaw is provider-agnostic. Pick based on priorities — quality, cost, speed, or privacy:

### By Provider

| Provider | Setup Command | Strengths | Cost |
|----------|--------------|-----------|------|
| Anthropic | `setup-token --provider anthropic` | Strong reasoning, tool safety, prompt injection resistance | API usage fees |
| OpenAI | `setup-token --provider openai` | Wide model selection, WebSocket streaming | API usage fees |
| Ollama | Local install | Free, private, no API keys, self-hosted | Hardware only |
| Kilo Code | `setup-token --provider kilocode` | Managed gateway, easy onboarding | Subscription |
| xAI / Grok | `setup-token --provider xai` | Alternative reasoning models | API usage fees |
| Moonshot / Kimi | `setup-token --provider moonshot` | Built-in web search with citations | API usage fees |
| MiniMax | `setup-token --provider minimax` | M2.7 model catalog (v2026.3.28+) | API usage fees |
| Vercel AI | `setup-token --provider vercel-ai` | Gateway with model routing | Varies |

### By Use Case

| Use Case | Suggested Starting Point | Thinking | Context | Notes |
|----------|-------------------------|----------|---------|-------|
| Personal assistant | Sonnet 4.6 or Ollama | adaptive | default | Sonnet balances quality and cost well |
| Family/team | Sonnet 4.6 or Haiku 4.5 | adaptive | default | Haiku keeps costs low with multiple users |
| Enterprise | Sonnet 4.6+ | adaptive | default | Choose based on compliance and budget |
| Developer | Sonnet 4.6 | adaptive | 1M | Good coding, reasonable cost; upgrade to Opus if budget allows |
| Public chatbot | Haiku 4.5 or Ollama | low/off | default | High volume — cost control is critical |
| Budget-conscious | Haiku 4.5 | low | default | Cheapest API option with good quality |
| Speed-sensitive | Haiku 4.5 | off | default | Fastest responses |
| Privacy-focused | Ollama (local) | n/a | varies | No data leaves your machine |
| Maximum quality | Opus 4.6 | adaptive | 1M | Best reasoning but highest cost — use when quality justifies it |
