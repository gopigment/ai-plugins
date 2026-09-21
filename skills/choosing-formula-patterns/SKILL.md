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

| Business Requirement | Patterns to Combine | Key Modifiers / Functions |
| --- | --- | --- |
| Pull actuals from transaction data | TL aggregation via the list's Month property | `BY SUM/AVG` over `'TL'.'Month'` + `ITEM` |
| Actual vs Plan in one metric | Version time windows + unified reporting | `IF('Is_Actual', ..., ...)` with `Is_Actual`/`Is_Plan` booleans |
| Driver-based forecasting | SWITCH engine + growth assumptions | `SWITCH` on method ID + `Is_Plan` guard |
| Single-metric roll-forward (cash, one balance line) | Compounding chain in one metric | `PREVIOUS(Month)` + `IFDEFINED` seed |
| Multi-metric balance roll-forward (inventory, cash, loan balances) | Cross-metric iterative chain — load `skill:iterating-with-previous-and-cycles`, create the cycle **before** writing PREVIOUSOF formulas | Prior-period ending via `PREVIOUSOF('Ending …')`; ending = beginning + inflows − outflows — **not** `PREVIOUS(Month)` or `[SELECT: Month - 1]` |
| Prior year baseline | Time offset lookup | `[SELECT: Month-12]` (not PREVIOUS) |
| Grow a prior-year baseline by a rate on top on data by Month | Time-dimensioned assumption + broadcast | `[SELECT: Month-12]` × `(1 + 'Growth %'[BY CONSTANT: Month.Year])` |
| Allocate annual total to months | Period allocation | `[BY SPLIT: Month.Year]` (not BY CONSTANT) |
| Replicate a rate/flag to all months | Constant broadcast | `[BY CONSTANT: Month.Year]` |
| Override-first staging | Manual override priority | `IFBLANK('Override', 'Calculated')` |
| Gate values to plan periods only | Plan gating | `IF('Is_Plan', expr, BLANK)` |
| Date-range presence (active periods) | Temporal presence | `PRORATA(Month, Start, End+1)` + `ISDEFINED` |
| Aggregate by property hierarchy | Dimensional rollup | `[BY: Dimension.'ParentProperty']` |
| Aggregate with mapping metric | Mapped allocation/rollup | `[BY Method: SourceDim -> MappingMetric]` |
| Cross-app data consumption | Library references | `'SourceApp'::'Metric'` + `RESETACCESSRIGHTS` on PULL |
| Fill gaps in time series | Forward fill | `FILLFORWARD('Input', Month)` as cleaning layer |
| "In/for a specific period or item" (target keeps that dimension) | Point-in-time lookup, sparse | `[FILTER: Dim = 'Dim'."Item"]` — **not** `[SELECT: ...][ADD CONSTANT: ...]`, which broadcasts the value to every item on that dimension instead of restricting to just this one |
| Collapse a dimension entirely by a flag (target drops that dimension) | Selective extraction | `[SELECT: Account.'IsRevenue']` |
| Tiered/banded lookup (tax brackets, commissions) | Segmentation pattern | `IF` + `[REMOVE FIRSTNONBLANK: Tier]` |
| Access rights metric | Security formula | `IFDEFINED('Users roles', ACCESSRIGHTS(...))` with BLANK for denied |

## What to Read Next

For domain-specific pattern combinations (OPEX, Workforce, Revenue, P&L, FX, Balance Sheet), see `skill:solving-financial-planning` and `skill:solving-workforce-planning`. For inventory and supply chain roll-forwards, read `skill:solving-supply-chain` **before** writing formulas.

- `skill:writing-pigment-formulas` — syntax reference
- `skill:using-formula-modifiers` — modifier mechanics
- `skill:using-formula-functions` — function signatures
- `skill:writing-performant-formulas` — pre-delivery checklist
- `skill:iterating-with-previous-and-cycles` — PREVIOUS/PREVIOUSOF cycles
