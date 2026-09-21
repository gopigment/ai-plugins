---
name: forecasting-in-pigment
description: "Execution skill. Use when creating forecasts or choosing between growth-rate hypotheses and statistical forecasting functions."
metadata:
  skill_path: /skills/forecasting-in-pigment/SKILL.md
  base_directory: /skills/forecasting-in-pigment
---

# Forecasting in Pigment

**Mandatory prerequisite:** every forecast model MUST have a Version dimension with Actual and Forecast items. Load `skill:building-versions-and-planning-cycles` before creating any metrics.

For the correct time-offset formula pattern (`[SELECT: Month-12]` vs `PREVIOUS`), load `skill:choosing-formula-patterns`.

Two forecasting approaches with different setup effort, accuracy, and use cases.

## Choose Among the Forecasting Approaches

| Approach | When to use | Setup effort | Accuracy |
| --- | --- | --- | --- |
| **Growth-rate hypotheses** | Simple budgets, top-down targets, early-stage planning with limited history | Low | Low to medium (depends on assumption quality) |
| **Statistical formula functions** | Formula-driven forecasts with full modeler control over method and parameters | Medium | Medium to high (depends on data quality and function choice) |

### Approach 1 — Growth-Rate Hypotheses

Apply a growth rate to historical actuals or a base period.

```pigment
'Forecast Revenue' =
  IF(
    'Is_Plan',
    'Last Actual Revenue' * (1 + 'Growth Rate Input'),
    'Actual Revenue'
  )
```

**When to prefer:**

- Less than 12 months of historical data
- Business drivers well understood and stable
- Management sets targets top-down (e.g. "grow 10% next year")
- Planning process values simplicity over precision

**Limitations:** does not capture seasonality, trends, or non-linear patterns.

### Approach 2 — Statistical Formula Functions

Built-in forecasting functions in metric formulas; modeler controls every parameter.

**Function selection decision tree:**

1. Does the data have a **seasonal pattern** (repeating highs/lows at regular intervals)?
   - No → step 2
   - Yes → step 3
2. Does the data show a **trend** (consistently increasing or decreasing)?
   - No trend → `SIMPLE_EXPONENTIAL_SMOOTHING` (level only; Alpha 0-1)
   - Linear trend → `FORECAST_LINEAR` (Value, Dimension) or `DOUBLE_EXPONENTIAL_SMOOTHING` (Value, Dimension, Alpha, Beta)
3. Seasonal pattern type?
   - Additive or unsure → `FORECAST_ETS` (Value, Dimension, Seasonality) or `SEASONAL_LINEAR_REGRESSION` (Value, Dimension, SeasonalPeriod)
   - Strong multiplicative → `FORECAST_ETS` (closest built-in option)

**Function selection guide:**

| Function | Pattern detected | Best for | Key parameters |
| --- | --- | --- | --- |
| `FORECAST_LINEAR` | Linear trend only | Steady growth/decline without seasonality | Value, Dimension |
| `SIMPLE_EXPONENTIAL_SMOOTHING` | Level (no trend) | Stable series, noise reduction | Value, Dimension, Alpha (0-1) |
| `DOUBLE_EXPONENTIAL_SMOOTHING` | Level + linear trend | Trending series without seasonality | Value, Dimension, Alpha, Beta |
| `SEASONAL_LINEAR_REGRESSION` | Linear trend + seasonality | Monthly/quarterly data with repeating seasonal patterns | Value, Dimension, SeasonalPeriod |
| `FORECAST_ETS` | Level + trend + seasonality | Complex seasonal data (additive Holt-Winters) | Value, Dimension, Seasonality |

**Parameter guidance:**

- **Alpha** (smoothing): 0.7-0.9 reacts faster to recent changes; 0.1-0.3 smoother forecasts. Start with 0.3.
- **Beta** (trend smoothing): similar range as Alpha; lower values dampen trend changes.
- **Seasonality / SeasonalPeriod**: must match data's natural cycle. 12 for monthly/yearly, 4 for quarterly, 52 for weekly.
- All functions require sufficient historical data on the time dimension. At least 2 full seasonal cycles recommended for seasonal functions (24 months for monthly).

For detailed function syntax and examples, see `skill:using-formula-functions` (forecasting functions section).

**Example — monthly sales with yearly seasonality:**

```pigment
'Forecast Sales' =
  IF(
    'Is_Plan',
    FORECAST_ETS('Actual Sales', Month, 12),
    'Actual Sales'
  )
```

**When to prefer:** modeler wants full control, transparent formula, moderate data volume with well-understood patterns.

## Combining Approaches

Statistical forecast + manual overrides:

```pigment
'Final Forecast' =
  IFBLANK(
    'Manual Override',
    FORECAST_ETS('Actuals', Month, 12)
  )
```

## Best Practices

- Validate forecasts against holdout data when possible
- Match forecast granularity to planning granularity
- For seasonal functions, provide at least 2 full cycles of history
- Document approach and parameters for reproducibility
- Forecasts typically apply to Budget or Forecast versions, not Actuals; use `IF('Is_Plan', ...)` to guard forecast formulas
