# OPEX Planning – Application Architecture & Patterns

## Purpose

Reusable blueprint for an OPEX (operating expense) planning application built around **driver-based forecasting** at a fixed planning grain (e.g. Entity × Department × PnL_Account × Version × Month), with **user overrides** on top of calculated forecasts.

- Logic is usually **simple**: a small set of predefined forecasting methods (Prior Year, Last X months average, % of Revenue, $ per Headcount, Manual input) and a central engine that computes forecast values per line; users can override where needed.
- Main **complexity is dimensionality**: all planning dimensions (Entity, Department, PnL_Account, Version, Month, and often Line) are in the metric structure, unlike Workforce Planning where many axes use BY-arrow card mappings.

**When to use:** building or extending an OPEX app that pulls actuals and drivers from other apps via PULL_ metrics, lets users choose a forecasting method per line, and produces forecast + actual + variance for reporting (PUSH_*).

**When not to use:** pure manual OPEX with no driver-based methods (see **OPEX Planning – Forecasting Methods & Engine** for the method layer); apps with all data from in-app lists can still reuse the engine and folder patterns.

---

## Core Concepts

- **Planning dimensionality in structure**: Entity, Department, PnL_Account, Version, Month (and Line) are in the structure of most OPEX metrics. Reporting slices use the same dimensions or their parents (BY: PnL_Account.PnL_Account Category).
- **PULL from Library**: Data comes from upstream PULL_ metrics (see Data foundation below and `skill:sharing-data-between-applications` for the PUSH_/PULL_ pattern). Set-up board provides formula hints to connect each PULL_* to its source.
- **Actual vs Plan windows**: Version properties (Window Start Month, Last Actuals Month, Window End Month) drive Boolean metrics (e.g. PUSH_DH_View_Load_Actual, PUSH_DH_View_Load_Plan). All forecast logic excludes actual months and filters to plan months.
- **Method per line**: Forecasting Method dimension (e.g. PY Values, Last X months Avg, % of Revenue, $ per Headcount, Manual Input) with ID (Integer for SWITCH), Need Input in Parameter? (Boolean), Parameter Unit. User selects method per line via INP_Forecasting_Method. Central engine SWITCHes on method ID.
- **Parameters**: INP_Parameter (and config SET_* metrics) supply method-specific values. One parameter metric, populated by IF/SWITCH based on selected method.
- **Overrides on top**: Calculated forecast is the base; users override where needed. Override structure is part of the same engine.
- **Line & Newline**: Line dimension for multiple forecast lines per (Entity, Department, PnL_Account, Version). Newline list (Entity, Department, PnL_Account) for "Add New Combination" → mapped into Line via FIL_NewlineAdded.
- **Naming conventions**: INP_ (input/driver), CALC_ (intermediate), OUT_ (primary output), PULL_/PUSH_ (cross-app), FIL_ (Boolean filters), REP_ (reporting-only), SET_ (settings/config).

---

## Pipeline

```
External apps (PUSH_*)
  → PULL_* (PnL actuals, headcount plan, revenue plan) + window flags (PUSH_DH_View_Load_Actual/Plan)
  → Configuration (SET_*: scope, default methods, validation options)
  → Line creation (Newline → FIL_NewlineAdded → Line dimension)
  → Inputs per line: INP_Forecasting_Method, INP_Parameter, INP_YoY%, INP_Validate, etc.
  → Method-specific CALC_* (CALC_PYValues, CALC_LastXMonthsAVG, CALC_%ofRevenue, CALC_$perHeadcount, CALC_Manual_Input)
  → CALC_Forecast = SWITCH(method ID, …) * YoY/blank modifier [EXCLUDE: Actual months]
  → OUT_FC → REMOVE Line for sharing
  → OUT_Act_FC, REP_FC_Last_Actual_Month, PUSH_OP_OPEX_Plan_Data
  → Boards: Set-up, Manager/Controller Input, OPEX Report
```

**Data foundation:** No in-app OPEX transaction lists. Data comes from PULL_ metrics (see `skill:sharing-data-between-applications` for the pattern). Typical PULL_* metrics:

- PULL_CR_PnL_Data_Actual (Entity × Department × PnL_Account × Version × Month)
- PULL_WF_Headcount_Plan_Data (Department × Entity × Version × Month)
- PULL_Rev_Revenue_Plan_Data (Department × Entity × Version × Month)

**Window flags:** `PUSH_DH_View_Load_Actual` and `PUSH_DH_View_Load_Plan` gate which months are actual vs plan per Version. See **OPEX Planning – Forecasting Methods & Engine** for window flag formulas and method interactions.

---

## Patterns

- **Input & config:** INP_Forecasting_Method, INP_Parameter, INP_YoY%, INP_YoY_Month, INP_Description, INP_Validate; SET_* for defaults and scope; FIL_Has_Actuals, SET_Fill_combo_withactuals?, SET_FCMethod_by_PnlAccount.
- **Row control:** FIL_NewlineAdded (Newline → Line), FIL_Rowfilter (visibility: new lines, actual-only, validation gating); CDFM_ID = INP_Forecasting_Method.ID for SWITCH.
- **Method-specific CALC\_*:** CALC_Actuals, CALC_PYValues, CALC_LastXMonthsAVG, CALC_%ofRevenue, CALC_$perHeadcount, CALC_Manual_Input (dims: Department, PnL_Account, Version, Entity, Month, Line).
- **Engine:** CALC_Forecast = SWITCH(INP_Forecasting_Method.ID, 1→PY, 2→LastXAvg, 3→%Rev, 4→$HC, 5→Manual) * CALC_YoY_2_fixblanks [EXCLUDE: PUSH_DH_View_Load_Actual].
- **Output:** OUT_FC; OUT_Act_FC = OUT_FC[REMOVE: Line] + PULL_CR_PnL_Data_Actual; REP_FC_Last_Actual_Month (chart continuity); PUSH_OP_OPEX_Plan_Data = OUT_FC[REMOVE: Line].

Details of each method, YoY/blank, row lifecycle, and window usage: [OPEX Planning – Forecasting Methods & Engine](./opex_forecasting_planning_methods_engine.md).

---

Related skills: [OPEX Planning – Forecasting Methods & Engine](./opex_forecasting_planning_methods_engine.md) · [Centralizing Financial Reporting Metric (Nexus Pattern)](./finance_nexus_financial_statements.md)
