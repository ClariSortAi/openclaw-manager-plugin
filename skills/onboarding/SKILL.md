---
name: onboarding
description: This skill should be used when the user asks to "set up OpenClaw", "get started", "onboard me", "plan my setup", or "help me choose channels". Conducts an interactive interview, then generates a tailored deployment plan.
argument-hint: [describe your goals]
---

# OpenClaw Onboarding Wizard

This skill conducts a structured interview to understand the user's goals, environment, and constraints, then generates a personalized OpenClaw deployment journey with specific milestones, commands, and configurations.

## Interview Process

Conduct the interview in **three focused rounds**. After each round, acknowledge the answers before asking the next set. Do not ask all questions at once.

### Round 1: Who and Why

Ask these questions first:

1. **Experience level** — "Are you brand new to OpenClaw, upgrading from an older version, or migrating from another tool?"
2. **Primary use case** — "Which best describes your goal?"
   - Personal AI assistant (just me, on my own devices)
   - Family or small team (a few trusted people sharing one gateway)
   - Team/org deployment (10+ users, need access controls)
   - Developer tool (coding assistant, CI/CD integration, automation)
   - Chatbot/service (customer-facing or public-facing bot)

### Round 2: Channels and Environment

Continue with:

3. **Messaging platforms** — "Which platforms do you want to connect? Pick all that apply." List the relevant ones:
   - Consumer: WhatsApp, Telegram, Discord, Signal, iMessage/BlueBubbles, IRC
   - Enterprise: Slack, Microsoft Teams, Google Chat, Feishu/Lark, Mattermost
   - Other: Matrix, LINE, Nostr, Twitch, WebChat, Zalo
4. **Operating environment** — "Where will OpenClaw run?"
   - macOS (personal machine)
   - Linux server (systemd)
   - Windows WSL2
   - Docker / Kubernetes
   - Cloud VM (AWS, GCP, DigitalOcean, etc.)

### Round 3: Security, Model, and Automation

Continue with:

5. **Security posture** — "How locked down does this need to be?"
   - Relaxed (just me, trusted network)
   - Moderate (small group, some access controls)
   - Strict (multi-user, sandboxed, audited)
   - Enterprise (compliance requirements, secrets management, incident response plan)
6. **Model preference** — "Which AI provider do you want to use?"
   - Anthropic API key (Claude models — Opus for quality, Sonnet for speed/cost balance, Haiku for budget)
   - OpenAI API key (GPT models)
   - Ollama / local models (free, private, self-hosted)
   - Kilo Code gateway (managed hosting)
   - xAI / Grok
   - Moonshot / Kimi (includes web search)
   - Not sure yet / help me pick
7. **Automation goals** — "What do you want to automate?" (optional, can skip)
   - Scheduled messages / daily digests
   - Gmail/email processing
   - Code review or CI/CD notifications
   - Custom workflows with sub-agents
   - Nothing yet, just messaging for now

### Fast Track

If the user asks to skip the interview, wants "the simplest setup", or says "just get me started", skip directly to Template A (Personal AI Assistant) using their OS and a sensible default model (Sonnet 4.6 for API users, Ollama for local). Present it with a note that they can re-run `/onboarding` later to customize.

## Journey Synthesis

After collecting all answers, generate a personalized journey. Consult **[references/journey-templates.md](references/journey-templates.md)** for milestone templates and **[references/feature-matrix.md](references/feature-matrix.md)** for feature-to-use-case mapping.

### Output Format

Structure the journey as numbered milestones:

```
## Your OpenClaw Journey

Based on your answers:
- Use case: [summary]
- Channels: [list]
- Environment: [platform]
- Security: [level]

### Milestone 1: Install & Verify (Day 1)
[Specific commands for their OS]

### Milestone 2: Connect Your Channels (Day 1-2)
[Only the channels they selected, with links to channel-setup.md]

### Milestone 3: Lock It Down (Day 2-3)
[Security config matched to their posture]

### Milestone 4: [Use-case specific]
[Automation, sub-agents, team onboarding, etc.]

### What's Next
[Advanced features they didn't ask for but should know about]
```

### Tailoring Rules

Apply these rules when generating the journey:

- **New users**: Include every step, explain what each command does, add verification checks after each milestone
- **Upgraders**: Focus on breaking changes (tools.profile, ACP dispatch, iMessage deprecation), new features, and migration steps
- **Solo/personal**: Use `tools.profile: "full"`, skip sandbox, recommend `pairing` DM policy
- **Family/team**: Use `tools.profile: "messaging"`, enable `per-channel-peer` session isolation, sandbox with `ro` workspace
- **Enterprise/strict**: Full sandbox, `allowlist` policies, tool denials, SecretRef, HSTS, security audit cron
- **Docker/K8s**: Include health endpoint config, disk budgets, token via env var, `0.0.0.0` binding with auth
- **WSL2**: Include nvm sourcing, systemd setup, powershell wrapper notes
- **Each channel**: Include only setup steps for channels the user selected; reference the relevant section of channel-setup.md
- **Model selection**: Respect the user's provider choice. Do not default to the most expensive option. If "not sure", recommend based on use case — see the model recommendations table in [references/feature-matrix.md](references/feature-matrix.md). For budget-conscious users, recommend Sonnet or Haiku. For privacy-focused users, recommend Ollama.
- **Automation goals**: Map to specific cron/webhook/sub-agent recipes from the main [openclaw-manager SKILL.md](../openclaw-manager/SKILL.md) Recommended Workflows section

### Cross-References

When generating the journey, link to the existing plugin documentation:

- Installation steps: reference the main [SKILL.md](../openclaw-manager/SKILL.md) installation section
- Channel setup: reference [channel-setup.md](../openclaw-manager/channel-setup.md) for the specific channels selected
- Security: reference [security-checklist.md](../openclaw-manager/security-checklist.md) for the user's security level
- CLI commands: reference [cli-reference.md](../openclaw-manager/cli-reference.md) for command details
- Troubleshooting: reference [troubleshooting.md](../openclaw-manager/troubleshooting.md) for common issues

## Handling Edge Cases

- If the user gives vague answers ("I don't know"), suggest the most common path (personal assistant on macOS with WhatsApp + Telegram) and note they can adjust later.
- If the user wants channels not yet documented, note the plugin-based channel and link to OpenClaw docs at https://docs.openclaw.ai/channels.
- If the user is on a version older than v2026.3.1 (recommended for full feature support; v2026.2.12 is the minimum for security), make **upgrading** Milestone 0 before anything else.
- If the user describes a use case that doesn't fit the templates, adapt creatively — the templates are starting points, not constraints.
- If the user provides goals upfront (via argument-hint), skip redundant questions and jump to the relevant round.
