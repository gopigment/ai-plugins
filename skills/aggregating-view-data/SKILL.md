---
name: aggregating-view-data
description: "Execution skill. Use when a View needs totals or subtotals, when a value must roll up with something other than Sum (average, min, max, count, last), when a snapshot metric adds up across time instead of holding, or when a ratio, percentage, growth or variance metric shows a wrong total. Covers metric default aggregators, pivot versus hidden-dimension aggregation, the aggregator reference per value type, and Advanced Aggregators on Table Views."
metadata:
  skill_path: /skills/aggregating-view-data/SKILL.md
  base_directory: /skills/aggregating-view-data
---

# Aggregating View Data

Aggregation decides two things: what a **total cell** contains, and how a **visible cell** is computed when the metric carries dimensions the View does not display. It lives at two levels: **metric defaults**, set once on the metric, and **View overrides**, set per View with `tool:update_view_aggregations`.

Wrong aggregation never raises an error. It returns a plausible number, so check it whenever a metric is a snapshot, a rate, or a variance.

Read the current configuration with `tool:get_views` and `include: ["Rows", "Columns", "HiddenDimensionsAggregations"]`. Aggregation fields that fail validation are dropped silently, so re-read after every update.

## Decide Where the Aggregation Belongs

| Dimension placement | Visible | Configure with | Creates total cells |
| --- | --- | --- | --- |
| Rows | yes | pivot `aggregationConfigurations` | yes, subtotals and grand total |
| Columns | yes | pivot `aggregationConfigurations` | yes |
| Pages | no | `hiddenDimensionsAggregations` | no |
| Not in the View at all | no | `hiddenDimensionsAggregations` | no |

Hidden-dimension aggregation adds no row and no column. It only decides how the visible cells fold away the dimensions you are not showing: a metric on Country x Month displayed by Month alone still has to collapse Country. Setting a pivot aggregation on a page is the most common mistake and does nothing.

The mirror mistake is quieter. A dimension the request names is usually **visible**, so its aggregator belongs on that pivot; putting it in `hiddenDimensionsAggregations` leaves the total cells you were asked about computing their default. And a calendar dimension is temporal: `temporalDimensionsAggregator` governs Month, Quarter and Year, `otherDimensionsAggregator` never does.

`hiddenDimensionsAggregations` takes one entry per value field: `valueFieldId`, `temporalDimensionsAggregator`, `otherDimensionsAggregator`.

Whether totals are *displayed*, and where, belongs to the Layout panel (`tool:update_view_grid_layout`), not here. Aggregation decides what they compute.

## Set the Metric Defaults First

Every metric carries two default aggregators, set together through `tool:create_metric` / `tool:update_metric`: one for **temporal** dimensions, one for all the others. Get them right and most Views need no override; at View level, `Default` means "inherit the metric".

| Metric kind | Temporal default | Non-temporal default |
| --- | --- | --- |
| Additive flow (revenue, cost, units) | `Sum` | `Sum` |
| Snapshot (headcount, inventory, balance sheet) | `Last` | usually `Sum` |
| Reference or lookup (owner, status, category) | `First`, `Last`, or `Any` | same |
| Ratio, percentage, growth, variance | no correct simple default, see below | same |

Twelve monthly headcounts of 50 must read 50 for the year, not 600. Summing a snapshot across time is the classic wrong number.

Override at View level only when that View genuinely needs different behavior. A default that is wrong everywhere belongs on the metric, not patched View by View.

## Choose a Simple Aggregator

| Value type | Available aggregators |
| --- | --- |
| Decimal / Integer | `Sum`, `Avg`, `Min`, `Max`, `Median`, `Stdevp`, `Stdevs`, `First`, `Last`, `FirstNonBlank`, `LastNonBlank`, `FirstNonZero`, `LastNonZero`, the `Count` family, `Blank`, `Default` |
| Boolean | `Any`, `All`, `First`, `Last`, `FirstNonBlank`, `LastNonBlank`, the `Count` family, `Blank`, `Default` |
| Text | `TextList`, `First`, `Last`, `FirstNonBlank`, `LastNonBlank`, the `Count` family, `Blank`, `Default` |
| Date | `Min`, `Max`, `First`, `Last`, `FirstNonBlank`, `LastNonBlank`, the `Count` family, `Blank`, `Default` |
| Dimension, permission-like, access-right-like | `First`, `Last`, `FirstNonBlank`, `LastNonBlank`, the `Count` family, `Blank`, `Default` |

The `Count` family is `Count`, `CountAll`, `CountUnique`, `CountBlank`. `OnlyOneNotNull` and `BitAnd` also exist; use them only when the business meaning is explicit.

## Use Advanced Aggregators for Ratios and Growth

A rate, a percentage, a margin or a relative variance has **no correct simple aggregator**: the total of a ratio is not the sum of the ratios, nor their average. It needs an **Advanced Aggregator**, which recomputes the operation at every aggregated cell from two operand value fields.

Operations: `Ratio`, `Growth`, `Product`, `Sum`, `Difference`, `AbsoluteDifference`, `AbsoluteGrowth`.

Constraints, all hard:

- **Table Views only.** Views on a Metric or on a List support simple aggregation only.
- Exactly **two operands**, both numeric **metric** value fields, no self-reference.
- It is a **View configuration**: the aggregator itself creates nothing. The ratio row still needs its own metric — create it first, then aggregate its value field.
- **Not a display option.** `showValueAsConfiguration` (percent of another metric, percent of total, running total) restyles a value that already exists; it neither creates the value nor changes what a total computes. Any value that is one metric over another needs its own ratio metric plus a `Ratio` aggregator — never a show-value-as setting, never a bare formula left to aggregate itself.

### Detect a ratio-like metric when you add it

Run this whenever you add a metric to a Table View through `tool:update_view_values`. It is ratio-like if the **name** hints at it (`%`, `rate`, `ratio`, `margin`, `growth`, `variance`, `GM%`) or the **formula** divides or compares two metrics (`A / B`, `DIVIDE`, `(A - B) / B`). Use `tool:search_metrics_and_lists` with `show_details: true` to read the formula when unsure.

- **Ratio / percentage** -> operation `Ratio`, with **A** the numerator and **B** the denominator, exactly as in the formula (`GM% = Gross Margin / Revenue`).
- **Growth / relative variance** -> operation `Growth`, with **A** the minuend of `(A - B) / B` and **B** the base.

### Wire it in the same editing pass

1. Add the two operand metrics to the **Table block** if they are missing.
2. `tool:update_view_values` with **three** value fields: the ratio metric plus both operands. The operands may be `displayed: false`; keep them in `values` because the aggregator reads them.
3. `tool:update_view_aggregations` with `type: Advanced` on the **ratio** value field: `pivotAggregations` for the visible Rows and Columns, and `hiddenDimensionsAggregations` for the dimensions on Pages or not shown. Never leave the default `Sum`.

Repeat for **every** Table View that shows that metric. Nothing propagates from one View to another.

## Avoid the Common Mistakes

- Setting a pivot aggregation on a **page** dimension. Page-only dimensions use `hiddenDimensionsAggregations`.
- Expecting `hiddenDimensionsAggregations` to produce subtotal rows. It never creates a cell.
- `Sum` on a snapshot across time. Use `Last`.
- `Sum` or `Avg` on a rate or a percentage. Use Advanced Aggregator `Ratio` with the same numerator and denominator as the formula.
- `Sum` on a growth or relative variance. Use Advanced Aggregator `Growth` with the same A and B as the formula.
- Advanced aggregation on a **Metric View**. Table Views only.
- Faking a ratio by adding the same operand twice, or by putting an Advanced Aggregator on a **Calculated Item**. Create a real ratio metric with a Pigment formula, then aggregate its value field. Calculated Items are for derived dimension rows and columns.
- Overriding at View level when the metric defaults were already correct. Prefer `Default`.
