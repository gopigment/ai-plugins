# Finance Functions

> Parameter order differs from Excel. Rate is 1st arg in NPV/XNPV. Guess is 2nd in IRR/XIRR. RankingDimension is 4th, after ComputeAllCells.

---

## NPV

`NPV(Rate, CashFlows [, ComputeAllCells] [, RankingDimension])`

- Rate: per-period discount rate (e.g. 0.1 = 10%). Can be dimensioned for variable rate.
- CashFlows: negative = payments, positive = income
- ComputeAllCells: FALSE (default) = first non-empty item only; TRUE = every item
- RankingDimension: required when CashFlows has multiple dimensions
- **Excludes initial investment** (add separately if needed)
- Ignores blanks/text

```pigment
NPV(0.10, 'Cash Flows')
NPV(0.12 / 12, 'Monthly Cash Flows')
NPV(0.10, 'Cash Flows by Country', TRUE, Year)
```

---

## XNPV

`XNPV(Rate, CashFlows [, ComputeAllCells] [, RankingDimension] [, DaysUsed])`

- DaysUsed: Date Property of RankingDimension; mandatory if not a Calendar dim
- Uses actual dates: Σ Pᵢ/(1+r)^((dᵢ−d₁)/365)
- **Includes all payments** (unlike NPV)
- Dates must be chronological or returns BLANK; blank dates → payment ignored

```pigment
XNPV(0.10, 'Investment Cashflow', TRUE)
XNPV(0.10, 'Cashflow by Country', TRUE, Month, Month.'Start Date')
```

---

## IRR

`IRR(CashFlows [, Guess] [, ComputeAllCells] [, RankingDimension])`

- Guess: default 0.1; must be > -1; should rarely exceed 100
- Cash flows must include at least one negative AND one positive
- Newton-Raphson; 200 iterations max → BLANK if no convergence
- All-positive or all-negative → BLANK

```pigment
IRR('Cash Flows')
IRR('Cash Flows', 0.5, TRUE)
IRR('Payment Per Country', 0.1, FALSE, 'Fiscal Year')
```

---

## XIRR

`XIRR(CashFlows [, Guess] [, ComputeAllCells] [, RankingDimension] [, DaysUsed])`

- Guess: default 0.1; must be > -1; should rarely exceed 100
- RankingDimension must be a Calendar time dimension or have a Date property (via DaysUsed)
- Same date requirements as XNPV
- ComputeAllCells=FALSE: items without payment ignored; TRUE: empty = 0

```pigment
XIRR('Cash Flow', 0.1, TRUE)
XIRR('Portfolio'.'Cash Flow', 0.12, TRUE, 'Transaction', 'Transaction'.'Date')
```

---

## NPV vs XNPV / IRR vs XIRR

| Aspect | NPV / IRR | XNPV / XIRR |
| --- | --- | --- |
| Cash flow timing | Periodic (evenly spaced) | Irregular (specific dates) |
| Use case | Regular intervals | Transaction-based |
| Performance | Faster | Slower |
| Initial investment | NPV excludes it | XNPV includes all |

---

## Common Patterns

```pigment
// Project evaluation
IF(NPV(0.10, 'Project Cash Flows', TRUE, Year) > 0, "Accept", "Reject")

// Compare IRR to hurdle rate
IF(IRR('Investment Cash Flows', 0.1, TRUE, Year) > 'Hurdle Rate', "Invest", "Pass")

// Monthly to annual rate adjustment
NPV('Annual Discount Rate' / 12, 'Monthly Cash Flows')
```
