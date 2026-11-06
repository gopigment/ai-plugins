---
name: analyzing-views
description: >
  Guide for determining which filters apply and how — covering both on-board queries (where the view
  is provided in user visible context) and off-board queries (where the view is not on the board the user has open, or no board is open). Use when
  the query involves filtering, slicing, grouping, or applying board page selectors — before planning
  filters, or before calling the get_views tool or search_views tool. Flow-specific rules live in on-board.md and off-board.md,
  so only the ones that apply are read.
metadata:
  base_directory: /skills/analyzing-views
  includes:
    - "SKILL.md"
    - "*.md"

---

# Analyzing views

This skill applies once you have retrieved a view — either from a widget of the board the user has open, in the User Visible Context (UVC), or found by search when the view is not on that board.

The two files below are **not loaded automatically**. Before planning any filter, read the one that matches your flow with `tool:read_file` — it is **required**:

- **[on-board.md](./on-board.md)** — the target view is available in the UVC: a widget of the board the user has open shows it. The board's active page selectors are constraints you must honour.
- **[off-board.md](./off-board.md)** — the target view is not available in the UVC: no board is open, or the view was found by searching beyond the open board (a board fallthrough). Nothing tells you which scope to apply, so you have to choose one and say which.

The rest of this file applies to both.

## Concepts

**Pivots**: how a view structures its data — dimensions and properties can be added as rows, columns, or interactive page selectors. Page selectors drive filters; row/column pivots drive the default breakdown grain. There are four page selector pivot types — knowing which type determines how to apply a filter:

- **Dimension** — the selector directly uses a dimension (e.g. `Teams`, `Month`, `Version`). Apply filters directly on that dimension.
- **Grouping** — the selector uses a *property on a dimension* to aggregate values (e.g. `Month → Year`, `Country → Region`, `Employee → Department`). The underlying data is at the finer granularity; apply as a filter on the property dimension (e.g. filter on `Month > Year`, not on `Month` directly).
- **Mapped** (`kind: Joined` in the view configuration) — the selector is derived from a *mapping metric* that translates one dimension into another (e.g. `Departments via MAP_Teams to Departments`). The block is dimensioned by the source (e.g. Teams), not the target (Departments). Cannot filter directly — see *Filtering on Mapped pivots*.
- **Slice** — the selector is derived from a *slice configuration* that groups items from source dimensions into named subsets (e.g. `Americas`, `EMEA` via a Region Grouping). Filtering on Slice pivots is not currently supported.

---

## Reading a view's configuration

The view configuration holds the full pivot configuration for each page selector: pivot type (Dimension, Grouping, Mapped, or Slice), the source dimension ID, the mapping metric ID (Mapped pivots only), and the property path (Grouping pivots only). It is the only source of pivot type information — you need it before any filter planning. It comes from `tool:get_views`, unless a tool response already in the conversation includes this view's configuration: then plan from that response and do not call `tool:get_views` again.
Use when: you have a specific view ID (from UVC or chosen via `tool:search_views`) and need to plan which filters to apply — e.g. "show me revenue by territory for Q3" on a board, or after selecting a view off-board.

On reading the view configuration, scan each page selector entry and build a plan:

- Check the pivot type for each selector. The type field tells you how to wire the filter:
  - **Dimension**: note the dimension ID — filter applies directly.
  - **Grouping**: note the source dimension ID and property path — filter applies on the property, not the dimension.
  - **Mapped** (`Joined`): note the mapping metric ID — a separate fetch of the mapping metric is required before filtering.
  - **Slice**: flag immediately — filtering is not currently supported. Note which selector is active and skip it. When it carries a selection (an active board selector, or a board's default selection off-board), tell the user which one and that the fetched result does not honour it.
- Flag any selector whose type is Mapped, Grouping, or Slice — these require extra steps or cannot be applied.
- If a page selector's pivot type is unclear or the required fields are missing, stop and ask before proceeding.

---

## Applying filters

### Filtering on Mapped pivots

Mapped dimensions cannot be filtered directly — they are added dynamically via a mapping metric, and the relationship can vary across dimensions (e.g. an Employee's Team may change by Month). Follow this procedure:

1. **Identify the mapping metric** from the view's pivot config. The config only gives its id (`mappingMetricId`): get its name and dimensions with `tool:search_metrics_and_lists`, passing the id in `block_id_exact_match` with `show_details: true`.
2. **Fetch the mapping metric**, broken down by all its dimensions. Each cell value is an item from the target dimension. Keep the cells whose value is one of the selected target items — e.g. for `L3 Manager = "Sarah Chen"`, the (Employee, Week) pairs where `Emp_Manager L3_Week = "Sarah Chen"`. Several selected target items are handled in this same step — no fetch per item. When the mapped dimension is only a breakdown (e.g. a Mapped row or column pivot), with no item selected, keep all cells. If no cell is kept, the filtered result is empty: say so and do not fetch the original metric — an empty `item_filters` list is ignored and would return unfiltered data.
3. **Fetch the original metric**, depending on how many dimensions the mapping metric spans:
   - **One dimension, and the mapped dimension is only a filter** (e.g. Employee → Department, filtered to Sales): pass the kept items as `item_filters` on that dimension. The result is already filtered.
   - **Several dimensions** (e.g. Employee × Week), **or the mapped dimension is a breakdown** (a Mapped row or column pivot, or e.g. "by Department"): a merge of the original metric and the mapping metric is required, as `item_filters` apply to each dimension separately. Break the original metric down by all the mapping metric's dimensions, plus the breakdown the user asked for, and pass the kept items of each dimension as `item_filters` to narrow the fetch. Then, in one code execution, merge it with the kept mapping cells on all the mapping metric's dimensions and aggregate to the requested granularity, grouping by the mapping value when the mapped dimension is a breakdown.

**Sister filters**: when fetching a Mapped selector's mapping metric, do not apply other Mapped selectors' values. Resolve each Mapped selector separately, then apply all of them to the target metric.
Example: for Mapped selectors Manager and Region, do not filter the Manager mapping by Region, or the Region mapping by Manager.

Constraints: never pass mapped dimensions directly into the data fetch. Apply the other filters on dimensions the two metrics share (e.g. the user's time filter) to both fetches — the mapping may be time-dependent.

### Disambiguating filter candidates

When the user requests a filter and multiple dimensions or properties in the view could represent that concept:

- Restrict candidates to dimensions in the resolved view's page selectors and row/column groupings. Only surface what the view exposes.
- If exactly one candidate exists, use it automatically.
- If multiple exist (e.g. `L2 Territory Owner` and `L3 Territory Owner`, or `Account > Industry` vs `Account > Parent Industry`), present a short list and ask the user to choose. Never silently default.
- If zero view-scoped candidates exist, expand to full metric dimensions and apply the same flow.
- After user selection, restate the chosen mapping before fetching. If the user later corrects, re-fetch and label the response as a corrected run.

---

## Result transparency

After every fetch where at least one filter was applied, tell the user which filters were used. Append a plain-English line: *Fetched using: [filter 1], [filter 2], …*

- State filters in plain business language. For example: write *"filtered to EMEA"*, not *"filter applied: dimension=*`Territory_Region`*, value=*`EMEA_001`*"*.
- If a filter value was resolved to a near-match or canonical label, note the substitution inline.
- If a disambiguation choice was made, state it: *Kyle Nichols as L2 Territory Owner.*
- If no filters were applied, omit the line entirely — except off-board, where off-board.md requires stating the scope.
- Do not include audit fields, internal process details, or fallback logic in user-facing responses.
