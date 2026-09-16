---
name: creating-metrics-and-tables
description: "Execution skill. Load it before the first `create_metric` or `update_metric` call; never create or change a metric without loading it first. Use when creating metrics or tables, selecting metric data types, or finalizing metric structure."
metadata:
  skill_path: /skills/creating-metrics-and-tables/SKILL.md
  base_directory: /skills/creating-metrics-and-tables
---

# Creating Metrics and Tables

A metric is a sparse multidimensional grid: dimensions define the axes, each cell holds one value per combination. Metrics are input, formula-driven, or imported. Whether a given dataset belongs in a metric, a transaction list or a dimension is settled in `skill:understanding-pigment-modeling` before you get here.

## Workflow

1. **Resolve the dimension IDs.** Every structural dimension must already exist and be populated. Look them up with `tool:search_metrics_and_lists`; never pass a guessed ID. If a dimension is missing, create it first (`skill:creating-dimensions-and-hierarchies`).
2. **Pick the structure.** See *Choose the Grain* below. Only dimension lists and their subsets can be structural.
3. **Pick the data type.** Number for quantitative measures and the default for planning; Date for per-cell dates; Text for descriptive content; Dimension for a categorical lookup per cell; Boolean for per-cell flags.
4. **Pick the default aggregators** in the same call. A snapshot metric that rolls up with `Sum` across time reports twelve times its real value and raises no error. Both defaults are set through `tool:create_metric`; the rules are in `skill:aggregating-view-data`.
5. **Set the default format** in the same call where the metric is user-facing (`skill:formatting-and-highlighting`).
6. **Validate the formula first** with `tool:validate_formula` (`skill:writing-pigment-formulas`). Do this before creating the metric.
7. **Create** with `tool:create_metric`, including the formula in the same call. Batch independent metrics together. Input metrics have no formula; populate them with `tool:set_metric_input` or an import. To change a formula later, use `tool:update_metric`.
8. **Verify** with `tool:query_data` that the values are plausible before moving on.

Create source and input metrics before the calculated metrics that reference them, and tables after every metric they display. The full dependency chain is in `skill:building-a-full-application`.

## Restrict Structure to Dimension Lists

Only **dimension lists** (including subsets) can appear in a metric's structure. Transaction lists never can: aggregate them into the metric through a formula instead. To aggregate one over time, the list must already carry a Month property mapping its date to the calendar; add it before writing the formula (`skill:creating-transaction-lists`).

If a block will not go into a metric's structure, it is a transaction list, not a dimension list.

## Choose the Grain

**For an output metric**, the grain is what the spec says it reports on, plus Version wherever it holds plan data.

**For an assumption or driver metric** (growth %, attrition rate, price uplift, headcount ratio), the grain is the grid the user fills in, not the grid of the metric it feeds:

- **Add the time dimension the assumption varies on.** A driver used across a multi-year horizon needs one cell per year; leaving time off makes the value constant forever and unplannable.
- **Do not inherit the target's time grain.** A driver keeps its own grain even when it feeds a finer metric; the formula bridges the gap, for example a yearly rate onto months with `[BY CONSTANT: Month.Year]`. Match the modifier to the grain you chose, and reference the metric directly when the grains already agree.
- **Add Version** whenever the assumption differs between Budget, Forecast and Actual.
- **Add business axes only where the value genuinely differs** (per Account, per Cost Center). A single company-wide rate stays free of them.

A scalar assumption with no dimensions at all is almost always wrong in a planning model.

## Challenge Metrics Past Five Dimensions

Each additional dimension multiplies cell count and calculation cost. **Five is the recommended ceiling.** Past it, ask which axes are truly independent rather than derivable from others, whether the metric should split into several, and whether the granular detail belongs in a transaction list aggregated upward. Sparse grids that are mostly blank and performance complaints on wide metrics are the symptoms.

## Create Tables for Multi-Metric Reporting

A table groups two or more metrics that share dimensions so they display and compare together: a P&L stacked vertically, Budget against Actual against Variance, a dashboard where several KPIs share Month and Department. Component metrics can have different dimensions; Pigment aggregates and allocates to show them side by side.

Tables are for presentation and comparison, never for storing data. They sit at the end of the dependency chain, so every component metric must exist first. Use `tool:create_table` to create one, `tool:search_tables` to inspect it, and `tool:update_table` to change its name, description or metric composition. A table can also carry its own formulas over the metrics inside it.

**Any table line that divides one metric by another** -- a margin, a rate, a share of a total -- needs an aggregator of its own or it reads wrong in every total cell. Wire it in the same editing pass (`skill:aggregating-view-data`).

## Verify

- [ ] Every structural dimension exists as a dimension list and is populated
- [ ] Data type matches the cell content, and both default aggregators are set deliberately
- [ ] Source and input metrics exist before the calculated metrics referencing them
- [ ] Dimensionality is five or fewer, or the excess is justified in the spec
- [ ] Assumption metrics carry their own planning grain, not the target's
- [ ] Ratio and variance lines on tables have an Advanced Aggregator
- [ ] `tool:query_data` returns values of a plausible magnitude
