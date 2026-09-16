---
name: solving-financial-planning
description: "Planning skill. FP&A use cases: budget/expense planning, revenue forecasting, P&L reporting, multi-currency, financial consolidation. Each use case has inline guidance; load sub-files only for advanced patterns."
metadata:
  skill_path: /skills/solving-financial-planning/SKILL.md
  base_directory: /skills/solving-financial-planning
  includes:
    - "*.md"
---

# Financial Planning & Analysis (FP&A)

> Prerequisites: Version dimension exists (`skill:building-versions-and-planning-cycles`). Calendar configured (`skill:setting-up-calendar`).

---

## Budget & Expense Planning

**When:** annual budget, departmental cost plans, cost center budgets, simple OPEX.

**Dimensions:** Version, Month, Department (or Cost Center), Account (Chart of Accounts). Add Entity if multi-entity.

**Structure:** one input metric for actuals (imported), one for budget/forecast (manual input or formula-driven). Calculated metrics for variances and totals. Keep formulas flat; separate actuals from plan in distinct metrics.

**Forecasting approaches:** prior year with growth rate, manual input filtered to plan months, or IF-based mix per Version. No SWITCH engine, no method dimension.

**Reporting:** combine actuals and forecast in a single metric using Is_Actual. Table block with one row per account category. Views: Month (columns) × Department or Account (rows).

For prior-year growth formulas, see `skill:choosing-formula-patterns` for the correct modifier pattern.

**Extend to full OPEX engine when:** users need 3+ forecasting methods selectable per line (e.g. PY values, moving average, % of revenue, $ per headcount). Load [#1 Architecture](./opex_planning_application_architecture.md) + [#2 Engine](./opex_forecasting_planning_methods_engine.md).

---

## Revenue Planning

**When:** revenue forecast, sales pipeline, ARR/MRR, bookings, growth-based projections.

**Dimensions:** Version, Month, Product (or Segment), Customer (or Territory). Add Entity if multi-entity.

**Structure:** use a transaction list for deal-level or booking-level data; aggregate into planning metrics. Keep driver assumptions (growth rates, conversion rates, churn rates) in dedicated input metrics. Separate actuals from forecast metrics.

**Forecasting approaches:** growth-based (prior year × rate), ARR/MRR waterfall (new + expansion - churn + prior), pipeline conversion (value × rate by stage), or bookings spread over contract duration with PRORATA.

**Reporting:** combine actuals and forecast using Is_Actual. Views: Month (columns) × Product or Customer (rows). Use calculated items for variance (Forecast vs Budget, YoY).

No dedicated advanced pattern exists yet. For complex multi-stage pipeline with probability weighting or territory-based capacity planning, build from these principles and `skill:writing-pigment-formulas`.

---

## P&L & Variance Reporting

**When:** actual vs budget comparison, P&L statement, variance analysis from a single data source.

**Dimensions:** Version, Month, Account (with Category parent hierarchy), Department. Add Entity and Data Type if needed.

**Structure:** actuals metric (imported from GL or transaction list), budget metric (input), forecast metric (formula or input). Variance = Budget - Actual. Use Account.Category property to group into P&L lines (Revenue, COGS, OPEX). Use Operator property on Category (1 for revenue, -1 for expenses) to normalize signs.

**Reporting:** Table block with P&L line metrics as rows. Use calculated items for variance columns (Budget vs Actual, % variance). Views: Month (columns), filter by Version.

**Mandatory:** even for single-source P&L, create a `DATA_*` staging metric between raw input and the reporting output, and a `*_Check` reconciliation metric that verifies the staging total equals the sum of statement lines.

**Extend to Nexus when:** the P&L must consolidate data from multiple upstream apps (Revenue, OPEX, Workforce) and/or ERP, **or when the request asks for a P&L reporting hub, nexus, or statement pipeline, even single-source.** Load [#4 Nexus](./finance_nexus_financial_statements.md). The Nexus sub-file covers P&L only; BS/CF can be adapted from the same pattern.

---

## Multi-Currency

**When:** multiple entities with different functional currencies need conversion to a group reporting currency.

**Simple approach:** for a single app needing basic FX, create a local rate input metric and multiply amounts. No Hub engine required.

**Extend to FX Hub when:** the workspace has multiple apps sharing the same FX rates and entity-currency mappings. Load [#3 FX Hub](./fx_currency_conversion.md). Requires a Hub app (see `skill:architecting-multi-application-solutions`).

---

## Extending an Existing OPEX Engine

**When:** adding or changing forecasting methods in an app that already has the driver-based engine structure (SWITCH on method ID, INP_Forecasting_Method, CALC_* per method).

Load [#2 Engine](./opex_forecasting_planning_methods_engine.md) only. Do not load #1.
