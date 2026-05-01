# Common Logseq CLI Query Patterns

A collection of copy-paste ready query patterns for common use cases.

## Setup

All examples assume:
- Graph name: `"LSEQ 2025-12-15"` (replace with your graph)
- Using `-p` flag for readable output
- Running in Claude Code (no `dangerouslyDisableSandbox` needed if your graph path is in `sandbox.filesystem.allowWrite` in `~/.claude/settings.json`)

## Basic Queries

### Get All Pages

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/title]]'
```

### Get All Pages (UUIDs and Titles Only)

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [:block/uuid :block/title]) :where [?b :block/title]]'
```

### Get Single Block by UUID

```bash
logseq query -g "LSEQ 2025-12-15" -p 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
```

### Get Multiple Blocks by UUID

```bash
logseq query -g "LSEQ 2025-12-15" -p 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy'
```

---

## Tag-Based Queries

### Find All Blocks with Specific Tag

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/tags ?t] [?t :block/title "contact"]]'
```

### Find Blocks with Multiple Tags (AND)

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/tags ?t1] [?t1 :block/title "project"] [?b :block/tags ?t2] [?t2 :block/title "active"]]'
```

### Find Blocks with Any of Multiple Tags (OR)

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/tags ?t] (or [?t :block/title "urgent"] [?t :block/title "important"])]'
```

### Find Blocks Tagged and Get Tag Names

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [:block/uuid :block/title {:block/tags [:block/title]}]) :where [?b :block/tags ?t] [?t :block/title "contact"]]'
```

---

## Text Search Queries

### Case-Insensitive Substring Search

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/title ?title] [(clojure.string/lower-case ?title) ?lower] [(clojure.string/includes? ?lower "search term")]]'
```

### Exact Title Match

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/title "Exact Title Here"]]'
```

### Title Starts With

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/title ?title] [(clojure.string/starts-with? ?title "Prefix")]]'
```

### Title Contains Multiple Keywords

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/title ?title] [(clojure.string/lower-case ?title) ?lower] [(clojure.string/includes? ?lower "keyword1")] [(clojure.string/includes? ?lower "keyword2")]]'
```

### Combined Tag and Text Search (Real Example)

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/tags ?t] [?t :block/title "contact"] [?b :block/title ?title] [(clojure.string/includes? ?title "lin")]]'
```

This finds blocks:
- Tagged with "contact"
- AND containing "lin" in the title

---

## Property-Based Queries

### Find Blocks with Specific Property

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :user.property/Status]]'
```

### Find Blocks Where Property Equals Value

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :user.property/Status "active"]]'
```

### Find Blocks with Property (Try Both Namespaces)

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where (or [?b :user.property/PropertyName] [?b :logseq.property/PropertyName])]'
```

### Get Property Values

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find ?b ?val :where [?b :user.property/Status ?val]]'
```

### Filter by Numeric Property

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :user.property/Priority ?p] [(> ?p 5)]]'
```

---

## Property Schema Discovery

Properties in DB graphs have types. **Reference properties** (`:db.type/ref`) store entity IDs, not strings — querying them with a plain string match returns nothing. Discover a property's type before querying it.

### Get Schema for a Specific Property

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?p [:db/ident :db/valueType :db/cardinality :block/title]) :where (or [?p :db/ident :user.property/status] [?p :db/ident :logseq.property/status])]'
```

Key fields returned:
- `db/valueType` — `:db.type/string`, `:db.type/ref`, `:db.type/number`, `:db.type/boolean`, `:db.type/instant`
- `db/cardinality` — `:db.cardinality/one` (single value) or `:db.cardinality/many` (multiple values)

### Discover All User Property Schemas

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?p [:db/ident :db/valueType :db/cardinality :block/title]) :where [?p :db/ident ?ident] [(namespace ?ident) ?ns] [(= ?ns "user.property")]]'
```

### Get All Possible Values for a Reference Property

For `:db.type/ref` properties (e.g. status, priority), fetch the actual options:

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?val [:block/title]) :where [_ :logseq.property/status ?val]]'
```

### Get a Tag's Associated Properties

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?tag [:logseq.property.class/properties]) :where [?tag :block/title "Task"]]'
```

---

## Type-Aware Property Queries

### Reference Property (`:db.type/ref`) — Entity Join Required

**Wrong** — returns nothing because status values are entities, not strings:
```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :logseq.property/status "Todo"]]'
```

**Correct** — join through the entity to get its title:
```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :logseq.property/status ?s] [?s :block/title "Todo"]]'
```

Multi-cardinality reference (OR across multiple values):
```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :logseq.property/priority ?p] (or [?p :block/title "High"] [?p :block/title "Urgent"])]'
```

Try both namespaces when the property might be user-defined or built-in:
```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where (or-join [?b] (and [?b :user.property/status ?s] [?s :block/title "Done"]) (and [?b :logseq.property/status ?s] [?s :block/title "Done"]))]'
```

### Number Property (`:db.type/number`) — Use Comparison Operators

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :user.property/year ?n] [(> ?n 2020)]]'
```

Bind result first when combining comparisons (avoids predicate nesting errors):
```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :user.property/year ?n] [(>= ?n 2020)] [(<= ?n 2025)]]'
```

### Boolean Property (`:db.type/boolean`) — Use `true`/`false` Literals

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :user.property/published true]]'
```

### Property Type Reference Table

| `:db/valueType` | Example usage | Query pattern |
|---|---|---|
| `:db.type/string` | `"Project Title"` | `[?b :prop "value"]` |
| `:db.type/ref` | Status, Priority (entity) | `[?b :prop ?ref] [?ref :block/title "value"]` |
| `:db.type/number` | Year, count | `[?b :prop ?n] [(> ?n 2020)]` |
| `:db.type/boolean` | Published, done | `[?b :prop true]` |
| `:db.type/instant` | Timestamps (ms) | `[?b :prop ?d] [(> ?d 1735689600000)]` |

---

## Tag Class Inheritance

Find blocks tagged directly OR tagged with any subclass that extends a parent class. Useful for task systems where `#Bug`, `#Feature`, etc. extend `#Task`.

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where (or-join [?b] (and [?b :block/tags ?t] [?t :block/title "Task"]) (and [?b :block/tags ?child] [?child :logseq.property.class/extends ?parent] [?parent :block/title "Task"]))]'
```

Combined with status filter (tested pattern from logseq task viewer):
```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [:block/uuid :block/title {:logseq.property/status [:block/title]} {:logseq.property/priority [:block/title]}]) :where (or-join [?b] (and [?b :block/tags ?t] [?t :block/title "Task"]) (and [?b :block/tags ?child] [?child :logseq.property.class/extends ?parent] [?parent :block/title "Task"])) [?b :logseq.property/status ?s] [?s :block/title "Todo"]]'
```

Note: Nested pull specs `{:attr [:block/title]}` resolve ref attributes to their titles inline, avoiding `{:db/id N}` in results.

---

## Date/Time Queries

### Find Blocks Created After Date

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/created-at ?date] [(> ?date 1735689600000)]]'
```

*Note: 1735689600000 = 2025-01-01 00:00:00 UTC*

### Find Blocks Created in Date Range

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/created-at ?date] [(> ?date 1735689600000)] [(< ?date 1738368000000)]]'
```

### Find Recently Updated Blocks

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [:block/uuid :block/title :block/updated-at]) :where [?b :block/updated-at ?date] [(> ?date 1749340000000)]] | head -20'
```

### Sort by Creation Date (Most Recent)

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [:block/uuid :block/title :block/created-at]) :where [?b :block/created-at]]' | jq 'sort_by(.["block/created-at"]) | reverse | .[0:10]'
```

---

## Combined Queries

### Tag + Property Filter

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/tags ?t] [?t :block/title "project"] [?b :user.property/Status "active"]]'
```

### Tag + Text Search + Date Filter

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/tags ?t] [?t :block/title "meeting"] [?b :block/title ?title] [(clojure.string/includes? ?title "review")] [?b :block/created-at ?date] [(> ?date 1735689600000)]]'
```

### Complex OR with Multiple Conditions

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/title] (or-join [?b] (and [?b :block/tags ?t] [?t :block/title "urgent"]) (and [?b :user.property/Priority "high"]))]'
```

---

## Reference and Relationship Queries

### Find Blocks That Reference Specific Page

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :where [?b :block/refs ?ref] [?ref :block/title "Referenced Page"]]'
```

### Get Block with All References Expanded

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [:block/uuid :block/title {:block/refs [:block/title]} {:block/tags [:block/title]}]) :where [?b :block/title "Page Name"]]'
```

### Find Blocks with Specific Relationship Property

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [:block/uuid :block/title {:user.property/Relations-bsQ-MeUO [:block/title]}]) :where [?b :user.property/Relations-bsQ-MeUO]]'
```

---

## Aggregation and Counting

### Count Blocks with Tag

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (count ?b) :where [?b :block/tags ?t] [?t :block/title "contact"]]'
```

### Count All Pages

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (count ?b) :where [?b :block/title]]'
```

### List All Unique Tag Names

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find ?tag-title :where [?b :block/tags ?t] [?t :block/title ?tag-title]]'
```

### Count Blocks Per Tag

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find ?tag-title (count ?b) :where [?b :block/tags ?t] [?t :block/title ?tag-title]]'
```

---

## Advanced Pull Patterns

### Pull Specific Fields Only

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [:block/uuid :block/title :block/created-at :block/updated-at]) :where [?b :block/title]]'
```

### Pull with Nested Entity Expansion

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [:block/uuid :block/title {:block/tags [:block/title :block/uuid]} {:block/refs [:block/title]}]) :where [?b :block/tags]]'
```

### Pull All User Properties

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [:block/uuid :block/title :user.property/*]) :where [?b :block/title]]'
```

---

## Debugging Queries

### Test if Graph is Accessible

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find ?b :where [?b :block/uuid]]' | head -5
```

### Check if Specific Tag Exists

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find ?t :where [?t :block/title "contact"]]'
```

### Find Blocks with Any Tag

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [:block/uuid :block/title]) :where [?b :block/tags]]' | head -20
```

### List All Property Names in Graph

```bash
logseq query -g "LSEQ 2025-12-15" -p '[:find ?prop :where [?b ?prop] [(namespace ?prop) ?ns] [(= ?ns "user.property")]]'
```

---

## Claude Code Integration Examples

### Standard Query Pattern

```typescript
Bash({
  command: 'logseq query -g "LSEQ 2025-12-15" -p \'[:find (pull ?b [*]) :where [?b :block/tags ?t] [?t :block/title "contact"]]\'',
  description: "Find all contacts in Logseq",
})
```

### Search with Result Limit

```typescript
Bash({
  command: 'logseq search "search term" -g "LSEQ 2025-12-15" -l 10',
  description: "Search for 'search term' (limit 10 results)",
})
```

### Query with Piped Output Processing

```typescript
Bash({
  command: 'logseq query -g "LSEQ 2025-12-15" -p \'[:find (pull ?b [:block/uuid :block/title]) :where [?b :block/title]]\' | head -20',
  description: "Get first 20 pages from Logseq",
})
```

---

## Using Built-in Rules

The Logseq CLI automatically injects DataScript rules from `logseq.db.frontend.rules`. You can use these in your queries:

### block-content Rule

```bash
# Search using built-in block-content rule
logseq query -g "LSEQ 2025-12-15" -p '[:find (pull ?b [*]) :in $ % ?search-term :where (block-content ?b ?search-term)]' "search term"
```

**Note:** This is similar to the `-t` (title-query) flag but gives more control.

### Custom Queries with Rules

Rules are automatically added as the second `:in` parameter:
```clojure
[:find (pull ?b [*])
 :where [?b :block/title ?t]]

# Becomes internally:
[:find (pull ?b [*])
 :in $ '%              ; Rules injected here
 :where [?b :block/title ?t]]
```

This means you can reference any rule defined in `logseq.db.frontend.rules/db-query-dsl-rules`.

---

## Tips and Tricks

### Use jq for JSON Processing

```bash
# Pretty print results
logseq query -g "GRAPH" -p 'QUERY' | jq '.'

# Extract just titles
logseq query -g "GRAPH" -p 'QUERY' | jq '.[] | .["block/title"]'

# Filter results
logseq query -g "GRAPH" -p 'QUERY' | jq '.[] | select(.["block/title"] | contains("keyword"))'

# Count results
logseq query -g "GRAPH" -p 'QUERY' | jq 'length'
```

### Testing Query Incrementally

```bash
# Start simple
logseq query -g "GRAPH" -p '[:find ?b :where [?b :block/uuid]]'

# Add title
logseq query -g "GRAPH" -p '[:find ?b :where [?b :block/title]]'

# Add pull pattern
logseq query -g "GRAPH" -p '[:find (pull ?b [:block/title]) :where [?b :block/title]]'

# Add condition
logseq query -g "GRAPH" -p '[:find (pull ?b [:block/title]) :where [?b :block/title ?t] [(clojure.string/includes? ?t "search")]]'
```

### Saving Queries for Reuse

```bash
# Save to variable in fish shell
set contact_query '[:find (pull ?b [*]) :where [?b :block/tags ?t] [?t :block/title "contact"]]'

# Use the variable
logseq query -g "LSEQ 2025-12-15" -p $contact_query
```

---

## Date Timestamp Conversion

### Common Timestamps

```
2025-01-01 00:00:00 UTC = 1735689600000
2025-06-01 00:00:00 UTC = 1748736000000
2025-12-01 00:00:00 UTC = 1764566400000
```

### Convert Timestamp to Date (Fish Shell)

```fish
date -r (math "1746842700308 / 1000")
```

### Get Current Timestamp for Queries

```fish
set current_time (math (date +%s) "*" 1000)
logseq query -g "GRAPH" -p "[:find (pull ?b [*]) :where [?b :block/created-at ?date] [(> ?date $current_time)]]"
```

---

## Common Mistakes to Avoid

❌ **Using `--` instead of `-p`:**
```bash
logseq query -g "GRAPH" -- 'QUERY'  # Returns ()
```

❌ **Forgetting to quote graph name:**
```bash
logseq query -g LSEQ 2025-12-15  # Fails
```

❌ **Wrong quote nesting:**
```bash
logseq query -g "GRAPH" -p "[:find (pull ?b [*]) :where [?b :block/title "test"]]"  # Syntax error
```

✅ **Correct patterns are shown in examples above**

> **Sandbox note:** `dangerouslyDisableSandbox` is not needed if your graph path is in `sandbox.filesystem.allowWrite` in `~/.claude/settings.json`.
