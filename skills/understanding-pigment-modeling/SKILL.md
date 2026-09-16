---
name: understanding-pigment-modeling
description: "Foundational skill. Read first before any modeling task. Provides the mental model (in-memory sparse multidimensional engine), block-type taxonomy, engine vocabulary, common decision mistakes, and hard rules."
metadata:
  skill_path: /skills/understanding-pigment-modeling/SKILL.md
  base_directory: /skills/understanding-pigment-modeling
---

# Understanding Pigment Modeling

Read this before any modeling task.

## Mental Model

Pigment is an in-memory, sparse, multidimensional engine. An Application is a collection of typed **blocks**. Two orthogonal layers (Native Scenarios for what-if sandboxes, Snapshots for frozen copies) apply across all blocks.

| Block type | What it is | When to use |
| --- | --- | --- |
| **Dimension list** | Analysis axis with unique **items** (rows) and typed **properties** (columns: Number / Date / Text / Bool / Dimension). A dimension placed in a metric's grid is a **structural dimension**. | Anything you slice metrics by: Country, Product, Employee, Account |
| **Calendar** | Built-in time dimensions: Month, Quarter, Year, Date. | Always reuse. Never recreate time lists. |
| **Version Dimension** | Dimension holding Budget / Actual / Forecast items, with switchover and gating properties. | Any planning cycle. See `skill:building-versions-and-planning-cycles`. |
| **Metric** | Multidimensional sparse grid (Number / Date / Text / Dim / Bool). Each **cell** = one value at one combination of structural items. Blank cells are not stored (**sparsity**). Formulas evaluate within a dimensional context called **scope**. | Anything you compute, plan, or report |
| **Transaction List** | High-volume row store. Items not unique. **Never structural.** | Granular events (GL, orders, HRIS) to aggregate into metrics with `BY` |
| **Table** | Groups metrics sharing dimensions, with calculated rows/columns. | P&L, Balance Sheet, multi-metric reporting |
| **Board** | Container page of Widgets. | Dashboards, reports, input screens |
| **View** | Configured visual of a Metric or Table (pivots, filters, sort, display mode: Grid / Chart / KPI). Reusable across Boards. | Any reusable data presentation |
| **Folder** | Organizational only. No logic. | Sorting and governance |

## Invariants

These are never negotiable.

**Structure**

1. Only **dimension lists** can be structural. Transaction lists never. Aggregate with `BY`.
2. A metric cell = one item per structural dimension. Blank cells are not stored (sparsity).
3. Every app with planning metrics gets a Version Dimension. Do not skip or defer.
4. List Subsets delete data irreversibly on membership change. Prefer filters unless the use case is clear.

**Naming and organization**

5. Never use `.`, `:`, `'`, or `"` in block names. Prefer ASCII.
6. Never place a block at the root level. Use numbered folders.
7. Folders are inert. They affect discovery, not calculation.

**Build order**

8. Calendar (verify/configure) → Dimensions → Version Dimension → Transaction Lists → Metrics (input, then calculated) → Tables → Boards. Always create prerequisites before dependents.
9. Never recreate time dimensions. Reuse the app calendar.

**Formula safety**

10. Never hard-code dimension item references for Time, Version, countries, or planning periods. Use a Dimension-typed input metric. Stable type/class/category items (e.g. `'FX Rate Types'."AVG"`) may stay hard-coded.
11. Never use `DATE()` with literal year/month for period bounds. Use a Date-typed input metric.
12. When T&D is active: never use a disconnected dimension as the property type on a connected dimension. Ask the user about connectivity if unknown.

**Structural changes on existing blocks**

13. A structural change (adding/removing a dimension, changing type) does not automatically propagate to the metrics that reference the changed one. Pigment aligns mismatched dimensions silently (broadcast/collapse) instead of failing, which can produce wrong numbers with no visible error. Before editing, use `tool:get_data_dependency_tree` (direction `Sources`) to trace back to the originating metric(s) and transaction list, so you know what grain of detail already exists upstream. Before declaring such a change done, use `tool:get_data_dependency_tree` (direction `Usages`) to find dependent formulas, then `tool:validate_formula` (with `target.metric_id`) on each to catch a silent mismatch — see `skill:writing-pigment-formulas` ("Structural Dimension Changes") for the full workflow.
14. Removing a dimension from a metric's structure is lossy and irreversible. Unlike adding a dimension, once the metric no longer stores that grain the historical detail cannot be recovered from the metric itself. Say so explicitly when proposing the change, even if the detail still exists upstream (e.g. on a source metric or transaction list).

## Decisions the Agent Gets Wrong Most Often

| Decision | Choose A when | Choose B when |
| --- | --- | --- |
| **Dimension** vs **Transaction List** | Unique items you slice by | High-volume events, no uniqueness, not structural |
| **Dimension-typed property** vs **simple property** | Values form a finite set you may slice or aggregate by (create a dimension list, reference it as property type) | Free text, measure, date, or boolean that is never a slicing axis |
| **Metric** vs **Transaction List** | Aggregated planning / reporting values | Atomic events from ERP / CRM (then aggregate with `BY`) |

When unsure about property types: do not default categorical fields to Text. If the values form a finite set that could become a slicing axis, make it a Dimension.
