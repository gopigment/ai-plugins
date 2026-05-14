---
name: creating-and-editing-pigment-views
description: Always use this skill when creating or editing Views, or needing to pick a View.
metadata:
  skill_path: /creating-and-editing-pigment-views/SKILL.md
  base_directory: /creating-and-editing-pigment-views
  includes:
    - "*.md"
---

# How to Use This Skill

**Progressive Disclosure Pattern**: This `SKILL.md` provides an overview. Most details live in supporting files.

## UI and tool semantics (Views)

These notes align the Pigment UI with view creation and edition tools

### `values` (value fields)

Each entry describes what appears in the cells :

- Metrics → one metric
- Tables → one value field per table metric
- List → one value per property

Value labels in Pivot panel in the UI are different:

- for Tables: "Metrics"
- for Lists: "Properties"

### `metricsLocation` (Metric & Table views)

**API enum** (C# `MetricsLocation`): **`Columns` (1)**, **`Rows` (2)**, **`Pages` (3)** — which axis carries the **metrics** in the pivot. **Dimensions** go under **`rows`**, **`columns`**, or **`pages`**. **Metrics** are chosen in **`values`**; they are **not** duplicated into the arrays for dimensions. Their axis is set only via **`metricsLocation`**.

**KPI rule**: `metricsLocation` for a KPI View MUST NOT be `Rows`. KPIs have no row pivots, so Rows yields a broken layout. Use `Columns` (default) or `Pages`.

### `pivotFieldId`

Every **pivot field** you add in `pages`, `rows`, or `columns` gets a **stable id** (GUID) in the View. **`pivotFieldId`** in other structures (filters, sorts, etc.) points to that pivot.

### `listPropertyPath` on pivots (grouping / hierarchy)

`ListPropertyPath` is the **technical** name of **properties** (e.g. in the UI: `Month > Year`, `Country > Region`).

### Other

- Do **not** create views on **sublists**.
- The **widget’s** `display_type` (KPI / Grid / Chart) is **not** stored on the View; configure it on the Board widget.

---

# CRITICAL RULES

- **Values/Rows/Columns/Pages field `id`** — Must always be a **freshly generated UUID**, never the metric ID. Each value entry needs its own unique identifier distinct from the metric/property it references.
- **New View (greenfield)** — Use **`tool:create_view`**
- **If board in user context and user wants to change a View** — **Do not** rewrite that View in place and rely on the agent to “validate” for them: use **`create_draft_view`** from that View, edit the **Draft** with **`update_draft_view`** (and related tools), keep the widget pointing at the **original** View, and use **`update_view_widget_overrides`** so only **this** session/user sees the Draft on the widget until they **save** in the Board UI. Details: [view_widgets.md](../designing-pigment-boards/view_widgets.md).
- **Bulk-save protocol** if `tool:save_draft_views` is available — after creating or editing one or more Draft Views:
  1. List the draft view names and ask the user for explicit confirmation before saving.
  2. Once confirmed, call `tool:save_draft_views` once with all draft view IDs.
  3. Report each result: view name, resulting ID, and whether it was merged or promoted to a new view.
- When editing a View in the context of a widget on a Board, you must:
  - use the **draft + override workflow** to allow safe, user-specific preview before committing changes that affect all users.
  - read: [view_widgets.md](../designing-pigment-boards/view_widgets.md).
- **Name (first signal)** — **"View 1"** and similar are often **placeholders**. Prefer **`create_view`** with a real name and pivots aligned to **this** widget and **other widgets on the same board** unless the existing View already fits.
- **View you created earlier in this run (not yet shared / saved on a board)** — You may **`update`** it directly without a Draft when that matches product flow.
- **Shared / external View (other users, other boards)** — Prefer **Draft** (or a **new** View) before overwriting something others rely on or displayed in another board, except if asked explicitely.

# Definitions

## Views

A View is **how** a Block (Metric, List, Table) is shown: pivots, filters, sort, display. One Block, many Views.

## Draft Views

A **private** working copy to **preview** edits before they hit an existing view. On **boards**, use a Draft + **widget overrides** when changing the **live** View behind a widget—see [view_widgets.md](../designing-pigment-boards/view_widgets.md). **Not** a substitute for **`create_view`** when you need a **new** View. **Save via bulk-save protocol** — list the draft names, wait for user confirmation, then call `tool:save_draft_views` once with all draft view IDs

---

# Reuse vs create

`tool:get_block_views` helps **spot** candidates; **there is no hard rule** to “find similar views” before you create. **Creating** is **normal** when names are generic or pivots do not match the board. Details: [relevant_views.md](../designing-pigment-boards/relevant_views.md), [view_design_process.md](./view_design_process.md).

# View naming

**Grid** — no widget suffix. **Chart** — add chart type, e.g. `… - Waterfall`. **KPI** — suffix ` - KPI`.

---

# View Design Process

Must read: [view_design_process.md](./view_design_process.md).

# View components, filters, sort, aggregators

- [view_components.md](./view_components.md)
- [view_filtering.md](./view_filtering.md)
- [view_sorting.md](./view_sorting.md)
- [view_display_modes.md](./view_display_modes.md)
- [view_pivoting.md](./view_pivoting.md)
- [view_aggregators.md](./view_aggregators.md)
