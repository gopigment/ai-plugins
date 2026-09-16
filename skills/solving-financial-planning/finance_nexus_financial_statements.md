# Core P&L Reporting Module – Nexus Pattern

## Purpose

How to build a **P&L reporting hub** that acts as the central nexus: pulls actual data from ERP and plan data from other planning apps (Revenue, OPEX, Workforce) into a single consistent structure, applies FX conversion, and feeds reporting metrics and tables. Produces a robust **Actual + Budget/Plan P&L** at monthly grain with built-in reconciliation checks. Modular so new planning apps plug into the Nexus layer.

**When to use:** building a central P&L reporting application consolidating ERP actuals, budget, and plan/forecast from one or more planning apps; need a single reporting metric supporting Actual vs Plan views via a Data Type dimension; extensible dimensionality (Entity, Department, plus optional Product, Customer, Project, Cost Center).

**When not to use:** building only a planning app with no central reporting hub; Balance Sheet or Cash Flow reporting (use dedicated 3-statements skill); single data source with no Actual + Budget + plan streams.

---

## Core Concepts

- **Base grain**: Entity × Version × Month × PnL_Account × Department; extended with user-confirmed extra dimensions.
- **Data layer**: Raw inputs (ERP P&L actuals, Budget load, optional plan data from other apps) kept close to source structure.
- **Staging layer**: Metrics reshaping raw data onto common grain and normalizing signs (e.g. `DATA_PnL_00_GL`, `DATA_PnL_02_Rollup`).
- **Nexus layer**: Plug-and-play hub with separate metrics for Actuals (`Nexus_01`), Budget (`Nexus_02`), each plan source (`Nexus_03_*`), combined Plan (`Nexus_04`), and unified Actual + Plan (`Nexus_99`) by Data Type.
- **Data Type**: Dimension (e.g. Actual, Forecast) to pivot the unified metric for both Actual and Plan views.
- **REP_PnL_Data**: Final P&L metric in reporting currency (FX applied); single source for all statement line metrics.
- **Operator**: Property on PnL_Account or PnL_Account Category (1 or -1) to normalize accounting signs; used in staging, not hard-coded.
- **`PULL_*`**: Metrics bringing data from Data Hub or other apps. For Library activation, see `skill:sharing-data-between-applications`.

> **Prerequisites:** Calendar with Month (linked to Quarter/Year). Access to ERP P&L actuals (e.g. `LOAD_PnL_GL_ERP`), Budget P&L (e.g. `LOAD_PnL_Budget`), and optionally plan data from other apps. Names are illustrative; use `tool:search_metrics_and_lists` or `tool:semantic_search` to discover actual block names. If any prerequisite unavailable, create placeholder metrics at the same grain.

---

## Pipeline

1. **Confirm extra dimensions**: ask user which additional breakdowns (Product, Customer, Project, Cost Center, none). Add consistently to staging, Nexus, unified, FX, and table metrics.

2. **Define dimensions**
   - **Time**: Month (Start Date, End Date, Start of Next Period, Year, Quarter), Quarter, Year
   - **Version**: Items e.g. Actuals, Budget, Forecast v1/v2; properties (Dimension → Month): Last Actuals Month, Window Start Month, Window End Month
   - **Other axes**: Entity, Department, Reporting Currency; Data Type (Actual, Forecast) used only in Nexus_99 and reporting
   - **PnL_Account Category**: category flags (Revenue, COGS, Operating Expenses, etc.); **Operator** (Integer): 1 for Revenue/Other Income, -1 otherwise
   - **PnL_Account**: Name, optional Display Name; PnL_Account Category; **Operator** = IFBLANK(PnL_Account.'PnL_Account Category'.Operator, 1); optional PnL_EBITDA
   - **Extra dimensions** (only if user confirmed)

3. **Data staging**: map ERP GL into `DATA_PnL_00_GL` at base grain (BY from transaction list). Normalize signs in `DATA_PnL_02_Rollup` = `DATA_PnL_00_GL` * PnL_Account.Operator * -1.

4. **Nexus metrics**
   - `PnL_Nexus_01_Actual_Data`: staging actuals with Version added, gated by actual-months view/pull
   - `PnL_Nexus_02_Budget_Data`: budget load with Version added, gated by plan view
   - Plan plugs: `PULL_RE_Revenue_Plan_Data`, `PULL_OP_OPEX_Plan_Data`, `PULL_WF_Workforce_Plan_Data` (BLANK or PULL from apps)
   - `PnL_Nexus_03_*/Plan_Data`: IF Version not Actuals/Budget then respective PULL_*, else BLANK
   - `PnL_Nexus_04_Plan_Data`: IF Version = Budget then Nexus_02; else sum of Nexus_03_…

5. **Unified Actual + Plan**: `PnL_Nexus_99_Actual_Plan_Data`: IF Data Type = Actual then Nexus_01 (ADD Data Type, BY SUM Actual); else Nexus_04 (ADD Data Type, BY SUM Forecast).

6. **FX reporting**: `REP_PnL_Data` = `PnL_Nexus_99` × FX rate (AVG for P&L). Dims: Reporting Currency, Entity, Version, PnL_Account, Month, Data Type, Department, [extra dims].

7. **Statement table metrics**: each line = filter on `REP_PnL_Data` by PnL_Account Category. Derived lines (Gross Margin) = sum of relevant lines × Category Operator [REMOVE: PnL_Account].

8. **Reconciliation check (mandatory)**: `PnL_Tbl_Check = 'REP_PnL_Data'[REMOVE: PnL_Account] - (sum of all statement lines)`. If not zero, pipeline has a gap. Not optional.

9. **P&L table**: Rows: PnL line or category; Pages: Entity, Department, extra dims; Columns: Month, Data Type, Version; Values: PnL table metrics.

---

## Patterns

### Staging: GL to common grain and sign normalization

```text
DATA_PnL_00_GL =
  'LOAD_PnL_GL_ERP'.Amt_LC
  [BY: Month, Account, Entity, Department
       /* + each extra dimension from source */]

DATA_PnL_02_Rollup =
  'DATA_PnL_00_GL' * PnL_Account.Operator * -1
```

Same dimensions for both. Operator drives sign; no hard-coded +/-.

**Sign conventions:** **Option A (ERP GL)** — source revenue negative, expenses positive → × Operator × -1. **Option B (mock, budget, plan)** — source all positive → × Operator only. Confirm source convention before staging.

**Operator formulas:**

- **PnL_Account Category**.Operator: `IF('PnL_Account Category'.IsRevenueCategory OR 'PnL_Account Category'.IsOtherIncomeCategory, 1, -1)`
- **PnL_Account**.Operator: `IFBLANK(PnL_Account.'PnL_Account Category'.Operator, 1)`

### Nexus Actual and Budget

```text
PnL_Nexus_01_Actual_Data =
  'DATA_PnL_02_Rollup'
  [ADD: Version]
  ['PULL_DH_View_Load_Actual']

PnL_Nexus_02_Budget_Data =
  'LOAD_PnL_Budget'
  [ADD: Version]
  [BY SUM: SET_Budget_Version]
  ['PULL_DH_View_Load_Plan']
```

### Nexus plan plugs

```text
PnL_Nexus_03_Revenue_Plan_Data =
  IF(
    NOT Version IN (SET_Actuals_Version, SET_Budget_Version),
    'PULL_RE_Revenue_Plan_Data',
    BLANK
  )
```

Same pattern for OPEX and Workforce. If no separate Revenue app, can use Budget filtered to Revenue category.

### Nexus combined Plan

```text
PnL_Nexus_04_Plan_Data =
  IF(
    Version = SET_Budget_Version,
    'PnL_Nexus_02_Budget_Data',
    'PnL_Nexus_03_Revenue_Plan_Data'
    + 'PnL_Nexus_03_OPEX_Plan_Data'
    + 'PnL_Nexus_03_Workforce_Plan_Data'
  )
```

### Unified Actual + Plan by Data Type

```text
PnL_Nexus_99_Actual_Plan_Data =
  IF(
    Is_Actual,
    'PnL_Nexus_01_Actual_Data'
      [ADD: 'Data Type']
      [BY SUM: SET_Actual_Data_Type],
    'PnL_Nexus_04_Plan_Data'
      [ADD: 'Data Type']
      [BY SUM: SET_Forecast_Data_Type]
  )
```

### FX reporting metric

```text
REP_PnL_Data =
  'PnL_Nexus_99_Actual_Plan_Data'
  * 'PULL_DH_FX_Rates'[SELECT: 'FX Rate Types'."AVG"]
```

For full FX engine design, see [FX currency conversion (Hub pattern)](./fx_currency_conversion.md).

### Statement line from REP_PnL_Data

```text
PnL_Tbl_01_Revenue =
  'REP_PnL_Data'
  [FILTER: PnL_Account.IsRevenueCategory]

PnL_Tbl_02_Gross_Margin =
  (
    'PnL_Tbl_01_Revenue'
    + 'PnL_Tbl_01_Cost_Of_Goods_Sold'
  )
  * PnL_Account.'PnL_Account Category'.Operator
  [REMOVE: PnL_Account]
```

Other lines (OPEX, EBITDA drivers, Net Income) follow the same pattern: filter by category or aggregate with Operator, then REMOVE PnL_Account.

---

## Pitfalls

- **Extra dimensions**: add everywhere from the start; retrofitting is error-prone.
- **Operator consistency**: wrong sign on PnL_Account/Category breaks all statement totals. Pair input signs with the Staging pattern.
- **View/pull logic**: Nexus_01 and Nexus_02 depend on view/pull metrics defining actual vs plan months per Version; align with Version properties.
- **Plan plugs**: if an app doesn't exist yet, use BLANK or placeholder; do not build that app inside this skill.
- **Single grain**: all Nexus and Rep metrics share the same dimension set; mismatched grains cause wrong totals.
- **Reconciliation check**: `PnL_Tbl_Check` is mandatory. If not zero, investigate missing categories or sign errors before proceeding.

---

Related skills: [FX currency conversion (Hub pattern)](./fx_currency_conversion.md) · [OPEX Planning – Architecture & Patterns](./opex_planning_application_architecture.md) · [OPEX Planning – Forecasting Methods & Engine](./opex_forecasting_planning_methods_engine.md)
