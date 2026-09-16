---
name: using-search-tools
description: >-
  Execution skill. Choosing between the discovery tools and calling them cheaply: why
  breadth is free but call count is not, and lineage versus full impact analysis. Load
  before exploring an application, resolving block names, or checking what a change breaks.
metadata:
  skill_path: /skills/using-search-tools/SKILL.md
  base_directory: /skills/using-search-tools
---

# Using Search and Dependency Tools

Search tools let you discover blocks; dependency tools let you trace formula lineage and impact. Using the wrong one wastes tokens and returns poor results.

---

## What a Search Call Costs

Every search call pays a **fixed cost**: the tool loads the application catalog before it filters. That cost does not depend on how broad your query is, how many patterns you pass, or how many results come back. A call that returns zero results costs the same as one that returns a hundred. On a large application this is several seconds per call.

Two consequences drive everything below:

1. **Breadth is free, call count is not.** One call carrying fifteen name patterns costs the same as one call carrying one. Never spread names across calls.
2. **Turns are the real cost.** Parallel calls in the same turn overlap, so their wall time is the slowest one, not the sum. Sequential calls across turns each add a full model round trip on top of the search.

So: name everything you need up front, put it in as few calls as possible, and fire whatever remains in a **single turn**.

---



## Tool Selection


| You want to…                                                                                                 | Use                                                                                                | Cost                                                 |
| ------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| Find blocks by exact name, kind, folder, formula content, data type, or UUID                                 | `tool:search_metrics_and_lists`, `tool:search_folders`, `tool:search_tables`                                          | Low per call;                                        |
| List all blocks (paged catalog) or apply combined structural filters                                         | `tool:search_metrics_and_lists`, `tool:search_folders`, `tool:search_tables` (empty request, increment `page_number`) | Low per call                                         |
| Look up a block you already have the UUID for                                                                | `tool:search_metrics_and_lists`, `tool:search_folders`, `tool:search_tables`                                          | Low per call                                         |
| Find blocks by concept/topic when you do not know the exact name                                             | `tool:semantic_search`                                                                             | Medium per call                                      |
| Trace multi-hop formula lineage (what feeds a metric, or what consumes it)                                   | `tool:get_data_dependency_tree`                                                                    | Low per call— recursive graph traversal; no AI       |
| Full impact analysis before deleting/renaming a metric (all consumers: formulas, views, boards, automations) | `tool:get_metric_dependencies`                                                                     | Low per call— flat inventory lookup; no AI           |
| Other approaches failed, or no other tool fits                                                               | `tool:search`                                                                                      | Highest — LLM expert reads blocks and produces prose |


**Rule of thumb**: Always prefer `tool:search_metrics_and_lists`, `tool:search_folders`, or `tool:search_tables` when you can express the query as a concrete filter. Fall back to `tool:semantic_search` when you only have a concept.

**Parallelization**: all search and dependency tool calls are independent and can be batched in a single turn. When you need multiple pieces of information (e.g. list dimensions + list metrics + check dependencies), issue them all at once instead of sequentially.

## Common Workflows



### Discover an application's structure

Do this as one wide sweep, then work from what came back. Do not scan folder by folder.

1. In a single turn, issue `tool:search_folders` and `tool:search_metrics_and_lists` with `kind: ["Dimension"]`
2. Read the returned catalog. It already tells you the folder layout, the dimension names and where they live
3. `tool:search_metrics_and_lists` with `kind: ["Metric"]` and `parent_folder_regex_search` covering the areas you care about → scan metrics by area
4. `tool:semantic_search` or `tool:get_data_dependency_tree` for targeted questions about computation logic



### Resolve a known list of blocks

This is the common case once you know what you are looking for.

1. Write down every block name you need, for the whole task
2. **One** `tool:search_metrics_and_lists` with all of them in `friendly_name_regex_search`, anchored with `^` and `$`, plus `kind` and `show_details: true`
3. Read the formulas from the details column. You now have the IDs and the logic; do not search again



### Find the right metric for a vague user request

1. `tool:semantic_search` with topic phrases from the user's question + `kind: ["Metric"]`
2. If no good match, broaden topics or try `tool:search_metrics_and_lists` with a name regex



### Find the right metric when you know its name (or part of it)

1. `tool:search_metrics_and_lists` with multiple regex search terms + `kind: ["Metric"]`
2. If no good match, try a shorter substring, or fall back to `tool:semantic_search`



### Understand a metric before editing it

1. `tool:search_metrics_and_lists` with `block_id_exact_match` + `show_details: true` → get formula and dimensions
2. `tool:get_data_dependency_tree` direction Sources → see what feeds it
3. `tool:get_metric_dependencies` → see what depends on it (impact analysis)



### Inventory a folder subtree before moving blocks

1. `tool:search_folders` → build the tree from the catalog (paginate if needed)
2. Resolve the source root by name; disambiguate duplicates with full path
3. Keep blocks whose folder path is under that root (from catalog paths, not `parent_folder_regex_search` alone)
4. Freeze folder tree + block IDs in the spec before `tool:create_folder` or `tool:move_blocks`



### Find unused blocks

1. `tool:search_metrics_and_lists` with `kind: ["Metric"]` → page through all metrics (increment `page_number`)
2. For each candidate, call `tool:get_metric_dependencies` → if no usages (no formula references, no views, no boards, no automations), the metric is unused
3. Batch dependency checks in parallel (one call per metric, all in a single turn)
4. Cross-check with `tool:get_data_dependency_tree` direction Usages → confirm no downstream formula consumers
5. Flag metrics with zero references as candidates for deletion; present to user for confirmation before removing

---



## Query Formulation



### tool:search

- One short, focused sentence per call; ask about one thing at a time
- Good: `"how are revenues computed?"` / `"what feeds into 2026 ARR?"`
- Bad: `"tell me everything about the revenue model, its inputs, outputs, formulas, and boards"` (too broad)
- Bad: `"ARR"` (too vague; needs a question)
- Bad: `"list all metrics related to churn"` (that is a `tool:search_metrics_and_lists` job)



### tool:semantic_search

- Short phrases of 2–4 words each; one concept per array element; multiple entries are OR-combined
- Always add `kind` when you know the block type
- Good: `topics: ["gross profit margin"], kind: ["Metric"]`
- Good: `topics: ["employee cost", "salary expense"], kind: ["Metric"]`
- Bad: `topics: ["the metric that computes gross profit margin by subtracting COGS from revenue"]` (too long)
- Bad: `topics: ["region geography country city continent"]` (split into separate entries)

---



## Dependency Tools



### tool:get_data_dependency_tree

Recursive traversal of formula-based dependencies. Returns a tree rooted at `start_block_id`.

- **Direction** `Sources`: walk toward blocks the metric references in its formula (what feeds it)
- **Direction** `Usages`: walk toward blocks that reference the metric in their formulas (what consumes it)
- Default depth is 5 hops; set `max_traversal_depth` to limit
- Use `end_block_id` to find the path between two specific blocks (ignores depth limit)
- Only metrics and transaction lists appear; dimensions are excluded

**When to use**: understanding the full calculation chain before editing a formula, verifying upstream inputs, tracing how data flows through the model.

### tool:get_metric_dependencies

Flat inventory of everywhere a metric is referenced. Grouped by consumer kind: formulas, list properties, tables, boards, views, automations, variables, ARM metrics.

- Does NOT walk multi-hop chains (use `tool:get_data_dependency_tree` for that)
- Shows ALL reference types (not just formula dependencies): boards, views, imports, automations

**When to use**: impact analysis before deleting, renaming, or structurally changing a metric. Answers "what will break if I touch this?"

### Choosing between them


| Question                                                          | Tool                                           |
| ----------------------------------------------------------------- | ---------------------------------------------- |
| What feeds this metric? (upstream chain)                          | `tool:get_data_dependency_tree` direction `Sources` |
| What consumes this metric through formulas? (downstream chain)    | `tool:get_data_dependency_tree` direction `Usages`  |
| Will anything break if I delete/rename this metric? (full impact) | `tool:get_metric_dependencies`                      |
| Is there a formula path between metric A and metric B?            | `tool:get_data_dependency_tree` with `end_block_id` |


---



## Anti-Patterns



### These cost the user seconds

1. **Splitting names across calls or turns** — the worst of the lot. Four consecutive turns searching `^Revenue$`, then `^Gross Margin$`, then `^Headcount$`, then `^Total Cost$` cost four searches and four model round trips, where one call with four patterns would have done. If two of your searches differ only by their regex list, they were one call.
2. **Scanning folder by folder** — one call per area multiplies the fixed cost. Pass every folder pattern in one `parent_folder_regex_search` list, or sweep the whole catalog once.
3. **Searching sequentially when the queries are independent** — if a search does not depend on the result of another, they belong in the same turn.



### Wrong tool for the job

- **Using** `tool:search` **to get a block list** — it returns prose reasoning, not structured lists.
- **Using** `tool:search` **to trace dependencies** — use `tool:get_data_dependency_tree` for precise multi-hop chains.
- **Using** `tool:get_data_dependency_tree` **for impact analysis** — it only shows formula references; use `tool:get_metric_dependencies` to see boards, views and automations too.



### Malformed queries

- **Using** `tool:semantic_search` **with full sentences** — the embedding model works best on 2–4 word phrases.
- **Bundling multiple questions** — one question per `tool:search` call, one concept per `tool:semantic_search` topic entry.
- **Omitting** `kind` when you clearly know the block type — wastes ranking capacity on irrelevant types.



### Wasted context, and risk

- `show_details=true` **on broad scans** — it does not slow the call down, but it shrinks the page from 100 blocks to 25 and floods the context. Scan compact first, then detail the blocks you care about.
- **Re-searching blocks you just created, or blocks already in context** — trust the tool responses you already have.
- **Skipping impact analysis before deletion** — always call `tool:get_metric_dependencies` before deleting a metric.
- **Scoping a subtree move by parent folder name or semantic search** — `parent_folder_regex_search` matches immediate parent only; name or topic is not folder membership. Inventory by full path first.

