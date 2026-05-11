# View Design Process

## Step 1: Define your ideal View

Before looking at existing Views, clarify what you need:

1. What data should be shown? (which Metric, List, or Table?)
2. How should it be broken down? (which Dimensions for rows/columns/pages?)
3. What filtering is needed? (default items, filters?)
4. What sorting is needed?

Refer to [view_components.md](./view_components.md), [view_filtering.md](./view_filtering.md), and [view_sorting.md](./view_sorting.md) for detailed guidance on value fields, pivots, filtering, and sorting.

### Display Modes

See [view_display_modes.md](./view_display_modes.md) for display mode definitions and block type rules.

## Step 2: Search for existing Views

Look for existing Views on the Block that match your intent. Must read: [relevant_views.md](../designing-pigment-boards/relevant_views.md). Use `get_block_views` to retrieve and evaluate candidates.

## Step 3: Decide: reuse, edit, or create

Compare the best existing View against your ideal design from step 1:

- **Matches your design** → use it as-is
- **Close but needs changes** → create a Draft from the existing View, then edit the Draft to match your ideal (pivots to add/remove, filters to adjust, sorting to change)
- **No good match** → create a new Draft View following your ideal design

In all cases where you create or edit, work exclusively with Draft Views. Never merge — leave the Draft for the user to review.

View creation and modification uses an **incremental approach**: create a basic Draft View first, then configure it step-by-step through multiple operations until it matches your design.

When editing a View within a widget on a Board, you must refer to [view_pivoting.md](./view_pivoting.md) to determine whether pivots should be placed in Columns or Rows

## Step 3b: Apply a View Template (Grid only)

View templates only apply to **Grid** display mode (Table / List / Spreadsheet). Skip this step for Charts and KPIs.

If `tool:get_all_view_templates` is available:

**After creating a new Draft View in Grid display mode:**
- Call `tool:get_all_view_templates` to list available templates
- Pick the most appropriate one
- Apply it silently — no need to ask the user first
- If no template clearly fits, skip and proceed to Step 4

**When editing an existing Grid view (not a creation flow):**
Apply the most suitable existing template :
- When the View has no custom formatting
- The Pivots (Pages, Rows, Columns, MetricLocation) have been changed

## Step 4: Validate the returned View

After creating or editing a Draft View, check that the returned object matches what you sent in the request. If fields are missing or different, the backend performed silent sanitization (e.g., invalid pivots or filters were dropped).

If the tool returns an error or the response doesn't match your request, refer to [view_troubleshooting.md](./view_troubleshooting.md).
