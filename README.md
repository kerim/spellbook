# spellbook

A Claude Code plugin marketplace by P. Kerim Friedman.

## Plugins

| Plugin | Description |
|--------|-------------|
| [logseq-cli](plugins/logseq-cli/) | Query Logseq DB graphs via `@logseq/cli` |

---

## logseq-cli

A skill that teaches Claude how to query [Logseq](https://logseq.com) DB graphs via `@logseq/cli` — writing Datalog queries against Logseq's DataScript database, parsing EDN/JSON output, handling property types and tag inheritance, and building practical pipelines without opening Logseq Desktop.

### Installation

First, register this marketplace with Claude Code (one-time setup):

```
/plugin marketplace add kerim/spellbook
```

Then install the logseq-cli plugin from it:

```
/plugin install logseq-cli@spellbook
```

Once installed, invoke the skill as `/logseq-cli:logseq-cli`.

### Sandbox configuration

`@logseq/cli` needs permission to run and access your graph directory. Add to `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(logseq:*)",
      "Bash(jet:*)"
    ]
  }
}
```

Your graph directory also needs to be in the filesystem allowlist. Run `! claude /sandbox` to manage sandbox settings.

### Usage

Ask Claude anything that involves reading your Logseq graph:

- "Show me all tasks marked Doing"
- "List pages tagged with #Project"
- "Find blocks where the status property is In Progress"
- "Query my graph for all journal entries this week"
- "What properties does the Task class define?"

The skill covers:

- Basic query syntax and common patterns
- Property type system (strings, refs, dates, URLs, node types)
- Tag and class inheritance (`or-join` + `:logseq.property.class/extends`)
- Parsing EDN output with `jet` or the `-p` flag
- Troubleshooting common errors

### Examples

See [`plugins/logseq-cli/skills/logseq-cli/examples/common-queries.md`](plugins/logseq-cli/skills/logseq-cli/examples/common-queries.md) for copy-paste query patterns.

### Requirements

- [`@logseq/cli`](https://github.com/logseq/logseq/tree/master/packages/logseq-cli) v4+
- [`jet`](https://github.com/borkdude/jet) (for EDN → JSON conversion)
- A Logseq DB graph (not a markdown/file graph)

## License

MIT
