# View Widgets

When creating or editing View Widgets on a Board, each widget must point to a View. How you configure the widget depends on whether the View needed modifications.

## Display Modes

See [view_display_modes.md](../designing-views/view_display_modes.md) for display mode definitions.

**Board-level page filters:** If the Board should drive several widgets with the same page selector (e.g. Year = FY25), **each** underlying View must include a **compatible** page on that dimension. Otherwise the board selector will not filter that widget. See **[board_pages.md](./board_pages.md)** (Board-to-Widget Page Compatibility Rule).

**⚠️ CRITICAL Display Type / Block Type Rules:**

The widget `display_type` MUST match the underlying block type:

- **Widgets on views on List blocks** → MUST use the **List** display type
- **Widgets on views on Metric blocks** → MUST NOT use the List display type (use Table, Chart, Kpi, or Spreadsheet instead)
- **Widgets on views on Table blocks** → MUST NOT use the List display type (use Table, Chart, Kpi, or Spreadsheet instead)

## Creating a View Widget

**⚠️ CRITICAL (order of operations — every View widget):** Do **not** create or point a **View** widget, or call **`tool:set_widget_preview`**, until the underlying View (or Draft) is **valid for the `display_type` you set on the widget** and for the **Block** (the **Display Type / Block Type Rules** above, plus [view_display_modes.md](../designing-views/view_display_modes.md)).

Read the View (or Draft) and confirm **block ↔ `display_type`** and **view_display_modes** rules (e.g. **Kpi** → no row pivots and `metricsLocation` MUST NOT be `Rows` (use `Columns` or `Pages`); **List** blocks only **List** display). If invalid, fix or **create** a suitable View (`create_view` or Draft) **before** the widget. **Do not** trust `get_block_views` candidates without this check.

## Changing a View that is already on this Board

When the user asks to **modify** the View **currently shown** on a View widget (same board context):

1. If the ask is a **new** visualization on the block (not changing this widget's current View), use **`create_view`** instead.
2. Otherwise call **`tool:update_view`** / **`tool:update_view_chart_config`** / `tool:update_view_filters` / `tool:update_view_sorts` directly on the View id. The response tells you whether a **Draft was auto-created** and returns the **Draft View id**.
3. **If a Draft was auto-created**: use the **returned Draft View id** for any **further edits** to this view — re-running an update on the **original** View id would fork a **second** Draft. Leave the widget bound to the **original** View id and call **`tool:set_widget_preview`** pointing the widget at the **returned Draft View id**, so the widget **displays the Draft** for the current user. Then **propose to save** the Draft — see the **Bulk-save protocol** in `skill:designing-views` (list the Draft, ask for explicit confirmation, then call **`tool:save_draft_views`**); the user can still save or discard it in the Pigment Board UI.
4. **If the edit applied directly** (no Draft fork — e.g. you're editing a View you just created), nothing else to do; the widget already shows the latest content.

The agent does **not** replace the organization-wide widget target, and never saves Drafts **unilaterally**. When `tool:save_draft_views` is available it **proposes** to save — listing the Drafts and asking for explicit confirmation before calling the tool; otherwise the user saves or discards in the Pigment Board UI. See the **Bulk-save protocol** in `skill:designing-views`.

See also `skill:designing-views` (CRITICAL RULES).
