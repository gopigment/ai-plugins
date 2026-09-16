# Forecasting & Statistical Functions

---

## Choosing the Right Method

| Data pattern | Function |
| --- | --- |
| Stable, no trend, no seasonality | SIMPLE_EXPONENTIAL_SMOOTHING |
| Trend, no seasonality | DOUBLE_EXPONENTIAL_SMOOTHING or FORECAST_LINEAR |
| Seasonality with trend | FORECAST_ETS or SEASONAL_LINEAR_REGRESSION |
| Simple linear trend | FORECAST_LINEAR |

---

## FORECAST_ETS

`FORECAST_ETS(InputBlock, SeasonalityLength [, RankingDimension] [, Alpha] [, Beta] [, Gamma])`

- Additive Holt-Winters (triple exponential smoothing)
- SeasonalityLength: integer (e.g. 12 for monthly/yearly, 4 for quarterly)
- RankingDimension: optional if single Calendar dim; required otherwise
- Alpha (level) default 0.25; Beta (trend) default 0.1; Gamma (seasonal) default 0.25
- **If any of Alpha/Beta/Gamma specified, ALL 3 are required**
- Needs ≥ **2x seasonality** data points
- Fills forecast from first empty cell to end of time dimension
- Mid-series empty cells should be filled with plausible values (implementation does not optimize missing data)

```pigment
FORECAST_ETS('Sales', 12, Month)
FORECAST_ETS('Revenue', 4, Quarter, 0.3, 0.1, 0.1)
```

---

## FORECAST_LINEAR

`FORECAST_LINEAR(SourceMetric [, RankingDimension] [, AlternateMetric])`

- OLS linear regression (Y = ax + b)
- AlternateMetric: use as independent variable (X) for regression; forecast only computes where X is non-empty
- Fills from first empty cell forward
- Single data point → constant

```pigment
FORECAST_LINEAR('Monthly Revenue', Month)
FORECAST_LINEAR('Cost of Sales', Month, 'Sales per Month')
```

---

## SIMPLE_EXPONENTIAL_SMOOTHING

`SIMPLE_EXPONENTIAL_SMOOTHING(InputBlock [, RankingDimension [, Alpha]])`

- Alpha: default 0.5 (0–1)
- Before first non-blank → BLANK; after last → constant (last smoothed value)
- Mid-series blanks treated as 0
- Formula: sₜ = α·xₜ + (1−α)·sₜ₋₁

```pigment
SIMPLE_EXPONENTIAL_SMOOTHING('Price', Month)
SIMPLE_EXPONENTIAL_SMOOTHING('Sensor Reading', Day, 0.2)
```

---

## DOUBLE_EXPONENTIAL_SMOOTHING

`DOUBLE_EXPONENTIAL_SMOOTHING(InputBlock [, RankingDimension [, Alpha, Beta]])`

- Alpha: default 0.25; Beta: default 0.1 (both 0–1)
- After last value → linear extrapolation from last slope/intercept
- Mid-series blanks = 0

```pigment
DOUBLE_EXPONENTIAL_SMOOTHING('Monthly Growth', Month)
DOUBLE_EXPONENTIAL_SMOOTHING('User Count', Week, 0.3, 0.15)
```

---

## SEASONAL_LINEAR_REGRESSION

`SEASONAL_LINEAR_REGRESSION(InputBlock, SeasonalityLength [, RankingDimension])`

- Multiplicative decomposition: (A·x + B) × SeasonalityFactor(x)
- Seasonality must be integer > 1
- Needs ≥ 2x seasonality data points
- Mid-series blanks = 0

```pigment
SEASONAL_LINEAR_REGRESSION('Sales', 12, Month)
SEASONAL_LINEAR_REGRESSION('Revenue', 4, Quarter)
```

---

## Statistical Functions

```pigment
STANDARD_NORMAL_DISTRIBUTION(0)            // 0.3989 (PDF peak at z=0)
STANDARD_NORMAL_DISTRIBUTION(1.96)         // ~0.058

STANDARD_NORMAL_DISTRIBUTION_CUMULATIVE(0)      // 0.5
STANDARD_NORMAL_DISTRIBUTION_CUMULATIVE(1.96)   // 0.975
```

Both take Integer or Number input; return Number. BLANK input → BLANK.

---

## Parameter Tuning Guide

**Only consult this table when the user explicitly asks for a specific smoothing behavior (e.g. "make it more responsive") or gives you a value.** If they didn't, omit Alpha/Beta/Gamma entirely and let the function's documented default apply — do not infer a value from how volatile or stable the data looks.

| Parameter | 0.1–0.2 | 0.2–0.3 | 0.4–0.5 |
| --- | --- | --- | --- |
| **Alpha** (level) | Heavy smoothing | Moderate | Responsive |
| **Beta** (trend) | Smooth (most common) | Responsive | Very responsive |
| **Gamma** (seasonal) | Stable (most common) | Moderate | Changing seasonality |
