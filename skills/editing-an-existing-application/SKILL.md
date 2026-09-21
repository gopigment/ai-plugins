---
name: editing-an-existing-application
description: "Planning skill. Use when modifying, extending, refactoring, or restructuring an existing Pigment application. Covers discovery, impact analysis, change planning, and phased execution. Do not use for greenfield builds (use building-a-full-application instead)."
metadata:
  skill_path: /skills/editing-an-existing-application/SKILL.md
  base_directory: /skills/editing-an-existing-application
---

# Editing an Existing Application

When modifying an existing Pigment application (adding blocks, changing formulas, restructuring dimensions, reorganizing folders, deleting unused objects, or any combination), follow the two-stage workflow below.

---

## Stage 1 — Plan

During Stage 1, do NOT call any modeling tool and do NOT load any execution skill. Loading another planning skill is allowed. Use only discovery tools (`tool:search_metrics_and_lists`, `tool:search_folders`, `tool:search_tables`, `tool:semantic_search`, `tool:get_metric_dependencies`, `tool:get_data_dependency_tree`, `tool:list_issues`, `tool:calendar_get`, `tool:get_applications`, `tool:search_boards`) and the sections below to produce the spec.

### Discover the current state

Map the relevant portion of the application before deciding anything. Do not rely on assumptions.

- **Inventory** — find the blocks related to the change with `tool:search_metrics_and_lists`, `tool:search_folders`, or `tool:search_tables`, narrowing by block type; `tool:semantic_search` when names are vague; `tool:get_applications` for exploration across applications.
- **Upstream sources** — trace every block you will touch with `tool:get_data_dependency_tree` using `direction: "Sources"`. This is unconditional: deciding that no formula change is needed is itself a conclusion about upstream data, so it must be checked first.
- **Downstream dependents** — `tool:get_metric_dependencies` returns **referrers only** (`referringMetrics`, `referringLists`, `referringTables`, `referringBoards`, `referringViews`, `hasStructureDependency`). It tells you nothing about upstream sources, so an empty result never means "no dependencies", only that nothing points at this block. Blocks with many dependents need extra care.
- **Formula health** — call `tool:list_issues` and record the pre-existing errors, so you can tell inherited errors from ones you introduce.
- **Conventions in use** — naming (prefixes like INPUT_, CALC_, DATA_; PascalCase dimensions; numbered folders), folder structure, Version dimension and its items, calendar (`tool:calendar_get`), plus libraries (`tool:list_application_libraries`) and scenarios (`tool:list_scenarios`) when relevant. Adopt what the application already does; the structural invariants from `skill:understanding-pigment-modeling` still apply to new blocks. Do not impose standard conventions on a live application unless the user asks for cleanup.
- **Domain skill** — a planning skill exists for each domain: OPEX and budgeting, workforce and hiring, sales and pipeline, supply chain and inventory. If the application matches one, load it from the skills library headers before Stage 2, and read the sub-files it points to when the pattern needs them (e.g. the engine sub-file for OPEX method selection).

### Assess impact and risk

Classify the change so the spec can call out what deserves extra caution and explicit user sign-off:

- **Destructive or high-risk structural** — deleting a list, metric, table, or property (especially one holding user inputs); deleting or removing items (modalities/rows) from a dimension or transaction list referenced by formulas, data, or access rights; changing a property's type; converting an existing property to auto-generated ID — replaces all existing values with sequential integers; removing or replacing a dimension in a metric's structure; large formula rewrites affecting many blocks simultaneously; any change that could destroy existing data or break formulas referencing it. Treat anything not clearly non-destructive or reporting-only as high risk rather than assuming it is safe. Flag these explicitly in the spec and confirm the user has approved them knowingly before Stage 2.
- **Non-destructive structural** — adding a new list, property, metric, or list items; setting input values; updating a formula; adding a dimension to a metric's structure (nothing existing is dropped); renaming a referenced block; replicating or moving a folder subtree. Low risk of data loss, but still touches a model other users rely on — for the riskier items (many dependents, dimension changes, renames, large folder subtrees) run dependency checks before and after.
- **Reporting changes (views, boards)** — no structural risk; edit directly in the target app.

### Write the spec

For each block to change:

1. **Block name and type** (metric, list, property, view, board)
2. **Current state** (structure, formula, folder, name)
3. **Target state**
4. **Dependencies affected** (upstream and downstream)
5. **Risk tier** (Low / Medium / High)

Order changes by dependency: upstream first (dimensions, source metrics, properties), downstream last (calculated metrics, views, boards).

List which execution skills will be needed in Stage 2, and which steps are UI-only (conditional highlighting, access-right apply rules, scheduled imports) so the user knows what they must finish by hand.

### Replicate or move a folder subtree

When copying or recreating a folder tree under a new parent, use this spec shape instead of the per-block template above:

- **Inventory** — `skill:using-search-tools`: every folder and block in the source subtree (IDs, paths, parent/child links).
- **Spec as a tree** — per folder: path, children, block IDs to move. Disambiguate duplicate folder names by full path.
- **Scope** — only blocks under the source root; not by name, topic, or immediate parent folder name.
- **Names** — mirror source folder names unless the user asks to rename.

Then get the spec approved:

1. Record the change sequence with `tool:track_todo_list`, one item per phase in scope.
2. Present the spec and ask for approval with `tool:ask_user`. Silence is not approval — wait for an answer.
3. If the user changes the scope, update the spec and the todo list before editing anything.

---

## Stage 2 — Execute

> **STOP — do not enter Stage 2 until the user has approved the spec.**

### Execution principles

1. **Respect dependency order** — do not create a block before the blocks it depends on exist. The phases below are a recommended sequence, not a strict one; its hard constraints are that shared blocks are enabled before anything references them, calendar and dimensions precede metrics, metrics precede their formulas, and views and boards referencing a block are removed before the block itself.
2. **Parallelize within and across phases** — batch independent tool calls together. If you are creating 3 properties on different lists, batch them. If two phases have no dependency between them (e.g. updating formatting in phase 6 and renaming blocks in phase 9), run them concurrently.
3. **Do not re-read what you just created.** Trust tool responses. Do not call `tool:search_metrics_and_lists`, `tool:search_folders`, `tool:search_tables`, `tool:get_board`, `tool:get_views`, or other read tools to re-fetch a block you just created in the same session.
4. **Combine dependent steps.** When the next step depends on IDs from the current step (e.g. `tool:add_list_items` after `tool:create_list`), extract IDs from tool responses and emit the dependent calls in the immediately next turn.
5. **Load skills as needed** — pick each phase's execution skill from the skills library headers. You may load several at once if several phases will execute in the same batch. Do not artificially serialize skill loading.

### Execution order

Skip phases not needed for the current change.

1. Enable the libraries the change consumes, if it reads shared blocks
2. Calendar changes
3. Add or modify dimensions, list items, properties, including Version
4. Add or modify metrics, transaction lists, tables
5. Update formulas
6. Update formatting
7. Add or modify views
8. Add or modify boards
9. Reorganize folders and rename blocks (subtree replication: depth-first `tool:create_folder`, then `tool:move_blocks` from the approved inventory)
10. Update security

### Deletion order

Always respect the dependency chain:

1. Delete views and boards referencing the block
2. Delete downstream calculated metrics that depend on it
3. Delete the target block itself
4. Delete orphaned upstream blocks only if they have no other dependents
5. Remove empty folders last

Never delete a block that still has downstream dependents without updating or removing those dependents first.

---

## Verification checklist

After completing all changes:

- [ ] **Formula health** — `tool:list_issues`: no new errors introduced (compare against the pre-existing errors recorded during discovery)
- [ ] **Dependency integrity** — `tool:get_metric_dependencies` on every modified block: upstream/downstream chains intact
- [ ] **Naming consistency** — new/renamed blocks follow existing convention
- [ ] **Folder placement** — no blocks left in "No Folder"
- [ ] **Folder subtree** — nesting and placement match the approved spec; out-of-scope blocks unmoved
- [ ] **Views and boards** — if structural changes affected displayed metrics, confirm views/boards still render correctly
