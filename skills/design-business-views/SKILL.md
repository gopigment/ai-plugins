---
name: design-business-views
description: "Execution skill. Load it right after finishing a block people will read as a Grid (a new table, a set of metrics, a Transaction List) and whenever you create or restyle a view (`Table` or `List` display or even metrics), before the first `update_view_*` call. Covers pivot placement, page order, tree mode, column widths, metric names, header colors and recurring View patterns (tabular input View, default View of a Transaction List), even if the user said nothing about layout."
metadata:
  base_directory: /skills/design-business-views
  includes:
    - "SKILL.md"
---

# Grid View Layout Patterns

Companion to `skill:designing-boards-and-views`. Covers how to lay out a **Grid** View (`Table` or `List` display) so a human understands it at first glance: pivots, page order, tree mode, column widths, metric names, number formats, header colors, and the styling of known patterns.

- **Part 1 — General rules** apply to every Grid View you create or restyle. Read them once per task.
- **Part 2 — Patterns** are recurring View types. Recognize the one your task matches and apply it on top of Part 1. Where a pattern contradicts the default convention in `skill:designing-boards-and-views` (e.g. `metricsLocation: Rows` on a new Table View), the pattern wins.

Lay out the View right after the block it displays is finished: placement of dimensions and metrics is what makes a block readable.

## Gather Before You Lay Out

Collect these once, before the first `update_view_*` call:

| Need | Tool |
| --- | --- |
| Dimensions of each metric, and whether it has a formula | `tool:search_metrics_and_lists` with `block_id_exact_match` + `show_details: true` |
| Valid pivots, including grouping pages such as `Month > Year` or `Employee > Department` | `tool:get_available_pivots` |
| Size of a dimension (is it over 500 items?) | `tool:get_list_items` with `offset: 500`, `limit: 1` -- any item returned means more than 500 |
| Typical item-name length of a dimension | `tool:get_list_items` with `limit: 50`, then average the `name` lengths |
| Existing Views of the block and their state (`isDefaultView`, `hasCustomFormatting`, `hasCustomGridLayout`) | `tool:search_views` then `tool:get_views` |
| Page order and colors already used by sibling Views | `tool:get_views` with `include: ["Pages", "Formatting"]` |
| Current number format of each metric (A10) | `tool:search_metrics_and_lists` with `show_details: true` |

---

## Check your work

When working on designing views, always check the result before presenting it to the user. Use `tool:get_views` and verify each settings and if the complete work make sense in comparison to what you were trying to achieve.

## Part 1 — General Rules

### A1 · Rows hold the largest dimension

People scroll vertically, so put the dimension with the **most items** in Rows: Account (P&L), Employee, TBH, Product, Customer. When that dimension is long and has a parent, nest it under the parent (Region > Country) to make it readable: parent first, child second.

Tool: `tool:update_view_pivots` (Metric / Table Views) or `tool:update_list_view_pivots` (List Views).

### A2 · Columns hold time, a second large dimension, or the metrics

- On an input Table, Columns hold the **metrics** (`metricsLocation: Columns`) -- see B2.
- Otherwise Columns hold Time or the second-largest dimension.
- **Month in Columns or Rows → add the grouping page `Month > Year`** (copy `dimensionId` + `listPropertyPath` from `tool:get_available_pivots`). Without it the View shows every month of every year side by side.

### A3 · Pages: what goes there

Pages hold two kinds of selectors:

- **Selectors that narrow the numbers.** **Version** always goes in Pages when a metric carries it, never on an axis unless the View compares versions (then both, see `skill:designing-boards-and-views` *Narrow the Data*).
- **Selectors that narrow the items on the axes.** Year (when Month is on an axis), Cost Center / Department, Country. For a dimension already on an axis, use its **grouping** page (`Employee > Department`), not a separate dimension pivot.

Page selectors have a single or multi select option (`singleModality` on each page in `tool:update_view_pivots`). Single select is useful for dimensions like Version where summing does not make sense. Only apply single select when it makes business sense (Version, Currency). Never apply it for high cardinality dimensions (> 50 items)

### A4 · Pages: order

Order Pages from the dimension that changes least and has fewest items to the one that changes most:

1. **Version** -- it stays on its default (the current version) in most Views.
2. **Time** -- `Month > Year`, Year.
3. **Business** dimensions -- Department, Country, Entity, …

For the Pages axis this order replaces the general Time -> Business -> Comparison order in `skill:designing-boards-and-views`. Keep the **same page order across all Views of the same Board or application**: read a sibling View's `Pages` with `tool:get_views` and copy its order before setting yours.

### A5 · Tree mode for deep nesting

When Rows carry **3 or more nested pivots** (count the metrics as one when `metricsLocation: Rows`), switch to tree mode: `tool:update_view_grid_layout` with `displayMode: Tree` and `totalsInFirstRow: true` (it is not set automatically). Keep `Tabular` below 3.

### A6 · Row header width

Size the first column (row headers) so item names are readable. Default to **175 px** (about 20 characters): `tool:update_view_grid_layout` → `layout.columnWidths.rowHeaderColumnWidths`, one entry per row pivot (`pivotFieldId` from `tool:get_views` `Rows`), `pixels: 175`. In Tree mode only the first entry is used.

Do not try to fit the longest item: some names are far longer than the rest. Go wider than 175 px only when most names exceed 20 characters, and never past about a fifth of the View's width.

### A7 · Data column widths

- **Reporting View** (numbers in cells): keep the default width. Do not send `dataColumnsWidth`.
- **Metrics in Columns** (input Views, or wherever cell content varies by metric): size each metric to its content with `perMetricColumnWidthOverrides`, in `characters`, one entry per value field (`valueFieldId` from `tool:get_views` `Values`). There is no auto-fit, so estimate:
  - Dimension-typed metric: average item-name length of that dimension + 2.
  - Number / currency: the widest formatted value you expect (e.g. `1,250,000 €` → 12).
  - Date: 12. Boolean: 6. Text: 25, up to 40 for comments.
  - Never narrower than the metric's name when the name is 20 characters or fewer.
- **List Views** have no shared width: use `listColumnWidths` instead.

### A8 · Truncated headers

The tools cannot set the header row height: `rowHeight` only sets data rows. When a metric name is longer than its column, widen that column (A7) if the name is 20 characters or fewer. Otherwise leave it, and tell the user in your final message which headers are cut off so they can enlarge the header row in the View's Layout panel.

### A9 · Friendly names

Metrics are usually created with a naming convention that is not user friendly (`INP_Unit_Cost`, `CALC_Total_Cost`). Set a friendly name for **every** metric in the View: at least drop the prefix and suffix (`INP_`, `CALC_`, `RUNX_`, …) and replace underscores with spaces. Keep it short enough to fit the column header, 20 characters or fewer when possible: `INP_Useful_Life_Months` → `Useful Life (Months)`.

Tool: `tool:update_view_values`, `friendlyName` on each value. It renames the column in this View only; the metric is unchanged. The call replaces the whole value list, so echo every existing value `id` with its `sourceId`, `type` and `displayed`, and set names and order (B2 step 3) in the **same** call.

### A10 · Number formatting

By default, number metrics show a thousand separator and no decimals. A metric that is explicitly a percentage (see the name rules in `skill:formatting-and-highlighting`) is shown as a percentage instead.

Views have no number-formatting tool: the format lives on the **metric** and applies to every View showing it. So:

1. Read each Integer / Decimal metric's current `defaultFormat` with `tool:search_metrics_and_lists` (`block_id_exact_match` + `show_details: true`).
2. Leave alone any metric that already has a deliberate format (prefix, suffix, multiplier, or non-zero decimals the modeler chose).
3. For the others, call `tool:update_metric` with `default_format.numberFormatOptions`:
   - Plain number: `numFractionDigits: 0`, `noThousandSeparator: false`.
   - Percentage or Rates: `multiplier: 100`, `multiplierSuffix: "%"`, `numFractionDigits` as `skill:formatting-and-highlighting` says.

   Resend the metric's existing `numberFormatOptions` fields (prefix, suffix, `negativeValue`, …) alongside the ones you change, so they are kept.

### A11 · Header formatting

Do not impose a header color: keep Pigment's default header style. Users have their own visual conventions, and a color the agent picks on its own is noise.

Color headers only when the user asks for it, and keep header colors a View already has: `tool:update_view_formatting` replaces **all** overrides on every call, so read `Formatting` with `tool:get_views` first and resend the existing header overrides together with any pattern overrides (B1 step 4, B2 step 5, B3 step 5).

When the user asks for header colors: `tool:update_view_formatting`, one override per header, each with `area: Header` and `style.backgroundColor`:

- Row headers: one path element per row pivot, `{ pivotFieldId: <row pivot id>, values: [{ kind: "AllModalities" }] }`.
- Column headers: one path element per column pivot, the same way. When `metricsLocation: Columns`, add the metric header: `{ pivotFieldId: null, values: [{ kind: "AllModalities" }] }`.
- Corner cells naming the row dimensions: `area: RowAxisTitle`, `{ pivotFieldId: <row pivot id>, values: [] }`. They only show in Tabular mode with `showRowHeaders: true` (`tool:update_view_grid_layout`).

### A12 · Hide empty rows and columns

Pigment often displays a huge number of cells, many of them empty, which makes a View hard to read. Users rely heavily on the "Hide empty rows and columns" option of the UI: set it for them.

Tool: `tool:update_view_filters` with `filters.sparseSettings: { showEmptyRows: false, showEmptyColumns: false }`. `tool:create_view` does not accept it, so set it after creation. `rules` is required and replaces every existing rule: resend the View's current rules (read `Filters` with `tool:get_views`), or `[]` if it has none.

**Never hide empty cells on the axis that carries the metrics.** An empty metric must stay visible, so the user sees it exists:

- `metricsLocation: Rows` → `showEmptyRows: true`, `showEmptyColumns: false`.
- `metricsLocation: Columns` → `showEmptyRows: false`, `showEmptyColumns: true`.
- `metricsLocation: Pages` → both `false`.

---


## Part 2 — Patterns

### B1 · Default Modeling View

**When:** you finish a substantial piece of modeling (several connected metrics) and present it to the end user, usually a modeler, so they can review your work and validate the result. This is the **minimum** layout to reach before handing the work over.

**Recognize it:** good practice is to gather the metrics you built into a Table (`tool:create_table`) so a human can read them together. Whenever you do, apply this pattern to the Table's View. The metrics are mostly formulas linked to each other, not user inputs (for inputs, see B2).

**Steps**

1. **Pivots** (`tool:update_view_pivots`, pivots copied from `tool:get_available_pivots`):
   - Rows: A1. Columns: A2. Pages: A3, in A4 order.
   - Put the metrics on Rows or Columns (`metricsLocation`), almost never on Pages. Metrics should not be nested with a high cardinality dimension (>5).
   - For this pattern, don't put the largest-cardinality dimension in Rows or Columns, as it will not be legible: keep it in Pages, multi-select per A3, so the View shows the total across all items.
2. **Metric names:** **keep the metric names as they are.** Skip A9 for this pattern: the modeler reviewing the logic needs the real names, the ones the formulas reference.
3. **Metric order** (`tool:update_view_values`, echo every existing value `id`, all `displayed: true`): follow the calculation's dependency order, so each metric comes after every metric it references.
   - Read the formulas with `tool:search_metrics_and_lists` (`show_details: true`), or walk the chain with `tool:get_data_dependency_tree` (`direction: Sources`) from the final metric.
   - Inputs and drivers come first, then intermediate calculations, then final outputs.
4. **Buckets** (only when the View holds **more than 10 metrics**):
   - Group the metrics by calculation stage (e.g. drivers, intermediate, outputs) so the reader can spot each group at a glance.
   - Give each bucket its own `_10` background on its metrics' cells: `tool:update_view_formatting`, one override per metric, `{ pivotFieldId: null, values: [{ kind: "Modality", id: <valueFieldId> }] }`, `area: Both`. Rotate `Blue_10`, `Teal_10`, `Orange_10`, `Purple_10`. Do not use `area: Header` here: a header style aimed at a metric is saved but does not render.
   - Send the bucket overrides in **one** call, together with any header overrides the View already has (A11).
   - The tools cannot set borders: use background colors only.
5. **Widths** (`tool:update_view_grid_layout`): A6 and A7.
   - Metrics on Rows: names are not shortened in this pattern, so size the metrics column (`rowHeaderColumnWidths` entry with `pivotFieldId: null`) to fit most metric names.
   - Metrics on Columns: `perMetricColumnWidthOverrides` per A7.
6. **Hide empty cells:** A12, to keep the View manageable in volume.
7. **Name and placement** (`tool:search_views` on the Table, then `tool:get_views`):
   - Only the default View exists and it has no custom formatting or grid layout (`isDefaultView: true`, `hasCustomFormatting: false`, `hasCustomGridLayout: false`) → apply steps 1-6 to the **default View** itself. If the edit auto-creates a Draft, follow *Edit a Live View Safely* in `skill:designing-boards-and-views`.
   - Otherwise → `tool:create_view` with a name that says what the View shows (e.g. `Depreciation - review`), never a placeholder such as `View 1`, and apply steps 1-6 to it.

**Check:** `tool:get_views` with `include: ["Rows", "Columns", "Pages", "Values", "Filters", "GridLayout", "Formatting"]`:
- The metrics are on Rows or Columns, in dependency order, with their original names (no `friendlyName`).
- Empty rows or columns are hidden on the non-metric axis only.
- With more than 10 metrics, each bucket has its own header color.


### B2 · Tabular Input

**When:** the View is where users **enter several pieces of information about one item**: an employee's department, start date, and salary, rather than a single number spread across dimensions. The block is usually a Table of input metrics sharing one main dimension: Employee, TBH, Account, Request, Project, Contract, …

**Recognize it:** many metrics in the Table have **no formula** (`tool:search_metrics_and_lists` with `show_details: true`) and share one high-cardinality dimension.

**Steps**

1. **Pivots** (`tool:update_view_pivots`, pivots copied from `tool:get_available_pivots`):
   - Rows: the main input dimension only. Add its parent levels only when the hierarchy is deeper than 3 levels.
   - `metricsLocation: Columns`.
   - Pages, in A4 order: Version if the metrics carry it; `Month > Year` if they carry Month; then the business dimensions the metrics carry, or the main dimension's grouping pages (`Employee > Department`, `Employee > Country`) when those properties exist.
   - If the main dimension has **more than 500 items**, also add it to Pages so users can search for one item. Having the same dimension in Rows and Pages is supported.
2. **Metric names** (`tool:update_view_values`): a friendly name on every metric, as in A9.
3. **Metric order** (`tool:update_view_values`, echo every existing value `id`, all `displayed: true`): order the columns in the order a user fills them in. Descriptive and categorical fields first (Name, Department, Role), then dates (Start date, End date), then amounts and quantities, then formula metrics last. Steps 2 and 3 use the same tool, which replaces the whole value list: send names and order in **one** call.
4. **Widths** (`tool:update_view_grid_layout`): row header at 175 px (A6), and each metric sized to its content (A7), measured against the friendly name from step 2. A Dimension-typed metric is sized to the average item-name length of the dimension it points to.
5. **Separate inputs from calculations** (`tool:update_view_formatting`; it replaces all overrides, so read `Formatting` with `tool:get_views` first and resend the ones to keep):
   - Formula metrics: background `Grey_10` on the cells. Target with one path element `{ pivotFieldId: null, values: [{ kind: "Modality", id: <valueFieldId> }] }`, `area: Data`.
   - Totals: bold. Target with one path element `{ pivotFieldId: <row pivot id>, values: [{ kind: "Total" }] }`.
   - If sibling Views in the application already style formula metrics or totals, reuse their colors instead of these defaults.
   - Send these in one call, together with any header overrides the View already has (A11).
6. **Name and placement** (`tool:search_views` on the Table, then `tool:get_views`):
   - Only the default View exists and it has no custom formatting or grid layout (`isDefaultView: true`, `hasCustomFormatting: false`, `hasCustomGridLayout: false`) → apply steps 1-5 to the **default View** itself. If the edit auto-creates a Draft, follow *Edit a Live View Safely* in `skill:designing-boards-and-views`.
   - Otherwise → `tool:create_view` named `<Table name> input` and apply steps 1-5 to it.

**Tell the user** in your final message to turn on **Highlight input cells** in the View's settings. The tools cannot set it.

**Check:** `tool:get_views` with `include: ["Pages", "Columns", "Values", "GridLayout"]`:
- Version is the first page.
- Metrics are on Columns, in input order, each with a `friendlyName`.
- Every metric has a width override.
- Formula metrics look visibly different from input metrics.

### B3 · Transaction List: Default View

**When:** you build a new Transaction List, or the user asks you to format one. A Transaction List holds the loaded transactional data (GL entries, invoices, orders, …): one item per row, its properties in columns. Users read it to check what was loaded and to find specific lines.

**Recognize it:** the block is a List, usually built with `skill:creating-transaction-lists`, with many rows and properties that are mostly loaded, not calculated. Its View is a **List View**: there are no metrics, the properties are the value fields. Use `tool:update_list_view_pivots` for pivots, never `tool:update_view_pivots`.

**Steps**

1. **Pivots** (`tool:update_list_view_pivots`; read the current layout with `tool:get_views` first, as the call replaces it wholesale):
   - `rowGroupings: []`: a flat List, one row per transaction. The server adds the main List row.
   - `pages`, in A4 order, each identified by its `listPropertyPath` (friendly names):
     - The **load period** property (e.g. `Load Month`, `Period`) whenever the List has one. Always put it in Pages.
     - Then the app's key business dimensions that the List carries as properties: Department, Country, Customer, Type, …
     - Do not page by every Dimension-typed property. Pick the 2-4 a user filters by in this app's use case; the others stay as columns.
   - `propertyColumns`: every property to show, in the order from step 2.
   - generally follow rule A3
2. **Property order** (the `propertyColumns` order from step 1, sent in the same call):
   - Loaded properties (no formula) first, on the left, in the order of the source file: identifiers and dates, then categorical fields, then amounts and quantities.
   - Calculated properties (with a formula) last, on the right.
   - Read which properties have a formula with `tool:search_metrics_and_lists` (`block_id_exact_match` + `show_details: true`).
3. **Widths** (`tool:update_view_grid_layout`):
   - Item-name column: A6, `rowHeaderColumnWidths` with the main List row's id from `tool:get_views` `Rows`.
   - Property columns: A7, `listColumnWidths` in `characters`, one entry per column (`valueFieldId` from `visiblePropertyColumns` in the `tool:update_list_view_pivots` response or `tool:get_views` `Columns`). The call replaces every column width, so send them all.
4. **Totals in the first row:**
   - `tool:update_view_aggregations`: on the main List row pivot, one `Sum` configuration for every number type property. Do not total rates, prices, percentages or numeric IDs; a total of those means nothing. Send `hiddenDimensionsAggregations: []`.
   - `tool:update_view_grid_layout`: `totalsInFirstRow: true`. It can go in the same call as step 3.
5. **Separate loaded from calculated** (`tool:update_view_formatting`; it replaces all overrides, so read `Formatting` with `tool:get_views` first and resend the ones to keep):
   - Calculated properties: background `Grey_10` on header and cells. Target with one path element `{ pivotFieldId: null, values: [{ kind: "Modality", id: <valueFieldId> }] }`, `area: Both`.
   - Totals row: bold. Target with one path element `{ pivotFieldId: <main List row id>, values: [{ kind: "Total" }] }`.
   - Send these in one call, together with any header overrides the View already has (A11).
   - The tools cannot set borders. The grey background alone marks where calculated columns start.
6. **Name and placement** (`tool:search_views` on the List, then `tool:get_views`):
   - Only the default View exists and it has no custom formatting or grid layout (`isDefaultView: true`, `hasCustomFormatting: false`, `hasCustomGridLayout: false`) → apply steps 1-5 to the **default View** itself. If the edit auto-creates a Draft, follow *Edit a Live View Safely* in `skill:designing-boards-and-views`.
   - Otherwise → `tool:create_view` named `<List name> - transactions` and apply steps 1-5 to it.

Skip A9 (List properties have no View-level friendly name) and A12 (a List has no empty cells to hide).

**Check:** `tool:get_views` with `include: ["Rows", "Columns", "Pages", "GridLayout", "Formatting"]`:
- The load period, if any, is a page.
- Loaded properties come before calculated ones, and every column has a width.
- Totals sit in the first row, only on amounts and quantities.
- Calculated columns have a grey background.
