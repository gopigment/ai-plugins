# Performance Access Rights

## Introduction

Access Rights (AR) in Pigment control which users can see which data. While essential for security and governance, access rights can significantly impact performance, especially in large applications with complex permission structures.

This guide covers how access rights affect performance, optimization patterns for AR-heavy formulas, and the ISDEFINED(User) technique for eliminating unnecessary AR computation.

## Understanding Access Rights Performance Impact

### How Access Rights Work

**Basic flow**:

1. User requests data
2. System retrieves metric data
3. System applies access rights rules
4. System filters out cells the user cannot access
5. System returns visible data to user

**Performance cost**: Steps 3-4 add computation overhead.

### Access Rights Processing Steps

**Technical implementation**:

1. **Metrics data join**: All relevant metrics are joined together
2. **AR data join**: Result is joined with AR rules (most costly step)
3. **Cell filtering**: Cells with "no read" permission are removed

**Key insight**: The AR join (step 2) is the most expensive operation.

### When Access Rights Impact Performance

**High impact scenarios**:

- **Many metrics with AR**: Each metric requires AR evaluation
- **Complex AR rules**: Rules with multiple dimensions or conditions
- **Large user base**: AR computed for many users
- **Frequent AR changes**: Rules that change often
- **Dimension-heavy AR**: AR rules on high-cardinality dimensions

**Low impact scenarios**:

- **Few metrics with AR**: Limited AR evaluation needed
- **Simple AR rules**: User-only or single-dimension rules
- **Small user base**: AR computed for few users
- **Static AR**: Rules that rarely change

## The ISDEFINED(User) Pattern

### The Problem: Computing AR for All Users

**Anti-pattern**:

```pigment
'Revenue with AR' = 'Revenue'[AR: 'Revenue AR Rules']
```

**Problem**: Computes access rights for **all users in the system**, even users who don't have access to the application.

**Example**:

- Application has 50 users with access
- Workspace has 500 total users
- System computes AR for all 500 users
- 450 users will never see this data

**Performance impact**: 10x unnecessary computation.

### The Solution: ISDEFINED(User)

**Optimized pattern**:

```pigment
'Revenue with AR' =
  IFDEFINED(User,
    'Revenue'[AR: 'Revenue AR Rules']
  )
```

**How it works**:

1. `ISDEFINED(User)` checks if the current user has access to the application
2. AR computation only happens for users with application access
3. Users without access get BLANK (no computation)

**Performance impact**: Computes AR only for relevant users.

### Real-World Example

**Scenario**: Financial planning application with complex AR.

**Before ISDEFINED(User)**:

- 100 users with app access
- 1,000 total users in workspace
- 20 metrics with AR
- Computation time: 15 seconds per update

**After ISDEFINED(User)**:

```pigment
// Applied to all AR metrics
'Metric with AR' =
  IFDEFINED(User,
    'Base Metric'[AR: 'AR Rules']
  )
```

**Result**:

- Computation time: 1.5 seconds per update
- **Improvement**: 10x faster

### When to Use ISDEFINED(User)

**Always use when**:

- Metric has access rights applied
- Application has subset of total workspace users
- Performance is a concern

**Pattern to apply**:

```pigment
// Wrap any AR application
IFDEFINED(User, [metric with AR])
```

**Exception**: When all workspace users have app access (rare).

## Selective vs Broad Access Rights Application

### The Question

Should you apply AR to only the metrics that need it, or apply AR broadly across all metrics?

**Hypothesis**: Applying AR selectively might improve performance by reducing AR computation.

### The Reality

**Applying AR selectively does NOT significantly improve performance**.

**Why**: The expensive operation is the AR join (step 2), which happens regardless of how many metrics have AR. The final filtering step (step 3) is much cheaper.

**Example**:

- Table with 30 metrics on Account dimension
- Applying AR to 5 metrics vs 30 metrics
- Performance difference: Minimal (~5% improvement)

### Optimization Opportunity: Row Filtering

**Better approach**: Apply AR broadly to enable row-level filtering.

**How it works**:

1. If AR is applied to **all metrics**, the system can remove entire rows
2. If AR is applied selectively, the system must filter cell-by-cell

**Example**:

- Account dimension with 10,000 accounts
- User has access to 1,000 accounts
- With broad AR: Filter 9,000 rows early
- With selective AR: Filter cells individually

**Performance impact**: Row filtering is much faster than cell filtering.

### Best Practice

**Apply AR consistently**:

- If a dimension needs AR, apply it to all metrics on that dimension
- Enables row-level filtering optimization
- Simplifies governance and maintenance

**Don't apply AR selectively** to "save performance" - it doesn't work.

## User-Dimensioned Access Rights

### Special Case: User-Only AR Rules

**Pattern**: AR rules dimensioned only on User (no other dimensions).

**Example**:

```pigment
// AR rule: User dimension only
'Can View' = User.'Department' = "Finance"
```

**Optimization opportunity**: These rules can be inlined as constants, avoiding joins entirely.

**Performance impact**: Significantly faster than multi-dimensional AR rules.

### When to Use User-Only AR

**Use when**:

- Access is based solely on user attributes
- No data-dependent permissions needed
- Simple yes/no access patterns

**Example use cases**:

- Department-based access
- Role-based access
- Region-based access (if user has region attribute)

**Don't use when**:

- Access depends on data values
- Complex conditional permissions
- Row-level security needed

## Optimizing AR-Heavy Formulas

### Pattern 1: Scope Before AR

**Anti-pattern**:

```pigment
// AR applied to all data, then filtered
'Result' = 'Revenue'[AR: 'AR Rules'][FILTER: 'Product'.'Active' = TRUE]
```

**Optimized pattern**:

```pigment
// Filter first, then apply AR
'Result' = 'Revenue'[FILTER: 'Product'.'Active' = TRUE][AR: 'AR Rules']
```

**Why better**: AR computation on smaller dataset.

### Pattern 2: Aggregate Before AR

**Anti-pattern**:

```pigment
// AR at transaction level, then aggregate
'Customer Total' = 'Transaction Amount'[AR: 'Transaction AR'][BY: 'Transaction'.'Customer']
```

**Optimized pattern**:

```pigment
// Aggregate first, then apply AR
'Customer Total' = 'Transaction Amount'[BY: 'Transaction'.'Customer'][AR: 'Customer AR']
```

**Why better**: AR computation on aggregated data (fewer cells).

**Note**: Only works if AR rules are at Customer level, not Transaction level.

### Pattern 3: Apply AR Once

**Anti-pattern**:

```pigment
// Multiple AR applications
'Step 1' = 'Revenue'[AR: 'AR Rules']
'Step 2' = 'Step 1' * 'Growth Rate'[AR: 'AR Rules']
'Step 3' = 'Step 2' + 'Fixed Costs'[AR: 'AR Rules']
```

**Optimized pattern**:

```pigment
// AR applied once at the end
'Step 1' = 'Revenue'
'Step 2' = 'Step 1' * 'Growth Rate'
'Step 3' = 'Step 2' + 'Fixed Costs'
'Final with AR' = 'Step 3'[AR: 'AR Rules']
```

**Why better**: Single AR computation instead of three.

### Pattern 4: ISDEFINED(User) Wrapper

**Standard pattern**:

```pigment
// Always wrap AR in ISDEFINED(User)
'Metric with AR' =
  IFDEFINED(User,
    'Base Metric'[AR: 'AR Rules']
  )
```

**Benefit**: Eliminates computation for users without app access.

## Access Rights and Consolidation

### AR Never Occurs Before Consolidation

**Important**: Access rights are applied **after** consolidation, not before.

**Implication**: Consolidation performance is not affected by AR.

**Example**:

```pigment
// Consolidation happens first
'Total Revenue' = 'Revenue'[REMOVE: Product]

// AR applied after consolidation
'Total Revenue with AR' = 'Total Revenue'[AR: 'AR Rules']
```

**Performance**: Consolidation speed is the same regardless of AR.

## Monitoring AR Performance

### Signs of AR Performance Issues

1. **Slow table loads**: Tables with many AR metrics load slowly
2. **User-specific slowness**: Some users experience slower performance
3. **AR rule changes cause slowness**: Updating AR rules triggers slow recalculation
4. **Profiler shows AR overhead**: Long computation times on AR metrics

### Measuring AR Impact

**Test without AR**:

1. Create duplicate metric without AR
2. Compare computation time
3. Difference = AR overhead

**Example**:

- Metric with AR: 5 seconds
- Same metric without AR: 0.5 seconds
- AR overhead: 4.5 seconds (90% of time)

### Acceptable AR Overhead

**General guidelines**:

- Simple AR rules: 10-50% overhead
- Complex AR rules: 50-200% overhead
- Very complex AR: 200-500% overhead

**If overhead is excessive**:

- Review AR rule complexity
- Consider simplifying rules
- Use ISDEFINED(User) pattern
- Apply AR less frequently in computation chain

## Best Practices for Access Rights Performance

### 1. Always Use ISDEFINED(User)

```pigment
// Standard pattern for all AR metrics
IFDEFINED(User, 'Metric'[AR: 'Rules'])
```

### 2. Apply AR Consistently

- Apply to all metrics on a dimension, not selectively
- Enables row-level filtering optimization

### 3. Keep AR Rules Simple

- Prefer user-only rules when possible
- Avoid complex multi-dimensional rules
- Minimize conditions in AR formulas

### 4. Apply AR Once

- Don't apply AR at every step of computation
- Apply at the end of the computation chain

### 5. Scope Before AR

- Filter and aggregate before applying AR
- Reduces dataset size for AR computation

### 6. Monitor AR Overhead

- Profile metrics with AR
- Measure overhead regularly
- Optimize if overhead is excessive

### 7. Consider AR Architecture

- Design AR structure early
- Balance security needs with performance
- Document AR strategy

## Common AR Anti-Patterns

### Anti-Pattern 1: No ISDEFINED(User)

```pigment
// Bad: Computes for all users
'Revenue with AR' = 'Revenue'[AR: 'Rules']

// Good: Computes only for app users
'Revenue with AR' = IFDEFINED(User, 'Revenue'[AR: 'Rules'])
```

### Anti-Pattern 2: Multiple AR Applications

```pigment
// Bad: AR at every step
'Step 1' = 'A'[AR: 'Rules']
'Step 2' = 'Step 1' + 'B'[AR: 'Rules']

// Good: AR once at end
'Step 1' = 'A'
'Step 2' = 'Step 1' + 'B'
'Final' = 'Step 2'[AR: 'Rules']
```

### Anti-Pattern 3: AR Before Aggregation

```pigment
// Bad: AR then aggregate
'Total' = 'Detail'[AR: 'Rules'][REMOVE: Product]

// Good: Aggregate then AR (if rules allow)
'Total' = 'Detail'[REMOVE: Product][AR: 'Rules']
```

### Anti-Pattern 4: Complex AR Rules

```pigment
// Bad: Very complex AR rule
'AR Rule' =
  ('User'.'Department' = "Finance" AND 'Account'.'Type' = "Revenue") OR
  ('User'.'Role' = "Manager" AND 'Region' = 'User'.'Region') OR
  ('User'.'Level' > 5)

// Better: Simplify or split into multiple rules
'AR Rule Finance' = 'User'.'Department' = "Finance" AND 'Account'.'Type' = "Revenue"
'AR Rule Manager' = 'User'.'Role' = "Manager" AND 'Region' = 'User'.'Region'
'AR Rule Senior' = 'User'.'Level' > 5
```

## Real-World AR Optimization Example

### Scenario: Multi-Regional Planning Application

**Original implementation**:

```pigment
// AR applied to 30 metrics individually
'Revenue Metric 1' = 'Base Revenue 1'[AR: 'Region AR']
'Revenue Metric 2' = 'Base Revenue 2'[AR: 'Region AR']
// ... 28 more metrics
```

**Problems**:

1. No ISDEFINED(User)
2. AR applied 30 times
3. Complex AR rules

**Optimized implementation**:

```pigment
// Base metrics without AR
'Base Revenue 1' = [formula]
'Base Revenue 2' = [formula]
// ... 28 more metrics

// Single AR wrapper metric
'Revenue with AR' =
  IFDEFINED(User,
    'Base Revenue'[AR: 'Simplified Region AR']
  )

// Simplified AR rule
'Simplified Region AR' = 'Region' = 'User'.'Region'
```

**Results**:

- Computation time: 12s → 2s (6x faster)
- AR rule complexity: Reduced
- Maintenance: Easier (single AR application)

## See Also

- [Performance Formula Optimization](./performance_formula_optimization.md) - General formula optimization including AR
- [Performance Sparsity Deep Dive](./performance_sparsity_deep_dive.md) - ISDEFINED patterns
- [Pigment Application Modeling: Security](../modeling-pigment-applications/modeling_principles.md) - AR design patterns
