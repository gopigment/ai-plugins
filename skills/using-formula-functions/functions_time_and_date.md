# Time and Date Functions

> **No TODAY() function in Pigment.** Use a scheduled Metric-to-Metric import.

> **Converting a date to a dimension member (Month, Quarter, Year)?** Use `TIMEDIM(Date, TimeDimension)` from [functions_lookup_and_aggregation.md](./functions_lookup_and_aggregation.md).

> **Planning period bounds:** Do not use `DATE(YYYY, M, D)` for forecast horizon or switchover. Use Date-typed or Month-typed input metrics.

---

## Date Construction

### DATE

`DATE(Year, Month, Day)` → returns **Text** (UTC datetime string in DD/MM/YYYY format), not Date.

```pigment
DATE(2024, 3, 15)    // "15/03/2024 00:00:00 +00:00"
```

**Gotcha**: Returns Text, not Date. Display format can be controlled via Locale date time formatting. Use `DATEVALUE()` to parse text into Date type.

### DATEVALUE

`DATEVALUE(DateText, DateFormat)` → returns **Date**.

- Format specifiers: `y/yy/yyyy` (year), `M/MM/MMM/MMMM` (month, English only), `d/dd` (day)
- Missing month → defaults to January; missing day → defaults to 1st

```pigment
DATEVALUE("2024-03-15", "yyyy-MM-dd")
DATEVALUE("Mar 2024", "MMM yyyy")       // → 2024-03-01
```

---

## Date Extraction

| Function | Syntax | Returns | Notes |
| --- | --- | --- | --- |
| **DAY** | `DAY(Date)` | Integer 1–31 | |
| **MONTH** | `MONTH(Date)` | Integer 1–12 | |
| **YEAR** | `YEAR(Date)` | Integer | |
| **WEEKDAY** | `WEEKDAY(Date)` | Integer 0–6 | **0=Sunday**, 1=Mon, ..., 6=Sat; no return_type arg |

---

## Date Arithmetic

| Function | Syntax | Returns | Notes |
| --- | --- | --- | --- |
| **DAYS** | `DAYS(StartDate, EndDate)` | Integer | Difference (End − Start), not inclusive count |
| **MONTHDIF** | `MONTHDIF(StartDate, EndDate)` | Integer | Counts month boundary crossings; Jan 1→Jan 31 = 0; Jan 31→Feb 1 = 1 |
| **EDATE** | `EDATE(Date [, MonthOffset])` | Date | Adds/subtracts months; overshooting day → last day of target month |
| **EOMONTH** | `EOMONTH(Date [, MonthOffset])` | Date | Last day of month, optional offset |
| **STARTOFMONTH** | `STARTOFMONTH(Date [, MonthOffset])` | Date | First day of month, optional offset |

```pigment
DAYS(DATE(2024,3,1), DATE(2024,3,15))               // 14
EDATE(DATE(2024,1,31), 1)                            // Feb 29, 2024 (leap year)
EOMONTH(DATE(2024,3,15))                             // 2024-03-31
```

### Date expression shortcuts

```pigment
Month + 1                          // next month dimension item
(Month + 1).'End Date' - 1        // penultimate day of next month
```

---

## Period Functions (Require Pigment Calendar)

### INPERIOD

`INPERIOD(Date, TimeDimension)` → TRUE if date falls within period; **BLANK** (not FALSE) otherwise.

- TimeDimension: Day, Week, Month, Quarter, Half, Year
- Date must share the Time Dimension (typically via a stored Month property with formula `TIMEDIM(Date, Month)` on the TL)

```pigment
INPERIOD('Product'.'Expiration Date'[ADD: Month], Month)
IF(INPERIOD('Orders'.'Date', Quarter), 'Orders'.'Amount')
```

### DAYSINPERIOD

`DAYSINPERIOD(TimeDimension [, StartDate] [, EndDate] [, WorkingDays] [, Holidays])`

- Returns Integer per period
- TimeDimension: Week, Month, Quarter, Half, Year (**not Day**)
- WorkingDays: Boolean metric on Day of Week
- Holidays: Boolean metric on Day

```pigment
DAYSINPERIOD(Month)
DAYSINPERIOD(Month, 'Employee'.'Start Date', 'Employee'.'End Date')
DAYSINPERIOD(Month, DATE(2024,6,1), DATE(2024,12,31), 'Working Days', 'Holidays')
```

### PRORATA

`PRORATA(TimeDimension [, StartDate] [, EndDate] [, WorkingDays] [, Holidays])`

Returns proportion (0–1). **Start Date INCLUDED; End Date EXCLUDED.**

```pigment
PRORATA(Month)                                                  // 1 for each full month
PRORATA(Month, DATE(2020,6,15), DATE(2020,7,14))               // partial months
'Salary' * PRORATA(Month, 'Start Date', 'End Date' + 1)        // +1 for inclusive end
PRORATA(Month, 'Start Date')                                    // open-ended (no end)

// Resolving dates from a transaction list per entity:
PRORATA(Month, 'HRIS'.'Hire Date'[by firstnonblank: 'HRIS'.Employee], 'HRIS'.'Term Date'[by firstnonblank: 'HRIS'.Employee] + 1)
// Alternative using BY MIN for earliest date:
'HRIS'.'Hire Date'[BY MIN: 'HRIS'.Employee]
```

> **Proration vs Presence:**
>
> - Proration (fractional FTE): use raw dates → `PRORATA(Month, 'Hire Date', 'Term Date' + 1)` returns 0–1
> - Presence (0/1 headcount): use STARTOFMONTH → `PRORATA(Month, STARTOFMONTH('Hire Date'), STARTOFMONTH('Term Date' + 1))` returns 0 or 1
>   Do NOT use STARTOFMONTH when you need a fractional FTE value.

**Guidelines**:

- Add `+1` to End Date for inclusive end in FTE calculations
- Do NOT use `IFBLANK('Term Date', DATE(9999,12,31))`; use 2-arg form `PRORATA(TimeDim, StartDate)`
- Headcount pattern: `PRORATA(Month, STARTOFMONTH(start), STARTOFMONTH(end+1))` → 1 if present on last day

#### Presence / Boolean from PRORATA

```pigment
PRORATA(Day, 'Start Date', 'End Date' + 1)                     // numeric 0-1 on Day
ISDEFINED(PRORATA(Day, 'Start Date', 'End Date' + 1))          // Boolean TRUE/BLANK
IFDEFINED(PRORATA(Day, 'Start Date', 'End Date' + 1), 1)       // 1/BLANK flag
```

### NETWORKDAYS

`NETWORKDAYS(DateFrom, DateTo, WorkingDays, Holidays)`

- Returns Integer
- **DateTo is EXCLUDED** from count
- All weekdays are working days by default unless WorkingDays metric defines otherwise
- Requires native Pigment calendar

```pigment
NETWORKDAYS(Month.'Start Date', Month.'End Date', 'Working Days', 'Holidays')
```

---

## Period-to-Date

| Function | Syntax | Resets at | Equivalent |
| --- | --- | --- | --- |
| **MONTHTODATE** | `MONTHTODATE(Metric [, Agg])` | Month boundary | `CUMULATE(Metric, Day, Day.Month)` — requires **Day** dim |
| **QUARTERTODATE** | `QUARTERTODATE(Metric [, Agg])` | Quarter boundary | `CUMULATE(Metric, Day, Day.Quarter)` or `CUMULATE(Metric, Month, Month.Quarter)` — requires **Day or Month** dim |
| **YEARTODATE** | `YEARTODATE(Metric [, Agg])` | Year boundary | `CUMULATE(Metric, Day, Day.Year)` or `CUMULATE(Metric, Month, Month.Year)` or `CUMULATE(Metric, Quarter, Quarter.Year)` — requires **Day, Month, or Quarter** dim |

- Default Aggregation: SUM. Also: AVG, MIN, MAX
- Input Metric must be on a more granular time dimension than the reset period
- Alternatives exist via Show Value As / Calculated Items, but those cannot be referenced by other Metrics

```pigment
MONTHTODATE('Daily Sales')
QUARTERTODATE('Monthly Sales')
YEARTODATE('Monthly Sales')
```

---

## FILLFORWARD

`FILLFORWARD(Expression, Dimension [, ON clause])`

- Non-iterative single-pass fill of blanks with preceding non-blank value
- For backward fill: `FILLFORWARD(expr, dim ON RANK(dim.'Start Date', 0, DESC))`
- Fills entire Metric **before** applying IF conditions
- Cannot act on current Metric directly
- **Densifies sparse data** → performance impact
- Blanks remain if no prior non-blank exists

```pigment
FILLFORWARD('Price', Month)
FILLFORWARD('Exchange Rate', Date)
```

---

## SELECT vs PREVIOUS/PREVIOUSOF

| Case | Use | Example |
| --- | --- | --- |
| Time shift / comparison | SELECT | `'Actuals'[SELECT: Month-12]` |
| Same metric self-reference | PREVIOUS | `PREVIOUS(Month)` |
| Cross-metric iterative | PREVIOUSOF + cycle | `PREVIOUSOF('Ending Balance')` |

**Use PREVIOUS/PREVIOUSOF ONLY when current period's result depends on prior period's result.**

For everything else: running totals → CUMULATE; fill blanks → FILLFORWARD; MoM display → Show Value As; prior year same month → `[SELECT: Month-12]`.

---

## Inclusive/Exclusive Summary

| Function | Start Date | End Date |
| --- | --- | --- |
| DAYS | Reference | Reference |
| MONTHDIF | Reference | Reference |
| NETWORKDAYS | Included | **Excluded** |
| PRORATA | Included | **Excluded** |

---

## Common Patterns

```pigment
// Year-over-Year
'Revenue' - 'Revenue'[SELECT: Month-12]

// Transaction aggregation by period — use stored Month property on the TL
'Transactions'.'Amount'[BY: 'Transactions'.'Month']
// Month property on the TL has formula: TIMEDIM('Transactions'.'Date', Month)

// Business days in month
NETWORKDAYS(Month.'Start Date', Month.'End Date', 'Working Days', 'Holidays')

// Fill missing prices
FILLFORWARD('Product Price', Month)

// Plan from PY + growth
'Actual Revenue'[SELECT: Month-12] * (1 + 'Monthly Growth Rate')
```
