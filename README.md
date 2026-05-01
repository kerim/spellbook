# spellbook

A Claude Code plugin marketplace by P. Kerim Friedman.

## Plugins

| Plugin | Description |
|--------|-------------|
| [logseq-cli](#logseq-cli) | Query Logseq DB graphs via `@logseq/cli` |
| [logseq-db-plugin-api](#logseq-db-plugin-api) | Build Logseq plugins for DB graphs |

### Adding this marketplace

One-time setup — register spellbook with Claude Code:

```
/plugin marketplace add kerim/spellbook
```

---

## logseq-cli

A skill that teaches Claude how to query [Logseq](https://logseq.com) DB graphs via `@logseq/cli` — writing Datalog queries against Logseq's DataScript database, parsing EDN/JSON output, handling property types and tag inheritance, and building practical pipelines without opening Logseq Desktop.

### Install

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

The skill covers basic query syntax, property types, tag/class inheritance, EDN parsing, and troubleshooting.

See [`plugins/logseq-cli/skills/logseq-cli/examples/common-queries.md`](plugins/logseq-cli/skills/logseq-cli/examples/common-queries.md) for copy-paste patterns.

### Requirements

- [`@logseq/cli`](https://github.com/logseq/logseq/tree/master/packages/logseq-cli) v4+
- [`jet`](https://github.com/borkdude/jet) (for EDN → JSON conversion)
- A Logseq DB graph (not a markdown/file graph)

---

## logseq-db-plugin-api

A skill for building Logseq plugins that target **DB (database) graphs**. Covers core APIs, event-driven updates, tag detection, property management, Datalog patterns, and common pitfalls — organized into three layers: authoritative mirrored upstream docs, production-tested patterns, and related skill references.

### Install

```
/plugin install logseq-db-plugin-api@spellbook
```

Once installed, invoke the skill as `/logseq-db-plugin-api:logseq-db-plugin-api`.

### Usage

Ask Claude anything related to developing Logseq DB plugins:

- "Create a plugin that tracks tagged items"
- "How do I respond to database changes in real-time?"
- "Why is block.properties.tags unreliable?"
- "Write a Datalog query for tag hierarchies"
- "How do I define property types before using them?"

### Requirements

- Logseq 0.11.0+ (for DB graph support)
- `@logseq/libs` 0.3.0+
- A Logseq DB graph

### Mirrored upstream docs

The skill includes authoritative developer documentation mirrored verbatim from [`logseq/logseq libs/development-notes/`](https://github.com/logseq/logseq/tree/master/libs/development-notes). These files are licensed **AGPL-3.0** (see [`plugins/logseq-db-plugin-api/skills/logseq-db-plugin-api/references/logseq-official/LICENSE`](plugins/logseq-db-plugin-api/skills/logseq-db-plugin-api/references/logseq-official/LICENSE)). All other skill content is MIT.

#### Refreshing the mirror

When Logseq updates their developer docs, run from the repo root:

```bash
bash plugins/logseq-db-plugin-api/scripts/sync-logseq-docs.sh
```

The script does a shallow+sparse clone of `logseq/logseq` (only the `libs/development-notes/` subtree), copies seven specific markdown files into `skills/logseq-db-plugin-api/references/logseq-official/`, and appends a footer to each file recording the upstream commit SHA and fetch timestamp. It is idempotent — if the upstream HEAD SHA matches the stored `.last-synced-sha`, it exits immediately without rewriting anything.

The `upstream/logseq-repo/` directory it creates is gitignored and used only as a local cache; subsequent runs do `git pull --ff-only` instead of re-cloning.

After syncing, commit the updated files in `references/logseq-official/` and bump the version in `plugins/logseq-db-plugin-api/.claude-plugin/plugin.json`.

---

## License

MIT — except mirrored content under `plugins/logseq-db-plugin-api/skills/logseq-db-plugin-api/references/logseq-official/`, which is AGPL-3.0.
