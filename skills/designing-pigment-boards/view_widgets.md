# View Widgets

When creating or editing View Widgets on a Board, each widget must point to a View. How you configure the widget depends on whether the View needed modifications.

## Display Modes

See [display_modes.md](../creating-and-editing-pigment-views/display_modes.md) for display mode definitions.

**⚠️ CRITICAL Display Type / Block Type Rules:**

The widget `display_type` MUST match the underlying block type:

- **Widgets on views on List blocks** → MUST use the **List** display type
- **Widgets on views on Metric blocks** → MUST NOT use the List display type (use Table, Chart, Kpi, or Spreadsheet instead)
- **Widgets on views on Table blocks** → MUST NOT use the List display type (use Table, Chart, Kpi, or Spreadsheet instead)

## Creating a View Widget

### Case 1: The View needs no edit

Create the View Widget pointing directly to the existing View. No override needed.

### Case 2: The View needed an edit (a Draft View was created)

Must read and follow the [creating-and-editing-pigment-views](../creating-and-editing-pigment-views/SKILL.md) skill to create and edit a Draft View.

Then:

1. **Create the View Widget** pointing to the **original View** (not the Draft)
2. **Use `update_view_widget_overrides`** to create an override that makes the widget display the Draft View for the current user only

## Changing which View a Widget points to

Update the widget to point directly to the new View ID. Do not use overrides for this.

If the new View needed an edit, a Draft View was created for it. In that case, update the widget to point to the **original new View** (not the Draft), then use `update_view_widget_overrides` to override with the Draft.

## Critical: Overrides are a full replacement

`update_view_widget_overrides` performs a **full replacement** of all overrides on the board for the current user. If multiple widgets need overrides, you must pass all of them in a single call. Calling it per widget will wipe previously set overrides.

## After all widgets are set up

**Do NOT merge any Draft View.** Your job ends at creating drafts and setting overrides.

The user can review the changes in the Board UI and decide to merge each Draft View or discard it.
