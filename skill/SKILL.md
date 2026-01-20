---
name: logseq-cli
description: Interface with Logseq DB graphs using the @logseq/cli command-line tool. Use this skill when working with Logseq databases in Claude Code - for querying graphs, searching blocks, listing graphs, or executing datalog queries. Essential for integration workflows that need direct database access.
---

# Logseq CLI

Comprehensive guide for using the Logseq CLI (`@logseq/cli`) within Claude Code workflows. This tool provides command-line access to Logseq DB graphs for querying, searching, and managing graph data.

## When to Use This Skill

Use this skill when you need to:

- Query Logseq DB graphs with datalog queries
- Search for blocks in Logseq graphs
- List available graphs on the system
- Test query syntax before implementing in code
- Export or import graph data
- Integrate Logseq data into Claude Code workflows

**Trigger phrases:**
- "query my Logseq graph"
- "search Logseq for..."
- "list my Logseq graphs"
- "test this Logseq query"
- "access Logseq database"

**Related skills:**
- `logseq-db-knowledge` - Understanding Logseq DB structure
- `logseq-db-plugin-api-skill` - Building Logseq plugins

---

## Claude Code Integration (CRITICAL)

### Sandbox Configuration

**MANDATORY: All Logseq CLI commands require `dangerouslyDisableSandbox: true`**

The Logseq CLI needs read access to `~/Library/Application Support/Logseq/` to access database files. This location is blocked by Claude Code's default sandbox.

**Why this is safe:**
- Logseq CLI is a read-only query tool
- No network access required for local queries
- No destructive operations (unless using import/export explicitly)
- Similar to using `dangerouslyDisableSandbox` for npm/pnpm install

### Bash Tool Pattern

**Always use this pattern in Claude Code:**

```typescript
Bash({
  command: 'logseq query -g "LSEQ 2025-12-15" -p \'[:find (pull ?b [*]) :where [?b :block/title]]\'',
  description: "Query Logseq graph for all blocks with titles",
  dangerouslyDisableSandbox: true  // REQUIRED for database access
})
```

### Shell Considerations

**Fish Shell (User's Default):**
- Quote handling works the same as bash/zsh
- Use single quotes for datalog queries: `'[:find ...]'`
- Escape quotes inside query if needed: `\'[:find ...]\'`

---

## Core Commands Reference

### `logseq list`

**Purpose:** List all available Logseq graphs (both DB and File graphs)

**Syntax:**
```bash
logseq list
```

**Options:**
- `-h, --help` - Print help

**Example Output:**
```
DB Graphs:
LSEQ 2025-12-15
Logseq friends

File Graphs:
logseq db import
```

**Usage in Claude Code:**
```typescript
Bash({
  command: 'logseq list',
  description: "List available Logseq graphs",
  dangerouslyDisableSandbox: true
})
```

**Important Notes:**
- Graph names are case-sensitive
- Use exact names from this output in `-g` flag
- DB graphs support full datalog queries
- File graphs have limited query support

---

### `logseq query`

**Purpose:** Execute datalog or entity queries against Logseq graphs

**Syntax:**
```bash
logseq query [args] [options]
```

**Options:**
- `-a, --api-server-token` - API server token to query current in-app graph
- `-g, --graphs` - Local graph(s) to query (REQUIRED for local queries)
- `-p, --properties-readable` - Show property values instead of IDs (HIGHLY RECOMMENDED)
- `-t, --title-query` - Invoke query on `:block/title` only
- `-h, --help` - Print help

**CRITICAL Syntax Rules:**

✅ **CORRECT:**
```bash
logseq query -g "GRAPH NAME" -p 'QUERY'
```

❌ **WRONG (returns empty results):**
```bash
logseq query -g "GRAPH NAME" -- 'QUERY'
```

**The `-p` flag is REQUIRED for meaningful output. Without it, you'll see entity IDs instead of values.**

**Query Types:**

1. **Datalog Query:**
   ```bash
   logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/title]]'
   ```

2. **Entity Query (by UUID):**
   ```bash
   logseq query -g "LSEQ 2025-12-15" -p '681eb44c-27ae-4c56-a5e5-109219ad8466'
   ```

3. **Entity Query (by db/id):**
   ```bash
   logseq query -g "LSEQ 2025-12-15" -p '1108'
   ```

**Claude Code Example:**
```typescript
Bash({
  command: 'logseq query -g "LSEQ 2025-12-15" -p \'[:find (pull ?b [:block/uuid :block/title]) :where [?b :block/tags ?t] [?t :block/title "contact"]]\'',
  description: "Find all blocks tagged with 'contact'",
  dangerouslyDisableSandbox: true
})
```

---

### `logseq search`

**Purpose:** Simple text search across block titles

**Syntax:**
```bash
logseq search [search-terms] [options]
```

**Options:**
- `-a, --api-server-token` - API server token to search current graph
- `-g, --graph` - Local graph to search (REQUIRED)
- `-r, --raw` - Print raw response
- `-l, --limit` - Limit max results (default: 100)
- `-h, --help` - Print help

**Important:** Only searches `:block/title` - does not search block content

**Example:**
```bash
logseq search "contact" -g "LSEQ 2025-12-15" -l 50
```

**Claude Code Example:**
```typescript
Bash({
  command: 'logseq search "friedman" -g "LSEQ 2025-12-15"',
  description: "Search for blocks with 'friedman' in title",
  dangerouslyDisableSandbox: true
})
```

---

### `logseq show`

**Purpose:** Display graph metadata and debugging information

**Syntax:**
```bash
logseq show [graphs]
```

**Options:**
- `-h, --help` - Print help

**Example:**
```bash
logseq show "LSEQ 2025-12-15"
```

**Use Cases:**
- Check graph creation date
- Verify graph exists
- Debug graph access issues

---

### `logseq export`

**Purpose:** Export graph to Markdown format

**Syntax:**
```bash
logseq export [options]
```

**Options:**
- `-g, --graph` - Local graph to export (REQUIRED)
- `-f, --file` - File to save export (REQUIRED)
- `-h, --help` - Print help

**Example:**
```bash
logseq export -g "LSEQ 2025-12-15" -f "/tmp/claude/export.md"
```

**Claude Code Example:**
```typescript
Bash({
  command: 'logseq export -g "LSEQ 2025-12-15" -f "/tmp/claude/logseq-export-$(date +%Y%m%d).md"',
  description: "Export Logseq graph to Markdown",
  dangerouslyDisableSandbox: true
})
```

---

### `logseq export-edn` / `logseq import-edn`

**Purpose:** Export/import graph in EDN (Extensible Data Notation) format

**Export Syntax:**
```bash
logseq export-edn -g "GRAPH" -f "output.edn"
```

**Import Syntax:**
```bash
logseq import-edn -g "GRAPH" -f "input.edn"
```

**Use Cases:**
- Programmatic graph manipulation
- Backup/restore workflows
- Graph migration

---

### Other Commands

**`logseq append`** - Append text to current page (requires API token)

**`logseq mcp-server`** - Run MCP (Model Context Protocol) server for Claude integration

**`logseq validate`** - Validate DB graph integrity

---

## Query Syntax Patterns (Comprehensive)

### Basic Datalog Structure

```clojure
[:find WHAT-TO-RETURN
 :where CONDITIONS]
```

**Components:**
- `:find` - What to return (pull pattern, variables, etc.)
- `:where` - Conditions that must be true

**Variables:**
- Start with `?` (e.g., `?b`, `?t`, `?title`)
- Bind to entities/values in `:where` clauses

---

### Finding Blocks by Title

**Simple title match:**
```clojure
[:find (pull ?b [*])
 :where
 [?b :block/title "Exact Title"]]
```

**Case-insensitive substring search:**
```clojure
[:find (pull ?b [*])
 :where
 [?b :block/title ?title]
 [(clojure.string/lower-case ?title) ?lower]
 [(clojure.string/includes? ?lower "search term")]]
```

**Real example (tested and working):**
```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/tags ?t] [?t :block/title "contact"] [?b :block/title ?title] [(clojure.string/includes? ?title "lin")]]'
```

This finds all blocks:
1. Tagged with "contact"
2. With "lin" somewhere in the title (case-sensitive)

**Result:** 4 blocks (Leyla Erlinda Friedman, Arline Lederman, Laurent Abelin, 陳憶玲 Yiling Chen)

---

### Finding Blocks by Tags

**Single tag:**
```clojure
[:find (pull ?b [*])
 :where
 [?b :block/tags ?t]
 [?t :block/title "tag-name"]]
```

**Multiple tags (AND):**
```clojure
[:find (pull ?b [*])
 :where
 [?b :block/tags ?t1]
 [?t1 :block/title "tag1"]
 [?b :block/tags ?t2]
 [?t2 :block/title "tag2"]]
```

**Multiple tags (OR):**
```clojure
[:find (pull ?b [*])
 :where
 [?b :block/tags ?t]
 (or
   [?t :block/title "tag1"]
   [?t :block/title "tag2"])]
```

---

### Property Filtering

**Blocks with specific property:**
```clojure
[:find (pull ?b [*])
 :where
 [?b :user.property/PropertyName]]
```

**Property with specific value:**
```clojure
[:find (pull ?b [*])
 :where
 [?b :user.property/Status "active"]]
```

**Property namespaces:**
- `:user.property/` - User-defined properties
- `:logseq.property/` - Built-in Logseq properties

**Try both namespaces if unsure:**
```clojure
[:find (pull ?b [*])
 :where
 (or
   [?b :user.property/PropertyName]
   [?b :logseq.property/PropertyName])]
```

---

### Entity Queries

**By UUID:**
```bash
logseq query -g "GRAPH" -p '681eb44c-27ae-4c56-a5e5-109219ad8466'
```

**By db/id:**
```bash
logseq query -g "GRAPH" -p '1108'
```

**Multiple entities:**
```bash
logseq query -g "GRAPH" -p '1108 1109 1110'
```

---

### Advanced Patterns

**Text search with multiple conditions:**
```clojure
[:find (pull ?b [*])
 :where
 [?b :block/title ?title]
 [(clojure.string/lower-case ?title) ?lower]
 [(clojure.string/includes? ?lower "keyword1")]
 [(clojure.string/includes? ?lower "keyword2")]]
```

**Combining tags and properties:**
```clojure
[:find (pull ?b [*])
 :where
 [?b :block/tags ?t]
 [?t :block/title "project"]
 [?b :user.property/Status ?status]
 [(= ?status "active")]]
```

**Date filtering (if using date properties):**
```clojure
[:find (pull ?b [*])
 :where
 [?b :user.property/DueDate ?date]
 [(> ?date 20250101)]]
```

**Using or-join for complex OR conditions:**
```clojure
[:find (pull ?b [*])
 :where
 [?b :block/title ?title]
 (or-join [?b]
   (and [?b :block/tags ?t1] [?t1 :block/title "urgent"])
   (and [?b :user.property/Priority "high"]))]
```

---

### Pull Patterns

**Pull everything:**
```clojure
(pull ?b [*])
```

**Pull specific attributes:**
```clojure
(pull ?b [:block/uuid :block/title :block/created-at])
```

**Pull with nested entities:**
```clojure
(pull ?b [:block/uuid
          :block/title
          {:block/tags [:block/title]}])
```

**Common useful attributes:**
- `:block/uuid` - Block UUID
- `:block/title` - Block title (for pages/blocks)
- `:block/created-at` - Creation timestamp
- `:block/updated-at` - Last update timestamp
- `:block/tags` - Tags applied to block
- `:block/refs` - References in block
- `:user.property/*` - User properties
- `:db/id` - Database ID

---

## Workflows

### Query Development Workflow

**1. Start with simplest query:**
```bash
logseq query -g "GRAPH" -p '[:find (pull ?b [:block/uuid]) :where [?b :block/uuid]]'
```

**2. Add one condition at a time:**
```bash
logseq query -g "GRAPH" -p '[:find (pull ?b [:block/uuid :block/title]) :where [?b :block/title]]'
```

**3. Add filtering:**
```bash
logseq query -g "GRAPH" -p '[:find (pull ?b [:block/uuid :block/title]) :where [?b :block/title ?title] [(clojure.string/includes? ?title "search")]]'
```

**4. Expand pull pattern:**
```bash
logseq query -g "GRAPH" -p '[:find (pull ?b [*]) :where [?b :block/title ?title] [(clojure.string/includes? ?title "search")]]'
```

**5. Verify results before using in code**

---

### Debugging Failed Queries

#### Empty Results `()`

**Possible causes:**

1. **Wrong syntax (missing `-p` flag):**
   - ❌ `logseq query -g "GRAPH" -- 'query'`
   - ✅ `logseq query -g "GRAPH" -p 'query'`

2. **No matching data:**
   - Verify with simpler query
   - Check tag/property names are correct
   - Try case-insensitive search

3. **Wrong graph name:**
   - Run `logseq list` to verify
   - Graph names are case-sensitive

4. **Wrong property namespace:**
   - Try both `:user.property/` and `:logseq.property/`

**Debugging steps:**

```bash
# Step 1: Verify graph exists
logseq list

# Step 2: Test simplest possible query
logseq query -g "GRAPH" -p '[:find ?b :where [?b :block/uuid]]'

# Step 3: Add conditions one at a time
logseq query -g "GRAPH" -p '[:find ?b :where [?b :block/title]]'

# Step 4: Test specific condition
logseq query -g "GRAPH" -p '[:find ?t :where [?b :block/tags ?t] [?t :block/title "contact"]]'
```

---

#### Syntax Errors

**Common mistakes:**

❌ **Using `--` separator:**
```bash
logseq query -g "GRAPH" -- '[:find ...]'  # Returns empty
```

✅ **Correct syntax:**
```bash
logseq query -g "GRAPH" -p '[:find ...]'
```

❌ **Missing quotes around graph name:**
```bash
logseq query -g GRAPH -p '[:find ...]'  # Fails if graph has spaces
```

✅ **Always quote graph names:**
```bash
logseq query -g "GRAPH NAME" -p '[:find ...]'
```

❌ **Wrong quote nesting:**
```bash
logseq query -g "GRAPH" -p "[:find (pull ?b [*]) :where [?b :block/title "test"]]"
```

✅ **Use single quotes for query, double for strings inside:**
```bash
logseq query -g "GRAPH" -p '[:find (pull ?b [*]) :where [?b :block/title "test"]]'
```

---

#### Database Access Errors

**Error: "unable to open database file"**

**Cause:** Sandbox blocking access to `~/Library/Application Support/Logseq/`

**Solution:** Add `dangerouslyDisableSandbox: true` to Bash tool call

```typescript
Bash({
  command: 'logseq query -g "GRAPH" -p \'[:find ...]\'',
  description: "Query Logseq",
  dangerouslyDisableSandbox: true  // ← ADD THIS
})
```

**Important:** The Logseq desktop app does NOT need to be closed. The CLI can access databases regardless of whether the app is running.

---

## Data Model Understanding

### Block Structure in Results

**Example result:**
```clojure
{:block/uuid #uuid "681eb44c-27ae-4c56-a5e5-109219ad8466",
 :block/title "Leyla Erlinda Friedman",
 :block/created-at 1746842700308,
 :block/updated-at 1749344667454,
 :block/tags [{:db/id 137} {:db/id 1076}],
 :block/refs [{:db/id 20} {:db/id 137} ...],
 :user.property/Birthday-CfafO3cr {:db/id 251},
 :user.property/Relations-bsQ-MeUO [{:db/id 1099} {:db/id 1107}],
 :db/id 1108}
```

**Key fields:**
- `:block/uuid` - Globally unique identifier
- `:db/id` - Database-specific ID (changes between graphs)
- `:block/title` - Page/block title
- `:block/name` - Normalized title (lowercase, hyphenated)
- `:block/created-at` / `:updated-at` - Unix timestamps (milliseconds)

---

### Entity References

**Without `-p` flag:**
```clojure
:block/tags [{:db/id 137}]
```

**With `-p` flag (properties-readable):**
```clojure
:block/tags [{:db/id 137}]
```

Note: Even with `-p`, entity references still show `:db/id`. To see full data, you need to:

1. **Expand in pull pattern:**
```clojure
[:find (pull ?b [:block/uuid
                  :block/title
                  {:block/tags [:block/title]}])
 :where ...]
```

Result:
```clojure
:block/tags [{:block/title "contact"} {:block/title "person"}]
```

2. **Or query the entity separately:**
```bash
logseq query -g "GRAPH" -p '137'  # Query by db/id
```

---

### Property Namespaces

**User properties:**
- Format: `:user.property/PropertyName-HASH`
- Example: `:user.property/Birthday-CfafO3cr`
- Created by users in Logseq

**Built-in properties:**
- Format: `:logseq.property/PropertyName`
- Example: `:logseq.property/created-by-ref`
- System properties

**Best practice:** When querying unknown properties, try both:
```clojure
[:find (pull ?b [*])
 :where
 (or
   [?b :user.property/PropertyName ?val]
   [?b :logseq.property/PropertyName ?val])]
```

---

### Query Result Format Quirks (CRITICAL for Parsing)

**IMPORTANT:** Query results have inconsistent key formatting that causes parsing errors if not handled correctly.

#### Property Identifier Prefix Mismatch

**In Queries (Required):** Property identifiers MUST have `:` prefix
```clojure
[:find (pull ?b [:user.property/ProjectStatus-IUJoj7Hs])
 :where [?b :user.property/ProjectStatus-IUJoj7Hs ?val]]
```

**In Results (No Prefix):** Property keys DON'T have `:` prefix
```javascript
{
  "user.property/ProjectStatus-IUJoj7Hs": {"db/id": 1143},
  "block/title": "My Block",
  "db/id": 5424
}
```

**Parsing Strategy:**
```javascript
// WRONG - assumes prefix exists
const value = result['`:user.property/ProjectStatus`'];  // undefined!

// CORRECT - check both formats
const value = result[':user.property/ProjectStatus'] ||
              result['user.property/ProjectStatus'];

// BEST - know your context
const value = result['user.property/ProjectStatus'];  // In results, no ':'
```

#### Result Structure Variations

**Pull Query Results:**

With `:limit 1`:
```javascript
{
  "data": [
    {"user.property/Status": {"db/id": 1143}},
    {"user.property/Status": {"db/id": 1134}},
    // ... multiple results
  ]
}
```

Access: `data[0]['user.property/Status']` NOT `data[0][0]['user.property/Status']`

**Find Query Results:**

Without pull:
```clojure
[:find ?title :where [?b :block/title ?title]]
```

Returns flat arrays:
```javascript
{"data": [["Title 1"], ["Title 2"], ["Title 3"]]}
```

Access: `data[0][0]` (nested array)

With pull:
```clojure
[:find (pull ?b [:block/title]) :where [?b :block/title]]
```

Returns object arrays:
```javascript
{"data": [{"block/title": "Title 1"}, {"block/title": "Title 2"}]}
```

Access: `data[0]['block/title']` (direct property access)

#### Entity Reference Values

**Query returns:**
```javascript
{
  "user.property/ProjectStatus": {
    "db/id": 1143
  }
}
```

**Keys to check for entity refs:**
- `'db/id'` (no colon - most common in results)
- `':db/id'` (with colon - less common but possible)

**Type Detection:**
```javascript
// Check both formats
if (value && typeof value === 'object' &&
    (value['db/id'] || value[':db/id'])) {
  // This is an entity reference
  valueType = ':db.type/ref';
}
```

#### Common Parsing Pitfalls

| Issue | Wrong Code | Correct Code |
|-------|------------|--------------|
| Property key prefix | `result[':user.property/X']` | `result['user.property/X']` |
| Nested array assumption | `data[0][0][key]` | `data[0][key]` |
| Entity ID key | `value[':db/id']` | `value['db/id'] \|\| value[':db/id']` |
| Block title key | `block[':block/title']` | `block['block/title']` |

#### Safe Parsing Function

```javascript
function safeGetProperty(obj, propWithoutColon) {
  // Try without colon first (most common in results)
  if (obj[propWithoutColon] !== undefined) {
    return obj[propWithoutColon];
  }
  // Try with colon (rare but possible)
  if (obj[':' + propWithoutColon] !== undefined) {
    return obj[':' + propWithoutColon];
  }
  return undefined;
}

// Usage
const status = safeGetProperty(result, 'user.property/ProjectStatus');
const dbId = safeGetProperty(value, 'db/id');
const title = safeGetProperty(block, 'block/title');
```

#### Real-World Example (Query Builder)

**Problem:** Property dropdown not appearing
**Root Causes Found:**
1. Query used `:user.property/X` (correct)
2. Code tried to access `result[':user.property/X']` (wrong - extra `:`)
3. Code assumed `data[0][0]` nesting (wrong - flat object array)
4. Type check looked for `':db/id'` (wrong - no `:` in result keys)

**Solution:**
```javascript
// Build query WITH colon prefix
const queryIdent = propertyIdent.startsWith(':') ?
                   propertyIdent : `:${propertyIdent}`;
const query = `[:find (pull ?b [${queryIdent}]) ...]`;

// Parse result WITHOUT colon prefix
const sampleValue = result.data[0][propertyIdent];  // No ':'!

// Check entity ref WITHOUT assuming colon
if (sampleValue['db/id'] || sampleValue[':db/id']) {
  // It's a reference property
}
```

---

### Timestamps

**Format:** Unix timestamp in milliseconds

**Example:**
- `:block/created-at 1746842700308`
- Human-readable: 2025-05-09 (approximately)

**Filtering by date:**
```clojure
[:find (pull ?b [*])
 :where
 [?b :block/created-at ?date]
 [(> ?date 1735689600000)]]  ; After 2025-01-01
```

**Converting timestamps:**
```bash
# In fish shell
date -r (math "1746842700308 / 1000")
```

---

## Technical Implementation Details

Based on the `@logseq/cli` source code (v0.4.2):

### Architecture

**Language:** ClojureScript compiled with [nbb-logseq](https://github.com/logseq/nbb-logseq)
**Database:** DataScript for queries (Clojure implementation of Datomic-like database)
**Entry Point:** `cli.mjs` → loads `src/logseq/cli.cljs`

### Query Processing

**Datalog Queries:**
- Detected by checking if query starts with `[` and contains `:find`
- Rules from `logseq.db.frontend.rules` are automatically injected
- Query structure: `[:find ... :where ...] + [:in $ '%]` (rules added automatically)
- Results are automatically unwrapped if only one `:find` binding

**Entity Queries:**
- Accept multiple formats:
  - UUID string: `"681eb44c-27ae-4c56-a5e5-109219ad8466"`
  - Integer db/id: `1108`
  - EDN `:db/ident` keyword: `:logseq.class/Tag`
- Resolved via `datascript.core/entity`
- Multiple entities can be queried: `logseq query 1108 1109 1110 -g "GRAPH"`

**Properties-Readable Flag (`-p`):**
```clojure
;; Implementation from query.cljs:32-48
(defn- readable-properties [ent]
  (->> (db-property/properties ent)
       (mapv (fn [[k v]]
               [k (cond
                    ;; Special handling for tags/classes
                    (#{:block/tags :logseq.property.class/extends ...} k)
                    (mapv :db/ident v)

                    ;; Set of entities → extract content
                    (and (set? v) (every? entity? v))
                    (set (map property-value-content v))

                    ;; Single entity → get ident or content
                    (entity? v)
                    (or (:db/ident v) (property-value-content v))

                    ;; Otherwise return as-is
                    :else v)]))
       (into {})))
```

This explains why even with `-p`:
- Tags still show as `{:db/id 137}` but with `:db/ident` when expanded in pull
- Entity references need explicit pull pattern expansion to see full data

### Built-in Rules

Queries automatically include DataScript rules:
- `block-content` - Used for text search in `:block/title`
- Other rules from `logseq.db.frontend.rules/db-query-dsl-rules`

Example using built-in rule:
```clojure
[:find (pull ?b [*])
 :in $ % ?search-term
 :where (block-content ?b ?search-term)]
```

### Database Access

**Local graphs:** SQLite files at:
- macOS: `~/Library/Application Support/Logseq/graphs/`
- Linux: `~/.config/Logseq/graphs/`
- Windows: `%APPDATA%/Logseq/graphs/`

**Connection:** Uses `logseq.db.common.sqlite-cli/open-db!`

### API Server Mode

**When using `-a` flag:**
- Connects to Logseq desktop app's HTTP API server (default: http://localhost:12315)
- Methods:
  - `logseq.db.datascript_query` - For datalog queries
  - `logseq.db.q` - For simple queries
- Requires HTTP API server enabled in Logseq app

### MCP Server

The CLI includes an MCP (Model Context Protocol) server:
```bash
logseq mcp-server -g "GRAPH"  # HTTP streamable server on 127.0.0.1:12315
logseq mcp-server -g "GRAPH" --stdio  # Stdio transport
```

**Purpose:** Integration with Claude and other AI assistants
**Modes:**
- HTTP Streamable (default)
- stdio transport (for direct integration)

---

## Error Handling

### Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| Empty results `()` | Missing `-p` flag | Add `-p` to command |
| Empty results `()` | Wrong graph name | Run `logseq list` to verify |
| "unable to open database file" | Sandbox blocking access | Add `dangerouslyDisableSandbox: true` |
| Syntax error | Wrong quote nesting | Use single quotes for query |
| "graph not found" | Typo in graph name | Copy exact name from `logseq list` |
| No results for property | Wrong namespace | Try both `:user.property/` and `:logseq.property/` |
| "Cannot compare :block/refs to user.property/X" | Missing `:` prefix in query | Add `:` before property ident in query |
| `undefined` when accessing result property | Used `:` prefix in result access | Remove `:` when accessing result keys |
| "Cannot read properties of undefined" | Wrong array nesting assumption | Use `data[0][key]` not `data[0][0][key]` for pull queries |
| Property type incorrectly detected as string | Checked for `':db/id'` with colon | Check both `'db/id'` and `':db/id'` formats |

---

### Validation Checklist

Before executing a query in code:

- [ ] Tested query with CLI first using `logseq query`
- [ ] Verified graph name with `logseq list`
- [ ] Used `-p` flag for readable output
- [ ] Added `dangerouslyDisableSandbox: true` in Claude Code
- [ ] Quoted graph name and query correctly
- [ ] Verified results are non-empty

---

## Quick Reference

### Common Commands Cheatsheet

```bash
# List graphs
logseq list

# Simple search
logseq search "term" -g "GRAPH"

# Basic query
logseq query -g "GRAPH" -p '[:find (pull ?b [*]) :where [?b :block/title]]'

# Query by tag
logseq query -g "GRAPH" -p '[:find (pull ?b [*]) :where [?b :block/tags ?t] [?t :block/title "TAG"]]'

# Query by UUID
logseq query -g "GRAPH" -p 'UUID-HERE'

# Export graph
logseq export -g "GRAPH" -f "output.md"

# Show graph info
logseq show "GRAPH"
```

---

### Claude Code Bash Tool Template

```typescript
Bash({
  command: 'logseq query -g "GRAPH NAME" -p \'QUERY HERE\'',
  description: "Describe what this query does",
  dangerouslyDisableSandbox: true
})
```

**Copy-paste ready example:**

```typescript
Bash({
  command: 'logseq query -g "LSEQ 2025-12-15" -p \'[:find (pull ?b [:block/uuid :block/title]) :where [?b :block/title]]\'',
  description: "Query all blocks with titles",
  dangerouslyDisableSandbox: true
})
```

---

### Most Common Query Patterns

**1. Find all pages/blocks:**
```clojure
[:find (pull ?b [*]) :where [?b :block/title]]
```

**2. Find by tag:**
```clojure
[:find (pull ?b [*]) :where [?b :block/tags ?t] [?t :block/title "TAG"]]
```

**3. Text search (case-insensitive):**
```clojure
[:find (pull ?b [*])
 :where
 [?b :block/title ?title]
 [(clojure.string/lower-case ?title) ?lower]
 [(clojure.string/includes? ?lower "search")]]
```

**4. Property filter:**
```clojure
[:find (pull ?b [*]) :where [?b :user.property/Status "active"]]
```

**5. Recent blocks:**
```clojure
[:find (pull ?b [*])
 :where
 [?b :block/created-at ?date]
 [(> ?date 1735689600000)]]
```

---

## Important Notes

1. **Sandbox requirement:** ALWAYS use `dangerouslyDisableSandbox: true` in Claude Code
2. **Critical syntax:** Use `-p` flag, NOT `--` separator
3. **Graph names:** Case-sensitive, use exact names from `logseq list`
4. **App status:** CLI works whether or not Logseq app is running
5. **Read-only:** Most CLI operations are read-only (except import/append)
6. **Fish shell:** Default shell, works identically to bash for these commands
7. **Testing first:** ALWAYS test queries with CLI before implementing in code
8. **Entity refs:** Use `-p` flag and expanded pull patterns to see referenced data
9. **Property namespaces:** Try both `:user.property/` and `:logseq.property/` if unsure
10. **Timestamps:** In milliseconds, use `(> ?date TIMESTAMP)` for filtering

---

## Integration with Other Skills

**Use with `logseq-db-knowledge`:**
- Understanding DB graph structure
- Property types and inheritance
- Block vs page differences

**Use with `logseq-db-plugin-api-skill`:**
- Testing queries before implementing in plugins
- Validating datalog syntax
- Understanding plugin query API

**Workflow example:**
1. Use this skill to explore data and test queries
2. Use `logseq-db-knowledge` to understand data model
3. Use `logseq-db-plugin-api-skill` to implement in plugin code

---

## Package Information

**NPM Package:** `@logseq/cli`
**Current Version:** 0.4.2
**License:** MIT
**Node Requirement:** >=22.17.0

**Source Code:**
- Repository: https://github.com/logseq/logseq
- Location: `deps/cli` directory
- Language: ClojureScript (compiled with nbb-logseq)

**Installation:**
```bash
npm install -g @logseq/cli
```

**Local Installation:**
Already installed at `/opt/homebrew/lib/node_modules/@logseq/cli/` (via Homebrew/npm global)

**Key Dependencies:**
- `@logseq/nbb-logseq` - Node.js Babashka for ClojureScript
- `better-sqlite3` - SQLite3 bindings
- `@modelcontextprotocol/sdk` - MCP server support
- `datascript` - Datalog database (transitive dependency)

**Development:**
See README at: `/opt/homebrew/lib/node_modules/@logseq/cli/README.md`
