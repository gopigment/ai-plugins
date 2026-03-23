# Finding Relevant Views

**Always reuse existing Views** instead of creating new ones. Existing Views preserve valuable customizations (formatting, calculations, pivot configurations) and avoid clutter. An 80% match that needs minor edits is better than creating from scratch.

**Key concept**: Display mode (KPI, Grid, Chart) is **NOT stored in the View** — it is set at the **Widget level**. Focus on whether the View's **pivot configuration** supports your intended display mode, not on how the View was previously used.

---

## Step 1: Define Your Intent

Before searching, clarify what you need:

```
Block: [Metric/List/Table name]
Display Mode: [KPI/Grid/Chart type]
Breakdowns: [Dimensions for rows/columns]
Pages: [Required item filters]
Purpose: [What question does this answer?]
```

**Example:**

```
Block: Revenue Metric
Display Mode: Grid (Table)
Breakdowns: Rows=Product Line, Columns=Month
Pages: Year=2024, Version=Actuals,Budget
Purpose: Compare actual vs budget revenue by product line over time
```

## Step 2: Retrieve and Evaluate Existing Views

Retrieve all Views for the Block using `get_block_views`. **Pass your display intent** using the `display_intent` parameter:
- `Kpi` — for single-value KPI display
- `Grid` — for table/grid display with rows and columns
- `Chart` — for chart/data visualization (optionally specify `chart_type`: Bar, Line, Pie, Waterfall, Org, Geo)

When a `display_intent` is provided, results are sorted by compatibility and limited to the top relevant Views. Focus your evaluation on the top results.

Refer to the tool description for the full list of returned fields.

### Evaluation criteria (in priority order)

**1. Pivot Configuration Match (most important)**

Check if rows, columns, and pages align with your intent:

- **Perfect** — all Dimensions align exactly. Select and use as-is.
- **Close** — most Dimensions align, minor edits needed (swap rows/columns, remove or add 1 pivot, change Quarter to Month). Select and edit.
- **Partial** — some Dimensions align but significant differences. Consider if editing effort is worth it.
- **Poor** — completely different Dimensions. Skip.

**Extra pivots are free to ignore (except for KPIs)**: for Grid and Chart intents, if a View has all the Dimensions you need _plus_ extra ones, treat it as a **Perfect** match — you simply remove the extra pivots with minimal effort. However, for **KPI intent**, any extra row pivot is a problem since KPIs must have no row pivots. Extra column pivots are acceptable (they produce multiple KPI cards).

**Pivot matching examples:**

```
Intent: Revenue KPI (no breakdowns)

View A: Revenue (no pivots)                        → Perfect - use as-is
View B: Revenue by Region (Region in Rows)         → Close - remove Region pivot
View C: Revenue by Product and Month (Product in Rows, Month in Cols) → Partial - remove both pivots
```

```
Intent: Revenue by Product (rows) and Month (columns)

View A: Product (rows), Month (columns)   → Perfect - use as-is
View B: Month (rows), Product (columns)   → Close - swap rows/columns
View C: Product (rows), Quarter (columns) → Close - change Quarter to Month
View D: Region (rows), Month (columns)    → Partial - change Region to Product
View E: Product (rows), no columns        → Close - add Month to columns
```

**2. Display Mode Compatibility (inferred from configuration)**

- **KPI intent**: View must have no row pivots. Views with 1-2 Column pivots can work well (they produce multiple KPI cards). Multiple Page pivots are fine — pages act as filters and don't affect the KPI layout.
- **Grid intent**: View with Row/Column pivots. Page pivots are fine (they act as filters). **Note**: Grid ↔ Spreadsheet conversion is NOT supported — these are fundamentally different display modes.
- **Chart intent**: View with `chartTypes`, or 1-2 Dimensions suitable for visualization. Views with >2 pivots in Rows or Columns need trimming for readability. Views with a time Dimension (Month, Quarter) are good candidates for trend charts.

Hints in View data:

- `chartTypes` non-empty → likely Chart
- Several `rows`/`columns` populated → likely Grid
- No Row Pivots and no `chartTypes` → likely KPI
- View name containing "Chart", "Trend", "Line", "Bar" suggests chart usage.

**3. Board Usage**

- 5+ Boards: proven, well-maintained — strong signal
- 2-4 Boards: validated by multiple users
- 1 Board: may be specialized
- 0 Boards: possibly outdated or abandoned

**4. Recency**

- Last 30 days: actively maintained
- Last 3 months: likely still relevant
- Over 1 year: check carefully, may be outdated

**5. Formatting & Customization**

Views with high `formatOverridesCount` or `conditionalFormattingCount` are worth reusing to preserve that effort.

**6. Name & Description (tie-breaker only)**

Only use as a tie-breaker when other criteria are equal. Clear names ("Revenue by Product Line - Monthly Trend") help confirm intent. Poor names ("View 1", "Test") are a weak negative signal. Do not let naming alone drive selection decisions.

## Step 3: Select the Best View

```
Does pivot configuration match your intent?
├─ Perfect or Close match → Candidate
├─ Partial match → Continue evaluation, weigh editing effort
└─ Poor match → Skip

Among candidates, pick the most valuable one:
1. Prefer richer Views: a View with formatting, conditional formatting, or more
   customization is a better starting point than a bare-bones exact match.
   Removing an extra pivot is cheap; recreating formatting from scratch is not.
2. Board usage (prefer higher — signals quality and maintenance)
3. Recency (prefer more recent)

Note: the selected View's Dimensions must be a SUPERSET of the intended
Dimensions. A View missing a Dimension you need is harder to fix than a View
with extra Dimensions you can remove.
```

If evaluating the match or picking the best View gets too hard, refer to [scoring_relevant_views.md](./scoring_relevant_views.md) for a detailed scoring system.

## Step 4: Decide to reuse the best View or create one from scratch

- If the best View selected in Step 3 is a good match → use it
- If the best View is a close match but needs editing → duplicate it then edit it. Only start from scratch if the cost of editing is higher than starting from scratch (refer to "How to assess the cost of edits")
- If no good match → create a new View. Use the best existing View as inspiration for how to structure rows, columns, and pages — even a poor match can reveal useful conventions for the Block.
- If uncertain → ask the user.

**Convention for new Views**: time Dimensions (Year, Quarter, Month) should generally go in **Columns**, with entity Dimensions (Product, Region, Department, etc.) in **Rows**.

**Naming new Draft Views**: When creating a Draft View with edits from the original, give it a clear name and description that reflect the intended purpose (e.g., "Revenue by Region - Quarterly Trend"). Good naming helps users understand the Draft's intent when they review it.

---

## How to assess the cost of edits

These edits modify the View's **pivot configuration** to support your intended display mode.

| Pattern                     | Edit                                                            | Difficulty                             |
| --------------------------- | --------------------------------------------------------------- | -------------------------------------- |
| Grid → KPI                  | Remove all Row pivots (Column pivots may stay)                  | Easy                                   |
| KPI → Grid                  | Add Dimensions to Rows and/or Columns                           | Easy                                   |
| Swap layout                 | Move Row dims to Columns and vice versa                         | Easy                                   |
| Change chart type           | Bar → Line, Line → Area, etc.                                   | Easy                                   |
| Change granularity          | Replace one Dimension with another (Quarter → Month)            | Moderate                               |
| Adjust filters              | Add/remove Dimensions from Pages                                | Moderate                               |
| Reorder multiple Dimensions | Rearrange several dims across Rows/Columns/Pages                | Moderate                               |
| Grid ↔ Chart conversion     | Restructure pivots for a different display paradigm             | Hard                                   |
| Add complex calculations    | Add YoY%, variance, or custom formulas                          | Hard                                   |
| Complete restructuring      | All Dimensions different from intent                            | Hard — find another View or create new |
| Grid ↔ Spreadsheet          | Not supported — these are fundamentally different display modes | **Impossible**                         |

---

## Examples

### Example 1: Revenue KPI

**Intent**: Revenue Metric, KPI display, no breakdowns, Pages: Year=2024 Version=Actuals

| View Name         | Pivot Config | Boards | Last Updated |
| ----------------- | ------------ | ------ | ------------ |
| Revenue KPI       | None         | 8      | 15 days ago  |
| Revenue by Region | Rows=Region  | 6      | 10 days ago  |
| Total Revenue     | None         | 0      | 6 months ago |

**Decision**: Select "Revenue KPI" — perfect pivot config, high usage, recently updated.

**If it didn't exist**: Select "Revenue by Region" over "Total Revenue". Despite needing a pivot removal (Region from Rows), it has high Board usage (6 vs 0) and is more recent. A View named "by Region" works fine as a KPI once you remove the pivot and set KPI display mode at widget level.

### Example 2: Sales Grid

**Intent**: Sales Metric, Grid display, Rows=Product Line, Columns=Quarter

| View Name                    | Pivot Config               | Boards | Last Updated |
| ---------------------------- | -------------------------- | ------ | ------------ |
| Sales by Product - Quarterly | Rows=Product, Cols=Quarter | 2      | 60 days ago  |
| Sales by Product - Monthly   | Rows=Product, Cols=Month   | 5      | 20 days ago  |
| Product Performance          | Rows=Product, Cols=Region  | 1      | 90 days ago  |

**Decision**: Select "Sales by Product - Quarterly" — exact pivot match outweighs the higher usage of the Monthly variant. Alternative: "Sales by Product - Monthly" if flexible on Month vs Quarter.

---

## Special Cases

**No good match**: Ask the user whether to adapt the closest match or create a new View. Explain what the closest option is and what edits it would need.
Example communication:

```
"No existing View matches 'Revenue by Region (Grid)'.
The closest match is 'Revenue by Product (Grid)' which shows a similar structure but a different Dimension.
Would you like to:
1. Use 'Revenue by Product' and adjust the breakdown?
2. Create a new View for 'Revenue by Region'?"
```

**Multiple equally good matches**: Break ties with pivot config closeness > Board usage > recency > name clarity.

**Outdated View (>1 year)**: If the Block is still active and the View is still on Boards, it's probably fine. If it's on 0 Boards, treat with caution and ask the user.
