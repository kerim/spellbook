# claude-plugins

A Claude Code plugin marketplace by P. Kerim Friedman. Currently includes a skill for querying [Logseq](https://logseq.com) DB graphs via `@logseq/cli` — teaches Claude how to write Datalog queries against Logseq's DataScript database, parse EDN/JSON output, handle property types and tag inheritance, and build practical pipelines without opening Logseq Desktop.

## Installation

```
/plugin marketplace add kerim/claude-plugins
/plugin install logseq-cli@claude-plugins
```

Once installed, invoke the skill as `/logseq-cli:logseq-cli`.

## Sandbox configuration

`@logseq/cli` needs read access to your Logseq graph directory. Add the path to your graph to `~/.claude/settings.json`:

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

## Usage

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

## Examples

See [`plugins/logseq-cli/skills/logseq-cli/examples/common-queries.md`](plugins/logseq-cli/skills/logseq-cli/examples/common-queries.md) for copy-paste query patterns.

## Requirements

- [`@logseq/cli`](https://github.com/logseq/logseq/tree/master/packages/logseq-cli) v4+
- [`jet`](https://github.com/borkdude/jet) (for EDN → JSON conversion)
- A Logseq DB graph (not a markdown/file graph)

## License

MIT
