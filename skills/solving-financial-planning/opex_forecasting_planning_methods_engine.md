# OPEX Planning – Forecasting Methods & Engine

## Purpose

How a **driver-based OPEX forecasting engine** works with several predefined methods (Prior Year, Last X months average, % of Revenue, $ per Headcount, Manual input) and one method per planning line. Covers: method selection (dimension + input metric + SWITCH), five calculation methods, global modifiers (YoY and blank handling), row lifecycle and validation, and Actual vs Plan windows.

**When to use:** building or extending an OPEX app with a central forecast metric (CALC_Forecast) switching on method ID; adding a new forecasting method; understanding how parameters, YoY, and Actual/Plan windows interact.

**When not to use:** purely manual OPEX with no driver-based methods. For app layout, PULL layer, folder structure, and naming, see **OPEX Planning – Application Architecture & Patterns**.

---

## Core Concepts

- **Method dimension**: Forecasting Method list with ID (Integer for SWITCH), Need Input in Parameter? (Boolean), Parameter Unit (Text).
- **SWITCH engine**: CALC_Forecast = SWITCH(method ID, 1→PY, 2→Avg, 3→%Rev, 4→$HC, 5→Manual) * modifier [EXCLUDE: Actual].
- **YoY modifier**: CALC_YoY_2_fixblanks: global multiplier applied after SWITCH for YoY uplift/decay and blank preservation.
- **Override-first**: After SWITCH result, override layer (IFBLANK(INP_Override, CALC_Forecast)) lets users adjust individual cells.
- **Window flags**: PUSH_DH_View_Load_Actual/Plan (Boolean per Version × Month). Engine EXCLUDEs actual months; methods FILTER to plan months.

---

## Pipeline

```
Method selection (INP_Forecasting_Method per line)
  → Method-specific CALC_* (one per method, IF-guarded)
  → SWITCH dispatches to selected method
  → × CALC_YoY_2_fixblanks (YoY + blank modifier)
  → [EXCLUDE: Actual months]
  → IFBLANK(INP_Override, CALC_Forecast) (override layer)
  → OUT_FC
```

---

## Patterns

### Method selection

**Forecasting Method dimension**: encodes each method with: ID (Integer, used in SWITCH), Need Input in Parameter? (Boolean), Parameter Unit (e.g. "months", "%", "$/HC"). Methods: PY Values (1), Last X months Avg (2), % of Revenue (3), $ per Headcount (4), Manual Input (5).

**INP_Forecasting_Method**: type: Dimension → Forecasting Method. Dims: Department, PnL_Account, Version, Entity, Line. Holds user-selected method per line. Optional prefill: when FIL_Has_Actuals and SET_Fill_combo_withactuals?, fill from SET_FCMethod_by_PnlAccount[BY: Line."1"].

**CDFM_ID** = INP_Forecasting_Method.ID (numeric code for SWITCH).

**Central engine: CALC_Forecast**

```
(
  SWITCH(
    'INP_Forecasting_Method'.ID,
    1, CALC_PYValues,
    2, CALC_LastXMonthsAVG,
    3, CALC_%ofRevenue,
    4, CALC_$perHeadcount,
    5, CALC_Manual_Input
  )
  * CALC_YoY_2_fixblanks
)
[EXCLUDE: 'PUSH_DH_View_Load_Actual']
```

**Override-first staging (mandatory):** after SWITCH result, apply override layer:

```
IFBLANK('INP_Override', CALC_Forecast)
```

Override input has same structure (Department, PnL_Account, Version, Entity, Month, Line). Either enable `overrideEnabled` on output metric or create dedicated override input metric. Applies uniformly to any method.

### The five calculation methods

All share planning dimensions (Department, PnL_Account, Version, Entity, Month, Line). Each wrapped in IF(INP_Forecasting_Method = Forecasting Method."MethodName", …). Note: Forecasting Method items are stable structural items; hard-coding is acceptable per modeling rule 10.

#### PY Values (Prior Year)

Use prior-year actuals (same month, year-1) as forecast base; optional YoY uplift via global modifier.

```
IF(
  'INP_Forecasting_Method' = 'Forecasting Method'."PY Values",
  IFDEFINED(
    'PULL_CR_PnL_Data_Actual'[ADD: Line][SELECT: Month - 12],
    'PULL_CR_PnL_Data_Actual'[ADD: Line][SELECT: Month - 12],
    PREVIOUS(Month, 12)
  )
)
```

SELECT: Month - 12 = same month last year. IFDEFINED with fallback PREVIOUS(Month, 12) for robustness.

#### Last X months average

Average of last X actual months as base for each plan month.

Inputs: INP_Parameter = SET_X_forMA when method = Last X months Avg. Helpers: CALC_Count_Last_Months_Actuals, CALC_Denominator_For_Avg.

```
IF(
  'INP_Forecasting_Method' = 'Forecasting Method'."Last X months Avg",
  (
    'PULL_CR_PnL_Data_Actual'[ADD: Line]
      [FILTER: CALC_Count_Last_Months_Actuals <= INP_Parameter]
      [REMOVE: Month]
  )
  /
  CALC_Denominator_For_Avg[ADD: Month][FILTER: 'PUSH_DH_View_Load_Plan']
)
```

Numerator: sum of actuals over last X months (filter by counter), REMOVE Month. Denominator: aligned to plan months.

#### % of Revenue

Forecast = given % of revenue plan.

Inputs: INP_Parameter = SET_%_forREV (e.g. 5 for 5%). PULL_Rev_Revenue_Plan_Data at Department × Entity × Version × Month.

```
IF(
  'INP_Forecasting_Method' = 'Forecasting Method'."% of Revenue",
  INP_Parameter / 100 * 'PULL_Rev_Revenue_Plan_Data'
)[FILTER: 'PUSH_DH_View_Load_Plan']
```

#### $ per Headcount

Forecast = $ per head × headcount plan.

Inputs: INP_Parameter = SET_$_forHC. PULL_WF_Headcount_Plan_Data at Department × Entity × Version × Month.

```
IF(
  'INP_Forecasting_Method' = 'Forecasting Method'."$ per Headcount",
  INP_Parameter * 'PULL_WF_Headcount_Plan_Data'
)[FILTER: 'PUSH_DH_View_Load_Plan']
```

#### Manual Input

Users type values by month; engine provides writable cells.

```
IFDEFINED(
  FIL_Rowfilter,
  IF(
    'INP_Forecasting_Method' = 'Forecasting Method'."Manual Input"
    AND 'PUSH_DH_View_Load_Plan',
    0
  )
)
```

Only when row visible (FIL_Rowfilter), method is Manual, and month is plan. Baseline 0 creates writable cells.

### Global modifiers: YoY and blank handling

**Inputs:** INP_YoY% (numeric per line, default 0), INP_YoY_Month (base Month of Year, e.g. IFDEFINED(INP_YoY%, Month."January")).

**CALC_YoY_2_fixblanks**: multiplier after SWITCH: (1) YoY uplift/decay (e.g. 1 + INP_YoY% for relevant months), (2) blank handling so method blanks are not turned into zeros. One place for YoY and blank logic; adding a new method does not require duplicating this.

### Row lifecycle and validation

**New line creation:** users add (Entity × Department × PnL_Account) via Newline list. FIL_NewlineAdded maps Newline into Line:

```
ISDEFINED(
  Newline.ID
  [BY: Newline.PnL_Account, Newline.Department, Newline.Entity]
)[ADD: Version][BY: Line."1"]
```

**FIL_Rowfilter**: Boolean controlling which lines appear. Combines: FIL_NewlineAdded; SET_Allowactuallineonly; SET_Onlyshownewlineaftervalidation; FIL_Has_Actuals; INP_Validate[SELECT: Line - 1]; ISDEFINED('INP_Forecasting_Method'[SET_OpexAccounts][SELECT: Line - 1]). Lines appear progressively.

**INP_Validate**: Boolean per line: IF(SET_Skip_Validation, BLANK, IFDEFINED('INP_Forecasting_Method', FALSE)). Used in FIL_Rowfilter and KPI widgets.

### Actual vs Plan windows

**Window flags (Version-based):**

- **PUSH_DH_View_Load_Actual:** Month >= Version.'Window Start Month' AND Month <= Version.'Last Actuals Month'
- **PUSH_DH_View_Load_Plan:** Month after Last Actuals Month and within Version.'Window End Month'

**Usage in engine:**

- CALC_Forecast: [EXCLUDE: 'PUSH_DH_View_Load_Actual'] → no forecast in actual months
- % of Revenue and $ per Headcount: [FILTER: 'PUSH_DH_View_Load_Plan']
- CALC_Manual_Input: AND 'PUSH_DH_View_Load_Plan'
- Reporting: OUT_Act_FC and REP_FC_Last_Actual_Month combine actuals and forecast using these flags

Changing Version properties moves the Actual/Plan boundary without changing any method formula.

### How to add a new method

1. Add an item to Forecasting Method dimension with new ID (e.g. 6) and metadata
2. Create CALC_NewMethod with same dimensions as other CALC_*. Formula: IF(INP_Forecasting_Method = Forecasting Method."NewMethod", <logic>)[FILTER: 'PUSH_DH_View_Load_Plan']
3. Wire parameters if needed in INP_Parameter or SET_*
4. Add case to CALC_Forecast SWITCH
5. YoY and blank handling in CALC_YoY_2_fixblanks unchanged unless new method needs special treatment

---

Related skills: [OPEX Planning – Application Architecture & Patterns](./opex_planning_application_architecture.md) · [Centralizing Financial Reporting Metric (Nexus Pattern)](./finance_nexus_financial_statements.md)
