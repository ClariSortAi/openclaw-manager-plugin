# OpenClaw Manager Plugin - Development Guide

## Project Overview

This is a **Claude Code plugin** (documentation-only skill) that provides Claude with expert knowledge about managing OpenClaw, an AI gateway for messaging platforms. There is no executable code -- the plugin works by loading structured markdown into Claude's context.

## Structure

- `skills/openclaw-manager/SKILL.md` -- Main skill definition (frontmatter + instructions)
- `skills/openclaw-manager/cli-reference.md` -- CLI command reference
- `skills/openclaw-manager/channel-setup.md` -- Channel setup guides
- `skills/openclaw-manager/security-checklist.md` -- Security hardening guide
- `skills/openclaw-manager/troubleshooting.md` -- Diagnostic workflows
- `.claude-plugin/plugin.json` -- Plugin manifest (bump version here)
- `.claude-plugin/marketplace.json` -- Marketplace listing metadata

## Development Conventions

- **No executable code.** This plugin is purely documentation files.
- Keep SKILL.md frontmatter `description` under 300 characters.
- All OpenClaw commands should be tested against a real installation before documenting.
- Default to the most secure configuration when giving examples.
- Always reference minimum safe version (currently v2026.2.12) in security contexts.
- Use consistent markdown formatting: code blocks for commands, tables for reference data.

## When Updating

1. Check the latest OpenClaw release notes at https://github.com/openclaw/openclaw/releases
2. Verify commands still work against the current OpenClaw version
3. Update `CHANGELOG.md` with changes under an `[Unreleased]` section
4. Bump version in `.claude-plugin/plugin.json` for releases
5. Test the skill locally: `claude --plugin-dir .` then `/openclaw-manager help`

## Key External References

- OpenClaw docs: https://docs.openclaw.ai
- OpenClaw GitHub: https://github.com/openclaw/openclaw
- ClawHub skill registry: https://clawhub.ai
- Security reporting: security@openclaw.ai
