---
name: using-formula-functions
description: "Execution skill. Use when drafting or debugging a Pigment formula that calls built-in functions. Covers all function signatures, parameters, return types, and usage examples grouped by category."
metadata:
  skill_path: /skills/using-formula-functions/SKILL.md
  base_directory: /skills/using-formula-functions
  includes:
    - "*.md"
---

# Pigment Formula Functions — Quick Reference

Pigment has its own proprietary formula language. These are NOT Excel/SQL/Python functions. Use only Pigment syntax.

## How to Use

Find the function below, then open the matching sub-file for full signatures, parameters, return types, gotchas, and examples. Match function name exactly (case-sensitive); pass parameters in documented order.

## Critical Global Rules

- **BLANK ≠ FALSE**: BLANK is not stored (sparse); FALSE is stored (dense)
- **Division by zero/BLANK** → BLANK automatically; no guard needed
- **AND / OR / NOT**: infix/prefix operators (`A AND B`, `NOT cond`), not function calls
- **Sparsity preference**: ISDEFINED > IFDEFINED > IFBLANK > IF(ISBLANK()). TRUE/FALSE are bare constants (no parens); FALSE densifies
- **No TODAY() function**: use scheduled Metric-to-Metric import
- **DATE() returns Text, not Date**; use DATEVALUE() for Text→Date

---

## Logical — [functions_logical.md](./functions_logical.md)

| Function | Syntax |
| --- | --- |
| IF | `IF(Cond, ValTrue [, ValFalse])` — 3rd arg optional (BLANK if omitted); both branches always evaluated |
| SWITCH | `SWITCH(Expr, Case1, Res1 [, ...] [, Default])` — cases must be scalar |
| AND | `cond1 AND cond2` — infix; BLANK propagates |
| OR | `cond1 OR cond2` — infix; TRUE short-circuits BLANK |
| NOT | `NOT cond` — prefix; NOT(BLANK) ≠ TRUE |
| TRUE / FALSE | Bare constants; FALSE densifies |
| ISDEFINED | `ISDEFINED(Val)` → TRUE/BLANK — **sparse** |
| IFDEFINED | `IFDEFINED(Block, IfDef [, IfBlank])` — **sparse** |
| IFBLANK | `IFBLANK(Metric, FillVal)` — fills blanks; constant fill densifies |
| ISBLANK | `ISBLANK(Val)` → TRUE/FALSE — **densifies**; avoid |
| ISNOTBLANK | `ISNOTBLANK(Val)` → TRUE/FALSE — **densifies**; avoid |
| ANYOF | `ANYOF(BoolBlock)` → scalar TRUE if any |
| ALLOF | `ALLOF(BoolBlock)` → scalar TRUE if all |
| IN | `Block IN (Item1, Item2)` or `Block IN (lo : hi)` — infix |

---

## Numeric — [functions_numeric.md](./functions_numeric.md)

| Function | Syntax | Notes |
| --- | --- | --- |
| ABS | `ABS(N)` | |
| SIGN | `SIGN(N)` | 1, 0, -1 |
| SQRT | `SQRT(N)` | Negative → BLANK |
| EXP | `EXP(N)` | e^N |
| LN | `LN(N)` | ≤0 → BLANK |
| LOG | `LOG(N)` | Base 10 only; negative → BLANK |
| SIN / COS | `SIN(N)` / `COS(N)` | Radians |
| MIN / MAX | `MIN(V1, V2, ...)` | Also works with Date |
| MOD | `MOD(N, Div)` | Remainder |
| QUOTIENT | `QUOTIENT(N, Div)` | Integer part |
| POWER | `POWER(N, P)` | |
| ROUND | `ROUND(N [, Digits])` | 0–14 digits; out of range → BLANK |
| ROUNDUP | `ROUNDUP(N [, Digits])` | Away from zero |
| ROUNDDOWN | `ROUNDDOWN(N [, Digits])` | Toward zero |
| TRUNC | `TRUNC(N [, KeptDigits])` | KeptDigits Integer only (not Metric); negative → BLANK |
| CEILING | `CEILING(N)` | Toward +∞; `CEILING(-2.36)` → -2 |
| FLOOR | `FLOOR(N)` | Toward -∞; `FLOOR(-2.36)` → -3 |
| CUMULATE | `CUMULATE(N, Dim [, GroupDim] [, Agg])` | Running total; supports ON for custom order |
| DECUMULATE | `DECUMULATE(N, Dim [, GroupDim])` | Inverse of CUMULATE |
| MOVINGSUM | `MOVINGSUM(Input, WinSize [, EndOff] [, Dim])` | Blanks ignored; integer window faster |
| MOVINGAVERAGE | `MOVINGAVERAGE(Input, WinSize [, EndOff] [, Dim])` | Blanks excluded from sum AND count |
| RANK | `RANK(Block [, Group] [, Dir] [, Ties])` | Skip Group with `""` / `0` / `false` |
| SPREAD | `SPREAD(Block, RankDim, N [, StartIdx])` | Equal split (1/N), not proportional |

---

## Text — [functions_text.md](./functions_text.md)

| Function | Syntax | Notes |
| --- | --- | --- |
| TEXT | `TEXT(Number)` | Number → Text |
| VALUE / NUMBER | `VALUE(Text)` | Text → Number; invalid → BLANK |
| LEN | `LEN(Text)` | Character count |
| LEFT | `LEFT(Text, Count)` | Negative → BLANK; overflow → all |
| MID | `MID(Text, Start, Count)` | 1-based; position 0 → BLANK |
| RIGHT | `RIGHT(Text, Count)` | |
| LOWER / UPPER | `LOWER(Text)` / `UPPER(Text)` | |
| PROPER | `PROPER(Text)` | Capitalize after non-alpha chars |
| TRIM | `TRIM(Text)` | Strip + collapse spaces |
| CONTAINS | `CONTAINS(Find, Search [, Start] [, CaseSens])` | **Substring first, haystack second**; default case insensitive |
| STARTSWITH | `STARTSWITH(Prefix, Text [, CaseSens])` | |
| ENDSWITH | `ENDSWITH(Suffix, Text [, CaseSens])` | |
| FIND | `FIND(Find, Search [, Start] [, CaseSens])` | 1-based; not found → BLANK |
| SUBSTITUTE | `SUBSTITUTE(Text, Old, New [, OccN])` | **Case sensitive** (only text fn that is) |
| & | `text1 & text2` | Requires TEXT() for non-text operands |

---

## Time & Date — [functions_time_and_date.md](./functions_time_and_date.md)

| Function | Syntax | Notes |
| --- | --- | --- |
| DATE | `DATE(Y, M, D)` | Returns **Text**, not Date |
| DATEVALUE | `DATEVALUE(Text, Format)` | Returns Date; English month names only |
| DAY / MONTH / YEAR | `DAY(Date)` etc. | Integer extraction |
| WEEKDAY | `WEEKDAY(Date)` | 0=Sun, 1=Mon, ..., 6=Sat |
| DAYS | `DAYS(Start, End)` | Difference (End − Start) |
| MONTHDIF | `MONTHDIF(Start, End)` | Month boundary crossings |
| EDATE | `EDATE(Date [, MonthOff])` | Date ± N months |
| EOMONTH | `EOMONTH(Date [, MonthOff])` | Last day of month |
| STARTOFMONTH | `STARTOFMONTH(Date [, MonthOff])` | First day of month |
| INPERIOD | `INPERIOD(Date, TimeDim)` | TRUE/BLANK |
| DAYSINPERIOD | `DAYSINPERIOD(TimeDim [, Start] [, End] [, WorkDays] [, Holidays])` | Integer; no Day dim; **Start incl, End excl** (+1 for inclusive) |
| PRORATA | `PRORATA(TimeDim [, Start] [, End] [, WorkDays] [, Holidays])` | 0–1; **Start incl, End excl** (+1 for inclusive) |
| NETWORKDAYS | `NETWORKDAYS(From, To [, WorkDays] [, Holidays])` | **End excluded** (+1 for inclusive); no weekend filter without WorkDays |
| MONTHTODATE | `MONTHTODATE(Metric [, Agg])` | Resets each month |
| QUARTERTODATE | `QUARTERTODATE(Metric [, Agg])` | Resets each quarter |
| YEARTODATE | `YEARTODATE(Metric [, Agg])` | Resets each year |
| FILLFORWARD | `FILLFORWARD(Expr, Dim [, ON])` | Non-iterative; densifies |

---

## Lookup & Aggregation — [functions_lookup_and_aggregation.md](./functions_lookup_and_aggregation.md)

### Lookup

| Function | Syntax | Notes |
| --- | --- | --- |
| ITEM | `ITEM(Val, Dim.'UniqueProp')` | Unique property only; faster |
| MATCH | `MATCH(Val, Expr)` | Any property (first match); **case-sensitive** |
| SHIFT | `SHIFT(Block, Offset)` | Returns Dimension item; out of range → BLANK |
| TIMEDIM | `TIMEDIM(Date, TimeDim)` | Date → Calendar element; fiscal-year aware |

### Aggregation

All return **scalar** (no dimensions). For dimensional aggregation use modifiers.

| Function | Syntax | Input | Blanks |
| --- | --- | --- | --- |
| SUMOF | `SUMOF(Block)` | Number | Ignored |
| AVGOF | `AVGOF(Block)` | Number | Ignored |
| MINOF | `MINOF(Block)` | Number/Date | Ignored |
| MAXOF | `MAXOF(Block)` | Number/Date | Ignored |
| COUNTOF | `COUNTOF(Block)` | Any | Non-blank only |
| COUNTALLOF | `COUNTALLOF(Block)` | Any | All incl. blank |
| COUNTBLANKOF | `COUNTBLANKOF(Block)` | Any | Blank only |
| COUNTUNIQUEOF | `COUNTUNIQUEOF(Block)` | Any | Unique non-blank |

---

## Finance — [functions_finance.md](./functions_finance.md)

Parameter order differs from Excel. Rate is 1st (NPV/XNPV). Guess is 2nd (IRR/XIRR).

| Function | Syntax | Notes |
| --- | --- | --- |
| NPV | `NPV(Rate, CFs [, AllCells] [, RankDim])` | Periodic; excludes initial investment |
| XNPV | `XNPV(Rate, CFs [, AllCells] [, RankDim] [, DaysUsed])` | Irregular dates; includes all |
| IRR | `IRR(CFs [, Guess] [, AllCells] [, RankDim])` | Needs mixed-sign CFs |
| XIRR | `XIRR(CFs [, Guess] [, AllCells] [, RankDim] [, DaysUsed])` | Irregular dates |

---

## Forecasting — [functions_forecasting.md](./functions_forecasting.md)

| Function | Syntax | Best for |
| --- | --- | --- |
| FORECAST_ETS | `FORECAST_ETS(Input, Season [, RankDim] [, α, β, γ])` | Trend + seasonality; if any α/β/γ given, all 3 required |
| FORECAST_LINEAR | `FORECAST_LINEAR(Metric [, RankDim] [, AltMetric])` | Simple linear trend |
| SIMPLE_EXPONENTIAL_SMOOTHING | `SIMPLE_EXPONENTIAL_SMOOTHING(Input [, RankDim [, α]])` | No trend, no seasonality |
| DOUBLE_EXPONENTIAL_SMOOTHING | `DOUBLE_EXPONENTIAL_SMOOTHING(Input [, RankDim [, α, β]])` | Trend, no seasonality |
| SEASONAL_LINEAR_REGRESSION | `SEASONAL_LINEAR_REGRESSION(Input, Season [, RankDim])` | Seasonality with trend |
| STANDARD_NORMAL_DISTRIBUTION | `STANDARD_NORMAL_DISTRIBUTION(z)` | PDF (z-score) |
| STANDARD_NORMAL_DISTRIBUTION_CUMULATIVE | `STANDARD_NORMAL_DISTRIBUTION_CUMULATIVE(z)` | CDF |

---

## Security

`ACCESSRIGHTS(ReadBoolean, WriteBoolean)` — use **BLANK** (not FALSE) for deny to preserve sparsity. See `skill:securing-with-access-rights` for full patterns and notes.

**Mandatory pattern** — always wrap in `IFDEFINED('Users roles', ...)`:

```pigment
IFDEFINED('Users roles', ACCESSRIGHTS(ISDEFINED(User), User.'Role' = SET_Admin_Role))
```

---

## Iterative — [functions_iterative_calculation.md](./functions_iterative_calculation.md)

| Function | Syntax | Notes |
| --- | --- | --- |
| PREVIOUS | `PREVIOUS(IterDim [, Offset])` | Single-block self-reference; max 10k items |
| PREVIOUSOF | `PREVIOUSOF(Metric [, Offset])` | Multi-block; requires cycle setup first |
| PREVIOUSBASE | Deprecated (Q2 2026) | Use PREVIOUSOF |

Only use for true circular dependencies. For time shifts use `[SELECT: Dim-N]`.
