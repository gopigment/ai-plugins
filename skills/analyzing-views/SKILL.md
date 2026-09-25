---
name: analyzing-views
description: >
  Guide for determining which filters apply and how — covering both on-board queries (where the view
  is provided in user visible context) and headless queries (where the view is not provided in the
  user visible context). Use when the query involves filtering, slicing, grouping, or applying board
  page selectors — before calling the get_views tool or search_views tool.
metadata:
  skill_path: /skills/analyzing-views/SKILL.md
  base_directory: /skills/analyzing-views
  includes:
    - "*.md"

---

# Analyzing views

This skill applies once you have retrieved a view — either via User Visible Context (UVC) when the user is on a board, or via `tool:search_views` followed by `tool:get_views` in a headless flow. Throughout this skill:

- Sections marked **[on-board only]** apply only when the target view is available in the UVC (user is on a board with active page selectors).
- Sections marked **[both flows]** apply in either case.

## Concepts

**User Visible Context (UVC)**: for the purposes of this skill, the relevant UVC is the board UVC — the structured block injected when the user is viewing a board. It contains the board ID, widget display names, widget names, view IDs, and active page selector values. If the UVC contains a board ID, widget display names, and view IDs, use these directly to identify the block and any active filters — no block search call is needed.
**Pivots**: how a view structures its data — dimensions and properties can be added as rows, columns, or interactive page selectors. Page selectors drive filters; row/column pivots drive the default breakdown grain (see *Display grain*). There are four page selector pivot types — knowing which type determines how to apply a filter:

- **Dimension** — the selector directly uses a dimension (e.g. `Teams`, `Month`, `Version`). Apply filters directly on that dimension.
- **Grouping** — the selector uses a *property on a dimension* to aggregate values (e.g. `Month → Year`, `Country → Region`, `Employee → Department`). The underlying data is at the finer granularity; apply as a filter on the property dimension (e.g. filter on `Month > Year`, not on `Month` directly).
- **Joined** — the selector is derived from a *mapping metric* that translates one dimension into another (e.g. `Departments via MAP_Teams to Departments`). The block is dimensioned by the source (e.g. Teams), not the target (Departments). Cannot filter directly — see *Filtering on mapped dimensions (Joined pivots)*.
- **Slice** — the selector is derived from a *slice configuration* that groups items from source dimensions into named subsets (e.g. `Americas`, `EMEA` via a Region Grouping). Filtering on Slice pivots is not currently supported — see *Unsupported pivot types* below.

---

## Reading a `tool:get_views` response

`tool:get_views` returns the full pivot configuration for each page selector: pivot type (Dimension, Grouping, or Joined), the source dimension ID, the mapping metric ID (Joined pivots only), and the property path (Grouping pivots only). This is the only call that surfaces pivot type information — it must be called before any filter planning.
Use when: you have a specific view ID (from UVC or chosen via `tool:search_views`) and need to plan which filters to apply — e.g. "show me revenue by territory for Q3" on a board, or after selecting a view in a headless flow.

`tool:search_views` (called with the block id in `block_ids`) returns slim selection metadata: view name, referring board names, and dimension names as plain strings. Joined and Dimension pivots return the same dimension name — the type distinction is lost. Use `tool:search_views` only to select the most relevant view, then follow with `tool:get_views` to get the full pivot configuration.
Use when: you found a block via search and need to pick the most relevant view before filter planning — e.g. headless query where no view ID is in context yet.

On reading the `tool:get_views` result, scan each page selector entry and build a plan:

- Check the pivot type for each selector. The type field tells you how to wire the filter:
  - **Dimension**: note the dimension ID — filter applies directly.
  - **Grouping**: note the source dimension ID and property path — filter applies on the property, not the dimension.
  - **Joined**: note the mapping metric ID — a separate fetch of the mapping metric is required before filtering.
  - **Slice**: flag immediately — filtering is not currently supported. Note which selector is active and skip it.
- Flag any selector whose type is Joined, Grouping, or Slice — these require extra steps or cannot be applied.
- If a page selector's pivot type is unclear or the required fields are missing, stop and ask before proceeding.

---

## Applying filters

### Board selector application [on-board only]

All active UVC page selectors are required constraints on the **final result** — the analysis must honour every selector. That does **not** mean every selector’s values should be passed into every intermediate fetch (see Joined pivots below).

For each active selector, look up its pivot type in the view's pivot configuration and apply accordingly:

- **Dimension** (direct): apply directly on the target dimension.
- **Grouping**: apply as a property filter on the source dimension using the property path from the pivot config.
- **Joined** (mapping): follow the mapped dimension procedure below.

**User override**: if the user explicitly requests different filter values or asks to ignore a selector, their instruction takes precedence. Only override when explicit — never infer an override from absence of mention.

**Multi-value selectors**: if the board selector has multiple selected values, fetch once per selected value, union the results, then aggregate. Never drop values to avoid splitting.

**Unsupported pivot type — Slice**: if an active UVC board selector has `kind: Slice`, a customer data fetch cannot replicate it. Disclose this explicitly to the user: state which selector is active and that the fetched result will not honour that constraint. Offer to proceed without it.

### Display grain [on-board only]

When the user asks about data on the current board/view, default the answer grain to the view's row/column pivots from `tool:get_views`. Only use a different grain if the user explicitly asks.

- **Dimension / Grouping** row or column pivots: pass them as `breakdown_dimensions`.
- **Joined** row or column pivots: do not pass the mapped dimension directly — resolve via the mapping metric (same procedure as Joined filters), then present at that mapped grain.

### Selector scope check [on-board only]

A board selector is only "in scope" for a metric if its target dimension appears in the resolved view's pivot config.

- **Not in pivot config** → first check whether this selector applies to a sibling metric (see Shared Entity Universe below). If no sibling relationship exists, silently ignore. Do not ask the user, do not attempt to apply.
- **In pivot config but mapping unclear** → hard stop. Ask clarification before returning any numbers.
- **Correction rule**: when re-applying a filter after a failed attempt, always re-consult the pivot configuration. Do not guess an alternative property path. The pivot config is the single source of truth — even when fixing mistakes.

### Shared entity universe [on-board only]

When a board selector applies to a sibling metric but NOT the target metric (its dimension is absent from the target metric's pivot config), the visible rows on the board are still constrained — all metrics in a widget share the same visible rows. The agent must replicate this behavior when fetching outside the platform.

**Detection**: cross-reference each active UVC selector's dimension against the target metric's native dimensions. If absent, the selector reaches this metric only through a shared dimension with a sibling.

**When detected**: do not apply the selector directly to the target metric.

1. Explain the mismatch — the selector doesn't directly apply to this metric, but on the board the data appears filtered because all metrics in a widget share visible rows.
2. Offer two options:
  - **(A) Full dataset** — fetch without the selector's constraint.
  - **(B) Board-matching view** — derive the entity set from the sibling metric where the selector applies, then filter the target metric to that set on the shared dimension.

**Procedure for option B**: the sibling metric's view ID is available in UVC for the same widget — call `tool:get_views` on it to retrieve its pivot config. Confirm the selector's dimension appears in that config → fetch the sibling with the selector applied → extract the entity set on the shared dimension → fetch the target metric filtered to that set only.

**Common pattern — ranking + value metric**: one metric defines which entities appear (e.g. top 10 by NNACV), another provides values (e.g. meetings count). Apply selectors to the ranking metric → use entity set as the ONLY filter on the value metric → do NOT re-apply selectors directly unless the value metric's own pivot config defines a mapping for that selector.

**Time selector trap**: time selectors (Snapshot Week, Quarter, Fiscal Period) are the most dangerous case. They often scope one metric ("pipeline as of this snapshot") but have completely different semantics on another ("only meetings in this one week"). If a time selector's dimension is not in the target metric's pivot config, never apply it directly. Be especially cautious with time-based selectors and flag the semantic difference when presenting options.

### Filtering on mapped dimensions (Joined pivots) [both flows]

Mapped dimensions cannot be filtered directly — they are added dynamically via a mapping metric, and the relationship can vary across dimensions (e.g. an Employee's Team may change by Month). Follow this procedure:

1. **Identify the mapping metric** from the view's pivot config.
2. **Fetch the mapping metric** (e.g. `Emp_Manager L3_Week`). Each cell value is an item from the target dimension.
3. **Filter the results**: find all source dimension item combinations where the value equals the user's desired target. E.g. for `L3 Manager = "Sarah Chen"`, find all (Employee, Week) pairs where `Emp_Manager L3_Week = "Sarah Chen"`.
4. **Apply source dimension filters** to the original metric using the identified items.

**Sister filters**: when fetching a Joined selector's mapping metric, do not apply other Joined selectors' values. Resolve each Joined selector separately, then apply all of them to the target metric.
Example: for Joined selectors Manager and Region, do not filter the Manager mapping by Region, or the Region mapping by Manager.

Constraints: never pass mapped dimensions directly into the data fetch. Confirm which source dimensions the mapping metric spans. If the mapping metric has a time dimension, apply the user's time filter to it as well — the relationship may be time-dependent.

### Disambiguating filter candidates [both flows]

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
- If no filters were applied, omit the line entirely.
- Do not include audit fields, internal process details, or fallback logic in user-facing responses.

**Board fallthrough disclosure**: if the result follows a board fallthrough and there were active UVC page selectors at the time, always note this regardless of whether filters were applied — disclose the drop explicitly: e.g. *"Note: this board's active filters (Region = EMEA) were not applied — the result is unfiltered."*
