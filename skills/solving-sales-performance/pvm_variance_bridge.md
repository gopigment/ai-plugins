# PVM Variance Bridge (Price / Volume / Mix)

A three-factor decomposition that explains year-over-year (or period-over-period) revenue change as the sum of Price Effect, Volume Effect, and Mix Effect.

---

## Three-Factor Decomposition

```
Price Effect + Volume Effect + Mix Effect = YoY Revenue Change
```

Both conventions are stated below. Confirm with the user which one they expect; Laspeyres (standard) is the default.

### Standard Laspeyres Convention

| Factor | Formula |
| --- | --- |
| **Price Effect** | `(P_cy − P_ly) × V_ly` |
| **Volume Effect** | `(V_cy − V_ly) × P_ly` |
| **Mix Effect** | `ΔRevenue − Price Effect − Volume Effect` (residual) |

Where:

- `P_cy` = current year unit price
- `P_ly` = last year (prior year) unit price
- `V_cy` = current year volume
- `V_ly` = last year volume
- `ΔRevenue` = Revenue_cy − Revenue_ly

Mix Effect captures the interaction between price and volume changes, plus any compositional shifts across products or segments.

---

## Dimensions

Ask the user which dimensions to include. The decomposition must be computed at the most granular intersection available, then aggregated up. Computing at an aggregate level first will produce incorrect mix effects.

---

## Pigment Idioms

### Revenue

```
'Sales TL'.'Revenue'[BY SUM: 'Sales TL'.'Month', 'Sales TL'.'Product', 'Sales TL'.'Country']
```

### Volume

Same pattern with the Volume column:

```
'Sales TL'.'Volume'[BY SUM: 'Sales TL'.'Month', 'Sales TL'.'Product', 'Sales TL'.'Country']
```

### Unit Price

Derived from revenue and volume:

```
Revenue / Volume
```

Or use `[BY AVG:]` if unit price is stored directly on the transaction list.

### Prior Year Values

Use `[SELECT: Year - 1]` or `[SELECT: Month - 12]` to reference the prior period. Do **NOT** use `PREVIOUS` — it references the prior item in the dimension order, which is not necessarily the prior year.

Examples:

- `Revenue[SELECT: Year - 1]` — revenue for the same product/country in the prior year
- `Volume[SELECT: Month - 12]` — volume 12 months ago

### Mix Denominator

Total volume without product breakout (for computing product mix shares):

```
'Volume'[REMOVE SUM: 'Product']
```

This gives the total volume across all products for each Country × Month, used to compute each product's share of total volume.

---

## Building the Metrics

1. **Revenue** — aggregated from transaction list (or existing metric).
2. **Volume** — aggregated from transaction list (or existing metric).
3. **Unit Price** — `Revenue / Volume`.
4. **Price Effect** — `(Unit Price − Unit Price[SELECT: Year - 1]) × Volume[SELECT: Year - 1]`
5. **Volume Effect** — `(Volume − Volume[SELECT: Year - 1]) × Unit Price[SELECT: Year - 1]`
6. **Mix Effect** — `(Revenue − Revenue[SELECT: Year - 1]) − 'Price Effect' − 'Volume Effect'`

---

## Reconciliation Check (Mandatory)

Create a check metric:

```
Revenue − Revenue[SELECT: Year - 1] − 'Price Effect' − 'Volume Effect' − 'Mix Effect'
```

This must equal **0** (or near-0 for floating-point rounding). Verify with `tool:fetch_metric_data` at the granular level and at totals.

If the check metric is non-zero beyond rounding tolerance, the decomposition has a bug. Debug by verifying each component at a single Product × Country × Month cell before looking at aggregates.
