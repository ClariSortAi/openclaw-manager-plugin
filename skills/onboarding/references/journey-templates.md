# Journey Templates

Pre-built milestone sequences for common user profiles. Select the matching template and customize based on interview answers.

## Template A: Personal AI Assistant (Solo User)

For a single user who wants an AI assistant on their phone/desktop via messaging apps.

### Milestone 1: Install & Verify
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
openclaw status
# Verify version is v2026.3.1+
```

### Milestone 2: Connect Channels
Set up 1-2 messaging platforms. Start with the one used most:
```bash
# Example: WhatsApp + Telegram
openclaw config set channels.whatsapp.dmPolicy pairing
openclaw channels login  # Scan QR code
openclaw config set channels.telegram.botToken "TOKEN"
openclaw config set channels.telegram.dmPolicy pairing
openclaw gateway restart
```
Then approve pairing from each platform.

### Milestone 3: Personalize
```bash
# Full tool access (solo user)
openclaw config set agents.defaults.tools.profile "full"

# Set your model (substitute your provider/model of choice)
openclaw models auth setup-token --provider <provider>
openclaw config set agents.defaults.model "<provider>/<model>"

# Examples by provider:
#   anthropic/claude-sonnet-4-6  (good balance of quality and cost)
#   anthropic/claude-opus-4-6    (highest quality, most expensive)
#   anthropic/claude-haiku-4-5   (budget-friendly, fast)
#   openai/gpt-4o               (OpenAI alternative)
#   ollama/llama3                (free, local, private)
#   kilocode/anthropic/claude-sonnet-4-6  (managed gateway)

# For reasoning-capable models, enable adaptive thinking
openclaw config set agents.defaults.params.thinkingLevel "adaptive"

openclaw gateway restart
```

### Milestone 4: Quality of Life
```bash
# Self-chat mode for WhatsApp (message yourself to trigger)
openclaw config set channels.whatsapp.selfChatMode true

# Daily summary cron
openclaw cron add --name "Daily Digest" --cron "0 8 * * *" \
  --message "Summarize anything important I missed overnight" \
  --channel whatsapp --to "self"
```

### What's Next
- Try the PDF tool for document analysis
- Explore ClawHub for community skills (`openclaw skills search`)
- Set up Gmail webhooks for email processing
- Configure sub-agents for specialized tasks

---

## Template B: Family / Small Team Gateway

For 2-10 trusted users sharing a single OpenClaw instance.

### Milestone 1: Install & Verify
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
openclaw status
```

### Milestone 2: Security First
Lock down before connecting any channels:
```bash
# Session isolation (each person gets their own context)
openclaw config set session.dmScope "per-channel-peer"

# Messaging-only tools (no shell/filesystem for others)
openclaw config set agents.defaults.tools.profile "messaging"

# Sandbox all sessions
openclaw config set agents.defaults.sandbox.mode all
openclaw config set agents.defaults.sandbox.workspaceAccess ro
openclaw config set agents.defaults.sandbox.scope agent

# Deny control plane tools
openclaw config set agents.defaults.tools.deny '["gateway","cron","sessions_spawn","sessions_send"]'

# Multi-user trust detection
openclaw config set security.trust_model.multi_user_heuristic true
```

### Milestone 3: Connect Channels
Set up channels with pairing so each family member approves individually:
```bash
openclaw config set channels.CHANNEL.dmPolicy pairing
openclaw config set channels.CHANNEL.groupPolicy allowlist
openclaw config set channels.CHANNEL.requireMention true
openclaw gateway restart
```
Each person messages the bot and receives a pairing code to approve.

### Milestone 4: Per-Person Tuning
Create a personal agent with elevated access for the admin:
```bash
openclaw agents add --id admin-agent
openclaw config set agents.list.admin-agent.tools.profile "full"
openclaw config set agents.list.admin-agent.sandbox.mode none
# Link admin identity to this agent
```

### What's Next
- Set up identity links so the same person on Slack and WhatsApp shares one session
- Configure allowlists for group chats
- Run `openclaw security audit --deep` monthly

---

## Template C: Team / Org Deployment (10+ Users)

For organizations with access control, compliance, and audit requirements.

### Milestone 0: Upgrade Check
```bash
openclaw status
# Must be v2026.3.1+. If not:
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw config validate
openclaw gateway restart
```

### Milestone 1: Infrastructure
```bash
# Install with systemd daemon
openclaw onboard --install-daemon

# Or Docker with health probes
# Liveness: GET /health
# Readiness: GET /ready
```

### Milestone 2: Security Hardening
```bash
openclaw config set gateway.bind loopback
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
openclaw config set gateway.mdns.mode off
openclaw config set gateway.security.hsts true

openclaw config set agents.defaults.tools.profile "messaging"
openclaw config set agents.defaults.tools.exec.security "deny"
openclaw config set agents.defaults.sandbox.mode all
openclaw config set agents.defaults.sandbox.workspaceAccess none
openclaw config set agents.defaults.sandbox.scope agent
openclaw config set agents.defaults.tools.deny '["gateway","cron","sessions_spawn","sessions_send"]'

openclaw config set session.dmScope "per-channel-peer"
openclaw config set security.trust_model.multi_user_heuristic true
openclaw config set fs.workspaceOnly true

# SecretRef for credential management
openclaw secrets plan
openclaw secrets apply

# Validate everything
openclaw config validate
openclaw security audit --deep
```

### Milestone 3: Channel Setup
Use allowlists (not pairing) for predictable access:
```bash
openclaw config set channels.slack.dmPolicy allowlist
openclaw config set channels.slack.allowFrom '["U_ADMIN_1","U_ADMIN_2"]'
openclaw config set channels.slack.groupPolicy allowlist
openclaw config set channels.slack.groups '["C_APPROVED_CHANNEL"]'
openclaw config set channels.slack.requireMention true
```

### Milestone 4: Monitoring & Compliance
```bash
# Automated security audit (weekly)
openclaw cron add --name "Weekly Security Audit" --cron "0 9 * * 1" \
  --message "Run openclaw security audit --deep and report findings" \
  --session isolated

# Session cleanup with disk budget
openclaw config set session.maintenance.maxDiskBytes 2147483648
openclaw config set session.maintenance.highWaterBytes 1717986918

# Log retention and redaction
openclaw config set logging.redactPatterns '["Bearer [A-Za-z0-9+/=]+","xoxb-[0-9A-Za-z-]+"]'
```

### Milestone 5: Remote Access
```bash
# Tailscale for secure remote access (not LAN binding)
openclaw config set gateway.auth.allowTailscale true
```

### What's Next
- Set up multi-agent routing (different agents for different teams)
- Configure plugin allowlists
- Create incident response runbook from security-checklist.md
- Regular credential rotation via `openclaw secrets plan`

---

## Template D: Developer Tool / Coding Assistant

For developers who want OpenClaw as a coding companion across messaging platforms.

### Milestone 1: Install
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
openclaw status
```

### Milestone 2: Developer Config
```bash
# Full tool access with coding profile
openclaw config set agents.defaults.tools.profile "coding"

# Set your preferred model (pick one)
openclaw models auth setup-token --provider <provider>
openclaw config set agents.defaults.model "<provider>/<model>"
# Sonnet 4.6 is a strong default for coding — fast, capable, reasonable cost
# Opus 4.6 for highest quality (expensive), Haiku for speed
# Ollama for local/private development

openclaw config set agents.defaults.params.thinkingLevel "adaptive"
openclaw config set agents.defaults.params.context1m true

# Workspace access for code
openclaw config set agents.defaults.sandbox.workspaceAccess rw
```

### Milestone 3: Connect Channels
Typically Slack (for team) + Discord (for community) + WhatsApp (for mobile):
```bash
# Slack for work
openclaw config set channels.slack.botToken "xoxb-..."
openclaw config set channels.slack.appToken "xapp-..."

# Personal channel with pairing
openclaw config set channels.whatsapp.dmPolicy pairing
openclaw channels login
```

### Milestone 4: Automation
```bash
# Sub-agents for specialized tasks
openclaw config set agents.defaults.subagents.maxSpawnDepth 3
openclaw config set agents.defaults.subagents.maxChildrenPerAgent 10

# CI/CD notification cron
openclaw cron add --name "Build Digest" --cron "0 17 * * 1-5" \
  --message "Summarize today's CI/CD build results and flag failures" \
  --channel slack --to "#dev-builds"
```

### What's Next
- Install the `diffs` plugin for code review
- Configure Gmail webhooks for PR notification processing
- Explore ClawHub for development-focused skills (`openclaw skills search`)
- Set up PDF tool for documentation analysis

---

## Template E: Docker / Kubernetes Deployment

For containerized deployments with orchestration.

### Milestone 1: Configure for Containers
```bash
# Bind to all interfaces (container networking)
openclaw config set gateway.bind "0.0.0.0"
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$OPENCLAW_GATEWAY_TOKEN"
```

### Milestone 2: Health & Monitoring
Built-in health endpoints (v2026.3.1+). Aliases: `/health` = `/healthz`, `/ready` = `/readyz`:
```yaml
# Kubernetes deployment snippet
livenessProbe:
  httpGet:
    path: /healthz
    port: 18789
  initialDelaySeconds: 10
readinessProbe:
  httpGet:
    path: /readyz
    port: 18789
  initialDelaySeconds: 5
```

### Milestone 3: Resource Management
```bash
# Disk budget for session storage
openclaw config set session.maintenance.maxDiskBytes 2147483648
openclaw config set session.maintenance.highWaterBytes 1717986918

# Validate config before deploying
openclaw config validate --json
```

### Milestone 4: Security in Containers
```bash
openclaw config set agents.defaults.tools.profile "messaging"
openclaw config set agents.defaults.sandbox.mode all
openclaw config set agents.defaults.tools.deny '["gateway","cron","sessions_spawn","sessions_send"]'
openclaw config set gateway.security.hsts true
```

### What's Next
- Set up persistent volume for `~/.openclaw/`
- Configure secrets via Kubernetes Secrets + SecretRef
- Add Prometheus metrics scraping via the audit --json endpoint
- Set up log aggregation from `/tmp/openclaw/`

---

## Template F: Upgrader (v2026.2.x to v2026.3.x)

For users upgrading through breaking changes.

### Milestone 0: Backup
```bash
cp -r ~/.openclaw ~/.openclaw.backup.$(date +%Y%m%d)
```

### Milestone 1: Upgrade
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw config validate --json
# Fix any invalid keys before restarting
```

### Milestone 2: Address Breaking Changes
```bash
# 1. tools.profile defaults vary across v2026.3.x
#    (v2026.3.2 introduced "messaging"; v2026.3.7 local onboarding fallback is "coding")
#    Existing installs keep their config, but verify explicitly:
openclaw config get agents.defaults.tools.profile
# Set explicitly for your use case:
openclaw config set agents.defaults.tools.profile "coding"

# 2. If both auth token and password are configured, set mode explicitly (v2026.3.7+)
openclaw config set gateway.auth.mode token

# 3. ACP dispatch is now enabled by default
#    Disable if not wanted:
openclaw config set acp.dispatch.enabled false

# 4. If using Zalo Personal, the login method changed:
openclaw channels login --channel zalouser

# 5. If using legacy iMessage, migrate to BlueBubbles
#    See channel-setup.md for BlueBubbles guide
```

### Milestone 3: Adopt New Features
```bash
# Adaptive thinking (Claude 4.6)
openclaw config set agents.defaults.params.thinkingLevel "adaptive"

# Config validation (run before every restart)
openclaw config validate

# PDF tool (uses your default model, or override)
# openclaw config set agents.defaults.pdfModel "<provider>/<model>"

# SecretRef for credential management
openclaw secrets audit
```

### Milestone 4: Security Catch-Up
```bash
openclaw security audit --deep --fix
# Review and apply any new recommendations
```

### What's Next
- Explore Telegram streaming and DM topics
- Try Feishu improvements (reactions, voice, doc tool)
- Set up health endpoints if running in Docker
- Review the full v2026.3.x changelog for features relevant to your setup

---

## Template G: Chatbot / Public-Facing Service

For customer-facing or community bots that receive messages from untrusted senders.

### Milestone 1: Install & Verify
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
openclaw status
```

### Milestone 2: Maximum Lockdown
Public-facing bots need the strictest security — untrusted input is the norm:
```bash
# Messaging-only tools (absolutely no shell, filesystem, or browser)
openclaw config set agents.defaults.tools.profile "messaging"
openclaw config set agents.defaults.tools.exec.security "deny"

# Full sandbox, no workspace access
openclaw config set agents.defaults.sandbox.mode all
openclaw config set agents.defaults.sandbox.workspaceAccess none
openclaw config set agents.defaults.sandbox.scope session

# Deny all control plane and elevated tools
openclaw config set agents.defaults.tools.deny '["gateway","cron","sessions_spawn","sessions_send","exec","browser","web_fetch"]'

# Restrict filesystem
openclaw config set fs.workspaceOnly true

# Per-session isolation (every conversation is independent)
openclaw config set session.dmScope "per-channel-peer"

# Multi-user trust heuristic
openclaw config set security.trust_model.multi_user_heuristic true
```

### Milestone 3: Channel Setup
Choose `open` policy only for the public channel; lock everything else:
```bash
# Public channel (e.g., Discord server or Telegram group)
openclaw config set channels.CHANNEL.dmPolicy open
openclaw config set channels.CHANNEL.allowFrom '["*"]'
openclaw config set channels.CHANNEL.requireMention true
# WARNING: open policy exposes the bot to anyone — ensure sandbox + tool denials are tight

# Admin channel (separate, locked down)
openclaw config set channels.ADMIN_CHANNEL.dmPolicy allowlist
openclaw config set channels.ADMIN_CHANNEL.allowFrom '["ADMIN_ID"]'
```

### Milestone 4: Model & Cost Control
Public bots can generate high volume. Choose a cost-effective model:
```bash
# Sonnet for quality/cost balance, Haiku for high-volume budget
openclaw models auth setup-token --provider <provider>
openclaw config set agents.defaults.model "<provider>/<model>"

# Consider Ollama for zero API cost at scale
# openclaw config set agents.defaults.model "ollama/<model>"

# Set session disk budget to prevent storage runaway
openclaw config set session.maintenance.maxDiskBytes 1073741824
openclaw config set session.maintenance.highWaterBytes 858993459
```

### Milestone 5: Monitoring
```bash
# Automated security audit
openclaw cron add --name "Daily Security Check" --cron "0 6 * * *" \
  --message "Run openclaw security audit --deep and flag any new issues" \
  --session isolated

# Session cleanup
openclaw cron add --name "Session Cleanup" --cron "0 3 * * *" \
  --message "Run openclaw sessions cleanup"
```

### What's Next
- Add rate limiting via reverse proxy (nginx, Cloudflare)
- Configure log redaction patterns for PII
- Set up alerting for security audit failures
- Consider Tailscale for admin-only remote access to the gateway
