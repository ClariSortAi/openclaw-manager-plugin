# OpenClaw Security Checklist

## Critical Version Requirement

**Minimum safe version: v2026.3.1**

Run `openclaw status` to check your version. If you are on anything older than v2026.3.1, upgrade immediately:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw config validate
openclaw gateway restart
```

The v2026.3.x line adds gateway auth bypass prevention, webhook auth enforcement, ACP sandbox inheritance, config backup permission hardening, SSRF DNS pinning, and macOS umask hardening on top of the 40+ fixes in v2026.2.12. For latest stable hardening and recovery tooling, prefer **v2026.5.7+**.

### Known Critical Vulnerabilities

#### v2026.1.29 Patches (January 30, 2026)

| CVE | Severity | Description | Fixed In |
|-----|----------|-------------|----------|
| CVE-2026-25253 | **Critical (CVSS 8.8)** | One-click RCE: visiting a malicious site or clicking a crafted link leaks the gateway auth token, giving attackers full gateway control | v2026.1.29 |
| CVE-2026-24763 | High | Command injection via crafted input | v2026.1.29 |
| CVE-2026-25157 | High | Command injection via crafted input | v2026.1.29 |

#### v2026.2.12 Emergency Patch (February 12, 2026)

Over 40 security vulnerabilities patched in a single emergency release:

| CVE | Severity | Description | Fixed In |
|-----|----------|-------------|----------|
| CVE-2026-26322 | **High (CVSS 7.6)** | SSRF in Gateway tool — explicit deny policies and hostname allowlists added | v2026.2.12 |
| CVE-2026-26319 | **High (CVSS 7.5)** | Missing Telnyx webhook authentication | v2026.2.12 |
| CVE-2026-26329 | High | Path traversal in browser upload | v2026.2.12 |

Additional v2026.2.12 hardening:
- SSRF protection: explicit deny policies and hostname allowlists
- Path traversal prevention: skill sync limited to `skills/` root directory
- Prompt injection mitigation: browser/web content shifted to "untrusted by default"
- Transcript compaction now strips detailed tool results to reduce prompt injection replay risk

**Breaking change:** POST `/hooks/agent` rejects `sessionKey` overrides by default.

#### Post-v2026.2.12 CVEs (February 2026)

| CVE | Severity | Description | Fixed In |
|-----|----------|-------------|----------|
| CVE-2026-27001 | High | Workspace path embedded into prompts without sanitization (prompt injection risk) | v2026.2.19 |
| CVE-2026-27004 | High | Session transcript content could leak across peer sessions in multi-user environments | v2026.2.19 |
| CVE-2026-27484 | High | Discord moderation authorization bypass: non-admin users could spoof sender identity | v2026.2.19 |
| CVE-2026-27485 | Medium | Symlink handling in local skill packaging could disclose local files | v2026.2.19 |
| CVE-2026-27488 | High | Cron webhook delivery SSRF: webhook targets could reach private/internal endpoints | v2026.2.19 |
| CVE-2026-27576 | Medium | ACP prompt-size checks missing in local stdio bridge | v2026.2.19 |

A January 2026 audit identified 512 total vulnerabilities (8 critical). Over 70 additional security fixes were released across v2026.2.12 and v2026.2.19.

#### v2026.3.x Security Hardening (March 2026)

| Area | Description | Fixed In |
|------|-------------|----------|
| Gateway auth | Security canonicalization with auth bypass prevention | v2026.3.2 |
| Webhook auth | Request auth enforcement for BlueBubbles and Google Chat | v2026.3.2 |
| Browser | Output boundary hardening | v2026.3.2 |
| Config backup | Permission enforcement (0600) for backup files | v2026.3.2 |
| SSRF | DNS pinning guard maintenance | v2026.3.2 |
| Node camera | URL download restrictions | v2026.3.2 |
| ACP sandbox | Inheritance enforcement for sub-agents | v2026.3.2 |
| Feishu webhooks | Ingress rate-limiting with stale-window pruning | v2026.3.1 |
| macOS | LaunchAgent umask hardening (077) | v2026.3.1 |
| WebSocket | Plaintext loopback-only enforcement | v2026.3.1 |
| Gateway | Closes repeated unauthorized request floods per connection | v2026.3.1 |
| Edit tools | Workspace boundary error fixes | v2026.3.1 |
| Compaction | Removed post-compaction audit injection message | v2026.3.1 |
| Browser redirects | Blocks private-network redirect hops in strict browser navigation flows | v2026.3.8 |
| `system.run` | Binds approved `bun`/`deno run` script operands to on-disk snapshots before execution | v2026.3.8 |
| Trusted-proxy WebSocket | Enforces browser origin validation for browser-originated connections to prevent CSWSH privilege escalation (`GHSA-5wcw-8jjv-m286`) | v2026.3.11 |
| Plugin loading | Disables implicit workspace plugin auto-load without explicit trust decision (`GHSA-99qw-6mr3-36qr`) | v2026.3.12 |
| Exec approval visibility and binding | Hardens approval prompts and parser/binding paths against Unicode obfuscation, ambiguous wrapper forms, and script-runner indirection (`GHSA-pcqg-f7rg-xfvv`, `GHSA-9r3v-37xh-2cf6`, `GHSA-f8r2-vg7x-gh8m`, `GHSA-57jw-9722-6rf2`, `GHSA-jvqh-rfmh-jh27`, `GHSA-x7pp-23xv-mmr4`, `GHSA-jc5j-vg4r-j5jx`) | v2026.3.12 |
| Owner-only command surfaces | Restricts `/config` and `/debug` to sender ownership boundaries even for otherwise authorized non-owner senders (`GHSA-r7vr-gr74-94p8`) | v2026.3.12 |
| Shared-token scope sanitization | Clears unbound client-declared scopes on shared-token WebSocket connects to prevent self-declared privilege expansion (`GHSA-rqpp-rjj8-7wv8`) | v2026.3.12 |
| Session and workspace boundary guards | Blocks admin-only persistent browser profile mutations from write-scoped `browser.request`, rejects public lineage override fields, and enforces sandbox session-tree visibility in `session_status` (`GHSA-vmhq-cqm9-6p7q`, `GHSA-2rqg-gjgv-84jm`, `GHSA-wcxr-59v9-rxr8`) | v2026.3.12 |
| Pairing scope caps | Caps issued/verified paired-device token scopes to approved device scope baselines (`GHSA-2pwv-x786-56f8`) | v2026.3.12 |
| WebSocket pre-auth hardening | Shortens unauthenticated handshake retention and rejects oversized pre-auth frames earlier (`GHSA-jv4g-m82p-2j93`, `GHSA-xwx2-ppv2-wx98`) | v2026.3.12 |
| Host exec environment hardening | Blocks inherited `GIT_EXEC_PATH` in sanitized host exec environments to prevent helper-path steering (`GHSA-jf5v-pqgw-gm5m`) | v2026.3.12 |
| Browser proxy attachment limits | Restores shared media-store size caps so oversized persisted proxy payloads are rejected (`GHSA-6rph-mmhp-h7h9`) | v2026.3.12 |
| Feishu, LINE, and Zalo webhook auth hardening | Requires stronger Feishu webhook validation and reaction context checks, signature validation for empty LINE probes, and pre-auth rate limiting for invalid Zalo webhook secrets (`GHSA-g353-mgv3-8pcj`, `GHSA-m69h-jm2f-2pv8`, `GHSA-mhxh-9pjm-w7q5`, `GHSA-5m9r-p9g7-679c`) | v2026.3.12 |
| Pairing bootstrap | Replaces shared credential exposure in `/pair` and `openclaw qr` with short-lived bootstrap tokens | v2026.3.12 |
| Channel routing allowlists | Slack/Teams/Zalo default to stable ID matching; mutable name matching is break-glass via `dangerouslyAllowNameMatching` | v2026.3.12 |
| Pairing setup codes | Bootstrap setup codes are enforced as single-use to prevent replay and silent privilege widening | v2026.3.13 |
| Telegram webhook auth | Validates webhook secret before parsing request bodies to reject unauthenticated payloads early | v2026.3.13 |
| iMessage remote attachments | Rejects unsafe remote attachment paths before spawning SCP to prevent shell metacharacter injection | v2026.3.13 |
| External content boundaries | Strips zero-width/soft-hyphen marker splitting tricks to preserve untrusted content boundaries | v2026.3.13 |
| Exec approval parsing | Expands fail-closed parsing for wrapper forms (`pnpm`, `env`, PowerShell `-File`/`-f`, Perl `-M`/`-I`, shell line continuation) | v2026.3.13 |
| Docker secrets handling | Prevents gateway token leakage through Docker build-context handling | v2026.3.13 (`v2026.3.13-1` tag path) |
| Browser relay migration safety | Removes legacy Chrome extension relay path/surfaces and standardizes browser profiles on supported attach flows (`existing-session` / `user`) | v2026.3.22 |
| Exec environment hardening expansion | Blocks additional host-exec env injection vectors (`MAVEN_OPTS`, `SBT_OPTS`, `GRADLE_OPTS`, `ANT_OPTS`, `GLIBC_TUNABLES`, `DOTNET_ADDITIONAL_DEPS`) and tightens Gradle redirect handling | v2026.3.22 |
| Windows media path hardening | Blocks remote-host `file://` and UNC/network path coercion in media loading paths to prevent SMB credential leakage | v2026.3.22 |
| Discovery fail-closed behavior | Rejects unresolved Bonjour/DNS-SD endpoints so TXT-only hints cannot steer routing or SSH auto-targeting | v2026.3.22 |
| Sandbox media dispatch hardening | Closes `mediaUrl`/`fileUrl` alias bypass paths so outbound tools/messages cannot escape media-root restrictions | v2026.3.24 |
| Trusted-proxy auth tightening | Rejects mixed shared-token trusted-proxy configs and removes implicit same-host local-direct fallback auth | v2026.3.31 |
| Owner-only tool HTTP invoke guard | Tightens `/tools/invoke` authorization so owner-only tools stay off HTTP invoke paths | v2026.3.31 |
| Handshake brute-force protection | Keeps shared-auth rate limiting active during WebSocket handshake attempts even with fake device-token candidates | v2026.3.31 |
| Exec environment sanitization expansion | Blocks additional proxy/TLS/Docker/Python package index env override vectors in approved host exec paths | v2026.3.31 |
| ACP dangerous-tool approvals | Replaces ACP dangerous-tool-name override behavior with semantic approval classes so indirect exec/control tools still require explicit approval | v2026.3.31 |
| Webhook secret comparison hardening | Replaces ad-hoc webhook secret comparisons with shared constant-time comparison helpers across BlueBubbles, Feishu, Mattermost, Telegram, Twilio, and Zalo | v2026.4.2 |
| Exec environment override hardening | Blocks additional host env override pivots for package roots, runtimes, compiler include paths, and credential/config locations | v2026.4.2 |
| Dotenv interpreter pin protection | Prevents workspace `.env` overrides of pinned Python interpreter env vars used by trusted helper paths | v2026.4.2 |
| Provider endpoint policy centralization | Consolidates native-vs-proxy classification, auth/header shaping, and TLS transport policy across provider HTTP/stream/websocket paths | v2026.4.2 |
| Session kill scope enforcement | Requires operator scopes and pre-lookup authorization for session-kill HTTP paths | v2026.4.2 |
| Slack interactive allowlist enforcement | Applies global owner `allowFrom` controls to Slack block actions/modals with stricter sender verification so interactive events cannot bypass allowlist intent | v2026.4.14 |
| Gateway-tool dangerous-flag guard | Blocks model-facing `config.patch`/`config.apply` from newly enabling flags surfaced as dangerous by `openclaw security audit` | v2026.4.14 |
| Local attachment canonical-path fail closed | Fails closed when attachment `realpath` canonicalization fails so path checks cannot silently degrade to non-canonical comparisons | v2026.4.14 |
| Browser SSRF enforcement expansion | Enforces browser SSRF policy on snapshot/screenshot/tab routes while preserving explicit strict-mode semantics | v2026.4.14 |
| Hook wake trust downgrade | Forces owner downgrade for untrusted `hook:wake` system events to prevent privilege confusion in wake hooks | v2026.4.14 |
| Teams SSO sender allowlist checks | Enforces sender allowlist policy on Teams SSO signin invokes | v2026.4.14 |
| Config snapshot alias redaction | Redacts `sourceConfig` and `runtimeConfig` aliases in config snapshot redaction paths | v2026.4.14 |
| Built-in tool name-collision hardening | Rejects client tool definitions whose names normalize-collide with built-in tools (or each other), preventing local `MEDIA:` trust inheritance via spoofed tool names | v2026.4.15 |
| Gateway auth hot-reload parity | Resolves active bearer auth per HTTP request/upgrade path so `secrets.reload` or config rotation immediately applies across `/v1/*`, `/tools/invoke`, and plugin routes | v2026.4.15 |
| MCP loopback auth/origin hardening | Uses constant-time bearer comparison on `/mcp` and rejects non-loopback browser-origin requests before auth evaluation | v2026.4.15 |
| QMD `memory_get` canonical path restrictions | Rejects arbitrary workspace markdown reads and only allows canonical memory files plus active indexed QMD documents | v2026.4.15 |
| Webchat media path hardening | Enforces local-root containment and rejects remote-host `file://` media embedding paths | v2026.4.15 |
| Busybox/toybox exec interpreter removal | Removes busybox and toybox from the list of safe-to-approve interpreter-like exec binaries so approval prompts cannot launder arbitrary commands through them | v2026.4.12 |
| Empty approver list approval bypass prevention | Prevents an empty approver list from inadvertently granting explicit approval authorization to unapproved callers | v2026.4.12 |
| Shell-wrapper detection broadening | Broadens shell-wrapper classification and blocks `env`-argv assignment injection so additional wrapper forms cannot bypass exec approval checks | v2026.4.12 |
| Restricted profile widening removal | Configured `tools.exec` / `tools.fs` sections no longer implicitly add those tools under restrictive profiles; use explicit `tools.alsoAllow` when intended | v2026.4.29 |
| Plugin state isolation | Plugin SDK keyed state stores use restart-safe plugin-isolated storage with TTL/eviction controls | v2026.4.29 |
| File-transfer path policy | Bundled file-transfer operations require default-deny per-node allowed paths, operator approval, canonical preflight checks, and symlink traversal is refused unless explicitly enabled | v2026.5.3 |
| Invalid config fail-closed behavior | Gateway startup and hot reload stop auto-restoring invalid config; `openclaw doctor --fix` owns safe repair/migration behavior | v2026.5.3 |
| Source-only plugin package rejection | Source-only plugin packages are rejected before runtime load so installs must provide runtime-ready package artifacts | v2026.5.3 |
| Windows host-env helper hardening | Windows audit/update helper resolution pins trusted install-root paths and blocks workspace dotenv redirection of `SystemRoot`, `WINDIR`, `LOCALAPPDATA`, and command-wrapper selection | v2026.5.4 |
| Docker Compose capability hardening | Bundled Compose drops `NET_RAW`/`NET_ADMIN` and enables `no-new-privileges` for the gateway container | v2026.5.5 |
| Channel/native command authorization | Native command handlers enforce owner boundaries, Telegram `accessGroup:*` applies to DM/group/native/callback paths, and LINE `dmPolicy: "open"` requires wildcard `allowFrom` | v2026.5.5-v2026.5.7 |
| Active Memory and inline skill authorization | Global Active Memory toggles require admin scope and inline skill tool dispatch runs through before-tool-call authorization hooks | v2026.5.7 |
| Beta provider SecretRef tightening | Provider `apiKey` config values resolve only through structured SecretRefs (`secrets.providers[id]` / `secrets.defaults`) instead of broad env-var-looking strings | v2026.5.12 beta |
| Beta Windows sandbox home blocking | Windows `USERPROFILE` joins blocked sandbox home roots so `.codex`, `.openclaw`, `.ssh`, and similar credential dirs are denied even if `HOME` points elsewhere | v2026.5.12 beta |

**Government advisories:**
- Belgium's Centre for Cybersecurity issued an emergency advisory classifying CVE-2026-25253 as critical
- The Dutch Data Protection Authority warned about serious cybersecurity and privacy risks and estimated ~20% of available plugins may contain malware
- Meta banned OpenClaw over security concerns

### Breaking Change: `auth: "none"` Permanently Removed

As of v2026.1.29, `gateway.auth.mode: "none"` is no longer accepted. All gateways require authentication. If you had `"none"` in your config, the gateway will refuse to start. Fix:

```bash
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
openclaw gateway restart
```

### Anthropic OAuth Tokens Blocked

Anthropic has officially blocked OpenClaw from using Claude OAuth tokens. You must use direct API keys:

```bash
# This will NOT work:
openclaw models auth  # OAuth flow for Anthropic is blocked

# Use API keys directly instead:
openclaw models auth setup-token --provider anthropic
# Then paste your API key from console.anthropic.com
```

## Quick Audit

```bash
openclaw security audit --deep
```

## Critical Settings

### 1. Gateway Binding
```bash
# Check current binding
openclaw config get gateway.bind

# Secure: Local only (default)
openclaw config set gateway.bind loopback

# Less secure: LAN access
openclaw config set gateway.bind 0.0.0.0  # NOT recommended
```

**Recommendation:** Keep `loopback` unless you need remote access. Use Tailscale for secure remote access.

### 2. Authentication Mode
```bash
# Use token auth (secure)
openclaw config set gateway.auth.mode token

# Generate strong token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"
```

If both `gateway.auth.token` and `gateway.auth.password` are configured (including SecretRefs), v2026.3.7+ requires an explicit `gateway.auth.mode` (`token` or `password`).

**Alternative: Trusted reverse proxy**
```bash
openclaw config set gateway.auth.mode trusted-proxy
# Requires an authenticated reverse proxy with TLS in front of the gateway
```

### 3. Tailscale Authentication

When `gateway.auth.allowTailscale` is enabled, OpenClaw verifies identity headers (`tailscale-user-login`) by resolving the client IP through the local Tailscale daemon:

```bash
openclaw config set gateway.auth.allowTailscale true
```

This allows secure remote access without exposing the gateway to the public internet. Tailscale nodes are verified against the local daemon rather than trusting headers blindly.

### 4. DM Policy
```bash
# Check current policy
openclaw config get channels.<channel>.dmPolicy

# Secure: Require pairing (default)
openclaw config set channels.slack.dmPolicy pairing

# Alternative: Allowlist only
openclaw config set channels.slack.dmPolicy allowlist
openclaw config set channels.slack.allowFrom '["U12345678"]'
```

**Never use `open` policy without understanding the risks.**

### 5. Group Policy
```bash
# Secure: Allowlist only
openclaw config set channels.slack.groupPolicy allowlist

# Require mentions in groups
openclaw config set channels.slack.requireMention true
```

### 6. Sandbox Mode

OpenClaw supports three workspace access levels:

| Level | Mount | Description |
|-------|-------|-------------|
| `none` | Inaccessible | Agent cannot access workspace |
| `ro` | `/agent` (read-only) | Agent can read but not write |
| `rw` | `/workspace` (read/write) | Agent has full workspace access |

Sandbox scope options:

| Scope | Description |
|-------|-------------|
| `agent` | Per-agent container (recommended) |
| `session` | Per-session container |
| `shared` | Single container for all (NOT recommended) |

```bash
# Enable sandboxing for all sessions
openclaw config set agents.defaults.sandbox.mode all

# Restrict workspace access
openclaw config set agents.defaults.sandbox.workspaceAccess none

# Set sandbox scope
openclaw config set agents.defaults.sandbox.scope agent
```

### 7. Tools Profile (v2026.3.2+)

The `tools.profile` setting establishes a base tool allowlist. In v2026.3.x, defaults can vary by onboarding path (for example, local onboarding fallback changed in v2026.3.7), so set this explicitly. This is a critical security boundary for multi-user gateways.

| Profile | Description |
|---------|-------------|
| `messaging` | Chat-only tools — no exec, no filesystem, no browser (recommended for untrusted/multi-user surfaces) |
| `coding` | Adds exec, filesystem, browser tools |
| `full` | No restrictions (personal use only) |

```bash
# Per-agent override
openclaw config set agents.list.support-bot.tools.profile "messaging"
openclaw config set agents.list.personal.tools.profile "coding"
```

### 8. Tool Execution Security (v2026.3.2+)

Control how exec/shell tools are handled:

```bash
# Deny all exec for untrusted agents
openclaw config set agents.defaults.tools.exec.security "deny"

# Require approval for each exec call
openclaw config set agents.defaults.tools.exec.security "ask"
```

### 9. Control Plane Tool Denials

In production, deny high-risk tools that allow agents to modify the gateway itself:

```json
{
  "agents": {
    "defaults": {
      "tools": {
        "deny": ["gateway", "cron", "sessions_spawn", "sessions_send"]
      }
    }
  }
}
```

Also restrict elevated tools (`exec`, `browser`, `web_fetch`, `web_search`) to trusted agents or use tool allowlists:

```bash
openclaw config set agents.defaults.tools.deny '["gateway","cron","sessions_spawn","sessions_send"]'
```

### 10. mDNS Discovery

OpenClaw can broadcast gateway presence via mDNS. Three modes:

| Mode | Description |
|------|-------------|
| `minimal` | Recommended. Omits `cliPath` and `sshPort` |
| `off` | Disabled entirely |
| `full` | Includes operational details (NOT recommended) |

```bash
# Set minimal mode (recommended)
openclaw config set gateway.mdns.mode minimal

# Or disable entirely
openclaw config set gateway.mdns.mode off

# Alternatively, disable via environment variable
export OPENCLAW_DISABLE_BONJOUR=1
```

### 11. Model Selection

Use modern, instruction-hardened models for bots with tool access. Larger models resist prompt injection better — this is a security property, not just a quality preference.

- **Strongest injection resistance**: Use the largest tier available from your provider (e.g., Anthropic Opus, OpenAI GPT-4o)
- **Good balance**: Mid-tier models (e.g., Anthropic Sonnet, OpenAI GPT-4o-mini) for moderate-risk surfaces
- **Local models (Ollama)**: May be less robust against sophisticated prompt injection — pair with strict tool denials and sandbox
- Avoid older or smaller models for tool-enabled agents facing untrusted inboxes

```bash
openclaw config set agents.defaults.model "<provider>/<model>"
```

### 12. Multi-User Trust Heuristic (v2026.2.24+)

Detects likely shared-user ingress and flags potential multi-user abuse on single-user channels:

```bash
openclaw config set security.trust_model.multi_user_heuristic true
```

### 13. HTTP Security Headers (v2026.2.23+)

When exposing the gateway over direct HTTPS (no reverse proxy), enable Strict-Transport-Security:

```bash
openclaw config set gateway.security.hsts true
```

### 14. Filesystem Restriction

Restrict agent file access to the workspace directory only:

```bash
openclaw config set fs.workspaceOnly true
```

### 15. SecretRef Credential Management (v2026.3.2+)

Use the SecretRef system instead of inline credentials:

```bash
# Audit all credential references
openclaw secrets audit

# Plan secret rotation
openclaw secrets plan

# Apply changes
openclaw secrets apply
```

SecretRef covers 64 credential targets across runtime collectors, onboarding flows, and audit surfaces.

## File Permissions

### Check Permissions
```bash
ls -la ~/.openclaw
```

### Fix Permissions
```bash
# Config directory: owner only
chmod 700 ~/.openclaw

# Config file: owner read/write only
chmod 600 ~/.openclaw/openclaw.json

# Credentials: owner only
chmod -R 600 ~/.openclaw/credentials/
```

Use full-disk encryption on the gateway host for an additional layer of protection.

## Security Hardening Checklist

### Version & Patches
- [ ] Running v2026.3.1 or later (recommend v2026.5.7+ for latest stable plugin repair, channel diagnostics, Codex OAuth recovery, and channel reliability hardening)
- [ ] `auth: "none"` not present in config (permanently removed in v2026.1.29)
- [ ] If both `gateway.auth.token` and `gateway.auth.password` exist, `gateway.auth.mode` is explicitly set (v2026.3.7+)
- [ ] If using `trusted-proxy`, shared-token/mixed-auth fallback assumptions are removed and same-host callers still present a valid token (v2026.3.31+)
- [ ] If upgrading to v2026.4.2+, run `openclaw doctor --fix` to migrate legacy `tools.web.x_search.*` and `tools.web.fetch.firecrawl.*` keys to plugin-owned paths
- [ ] Host exec policy is pinned explicitly (`agents.defaults.tools.exec.security`) rather than relying on defaults introduced by recent releases
- [ ] If using restrictive tool profiles with configured exec/filesystem settings, add explicit `tools.alsoAllow` entries where those tools are truly intended (v2026.4.29+)
- [ ] If upgrading to v2026.5.2+, run `openclaw doctor --fix` to migrate legacy thread-spawn config to `threadBindings.spawnSessions`
- [ ] If enabling file-transfer, configure `plugins.entries.file-transfer.config.nodes` as default-deny per node and keep `followSymlinks` disabled unless the node path policy has been reviewed
- [ ] Validate config before restart because v2026.5.3+ fails closed instead of auto-restoring invalid config
- [ ] If using Slack interactive buttons/modals, validate `channels.<channel>.allowFrom` / pairing-owner policy after upgrade to v2026.4.14+ (interactive events now enforce global owner allowlists)
- [ ] If agents can call model-facing gateway config tools, confirm dangerous-flag enablement is handled via authenticated operator workflows (v2026.4.14 blocks model-side escalation)
- [ ] After rotating gateway auth token/SecretRef, verify HTTP surfaces (`/v1/*`, `/tools/invoke`, plugin routes) require the new bearer without waiting for a gateway restart (v2026.4.15+)
- [ ] Using direct API keys, not Anthropic OAuth tokens
- [ ] POST `/hooks/agent` sessionKey override behavior reviewed (rejected by default since v2026.2.12)
- [ ] If installing/updating official plugins on the beta npm channel, prefer the `2026.5.3-1` core npm hotfix when install scans flag distant `process.env` / API-send references in compiled bundled-plugin packages
- [ ] After upgrading through `v2026.5.5`, verify Codex OAuth model routes still point where intended; current stable preserves working `openai-codex/*` PI routes and repairs bad rewrites with `openclaw doctor --fix`
- [ ] If using beta provider API-key config, migrate broad env-var-looking `apiKey` strings to structured SecretRefs before relying on startup/auth discovery
- [ ] Config validated before restart: `openclaw config validate`

### Network Security
- [ ] Gateway bound to loopback (127.0.0.1)
- [ ] Token authentication enabled
- [ ] Strong, random gateway token
- [ ] Tailscale used for remote access (not LAN binding)
- [ ] mDNS set to `minimal` or `off`

### Access Control
- [ ] DM policy set to `pairing` or `allowlist`
- [ ] Operators understand pairing setup codes are single-use (v2026.3.13+)
- [ ] Group policy set to `allowlist`
- [ ] Mention required in groups
- [ ] `dangerouslyAllowNameMatching` left disabled unless break-glass behavior is explicitly required
- [ ] `tools.profile` set to `"messaging"` for untrusted surfaces (v2026.3.2+)
- [ ] `tools.exec.security` set to `"deny"` or `"ask"` for non-personal agents
- [ ] Elevated tool access restricted
- [ ] Unknown senders cannot execute commands
- [ ] Control plane tools denied (`gateway`, `cron`, `sessions_spawn`, `sessions_send`)
- [ ] `fs.workspaceOnly` enabled for agents with file access

### Sandboxing
- [ ] Sandbox mode enabled (`all` or `non-main`)
- [ ] Workspace access restricted (`none` or `ro`)
- [ ] Sandbox scope set to `agent` (not `shared`)
- [ ] Docker isolation configured (if applicable)

### Credentials
- [ ] File permissions set to 600/700
- [ ] Config backups at 0600 permissions (enforced in v2026.3.2+)
- [ ] No credentials in environment variables (use config or SecretRef)
- [ ] Docker build contexts do not include OpenClaw state/secrets files
- [ ] API keys stored via `openclaw configure` or `openclaw secrets apply`
- [ ] Tokens rotated monthly (API keys, email credentials, messaging tokens)
- [ ] SecretRef audit clean: `openclaw secrets audit`

### Logging & Monitoring
- [ ] Sensitive data redaction enabled
- [ ] Custom redaction patterns added via `logging.redactPatterns` if needed
- [ ] Log retention policy defined
- [ ] Session transcripts secured

### Skill & Plugin Validation
- [ ] Every ClawHub skill audited before installation (review source, check author, verify VirusTotal)
- [ ] Only trusted plugins installed (they run in-process with full privileges)
- [ ] Plugin allowlist configured via `plugins.allow`
- [ ] Install-time dangerous-code findings are reviewed; avoid bypassing fail-closed safety overrides unless risk is explicitly accepted (v2026.3.31+)
- [ ] Aware of ClawHavoc supply chain attack: 1,184+ malicious skills confirmed, 2,419 removed from ClawHub
- [ ] ClawHub VirusTotal integration active (automatic scanning since Feb 2026)

## Configuration Examples

### Minimal Secure Setup
```json
{
  "gateway": {
    "bind": "loopback",
    "auth": {
      "mode": "token"
    },
    "mdns": {
      "mode": "minimal"
    }
  },
  "channels": {
    "slack": {
      "dmPolicy": "pairing",
      "groupPolicy": "disabled"
    }
  },
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-4-7",
      "tools": {
        "profile": "messaging",
        "deny": ["gateway", "cron", "sessions_spawn", "sessions_send"]
      },
      "sandbox": {
        "mode": "all",
        "workspaceAccess": "ro",
        "scope": "agent"
      }
    }
  },
  "fs": {
    "workspaceOnly": true
  }
}
```

### Multi-User Setup (More Restrictive)
```json
{
  "gateway": {
    "bind": "loopback",
    "auth": {
      "mode": "token"
    },
    "mdns": {
      "mode": "off"
    }
  },
  "channels": {
    "slack": {
      "dmPolicy": "allowlist",
      "allowFrom": ["U12345", "U67890"],
      "groupPolicy": "allowlist",
      "groups": ["C11111"],
      "requireMention": true
    }
  },
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-4-7",
      "sandbox": {
        "mode": "all",
        "workspaceAccess": "none",
        "scope": "agent"
      },
      "tools": {
        "deny": ["gateway", "cron", "sessions_spawn", "sessions_send"],
        "elevated": {
          "enabled": false
        }
      }
    }
  },
  "session": {
    "dmScope": "per-channel-peer"
  },
  "plugins": {
    "allow": ["voice-call"]
  }
}
```

## Threat Model

### What OpenClaw Can Do
- Execute shell commands
- Read/write files
- Access network
- Send messages on your behalf
- Browse the web
- Spawn sub-agents (configurable depth)

### Attack Vectors
1. **Prompt Injection** - Malicious content in messages
2. **Social Engineering** - Tricking bot into harmful actions
3. **Credential Theft** - Accessing stored credentials
4. **Session Hijacking** - Cross-user context leakage
5. **Malicious Plugins/Skills** - ~20% of third-party plugins may contain malware (Dutch DPA estimate). The ClawHavoc campaign distributed 1,184+ malicious ClawHub skills carrying Atomic macOS Stealer (AMOS) and Windows infostealers
6. **Token Leakage** - CVE-2026-25253 demonstrated gateway token exfiltration via crafted links
7. **SSRF** - CVE-2026-26322 and CVE-2026-27488 showed gateway and cron webhook SSRF reaching internal endpoints
8. **Session Leakage** - CVE-2026-27004 demonstrated transcript content leaking across peer sessions in multi-user setups

### Mitigations
- Keep OpenClaw updated to latest version (minimum v2026.3.1, recommended v2026.5.7+)
- Use `tools.profile: "messaging"` for untrusted surfaces
- Strict access control (pairing/allowlist)
- Sandboxing for untrusted users
- Session isolation (per-peer scope)
- Disable elevated tools for groups
- Use modern, instruction-hardened models (Opus 4.7 with adaptive thinking)
- Deny control plane tools in production
- Use SecretRef instead of inline credentials (`openclaw secrets audit`)
- Audit all third-party skills and plugins before installation
- Rotate credentials monthly
- Run in Docker container for process isolation with health endpoints
- Validate config before restart (`openclaw config validate`)

## Incident Response

### Containment
```bash
# 1. Stop gateway immediately
openclaw gateway stop

# 2. Lock down binding
openclaw config set gateway.bind loopback

# 3. Disable risky channels
openclaw config set channels.slack.dmPolicy disabled
```

### Credential Rotation
```bash
# 1. Rotate gateway token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"

# 2. Rotate remote/provider tokens
# Regenerate API keys via provider dashboards (console.anthropic.com, etc.)

# 3. Re-authenticate channels
openclaw channels logout
openclaw channels login

# 4. Rotate messaging platform tokens (Slack, Telegram, Discord)
# Generate new tokens from each platform's developer portal
```

### Investigation
```bash
# Check gateway logs
cat /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

# Review sessions
ls ~/.openclaw/agents/main/sessions/

# Check config changes
cat ~/.openclaw/openclaw.json

# Full audit
openclaw security audit --deep

# Machine-readable audit output
openclaw security audit --deep --json
```

## Reporting Vulnerabilities

Email: security@openclaw.ai

Use responsible disclosure - avoid public discussion before fixes are available.
