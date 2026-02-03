# Contributing to OpenClaw Manager Plugin

Thank you for your interest in contributing! This plugin helps Claude Code users manage OpenClaw installations, and community contributions make it better.

## Ways to Contribute

### Report Issues
- **Bug reports**: Plugin not triggering correctly, incorrect commands, outdated documentation
- **Feature requests**: New channels, additional troubleshooting scenarios, better guidance
- **Documentation improvements**: Typos, unclear instructions, missing information

### Submit Pull Requests
- Fix bugs or typos
- Add support for new OpenClaw features
- Improve troubleshooting guides
- Add new channel setup documentation
- Enhance security recommendations

## Development Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/ClariSortAi/openclaw-manager-plugin.git
   cd openclaw-manager-plugin
   ```

2. **Test locally with Claude Code**
   ```bash
   claude --plugin-dir .
   ```

3. **Make your changes** to files in `skills/openclaw-manager/`

4. **Test the skill**
   ```
   /openclaw-manager:openclaw-manager help
   ```

## Plugin Structure

```
openclaw-manager-plugin/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata (edit for version bumps)
├── skills/
│   └── openclaw-manager/
│       ├── SKILL.md             # Main skill - edit for core changes
│       ├── cli-reference.md     # CLI commands - update when OpenClaw changes
│       ├── troubleshooting.md   # Issues/fixes - add new solutions here
│       ├── channel-setup.md     # Channel guides - add new platforms here
│       └── security-checklist.md # Security - update recommendations
├── README.md
├── CHANGELOG.md
└── CONTRIBUTING.md
```

## Guidelines

### For Documentation Changes

- Keep commands accurate and tested against current OpenClaw versions
- Use consistent formatting (code blocks, tables, headers)
- Include both the command and expected output when helpful
- Update the relevant section, not just append to the end

### For SKILL.md Changes

- Keep the description under 300 characters for the frontmatter
- Ensure trigger phrases cover common user queries
- Test that Claude invokes the skill correctly

### For New Features

1. Open an issue first to discuss the change
2. Keep PRs focused on a single feature or fix
3. Update CHANGELOG.md with your changes
4. Test thoroughly with Claude Code

## Code of Conduct

- Be respectful and constructive
- Focus on the technical merit of contributions
- Help newcomers get started

## Questions?

Open an issue or start a discussion on GitHub. We're happy to help!

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
