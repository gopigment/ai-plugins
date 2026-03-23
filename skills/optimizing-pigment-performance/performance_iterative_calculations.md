# Performance Iterative Calculations

## Introduction

Iterative calculations—where each period depends on the previous period—are common in financial planning but can create significant performance challenges. Functions like PREVIOUS, PREVIOUSOF, CUMULATE, and FILLFORWARD require sequential computation that can become slow over long time horizons.

This guide covers **optimization strategies** for iterative calculations, subsetting techniques, and when to use alternative approaches. For the full technical spec (circular dependencies, PREVIOUS vs PREVIOUSOF, configuration, syntax, debugging), see [Iterative Calculation (PREVIOUS & PREVIOUSOF)](../writing-pigment-formulas/functions_iterative_calculation.md).

**⚠️ IMPORTANT — PREVIOUSOF Prerequisite:**
PREVIOUSOF can only be used on metrics that have iterative calculation enabled in the Pigment application settings. This configuration **cannot be done via AI tools** — the user must set it up in the Pigment UI. Before writing any formula with PREVIOUSOF, confirm with the user that iterative calculation is configured on the target metric. If not, instruct them to enable it first.

## Understanding Iterative Calculation Performance

### Why Iterative Calculations Are Slow

**Sequential dependency**: To compute period N, you must first compute periods 1 through N-1.

**Example**:

```pigment
'Ending Balance' = PREVIOUSOF('Ending Balance', 0) + 'Inflow' - 'Outflow'
```

**Computation sequence**:

- Month 1: 0 + Inflow₁ - Outflow₁
- Month 2: Month 1 result + Inflow₂ - Outflow₂
- Month 3: Month 2 result + Inflow₃ - Outflow₃
- ...
- Month 36: Month 35 result + Inflow₃₆ - Outflow₃₆

**Performance impact**: Cannot parallelize, must compute sequentially.

### Scope Loss in Iterative Calculations

Iterative calculations lose scope on the iterating dimension:

```pigment
'YTD Revenue' = 'Monthly Revenue'[CUMULATE: Month]
```

**Profiler result**: Scope lost on Month dimension.

**Why**: If Month 3 changes, Months 3-12 must be recomputed (sequential dependency).

### Performance Factors

**Time horizon**: Longer horizons = more sequential steps

- 12 months: Fast
- 36 months: Moderate
- 60 months (5 years): Slow
- 1,825 days (5 years daily): Very slow

**Granularity**: Finer granularity = more steps

- Monthly: 12 steps per year
- Weekly: 52 steps per year
- Daily: 365 steps per year

**Dimensions**: More dimensions = more cells to iterate

- 1 dimension (Month): Fast
- 2 dimensions (Month × Product): Moderate
- 3 dimensions (Month × Product × Region): Slow

**Data density**: Dense data = more cells to compute

- Sparse (10% cells): Fast
- Dense (90% cells): Slow

## Optimization Strategy 1: Subset Time Dimensions

### The Problem: Long Time Horizons

**Scenario**: 5 years of daily data for inventory tracking.

**Anti-pattern**:

```pigment
'Daily Ending Inventory' =
  PREVIOUSOF('Daily Ending Inventory', 0) + 'Purchases' - 'Sales'
```

**Performance**: 1,825 sequential days × all products × all warehouses = Very slow.

### The Solution: Subset to Relevant Periods

**Pattern**: Use dimension subsets to limit the iteration window.

**Implementation**:

```pigment
// Create a subset for recent periods only
// Subset: 'Recent Days' = Last 90 days

'Daily Ending Inventory' =
  PREVIOUSOF('Daily Ending Inventory'[FILTER: Day IN 'Recent Days'], 0) +
  'Purchases' - 'Sales'
```

**Performance**: 90 days instead of 1,825 days = 20x faster.

### When to Use Subsets

**Use subsets when**:

- Historical data doesn't change
- Only recent periods need iterative calculation
- Users only interact with recent periods

**Design and risks:** List Subsets have irreversible data-loss behavior when membership changes and require explicit mapping to the parent dimension. For when to recommend subsets vs filters or another list, data-loss warnings, and safe patterns (STORE/CALC, remap to parent), see [List Subsets (modeling)](../modeling-pigment-applications/modeling_subsets.md).

**Example use cases**:

- Rolling 90-day inventory
- Current fiscal year YTD
- Last 12 months cumulative
- Current quarter sequential calculations

### Creating Effective Subsets

**Time-based subset**:

```pigment
// Property on Month dimension
'Is Current Year' = YEAR(Month.'Date') = YEAR(TODAY())

// Use in formula
'YTD Revenue' = 'Monthly Revenue'[FILTER: Month.'Is Current Year'][YEARTODATE]
```

**Dynamic subset**:

```pigment
// Property on Day dimension
'Days Since Today' = DAYS(TODAY(), Day.'Date')
'Is Recent' = 'Days Since Today' <= 90

// Use in formula
'Recent Cumulative' = 'Daily Value'[FILTER: Day.'Is Recent'][CUMULATE: Day]
```

## Optimization Strategy 2: Use FILLFORWARD Instead of PREVIOUS

### When FILLFORWARD is Better

**FILLFORWARD**: Non-iterative blank filling (more efficient).

**PREVIOUS**: Iterative calculation (less efficient).

**Use FILLFORWARD when**:

- You only need to fill blanks with the last known value
- No calculation is needed at each step
- The logic is simple forward propagation

### Example: Status Propagation

**Anti-pattern using PREVIOUS**:

```pigment
'Current Status' =
  IFBLANK('Status Input', PREVIOUSOF('Current Status', "Unknown"))
```

**Problem**: Iterative, computes every period even if no change.

**Optimized using FILLFORWARD**:

```pigment
'Current Status' = 'Status Input'[FILLFORWARD]
```

**Improvement**: Non-iterative, much faster.

### Example: Employee Assignment

**Anti-pattern using PREVIOUS**:

```pigment
'Current Department' =
  IFBLANK('Department Change', PREVIOUSOF('Current Department', "Unassigned"))
```

**Optimized using FILLFORWARD**:

```pigment
'Current Department' = 'Department Change'[FILLFORWARD]
```

**When PREVIOUS is required**:

- Calculation at each step (e.g., balance + inflow - outflow)
- Conditional logic at each period
- Transformations that depend on previous value

## Optimization Strategy 3: Reduce Dimensionality

### The Problem: High-Dimensional Iterative Calculations

**Anti-pattern**:

```pigment
// 3 dimensions: Month × Product × Region
'Cumulative Sales' = 'Monthly Sales'[CUMULATE: Month]
```

**Performance**: Iterates for every Product × Region combination.

**If**: 1,000 products × 50 regions = 50,000 iteration chains.

### The Solution: Aggregate Before Iterating

**Pattern**: Reduce dimensions before iterative calculation.

**Implementation**:

```pigment
// Aggregate to fewer dimensions
'Total Monthly Sales' = 'Monthly Sales'[REMOVE: Product, Region]

// Iterate on smaller dataset
'Cumulative Total Sales' = 'Total Monthly Sales'[CUMULATE: Month]

// If needed, allocate back
'Allocated Cumulative' =
  'Cumulative Total Sales'[BY: 'Allocation Key']
```

**Performance**: 1 iteration chain instead of 50,000 = 50,000x faster.

**Trade-off**: Less granular (total only, not by Product × Region).

### When This Works

**Use when**:

- The cumulative total is what matters
- Product/Region-level cumulative not needed
- Reporting is at aggregate level

**Don't use when**:

- Need cumulative by Product × Region
- Granular analysis required
- Allocation would be complex

## Optimization Strategy 4: Alternative Calculation Methods

### Pattern 1: Pre-Compute Starting Points

**Anti-pattern**: Iterate from the beginning of time.

```pigment
'Ending Balance' = PREVIOUSOF('Ending Balance', 0) + 'Change'
```

**Problem**: If data goes back 10 years, iterates from year 1.

**Optimized**: Use a known starting point.

```pigment
// Import or calculate starting balance for current year
'Starting Balance' = 'Imported Starting Balance'

// Iterate only from current year
'Ending Balance' =
  IFBLANK(
    PREVIOUSOF('Ending Balance', 'Starting Balance') + 'Change',
    'Starting Balance'
  )
```

**Improvement**: Iterate only current year, not all history.

### Pattern 2: Use CUMULATE Instead of PREVIOUS

**When applicable**: Simple running totals without complex logic.

**Anti-pattern**:

```pigment
'Running Total' = PREVIOUSOF('Running Total', 0) + 'Value'
```

**Optimized**:

```pigment
'Running Total' = 'Value'[CUMULATE: Month]
```

**Why better**: CUMULATE is optimized for simple summation.

### Pattern 3: Period-Specific Calculations

**Anti-pattern**: Iterate across all periods for a single calculation.

```pigment
// Just need December YTD
'Dec YTD' = 'Monthly Revenue'[YEARTODATE]
```

**Problem**: Computes YTD for all months, even though only December is needed.

**Optimized**:

```pigment
// Calculate only December YTD
'Dec YTD' = 'Monthly Revenue'[SELECT: Month <= Month."Dec 25"][REMOVE: Month]
```

**Improvement**: Single aggregation instead of iterative calculation.

## Optimization Strategy 5: Granularity Trade-offs

### Consider Monthly Instead of Daily

**Scenario**: Cash flow forecasting with daily granularity.

**Anti-pattern**:

```pigment
// 1,825 days of iteration
'Daily Cash Balance' =
  PREVIOUSOF('Daily Cash Balance', 'Starting Cash') +
  'Daily Inflows' - 'Daily Outflows'
```

**Question**: Is daily granularity necessary?

**Optimized**: Use monthly granularity if acceptable.

```pigment
// 60 months of iteration
'Monthly Cash Balance' =
  PREVIOUSOF('Monthly Cash Balance', 'Starting Cash') +
  'Monthly Inflows' - 'Monthly Outflows'
```

**Improvement**: 30x fewer iterations (1,825 days → 60 months).

### Hybrid Approach: Monthly + Daily Detail

**Pattern**: Monthly for most periods, daily for current period.

```pigment
// Monthly for historical
'Historical Monthly Balance' =
  'Monthly Balance'[FILTER: Month < Month."Current Month"]

// Daily for current month only
'Current Month Daily Balance' =
  PREVIOUSOF('Current Month Daily Balance'[FILTER: Day IN 'Current Month'],
    'Historical Monthly Balance'[SELECT: Month = Month."Last Month"]) +
  'Daily Change'
```

**Benefit**: Fast historical calculation, detailed current period.

## Common Iterative Calculation Patterns

### Pattern 1: Inventory Balance

```pigment
'Ending Inventory' =
  PREVIOUSOF('Ending Inventory', 'Starting Inventory') +
  'Purchases' - 'Sales'
```

**Optimization**:

- Subset to recent periods
- Use monthly instead of daily if possible
- Reduce product dimensionality where appropriate

### Pattern 2: Cash Flow

```pigment
'Cash Balance' =
  PREVIOUSOF('Cash Balance', 'Opening Balance') +
  'Inflows' - 'Outflows'
```

**Optimization**:

- Pre-compute starting balance for current year
- Use monthly granularity
- Consider separate metrics for different cash accounts

### Pattern 3: Employee Headcount

```pigment
'Headcount' =
  PREVIOUSOF('Headcount', 0) + 'Hires' - 'Departures'
```

**Optimization**:

- Use FILLFORWARD for static assignments
- Aggregate by department before iterating
- Subset to current fiscal year

### Pattern 4: Loan Balance

```pigment
'Loan Balance' =
  PREVIOUSOF('Loan Balance', 'Principal') - 'Payment'
```

**Optimization**:

- Calculate only for active loans
- Use monthly payments instead of daily
- Pre-compute for historical periods

## Performance Monitoring

### Signs of Iterative Calculation Issues

1. **Timeouts**: Calculation doesn't complete
2. **Long computation times**: >10 seconds for simple inputs
3. **Profiler shows scope loss**: On time dimension
4. **User complaints**: Slow response when updating values

### Measuring Impact

**Before optimization**:

- Note computation time in profiler
- Count number of iteration steps (time periods)
- Check dimensionality (how many chains)

**After optimization**:

- Compare computation time
- Verify correctness
- Check scope in profiler

**Expected improvements**:

- Subsetting: 5-20x faster
- FILLFORWARD vs PREVIOUS: 10-50x faster
- Reduced dimensionality: 10-1000x faster
- Granularity change: 10-30x faster

## Best Practices Summary

1. **Subset time dimensions**: Limit iteration window to relevant periods
2. **Use FILLFORWARD when possible**: Non-iterative is faster
3. **Reduce dimensionality**: Aggregate before iterating
4. **Pre-compute starting points**: Don't iterate from the beginning of time
5. **Consider granularity trade-offs**: Monthly vs daily vs weekly
6. **Use CUMULATE for simple totals**: Optimized for summation
7. **Profile regularly**: Measure impact of optimizations
8. **Accept trade-offs**: Sometimes granularity or detail must be sacrificed

## When Iterative Calculations Are Unavoidable

Some calculations require iteration:

**Cash flow with complex logic**:

```pigment
'Cash Balance' =
  PREVIOUSOF('Cash Balance', 'Starting') +
  IF('Cash Balance' < 'Minimum', 'Credit Line Draw', 0) +
  'Inflows' - 'Outflows'
```

**Inventory with reorder logic**:

```pigment
'Inventory' =
  PREVIOUSOF('Inventory', 'Starting') +
  IF(PREVIOUSOF('Inventory', 'Starting') < 'Reorder Point', 'Order Quantity', 0) +
  'Receipts' - 'Sales'
```

**In these cases**:

- Optimize what you can (subsetting, dimensionality)
- Accept the performance cost
- Consider if the complexity is truly necessary

## See Also

- [Iterative Calculation (PREVIOUS & PREVIOUSOF)](../writing-pigment-formulas/functions_iterative_calculation.md) - Full spec: circular dependencies, configuration, syntax, debugging
- [Performance Scoping Patterns](./performance_scoping_patterns.md) - Understanding scope loss in iterations
- [Performance Formula Optimization](./performance_formula_optimization.md) - General formula optimization
- [Time and Date Functions](../writing-pigment-formulas/functions_time_and_date.md) - FILLFORWARD, SELECT vs PREVIOUS/PREVIOUSOF
