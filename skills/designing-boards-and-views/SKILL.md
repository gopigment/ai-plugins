---
name: designing-boards-and-views
description: "Execution skill. Use when creating or editing a Board, a dashboard, or a View -- board sections and layout on the 12-column grid, widgets and their sizing (KPI, chart, grid, text, spacer, action button), board page selectors and their defaults, view pivots (rows / columns / pages), breaking down by or per a dimension, filters, sorts, totals, chart type and configuration, cell formatting, and draft views."
metadata:
  skill_path: /skills/designing-boards-and-views/SKILL.md
  base_directory: /skills/designing-boards-and-views
---

# Designing Boards and Views

A Board is a dashboard made of widgets laid out on a 12-column grid. It describes **what is displayed**, never how it is calculated. Design it after modeling is done.

A View is **how** a Block (Metric, List, Table) is displayed: values, pivots, filters, sorts, aggregation. One Block carries many Views, and every data widget on a Board points at one.

Number formatting (decimals, currency, percent, K/M scaling) is set on the **Metric**, not the View: load `skill:formatting-and-highlighting`. Totals, subtotals and non-default aggregators are in `skill:aggregating-view-data` -- load it as soon as a widget must show a total, or a line whose value divides one metric by another.

Never create Views on sublists.

## Pick the Right Tool

| Goal | Tool |
| --- | --- |
| List or read Boards | `tool:search_boards` / `tool:get_board` |
| Create a Board | `tool:create_board` |
| Board settings (name, description, icon, color, width) | `tool:update_board` |
| Add, move, resize, retarget widgets | `tool:update_board_widgets` |
| Board page selectors and their defaults | `tool:update_board_page_configurations` |
| Show a Draft View to the current user only | `tool:set_widget_preview` / `tool:clear_widget_preview` |
| List existing Views of a block | `tool:search_views` |
| Read one View | `tool:get_views` (`include`: `Formatting`, `GridLayout`, `Rows`, `Columns`, `HiddenDimensionsAggregations`) |
| Discover which pivots are valid | `tool:get_available_pivots` |
| Create a View | `tool:create_view` |
| Pivots on Metric / Table Views | `tool:update_view_pivots` |
| Pivots on List Views | `tool:update_list_view_pivots` |
| Value fields (add, remove, hide) | `tool:update_view_values` |
| Totals and aggregators | `tool:update_view_aggregations` |
| Filters / sorts | `tool:update_view_filters` / `tool:update_view_sorts` |
| Chart type and options | `tool:update_view_chart_config` |
| KPI options | `tool:update_view_kpi_config` |
| Cell colors, bold, alignment | `tool:update_view_formatting` |
| Tree vs tabular, row height, totals position | `tool:update_view_grid_layout` |
| Name, description, template, `sharingStatus` | `tool:update_view` |
| Grid templates | `tool:get_all_view_templates` |
| Save draft Views | `tool:save_draft_views` |

## Build a Board in Four Steps

**1. Define the purpose and plan the pages.** State the Board's purpose in one or two sentences ("track Q1 2026 actuals against budget"). Search the metrics you intend to display to learn which dimensions they carry, then plan the page selectors you want at board level (Time, Version, Scenario, plus business dimensions) -- only those the underlying blocks actually have.

**2. Create the Board.** `tool:create_board` with a name, a description, an icon matching the intent, and a color. Default to full width, header showing icon and color. Never put the Board title or description in a Text widget; they are Board properties.

**3. Find or create the Views, then add the widgets.** Call `tool:search_views` once with every block in the story in `block_ids`, then judge reuse as described in *Reuse or Create* below. Align each View's pages with its siblings as you go, then add the widgets with `tool:update_board_widgets`.

**4. Set the board pages.** `tool:update_board_page_configurations` with the selectors planned in step 1 and their **default selected items** (default Year to FY 26, default Version to Actuals and Budget for a variance board). Confirm each widget is linked to the pages you care about, then check that **every dimension or modality the user named in the request appears as a board page with defaults** -- if the request named one and no View exposes it, go back and fix the View rather than finishing the Board without it.

## Make Board Pages Actually Filter

A board-level selector on dimension **D** only affects widgets whose View has a **compatible page** on D: either a simple page (`dimensionId` = D) or a grouping page whose `listPropertyPath` resolves to D. A View with no Year in Pages is simply not filtered by the board's Year selector, and the Board cannot force a dimension onto a View that does not expose it.

This drives the workflow: **align the Views before setting board pages**, not after. Reconcile after each View, not at the end.

**`pages` is not the leftovers after the axes.** Having a dimension in Rows or Columns is not enough for the Board Page selector on this dimension to apply to the View: that dimension must also be in the View's `pages`. Omitting a dimension from a View's `pages` is therefore a deliberate decision that this View will not follow the Board's selector on it. Decide it as you build each View, not after: that is what makes the Views align with each other, and every dimension you leave out is a selector the Board can never offer.

**A View you reuse owes the same page set as one you build.** A reused or duplicated View keeps whatever `pages` it was created with, and dropping it on a Board unchanged is the usual reason a Board ends up with no selectors at all. Read it with `tool:get_views`, then add the pages it lacks with `tool:update_view_pivots` before you add the widget.

Seeding gets you the dimensions **this** block carries. The Board also needs the ones its **other** Views carry, and those are often coarser: for every dimension **D** a sibling View exposes, this View owes a compatible page too.

- This block has D -> add a simple page on D. Pages plus Rows/Columns together is fine (see *Narrow the Data*).
- This block carries D only at a finer grain F (Month under Year, Country under Region) -> add the **grouping** page `F > ... > D`. A simple page on F is not compatible: a Month page does not follow a Year selector, `Month > Year` does. Seeding never produces this page, because F, not D, is what the block carries -- it is the one you have to add deliberately.
- Skip only when D is absent from the block at every grain.

Board selectors come from the union of the pages of the Views you add; you cannot invent one no View has. A widget whose View **does** have a compatible page can still opt out by unlinking that page, but only the user can do that in the Board UI: `tool:update_board_widgets` cannot set per-widget page links.

**A view filter, or a grouping page pinned to a fixed value, is not a board page.** It hard-codes the context instead of exposing it, and leaves the board selector empty. Every dimension the user named as a filter or a comparison must end as a board page **with `defaultModalityReferences` set** -- a page configuration with empty defaults renders as "All" and does not satisfy a request that named a value.

When the user names modalities ("for FY 26", "Actuals vs Budget"), those exact modalities are the defaults, single- or multi-select accordingly. Typical patterns: monthly review (Month = current, Version = Actuals), variance analysis (Month = current, Version = Actuals + Budget), annual planning (Year = next, Version = Budget + Forecast), scenario planning (Year = current, Version = Forecast, Scenario = the compared set).

Each entry of `pageConfigurations` needs a `pageIdentifier` (`pageIdentifierType` of `Dimension`, `Scenario` or `Variable`, plus the matching `dimensionId` or `variableId`), a `singleModality` boolean, and the defaults as `defaultModalityReferences`, each one `{ "type": "Fixed", "fixedValue": "<modality id>" }`. Set `pageDisplayMode` to `Show`, `Minimize` or `Hide`, and **never `Hide` unless the user asks for it**. The call **replaces the whole list**, so resend every page you want to keep.

## Choose the Widgets

Use **View widgets** for data, **Text widgets** for section titles and commentary, **Spacer widgets** between sections, and **ActionButtons** for navigation or imports. Do not use Images unless asked, and do not describe a navigation intent in text when an ActionButton does the job.

Every View widget requires a **View id**. There is no way to point a widget at a metric or a block directly, not even for a single-value KPI: create or pick a View first, configure it, then reference its id.

A Text widget's content is a list of rich text nodes: one node per paragraph, each holding its text in a child node. Both node types are spelled in PascalCase -- exactly `Paragraph` and `Text`, never lowercase -- and so are the font sizes (`Normal`, `H1`, `H2`, `H3`); the call is rejected outright on any other casing.

**The display mode is set on the widget, not on the View.** The widget carries `display_type` -- exactly `List`, `Table`, `Chart`, `KPI`, or `Spreadsheet`. There is no `Grid` value. It must match the block: Views on **List** blocks must use `List`; Views on **Metric** or **Table** blocks must not. Chart configuration lives on the View (`tool:update_view_chart_config`), not on the widget.

Validate the View against the intended `display_type` **before** creating the widget or calling `tool:set_widget_preview`. For KPI widgets specifically: no row pivots, and `metricsLocation` must be `Columns` or `Pages`, never `Rows`. A KPI shows as many cards as the View has columns, so do not put a large dimension in columns. To show several metrics side by side, prefer a single KPI widget on a View carrying all of them in `values` over several single-metric KPI widgets, even when single-metric Views already exist. KPI is unavailable on List blocks.

To change the View already shown on a widget, call the `update_view_*` tools on that View id; if a Draft is auto-created, follow *Edit a Live View Safely* below.

If the user asked for no new modeling blocks, a Table block is still fair game: it bundles existing metrics for display and is a layout container, not a structural object.

## Lay It Out

Content spans 12 columns. Pick **one** width for the whole Board and keep it: 12 columns for monitoring dashboards, horizontal comparison, KPI rows and dense time series; 10 or 8 columns centered for narrative boards that read like a document -- deep dives, reviews, approval flows. Never alternate widths within a Board.

Structure the Board in **sections** stacked vertically, each opened by a full-width Text widget with an **H2** title (never H1, bold, italic, or underline) and separated by a spacer. Put at most 3 widgets per row, symmetrical in height and width. Side-by-side sections are a rare pattern: two at most, equal height, usually one widget each.

The classic reporting structure is a KPI overview, then charts for visual analysis, then grids for detail. Adapt it to the request; readability, hierarchy and symmetry matter more than the template. Avoid large white gaps, misaligned widgets and unnecessary noise.

Widget titles are usually redundant: keep the text content but set `show_title` to false when the section title already explains the widget. If one widget in a section shows a title, all of them should. Section titles are the default; skip them only on a very simple Board or on an opening KPI row.

## Size the Widgets

Text and ActionButton heights follow the variant, where Text **Open** maps to `layout: Regular` and **Filled (Box)** to `layout: Box`. A Filled variant is always 1 taller than its Open equivalent.

| Widget | Variant | Content | Height |
| --- | --- | --- | --- |
| Text | Open | Title only (H2) | 2 |
| Text | Open | Title only (H1) | 3 |
| Text | Open | Link / normal text only | 2 |
| Text | Open | Title + body | 3-4 |
| Text | Open | Title + body + takeaway | 4 |
| Text | Filled (Box) | any of the above | Open + 1 |
| ActionButton | Button | any label | 2 |
| ActionButton | Card | any label | 10 |

| Widget | Height |
| --- | --- |
| Spacer | 1 (width 12) |
| KPI | 4-6, or 5-7 with a title |
| Chart | 11-18, or 12-18 with a title |
| Grid (`Table` or `List` display) | 11-24, or 12-24 with a title |

Chart and Grid heights depend on data complexity: rows, columns, legends, axis labels.

## Reuse or Create

Call `tool:search_views` to see what exists; this is a light pass, not a hard prerequisite. Pass `display_intent` (`Kpi`, `Grid`, or `Chart`, plus an optional `chart_type`) to get candidates sorted by compatibility. **On a List block use `Grid` or omit it** -- `Kpi`, `Chart` and any `chart_type` return an error, because a List only supports Grid display. Results are paginated and the intent only sorts within one page, so read the later pages before you conclude that nothing fits.

Judge the **name** first: `View 1`, `Test` and other generic defaults are product placeholders, so create a new View instead. A descriptive name plus pivots that fit this widget and its board siblings is a reuse signal.

Then judge the pivots: reuse when rows, columns and pages already match, or when one cheap edit (swap axes, remove a pivot, change granularity, add a missing page) closes the gap. Reusing does not let you inherit an empty `pages`: if the View is headed for a Board, adding the pages it lacks is part of the reuse, not a reason to reject the candidate. Prefer a richer View (formatting, conditional formatting) over a bare exact match; removing a pivot is cheap, recreating formatting is not. A View missing a dimension you need is harder to fix than a View with extra dimensions you can drop. For a KPI intent, an extra **row** pivot disqualifies a candidate; extra column pivots are fine. Restructuring a Grid into a Chart is expensive, and **Grid and Spreadsheet cannot be converted into each other at all** -- they are different display paradigms, so create a new View instead.

When unclear, create. Convention for a new View: time dimensions in Columns, entity dimensions in Rows, and on a Table `metricsLocation: Rows` -- but `Columns` when the View is headed for a KPI widget. Name Grids without a suffix, Charts with the chart type (`Revenue - Waterfall`), KPIs with ` - KPI`.

## Create a View

`tool:create_view` is a two-step flow:

1. Call `tool:create_view`. Leave `pivotLayout` **null** and the server builds a sensible default layout. To override, send all three axes (`rows`, `columns`, `pages`), each entry a seed of `dimensionId` plus optional `listPropertyPath`; use an empty array for an axis with no pivot. Half-specified layouts are rejected.
2. Refine with the `update_view_*` tools. None of those refinements are accepted by `tool:create_view`.

**Ids are server-assigned.** Omit `id` for a new pivot or value; echo back the id from a previous response to keep an existing one. Never invent a UUID.

**Cross-application blocks:** a View lives in the same application as its block. For a block coming from an enabled library, create the View in the library app, then call `tool:update_view` with `sharingStatus: Dataviz` -- otherwise a widget in the consuming app renders "This View doesn't exist anymore." `sharingStatus` needs its **own** `tool:update_view` call: it cannot travel with name, description or template, and it applies only to the canonical View, not to a Draft.

## Place the Pivots

`values` holds what appears in the cells: the metric for a Metric block, one entry per table metric for a Table, one entry per property for a List. Dimensions go in `rows`, `columns` or `pages`. Metrics are never duplicated into those arrays: their axis is set by **`metricsLocation`** (`Columns`, `Rows`, or `Pages`).

Any request to break down, split by, group by, show by, or see something "per" a dimension is a request to add a pivot. Call `tool:get_available_pivots` first and copy the id fields from the returned candidates:

- **Dimension**: copy `dimensionId`.
- **Grouping** (a parent-child level such as `Country > Region`): copy `dimensionId` and `listPropertyPath` verbatim; never retype a property path from a display name.
- **Joined** (mapped dimension): copy `dimensionId` and `mappingMetricId`. To show data the metric is not structured on, prefer a Joined pivot over editing the metric.
- **Slice**: copy `dimensionId` and `sliceConfigurationId`.
- **Scenario**: a scenario selector, with a null `dimensionId`.

`pivotSummary` is a human-readable hint for choosing; never send it back.

**Order within an axis drives nesting**: the first row pivot is the outermost grouping, the first column pivot is the top header band. Order pivots Time -> Business -> Comparison / Scenario, parents before children, and keep dimensions of the same hierarchy on the same axis (they cannot be split between rows and columns). A pivot carrying a filter goes last, in the innermost position; when several pivots are filtered only the first is guaranteed to hold.

**Allocation follows the widget's intended display type**, so settle the display type before placing pivots -- the display type itself lives on the widget, see *Choose the Widgets*. Which axis each type wants is documented per type under *Axis meaning by display type* in the `tool:update_view_pivots` description; read it there.

For a Grid, expose a multi-level hierarchy by adding **one pivot per level in Rows**, shallowest first; switch between tabular and tree rendering with `tool:update_view_grid_layout`.

## Narrow the Data: Pages Before Filters

When the user says "filter", "for FY 24", "focus on EMEA", "Actuals only", or asks to _compare_ a set of modalities ("Actuals vs Budget", "FY 24 vs FY 25"), they mean a **Page Selector**: put the dimension in `pages` and set its default items. This is also the only way the Board gets a selector for that dimension -- a Board can only offer selectors that its Views expose in `pages`. Use `filters[]` only for value-based logic (top 10 by revenue), exclusions, or property rules a page selector cannot express. Both can combine: "top 10 suppliers for 2024" is a Year page plus a `byValue` filter rule.

**A comparison dimension goes on the axis _and_ on Pages.** Putting Actuals/Budget or FY 24/FY 25 only in Rows or Columns is the most common board mistake: the values are visible, but the Board has no selector for them and no defaults can be set. Add the dimension to `pages` with the compared modalities as multi-select defaults, in the same editing pass, so the user can swap the compared set without editing the View.

This is not an exception for comparison dimensions. A dimension left out of `pages` is a dimension the Board's selector can never filter this View by.

**The same dimension on Pages and on Rows or Columns is supported and often correct.** Country in rows with Country on pages defaulted to France; Month in columns with `Month > Year` grouping on pages. Never ask the user to resolve this as a conflict.

Filters are applied after creation with `tool:update_view_filters`. Send the full rule set in `filters.rules` (it replaces the existing one); each rule carries exactly one of:

- **`byItems`** -- keep (`IsIn`) or exclude (`IsNotIn`) items of a pivoted dimension: `pivotFieldId`, `compareOperator`, `modalityIds`. One rule per pivot.
- **`byValue`** -- filter by a value field's cell values: `pivotFieldId`, `valueFieldId`, `compareOperator`, `values`, plus `projections`.
- **`byProperty`** -- filter a pivoted dimension by one of its list properties: `pivotFieldId`, `listPropertyPath`, `compareOperator`, `values`. `listPropertyPath` takes the property's **friendly name** (e.g. `"Name"`); technical names (`_name_XXXXXX`) are not resolved and fail with "property not found". Not supported when the View's underlying block is a List -- there, filter on the value field for that property with a `byValue` rule instead.

`compareOperator` takes the full tool enum name -- `Equal`, `GreaterThan`, `GreaterThanOrEqual` -- never a short form such as `Eq` or `Gt`. Each operator only works on some target types and expects its own number of `values` (`Between*` two, `IsBlank` none, `IsIn`/`IsNotIn` modality IDs on Dimension-typed targets only); the tool schema lists them per operator.

Two rules make or break filters. The `pivotFieldId` **must** come from `rows` or `columns`, never from `pages`; `byValue` and `byProperty` also need a dimension pivot, never the Scenario one. And a `byValue` rule needs exactly one `projection` per pivot on the **opposite** axis whose dimension the metric carries -- no more, no fewer -- naming which modality to compare on; the filtered pivot must be the innermost such pivot on its own axis. A wrong projection set or an operator invalid for the target's type fails the update; a rule with an empty selection or the wrong number of `values` is dropped and reported in `metadata.warnings`.

## Sort

Apply sorts after creation with `tool:update_view_sorts`, in priority order (first wins).

- **ByProperty** -- alphabetical or property-based; `property_friendly_name: "Name"` for A-Z, `None` for the manual dimension order.
- **ByMetricValue** -- rank by a metric; needs `pivot_field_id`, `value_field_id`, and `projections` for every pivot on the opposite axis (or `subtotal_pivot_field_ids`).

A top N is both a sort (ByMetricValue, `Desc`) and a `byValue` filter (`TopN`).

## Aggregate and Total

Anything about totals, subtotals, or a value rolling up with something other than `Sum` belongs to `skill:aggregating-view-data`; load it rather than guessing. Two rules matter enough to repeat here:

- A dimension on **Rows / Columns** creates visible totals through its pivot `aggregationConfigurations`; a dimension on **Pages** or absent from the View is folded by `hiddenDimensionsAggregations` and creates no total cell. A non-default roll-up is set on the pivot of the dimension it rolls up -- a per-period average on the period pivot, a per-entity one on the entity pivot -- not on whichever pivot is nearest, and not through `hiddenDimensionsAggregations` while that dimension is visible. Calendar dimensions are temporal, so when they *are* hidden they take `temporalDimensionsAggregator`, never `otherDimensionsAggregator`.
- **A ratio, percentage, growth or variance metric must never aggregate with `Sum` or `Avg`.** Every time you add one to a **Table** View, wire an **Advanced Aggregator** and its two operands in the same editing pass. Skipping it produces a wrong total with no error.

On Table Views, **remove** irrelevant metrics from `values` rather than hiding them; hidden metrics may still compute. Keep `displayed: false` only for Advanced Aggregator operands, filter targets, or sort targets.

## Edit a Live View Safely

Editing a View that other users or other boards rely on may auto-create a **Draft**, a private working copy. The tool response tells you and returns the Draft id.

1. Use the returned **Draft id** for all further edits; re-running an update on the original id forks a second Draft.
2. Leave the widget bound to the original View and call `tool:set_widget_preview` on the Draft so only the current user sees it.
3. Propose to save: list the draft names, ask for explicit confirmation, then call `tool:save_draft_views` once with all draft ids, and report for each whether it was merged or promoted. Never save unilaterally.

Drafts cannot be deleted by `tool:delete_views`; to discard one, tell the user to do it in the Board UI. A Draft is not a substitute for `tool:create_view` when a genuinely new View is needed.

## Validate

Compare the response with what you sent; dropped fields mean sanitization. Common causes: a filter `pivotFieldId` taken from `pages`, missing projections, a `pivotFieldId` that is not the innermost pivot on its axis, an operator incompatible with the metric type, a `listPropertyPath` that does not exist, or a dimension that is not on the underlying block.

An empty View usually means over-filtering, a block with no data, or `displayed: false` on every value. A View that is slow to load usually has too many breakdowns or too much volume: trim pivots, narrow the pages, and set `show_empty_rows` and `show_empty_columns` to false.
