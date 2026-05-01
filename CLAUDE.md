# Development Instructions for logseq-cli-skill

## Skill Development Workflow

This repo is a Claude Code marketplace plugin. Skill files live under `plugins/logseq-cli/skills/logseq-cli/`.

### Important Rules

1. **Only edit files in `plugins/logseq-cli/skills/logseq-cli/`** — never edit `~/.claude/` directly
2. **Update README.md** if functionality changes
3. **Ask user to review changes** before committing
4. **Bump `version` in `plugins/logseq-cli/.claude-plugin/plugin.json`** on every release
5. **Do not rename the `name` field in `.claude-plugin/marketplace.json`** — it is the user-visible install token (`logseq-cli@logseq-cli-skill`); renaming breaks existing installs

### File Structure

```
logseq-cli-skill/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog
├── plugins/
│   └── logseq-cli/
│       ├── .claude-plugin/
│       │   └── plugin.json       # Plugin manifest (bump version here on release)
│       └── skills/
│           └── logseq-cli/
│               ├── SKILL.md      # Edit this
│               └── examples/     # Edit these
├── .gitignore
├── README.md
├── LICENSE
└── CLAUDE.md                     # This file
```

### Local Development

Load the plugin without installing:

```fish
claude --plugin-dir ./plugins/logseq-cli
```

Skill invokes as `/logseq-cli:logseq-cli` when loaded via plugin.

### Before Committing

- [ ] Changes are in `plugins/logseq-cli/skills/logseq-cli/` only
- [ ] README.md updated if needed
- [ ] User has reviewed changes
- [ ] Version bumped in `plugin.json` (if releasing)
