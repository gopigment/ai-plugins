# Performance Profiler Usage

## Introduction

The Pigment formula profiler is a powerful computation tracking feature that helps you understand how metrics are calculated, where performance bottlenecks exist, and how scope propagates through your computation chains. Mastering the profiler is essential for effective performance optimization.

This guide covers practical profiler usage: what the profiler shows, how to interpret its output, and how to use it for performance debugging.

## What is the Profiler?

The profiler tracks the full cycle of metric computations from your input all the way to the final metric in your application. It shows:

- **Computation time** per metric
- **Scope propagation** through the computation chain
- **Dependencies** between metrics
- **Which metrics were recalculated** and why

## Accessing the Profiler

The profiler is available in the Pigment interface when you make changes to metrics. It automatically tracks computations triggered by your inputs or formula changes.

## Understanding the Scope Column

The most powerful feature of the profiler is the **Scope** column, which shows how scope propagates through your metrics.

### What is Scope?

**Scope** in Pigment means the part of a metric that is being computed because of a change in the metric itself or a metric it references in its formula.

**Example**: Imagine a metric with 2 dimensions: `Employee` and `Month`.

If your input affects only one employee in one month, Pigment should only calculate that intersection and leave the rest of the metric data untouched. This results in dramatically improved computation times.

All metrics that reference this initial metric should also have scoped computation, as long as you're not doing data transformations that require full recomputation.

### Scope Notation

The profiler displays the **outbound scope** (result of computation) on each dimension involved in the structure of every metric within the computation chain.

**Notation format**: `X/Y` where:

- `X` = number of dimensions with scope
- `Y` = total number of dimensions in the metric

**Examples**:

- `2/2` = Full scope across 2 dimensions
- `2/3` = Partial scope (2 dimensions scoped, 1 dimension added without scope)
- `0/3` = No scope (full recomputation required)

### Scope Color Codes

The profiler uses three color codes to represent scope behavior:

#### ⚫ Black Chip - Scope Preserved

**Meaning**: No change in scope. The metric simply carries forward the inbound scope it received, if any, without altering or extending it.

**What happens**: This scope will continue downstream to the next metrics in the chain.

**Example**: A simple formula like `'Revenue' * 1.1` where Revenue has scope `1/2` will also have scope `1/2` with a black chip.

**Interpretation**: ✅ Good - scope is being preserved efficiently.

#### 🔵 Blue Chip - Scope Introduction

**Meaning**: Positive scope change. The metric did not receive this specific inbound scope for one or more dimensions, but is adding it to the next dependent executions.

**What happens**: This newly introduced scope will be passed to downstream metrics, potentially triggering their recalculation.

**Example**:

```
Metric: 'Scoped Calc + Version'
Inbound scope: 2/3 (Employee and Month)
Formula applies values only to Version A
Result: Blue chip on Version dimension
Outbound scope: 3/3 with new scope on Version
```

**Interpretation**: ⚠️ Neutral - scope is being added, which is sometimes necessary but increases computation.

#### 🔘 Gray Chip - No Effective Change

**Meaning**: No change in values after metric computation. The metric computation yields the same result (zero delta), and no new values are passed downstream.

**What happens**: No downstream metrics will be recalculated since there are no outbound changes.

**Example**:

```
Metric filters data only for February
Upstream change: Employee 3 on January
Result: Gray chip (computation triggered but output unchanged)
```

**Interpretation**: ℹ️ Informational - computation was triggered but had no effect.

## Scope Patterns

### Full Scope (2/2)

When inputting a value into a metric, you receive **full scope** across all dimensions involved.

**Example**:

- Metric dimensions: `Employee`, `Month`
- Input: Change value for Employee "John" in "January"
- Scope: `2/2` (both dimensions scoped)
- Result: Only that one intersection is recomputed

**Best practice**: This is the ideal state. Try to preserve full scope through your computation chain.

### Partial Scope (2/3)

When a new dimension is added to a metric, you receive **partial scope**.

**Example**:

- Original metric: `Employee`, `Month` with scope `2/2`
- Formula: `'Original Metric' [ADD: Version]`
- Result: Scope `2/3` (Employee and Month scoped, Version not scoped)
- Computation: Must compute across all versions for the scoped Employee/Month

**Interpretation**: The original scope is preserved, but the metric's dimensionality has changed. More data points will be recalculated because of the additional dimension.

### No Scope (0/3)

Certain modeler actions can completely remove the scope of a metric.

**Common causes**:

1. **REMOVE modifier**: Aggregating across dimensions
2. **CUMULATE**: Cumulating across a dimension
3. **Complex transformations**: Calculations that require full recomputation

**Example**:

```pigment
// Calculate share of each employee/month vs total
'Revenue' / 'Revenue'[REMOVE: Employee, Month]
```

**Result**: Scope `0/3` - every cell must be recomputed because the denominator depends on all cells.

**Interpretation**: Sometimes unavoidable, but try to push these metrics to the end of the computation chain.

## When Scope Loss is Unavoidable

### CUMULATE and Time Functions

```pigment
'Monthly Revenue'[CUMULATE: Month]
```

**Result**: Scope lost on the `Month` dimension because Pigment needs to compute A to compute B (sequential dependency).

**Same applies to**:

- `PREVIOUS` or `PREVIOUSOF`
- `YEARTODATE`, `QUARTERTODATE`, `MONTHTODATE`
- `SHIFT` on a dimension

### REMOVE for Aggregation

```pigment
'Employee Revenue' / 'Employee Revenue'[REMOVE: Employee]
```

**Result**: Scope lost because the denominator requires computing across all employees.

**Why unavoidable**: To calculate the share, every cell needs to know the total, which depends on all cells.

### Complex Calculations Requiring Full Context

```pigment
// Ranking across all items
RANK('Sales'[REMOVE: Product])
```

**Result**: Scope lost because ranking requires knowing all values.

## Keeping Scope: Best Practices

### 1. Avoid Unnecessary REMOVE

**Anti-pattern**:

```pigment
'Metric'[REMOVE: Dimension] // when Dimension isn't needed downstream
```

**Better approach**: Keep the dimension if downstream metrics need it. Use mapping instead if there's a 1:1 relationship.

**Example**:

```pigment
// Instead of REMOVE to aggregate
'Transaction Amount'[REMOVE: Transaction ID]

// Consider using BY with a mapping
'Transaction Amount'[BY: 'Transaction'.'Customer']
```

### 2. Push Scope-Losing Metrics to the End

**Anti-pattern**:

```pigment
// Early in chain
Step 1: 'Revenue'[REMOVE: Product]
Step 2: 'Step 1' * 'Growth Rate'
Step 3: 'Step 2' + 'Fixed Costs'
// All subsequent steps lose scope
```

**Better approach**:

```pigment
// Keep scope as long as possible
Step 1: 'Revenue' * 'Growth Rate'
Step 2: 'Step 1' + 'Fixed Costs'
Step 3: 'Step 2'[REMOVE: Product] // Only at the end
```

### 3. Use Mappings Instead of REMOVE

When transitioning between dimensions with a 1:1 relationship, use mappings to maintain scope.

**Anti-pattern**:

```pigment
'Employee Salary'[REMOVE: Employee][ADD: Department]
```

**Better approach**:

```pigment
'Employee Salary'[BY: 'Employee'.'Department']
```

**Why better**: The BY modifier with a mapping can preserve scope better than REMOVE + ADD.

## Using the Profiler for Performance Debugging

### Step 1: Identify Slow Metrics

Look at the **computation time** column in the profiler. Sort by time to find the slowest metrics.

**What to look for**:

- Metrics taking >1 second
- Metrics with disproportionate computation time relative to their complexity
- Metrics that are surprisingly slow

### Step 2: Check Scope Propagation

For slow metrics, examine the **scope column**:

**Questions to ask**:

1. Does this metric have scope? (X/Y where X > 0)
2. Where was scope lost? (Look upstream for 0/Y)
3. Is the scope loss necessary?

### Step 3: Trace the Computation Chain

Follow the dependency chain in the profiler:

1. **Start with your input** - should show full scope
2. **Follow downstream** - watch for scope loss
3. **Identify the culprit** - which metric first loses scope unnecessarily?

### Step 4: Analyze Scope Loss Causes

When you find a metric with `0/X` scope, examine its formula:

**Common causes**:

- REMOVE modifier
- CUMULATE or time functions
- Complex aggregations
- Unnecessary full-table operations

### Step 5: Optimize and Re-Profile

After making changes:

1. Clear the profiler
2. Make the same input change
3. Compare computation times
4. Verify scope is better preserved

## Real-World Example: Debugging a Slow Calculation

### Scenario

A financial planning application has slow performance when users update monthly revenue forecasts.

### Profiler Analysis

```
Metric Chain:
1. 'Monthly Revenue Input' - 0.1s, scope 2/2 ⚫
2. 'Adjusted Revenue' - 0.2s, scope 2/2 ⚫
3. 'Revenue Share' - 5.2s, scope 0/3 🔘
4. 'Weighted Allocation' - 4.8s, scope 0/3 🔘
5. 'Final Forecast' - 3.1s, scope 0/3 🔘
```

### Problem Identified

Metric 3 ('Revenue Share') loses scope and causes all downstream metrics to lose scope.

**Formula**:

```pigment
'Adjusted Revenue' / 'Adjusted Revenue'[REMOVE: Product, Region]
```

### Solution

The share calculation needs to happen, but we can defer it:

**Optimized approach**:

```pigment
// Metrics 1-4: Keep scope
1. 'Monthly Revenue Input'
2. 'Adjusted Revenue'
3. 'Weighted Allocation' = 'Adjusted Revenue' * 'Weight'
4. 'Final Forecast' = 'Weighted Allocation' + 'Fixed Costs'

// Metric 5: Calculate share only at the end for reporting
5. 'Revenue Share' = 'Final Forecast' / 'Final Forecast'[REMOVE: Product, Region]
```

### Result

```
New Profiler Output:
1. 'Monthly Revenue Input' - 0.1s, scope 2/2 ⚫
2. 'Adjusted Revenue' - 0.2s, scope 2/2 ⚫
3. 'Weighted Allocation' - 0.3s, scope 2/2 ⚫
4. 'Final Forecast' - 0.2s, scope 2/2 ⚫
5. 'Revenue Share' - 5.0s, scope 0/3 🔘

Total time: 5.8s (was 13.4s)
Improvement: 57% faster
```

## Common Profiler Patterns

### Pattern 1: Cascading Scope Loss

**Symptom**: Scope lost early, all downstream metrics show `0/X`.

**Cause**: Unnecessary REMOVE or aggregation early in the chain.

**Solution**: Move aggregations to the end or use mappings.

### Pattern 2: Blue Chips Everywhere

**Symptom**: Many blue chips, increasing scope notation (2/3, 3/4, 4/5).

**Cause**: Repeatedly adding dimensions with ADD modifier.

**Solution**: Design metrics with final dimensionality from the start.

### Pattern 3: Gray Chips with Long Computation

**Symptom**: Gray chips but still slow computation times.

**Cause**: Computation is triggered but produces no output change.

**Solution**: Add filtering earlier to prevent unnecessary computation.

## Profiler Limitations

### What the Profiler Shows

- ✅ Computation time per metric
- ✅ Scope propagation
- ✅ Dependency chains
- ✅ Which metrics were recalculated

### What the Profiler Doesn't Show

- ❌ Memory usage
- ❌ Database query details
- ❌ Exact cell count computed
- ❌ Network latency

## Best Practices Summary

1. **Always profile before optimizing** - Don't guess where the bottleneck is
2. **Focus on scope preservation** - Black chips are good, aim for X/X scope
3. **Trace scope loss** - Find where scope is first lost unnecessarily
4. **Defer aggregations** - Push REMOVE and scope-losing operations to the end
5. **Use mappings** - Prefer BY with mappings over REMOVE + ADD
6. **Profile after changes** - Verify your optimization worked
7. **Document findings** - Note which patterns caused issues for future reference

## See Also

- [Performance Scoping Patterns](./performance_scoping_patterns.md) - Deep dive into scope propagation mechanics
- [Performance Formula Optimization](./performance_formula_optimization.md) - Formula-level optimization techniques
- [Performance Troubleshooting Workflow](./performance_troubleshooting_workflow.md) - Systematic performance audit methodology
