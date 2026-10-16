---
name: choosing-formula-patterns
description: "Planning skill. Use before writing formulas to decide which patterns and modifiers are needed for the use case. Maps business requirements to formula pattern combinations."
metadata:
  skill_path: /skills/choosing-formula-patterns/SKILL.md
  base_directory: /skills/choosing-formula-patterns
---

# Choosing Formula Patterns

Load this skill **before** writing formulas. It maps business requirements to the pattern combinations you need, then points you to the execution skills for implementation.

## Decision Framework

| Business Requirement | Patterns to Combine | Key Modifiers / Functions | When to use / why |
| --- | --- | --- | --- |
| Pull actuals from transaction data | TL aggregation via the list's Month property | `BY SUM/AVG` over `'TL'.'Month'` + `ITEM` | Standard way to turn transactional rows into dimensional (per Month) values |
| Actual vs Plan in one metric | Version time windows + unified reporting | `IF('Is_Actual', ..., ...)` with `Is_Actual`/`Is_Plan` booleans | Standard setup for any planning and reporting: one metric holds both, switched by version time windows |
| Driver-based forecasting | SWITCH engine + growth assumptions | `SWITCH` on method ID + `Is_Plan` guard | Each forecast method lives in its own independent metric, so logic can differ per method (typical for OPEX planning) |
| Single-metric roll-forward (cash, one balance line) | Compounding chain in one metric | `PREVIOUS(Month)` + `IFDEFINED` seed | Use when a single metric rolls forward on itself and has an opening value. Not for cross-metric chains (see next row) |
| Multi-metric balance roll-forward (inventory, cash, loan balances) | Cross-metric iterative chain — load `skill:iterating-with-previous-and-cycles`, create the cycle **before** writing PREVIOUSOF formulas | Prior-period ending via `PREVIOUSOF('Ending …')`; ending = beginning + inflows − outflows — **not** `PREVIOUS(Month)` or `[SELECT: Month - 1]` | Use when ending balance depends on other metrics that depend on the prior ending: needs a cycle, which a plain `PREVIOUS(Month)` cannot provide |
| Prior year baseline | Time offset lookup | `[SELECT: Month-12]` (not PREVIOUS) | Reading a past period is a lookup, not a recursion, so `SELECT` is cheaper and clearer than `PREVIOUS` |
| Grow a prior-year baseline by a rate on top on data by Month | Time-dimensioned assumption + broadcast | `[SELECT: Month-12]` × `(1 + 'Growth %'[BY CONSTANT: Month.Year])` | Use when the growth rate is set per year but applied to monthly data: broadcast the rate to every month |
| Allocate annual total to months | Period allocation | `[BY SPLIT: Month.Year]` (not BY CONSTANT) | Use for amounts: `BY SPLIT` divides the annual total across months. `BY CONSTANT` would repeat it in every month and overcount |
| Replicate a rate/flag to all months | Constant broadcast | `[BY CONSTANT: Month.Year]` | Use for rates and flags that apply identically to every month, not amounts (see previous row) |
| Override-first staging | Manual override priority | `IFBLANK('Override', 'Calculated')` | Performant way to let users override a calculated value; blank override falls back to the calculation |
| Gate values to plan periods only | Plan gating | `IF('Is_Plan', expr, BLANK)` | Restricts the calculation to plan periods, avoiding useless data points on high-volume metrics |
| Date-range presence (active periods) | Temporal presence | `PRORATA(Month, Start, End+1)` + `ISDEFINED` | Simplifies active-period logic (start/end dates) into one function instead of nested date comparisons |
| Aggregate by property hierarchy | Dimensional rollup | `[BY: Dimension.'ParentProperty']` | Use when the rollup follows a property of the dimension (e.g. Product to its parent Category) |
| Aggregate with mapping metric | Mapped allocation/rollup | `[BY Method: SourceDim -> MappingMetric]` | Use when the rollup needs a mapping metric between two dimensions (multi-dimensional mapping) |
| Cross-app data consumption | Library references | `'SourceApp'::'Metric'` + `RESETACCESSRIGHTS` on PULL | Inter-app formula syntax. `RESETACCESSRIGHTS` is not always necessary |
| Fill gaps in time series | Forward fill | `FILLFORWARD('Input', Month)` as cleaning layer | Makes user inputs simpler (enter a value once, it carries forward); combine with a formula override |
| "In/for a specific period or item" (target keeps that dimension) | Point-in-time lookup, sparse | `[FILTER: Dim = 'Dim'."Item"]` — **not** `[SELECT: ...][ADD CONSTANT: ...]`, | Use to restrict to one period or item while keeping the dimension. `[SELECT][ADD CONSTANT]` would broadcast the value to every item instead |
| Collapse a dimension entirely by a flag (target drops that dimension) | Selective extraction | `[SELECT: Account.'IsRevenue']` | Use when a flag decides which items to keep and the target drops that dimension |
| Tiered/banded lookup (tax brackets, commissions) | Segmentation pattern | `IF` + `[REMOVE FIRSTNONBLANK: Tier]` | Use for any bracket logic: tax, social charges, ramp-up |
| Access rights metric | Security formula | `IFDEFINED('Users roles', ACCESSRIGHTS(...))` with BLANK for denied | Returns BLANK for denied users so the metric works as an access-rights filter |
| Compound a base by several % growth rates | Closed-form compounding instead of a recursive `PREVIOUS` chain | `'Base' * EXP(CUMULATE(LN(1+'% Growth'), Month))` | Use only if growth is purely multiplicative (no adds, caps or resets), the base is constant and `1+g > 0`. Avoids the iterative `PREVIOUS` chain; otherwise keep `PREVIOUS(Month)` |
| Combine separate per-item calculations onto one dimension (e.g. WFP/OPEX → P&L by Account) | Additive `BY` onto target items instead of an `IF` chain, with a VAR metric holding each item | `'Calc 1'[BY SUM: Account."account1"] + 'Calc 2'[BY SUM: Account."account2"]` | Use instead of an `IF` chain when each calculation belongs to one item of the target dimension (e.g. payroll costs to chart of accounts, OPEX values to OPEX accounts, usually for another app) |

## What to Read Next

For domain-specific pattern combinations (OPEX, Workforce, Revenue, P&L, FX, Balance Sheet), see `skill:solving-financial-planning` and `skill:solving-workforce-planning`. For inventory and supply chain roll-forwards, read `skill:solving-supply-chain` **before** writing formulas.

- `skill:writing-pigment-formulas` — syntax reference
- `skill:using-formula-modifiers` — modifier mechanics
- `skill:using-formula-functions` — function signatures
- `skill:writing-performant-formulas` — pre-delivery checklist
- `skill:iterating-with-previous-and-cycles` — PREVIOUS/PREVIOUSOF cycles
