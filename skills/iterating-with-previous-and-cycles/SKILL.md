---
name: iterating-with-previous-and-cycles
description: "Execution skill. Use when writing PREVIOUS, PREVIOUSOF, or PREVIOUSBASE formulas, configuring iterative calculation cycles, or debugging circular dependency errors."
metadata:
  skill_path: /skills/iterating-with-previous-and-cycles/SKILL.md
  base_directory: /skills/iterating-with-previous-and-cycles
---

# Iterating with PREVIOUS and Cycles

Iterative calculations let a metric reference its own previous value along a dimension (typically time). Use them only for true metric-level circular dependencies; Pigment does not support cell-level circulars like Excel.

## Choose the Right Iterative Tool

| Scope | Function | When to use |
| --- | --- | --- |
| Single block | `PREVIOUS(Dimension [, Offset])` | All circular-referencing formulas are in one block |
| Multiple blocks | `PREVIOUSOF(Metric)` | Circular references span different blocks; requires an iterative calculation cycle |

Rule: loop within a single metric with PREVIOUS; loop across multiple metrics with PREVIOUSOF. PREVIOUSBASE is deprecated; use PREVIOUSOF instead.

## Write PREVIOUS Formulas

**Syntax**: `PREVIOUS(IteratingDimension [, Offset])`

The argument is the iterating **dimension**, not the metric. Offset defaults to 1; can be a positive integer or an Integer metric on the same dimensions.

Common error: `PREVIOUS('My Metric Name')` fails with `Unknown List: My Metric Name`. The fix: use `PREVIOUS(Month)` — the dimension, not the metric name.

```pigment
// Cash balance roll-forward (single metric)
Cash = PREVIOUS(Month) + Income - Expense
```

Use PREVIOUS only when the entire roll-forward lives in one metric. When Beginning and Ending are separate metrics, use PREVIOUSOF and a cycle (see below).

The engine computes the entire formula (including everything before and after PREVIOUS) before moving to the next cell. Post-processing (filtering) should happen in a **separate metric**.

## Avoid the IFBLANK + PREVIOUS Pitfall

```pigment
// BAD — returns blanks everywhere including where the filter is TRUE
IFBLANK('Source', PREVIOUS(Month))
['Filter Boolean']
```

**Why**: for earlier items where the filter is FALSE, the engine emits blanks. When TRUE, PREVIOUS points to a blank cell, propagating BLANK forward.

**Fix**: separate the PREVIOUS expression from the filter:

```pigment
// Metric 1 — 'Filled Source'
IFBLANK('Source', PREVIOUS(Month))

// Metric 2 — 'Filtered Result'
'Filled Source'['Filter Boolean']
```

## Set Up PREVIOUSOF Cycles

Before using PREVIOUSOF, create an iterative calculation cycle listing all participating metrics and the iterating dimension. PREVIOUSOF will not resolve without a cycle. Do not rewrite to PREVIOUS on itself; this changes the mathematical meaning.

### Programmatic workflow

1. Call `tool:list_cycles` to check if a cycle already exists.
2. Identify all metrics in the dependency chain and the iteration dimension (typically Month).
3. Create all participating metrics first (they must exist before being added to the cycle).
4. Call `tool:create_cycle` with `cycleName`, `iterativeDimensionId`, and all `metricIds`.
5. Only after the cycle is created, write PREVIOUSOF formulas. Writing PREVIOUSOF before creating the cycle produces errors.
6. Use `tool:update_cycle` to add or remove metrics from an existing cycle.
7. Verify all mutually dependent metrics participate.

### Constraints

- Maximum ~10 metrics per cycle (verify the current platform limit)
- All metrics must include the iteration dimension in their structure
- Iteration dimension must have fewer than 10,000 items
- Cannot combine or link two cycles together

### Standard balance pattern

Use this when beginning and ending balances are separate metrics (inventory, cash, loan balances). Create the cycle first, then write PREVIOUSOF formulas.

```pigment
// Beginning balance — after cycle exists
IFDEFINED(PREVIOUSOF('Ending Balance'), PREVIOUSOF('Ending Balance'), 'Opening Balance')

// Ending balance — inflows and outflows may be direct references or intermediate metrics
'Beginning Balance' - 'Outflows' + 'Inflows'
```

Do not substitute `'Ending Balance'[SELECT: Month - 1]` for PREVIOUSOF; it does not participate in the cycle and breaks cross-metric roll-forwards.

### Cash Balance from Sparse Revenue

```pigment
// Seed with 0 when no prior period exists; add net income each month
IFDEFINED(PREVIOUS(Month), PREVIOUS(Month), 0) + IFDEFINED('Net Income', 'Net Income', 0)
```

Use `IFDEFINED` to handle the first period (no prior balance) and months with no data.

## Apply the Reduction Heuristic (performance only)

Before collapsing a PREVIOUSOF cycle into PREVIOUS, confirm that separate Beginning/Ending metrics are not required. Many planning models expose both metrics for reporting, reorder logic, or drill-down views; in those cases keep the two-metric PREVIOUSOF + cycle pattern even if the math could collapse.

Only collapse when **all** of the following are true:

1. The user needs only one balance metric (no separate Beginning metric in outputs).
2. No other metric in the chain independently references a prior value.
3. You have verified the collapsed formula is equivalent after substitution.

```pigment
// Collapsed version (single metric only — often more performant)
'Ending Balance' = PREVIOUS(Month) + 'Movements'
```

When in doubt, prefer PREVIOUSOF + cycle. Collapsing changes which metrics exist in the model and is harder to extend later.

## Debug Iterative Formula Errors

When a cycle is active, Pigment builds a combined base formula. Three individually valid formulas can combine into an invalid base formula.

**Key rule**: modifiers (BY, SELECT, REMOVE, FILTER) are supported **as long as they do not reference the iterating dimension**. Window functions (CUMULATE, MOVINGAVERAGE, MOVINGSUM) are supported within cycle metrics.

```pigment
// BAD — REMOVE on the iterating dimension inside a cycle metric
'Incoming' = 'Beginning'[REMOVE: Month]
```

Error: "A dimension modifier using the iterating dimension Month can't be applied to a metric in the iterative calculation."

## Use FILLFORWARD Instead When Possible

FILLFORWARD is non-iterative and much faster. Use it when you only need to carry the last known value forward with no calculation at each step.

```pigment
// BAD — iterative carry-forward
IFBLANK('Status Input', PREVIOUSOF('Current Status'))

// GOOD — non-iterative, 10-50x faster
FILLFORWARD('Status Input', Month)
```

Use PREVIOUS/PREVIOUSOF only when each step involves a computation (balance + inflows - outflows, conditional reorder logic, etc.).

## Optimize Iterative Calculation Performance

Iterative calculations are inherently sequential (period N depends on 1..N-1). Cost drivers:

- **Iterating dimension length**: More items = more sequential steps
- **Number of other dimensions**: More dimensions = more iteration chains
- **Data density**: Dense data = more cells per step
- **Formula complexity**: Heavy sub-expressions multiply cost per step

### Optimization strategies (ordered by typical impact)

1. **Subset the iterating dimension** to relevant periods (current fiscal year, rolling 90 days). 5-20x faster.
2. **Use FILLFORWARD** for simple carry-forward logic. 10-50x faster.
3. **Aggregate before iterating** (REMOVE unnecessary dimensions before the iterative metric). 10-1000x faster.
4. **Pre-compute starting points** so the engine does not roll forward from the beginning of time.
5. **Consider coarser granularity**: monthly instead of daily (~30x fewer iterations).
6. **Keep sub-expressions light**: avoid densifying functions like ISBLANK inside PREVIOUS. Prefer `IFDEFINED(M, M, PREVIOUS(Month))` over `IF(ISBLANK(M), PREVIOUS(Month), M)`.
7. **Minimize PREVIOUS calls**: `PREVIOUS(Month) * (A + B)` is faster than `PREVIOUS(Month) * A + PREVIOUS(Month) * B`.
8. **Eliminate unused metrics from cycle configurations**: only include metrics in the actual dependency chain.

### Performance expectations

| Granularity | Periods/year | 5-year horizon |
| --- | --- | --- |
| Monthly | 12 | 60 (fast) |
| Weekly | 52 | 260 (moderate) |
| Daily | 365 | 1,825 (slow) |

## When Iterative Calculations Are Unavoidable

Some patterns require iteration:

- Balance roll-forwards with conditional logic (credit line draws, reorder points)
- Inventory with inflows/outflows where each period depends on the prior ending balance
- Loan amortization schedules

Optimize what you can (subsetting, dimensionality, granularity) and accept the remaining performance cost.
