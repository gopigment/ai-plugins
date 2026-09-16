# Numeric Functions

---

## Basic Math

| Function | Syntax | Returns | Notes |
| --- | --- | --- | --- |
| **ABS** | `ABS(Number)` | Absolute value | |
| **SIGN** | `SIGN(Number)` | 1, 0, or -1 | |
| **SQRT** | `SQRT(Number)` | Square root | Negative → BLANK |
| **EXP** | `EXP(Power)` | e^Power | Inverse of LN |
| **LN** | `LN(Number)` | Natural log (base e) | ≤0 → BLANK |
| **LOG** | `LOG(Number)` | Log base 10 **only** | Negative → BLANK; no custom base (unlike Excel) |
| **SIN** | `SIN(Number)` | Sine | Input in **radians** |
| **COS** | `COS(Number)` | Cosine | Input in **radians** |
| **MIN** | `MIN(V1, V2, ...)` | Smallest value | Also works with **Date** values |
| **MAX** | `MAX(V1, V2, ...)` | Largest value | Also works with **Date** values; `MAX('Qty', 0)` replaces negatives |
| **MOD** | `MOD(Number, Divisor)` | Remainder | `MOD(2, 5)` → 2 |
| **QUOTIENT** | `QUOTIENT(Number, Divisor)` | Integer quotient | Truncates toward zero |
| **POWER** | `POWER(Number, Power)` | Number^Power | `POWER(n, 0)` → 1 |

---

## Rounding

| Function | Syntax | Behavior | Negative example |
| --- | --- | --- | --- |
| **ROUND** | `ROUND(Number [, Digits])` | Nearest value | Standard rounding |
| **ROUNDUP** | `ROUNDUP(Number [, Digits])` | Away from zero | `ROUNDUP(-2.1)` → -3 |
| **ROUNDDOWN** | `ROUNDDOWN(Number [, Digits])` | Toward zero | `ROUNDDOWN(-2.9)` → -2 |
| **TRUNC** | `TRUNC(Number [, KeptDigits])` | Truncate decimals | KeptDigits is **Integer only** (not Metric); negative → BLANK |
| **CEILING** | `CEILING(Number)` | Toward +∞ (integer) | `CEILING(-2.36)` → **-2** |
| **FLOOR** | `FLOOR(Number)` | Toward -∞ (integer) | `FLOOR(-2.36)` → **-3** |

- Digits range: 0–14 for ROUND/ROUNDUP/ROUNDDOWN. Out of range or negative → BLANK
- Digits defaults to 0 for ROUND/ROUNDUP/ROUNDDOWN; can be Integer or Metric
- TRUNC KeptDigits is Integer only (not Metric); negative → BLANK; no documented upper limit
- TRUNC with no 2nd arg converts Number → Integer type
- **CEILING/FLOOR differ from ROUNDUP/ROUNDDOWN for negatives** (mathematical direction vs toward/away from zero)

---

## CUMULATE

`CUMULATE(Number, Cumulated Dimension [, Group Dimension] [, Aggregation])`

- **Aggregation**: SUM (default), AVG, MIN, MAX
- **Group Dimension**: resets accumulation (e.g. `Month.Year` resets yearly)
- **ON operator**: custom order: `CUMULATE(metric, 'Dim' ON 'Dim'.Priority)`

```pigment
CUMULATE('Monthly Sales', Month)                     // running total
CUMULATE('Quantity Sold', Month, Month.Year)         // reset each year
CUMULATE('Score', Employee ON Employee.'Hire Date')  // custom order
```

## DECUMULATE

`DECUMULATE(Number, Decumulated Dimension [, Group Dimension])`

Inverse of CUMULATE: difference between current and previous item.

```pigment
DECUMULATE('YTD Revenue', Month)
DECUMULATE(CUMULATE(x, dim), dim)   // returns original x
```

**ON operator** also supported for custom order.

---

## MOVINGSUM

`MOVINGSUM(Input, WindowSize [, EndOffset] [, Dimension])`

- **Input**: Integer or Number. Returns same type as Input.
- **WindowSize**: integer ≥1 or Integer Metric (integer faster for performance)
- **EndOffset**: default 0 (window ends at current); positive = future. Can be integer or Integer Metric.
- **Dimension**: defaults to time dimension if Input has exactly one
- **Blanks ignored** (not treated as 0); window shrinks at edges

```pigment
MOVINGSUM('Sales', 3)                    // 3-period rolling sum
MOVINGSUM('Revenue', 12, -1)             // 12-period window, offset by -1
MOVINGSUM('Sales', DynamicWindow, 0, Month)  // dynamic window from Metric
```

## MOVINGAVERAGE

`MOVINGAVERAGE(Input, WindowSize [, EndOffset] [, Dimension])`

Same parameters as MOVINGSUM. Returns **Number**.

- Blanks excluded from **both sum and count**: avg of (BLANK, 2) = 2; avg of (0, 2) = 1

```pigment
MOVINGAVERAGE('Sales', 3)
MOVINGAVERAGE('Revenue', 12, -1)
```

---

## RANK

`RANK(SourceBlock [, Group] [, Direction] [, Ties])`

- **SourceBlock**: number, integer, date, text, or Dimension. Text ranks alphabetically.
- **Group**: dimension that **resets** ranking (not the ranked dimension). Skip with `0`, `false`, or `""`. Can also be a **Metric**: items with same Metric value share a ranking group.
- **Direction**: ASC (smallest=1) or DESC (largest=1)
- **Ties**: MINIMUM (default), MAXIMUM, SEQUENTIAL, AVERAGE
- Returns **Integer** (1-based; no rank 0)

```pigment
RANK('Revenue', Product, DESC)                  // highest revenue = rank 1
RANK('Salary', Employee, ASC)                   // lowest salary = rank 1
RANK(Account.TAM, "", ASC)                      // skip Group with ""
RANK(Account.TAM, Region, ASC)                  // rank within each Region
RANK(metric, Month.Name & Team.Name)            // multi-dimension reset
RANK('Metric', 0, DESC, SEQUENTIAL)             // skip Group, sequential ties
RANK('Revenue', 'Product Category', DESC)       // Group as Metric: pass the metric directly, unmodified
```

**Critical**: Do NOT use the block's own dimension as Group; gives rank 1 everywhere.
`RANK(Month)` follows calendar order (fiscal-year aware).
**Group as Metric** means passing an existing categorical/dimension-typed metric as-is.
**Only add Ties when the requirement asks for specific tie-breaking** — omit it to get the MINIMUM default.

---

## SPREAD

`SPREAD(SourceBlock, RankingDimension, SpreadNumber [, StartingIndex])`

- **SpreadNumber**: integer or Metric; how many items to split across
- **StartingIndex**: List Property or Metric defining where spread begins
- **Equal distribution** (1/N each), not proportional
- Requires **ordered** dimension

```pigment
SPREAD('Quantity Sold', Month, 3)
SPREAD(10[BY: SET_Spread_Start_Month], Month, 6)
SPREAD('Annual Budget', Month, 12, Employee.'Start Month')
```

---

## Common Patterns

```pigment
CUMULATE('Monthly Revenue', Month, Month.Year)    // Year-to-Date
'Revenue' - 'Revenue'[SELECT: Month-1]            // Month-over-Month
MOVINGAVERAGE('Sales', 3)                         // 3-Month Moving Average
IF(RANK('Revenue', Product, DESC) <= 10, 'Revenue') // Top 10
MAX('Quantity', 0)                                // Floor at zero
```
