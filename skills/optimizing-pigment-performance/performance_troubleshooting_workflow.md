# Performance Troubleshooting Workflow

## Introduction

Systematic performance troubleshooting is essential for identifying and resolving bottlenecks efficiently. This guide provides a step-by-step methodology for conducting performance audits, from initial assessment to optimization and verification.

## The Performance Troubleshooting Process

### Overview

```
1. Establish Baseline
   ↓
2. Identify Bottlenecks
   ↓
3. Analyze Root Causes
   ↓
4. Prioritize Optimizations
   ↓
5. Implement Changes
   ↓
6. Verify Improvements
   ↓
7. Document Findings
```

## Step 1: Establish Baseline

### Objective

Measure current performance to establish a baseline for comparison.

### Actions

**1.1 Identify Problem Areas**

Ask users:

- Which operations are slow?
- Which boards/tables have issues?
- When does slowness occur?
- Which users are affected?

**1.2 Reproduce the Issue**

- Perform the slow operation yourself
- Note the exact steps to reproduce
- Measure time taken
- Document user context (role, permissions)

**1.3 Open the Profiler**

- Make the change that triggers slow calculation
- Open the profiler
- Review the computation chain
- Note computation times

**1.4 Document Baseline Metrics**

Record:

- Total computation time
- Number of metrics in chain
- Scope values (X/Y notation)
- Slowest metrics
- User feedback on perceived slowness

### Example Baseline Documentation

```
Issue: Revenue forecast update takes 15 seconds
User: Finance Manager
Operation: Update Q1 forecast for Product A
Profiler results:
  - Total chain: 25 metrics
  - Slowest: 'Revenue Share' (8.2s)
  - Scope loss: At metric 'Total Revenue' (0/3)
  - Total time: 15.3s
```

## Step 2: Identify Bottlenecks

### Objective

Find the specific metrics or operations causing slowness.

### Actions

**2.1 Sort Profiler by Computation Time**

- Identify metrics taking >1 second
- Focus on top 5 slowest metrics
- Note their position in the chain

**2.2 Check Scope Propagation**

For each slow metric:

- What is its scope? (X/Y)
- Where was scope lost?
- Is scope loss necessary?

**2.3 Examine Metric Formulas**

For slow metrics:

- Read the formula
- Identify modifiers used
- Check for anti-patterns
- Note complexity

**2.4 Check Metric Dimensions**

- How many dimensions?
- What is the cardinality of each?
- Total possible cells?
- Actual cells with data (sparsity)?

**2.5 Identify Patterns**

Common bottleneck patterns:

- Early scope loss
- Dense boolean metrics
- Unnecessary REMOVE operations
- Iterative calculations over long horizons
- Complex AR rules
- Unoptimized formula structure

### Example Bottleneck Analysis

```
Bottleneck identified: 'Revenue Share' metric

Formula: 'Revenue' / 'Revenue'[REMOVE: Product, Region]

Issues found:
1. Scope loss: REMOVE causes 0/3 scope
2. Position: Early in chain (metric 3 of 25)
3. Impact: All downstream metrics lose scope
4. Dimensions: Product (1000) × Region (50) × Month (24)
5. Computation: 1,200,000 cells recalculated

Root cause: Unnecessary early aggregation
```

## Step 3: Analyze Root Causes

### Objective

Understand why the bottleneck exists and what's causing it.

### Analysis Framework

**3.1 Scope Analysis**

Questions:

- Where is scope first lost?
- Is the scope loss necessary?
- Can scope-losing operations be deferred?
- Are there unnecessary REMOVE operations?

**3.2 Sparsity Analysis**

Questions:

- Are metrics sparse or dense?
- Are ISBLANK/ISNOTBLANK used?
- Could ISDEFINED be used instead?
- Are there dense boolean metrics?

**3.3 Formula Structure Analysis**

Questions:

- Is filtering done early?
- Are aggregations deferred?
- Is execution order optimized?
- Are there unnecessary intermediate calculations?

**3.4 Dimensional Analysis**

Questions:

- Are there too many dimensions?
- Is dimensionality higher than needed?
- Can dimensions be reduced?
- Are there high-cardinality dimensions?

**3.5 Iterative Calculation Analysis**

Questions:

- Are PREVIOUS/CUMULATE used?
- Over how many periods?
- Can the time horizon be subset?
- Can FILLFORWARD be used instead?

**3.6 Access Rights Analysis**

Questions:

- Are AR rules complex?
- Is ISDEFINED(User) used?
- Is AR applied multiple times?
- Can AR be simplified?

### Root Cause Documentation Template

```
Metric: [Metric Name]
Symptom: [What's slow]
Root Cause: [Why it's slow]
Contributing Factors:
  - Factor 1
  - Factor 2
  - Factor 3
Optimization Opportunities:
  - Opportunity 1
  - Opportunity 2
```

## Step 4: Prioritize Optimizations

### Objective

Determine which optimizations to implement first based on impact and effort.

### Prioritization Matrix

**High Impact, Low Effort** → Do First

- Add ISDEFINED(User) to AR metrics
- Use ISDEFINED instead of ISBLANK
- Add early FILTER clauses
- Remove unnecessary REMOVE operations

**High Impact, Medium Effort** → Do Second

- Defer aggregations to end of chain
- Restructure formula execution order
- Subset time dimensions for iterations
- Simplify AR rules

**High Impact, High Effort** → Do Third

- Redesign dimensional structure
- Split complex metrics
- Refactor computation chains
- Change granularity (daily → monthly)

**Low Impact** → Consider Later

- Micro-optimizations
- Cosmetic formula improvements
- Minor refactoring

### Prioritization Criteria

**Impact assessment**:

- How much time will this save?
- How many users are affected?
- How often is this operation performed?
- What is the downstream impact?

**Effort assessment**:

- How complex is the change?
- How many metrics need updating?
- Are there dependencies?
- What is the testing effort?

**Risk assessment**:

- Could this break existing functionality?
- Are there downstream dependencies?
- How easy is it to roll back?

### Example Prioritization

```
Optimization Options:

1. Add ISDEFINED(User) to 'Revenue Share'
   Impact: High (50% time reduction)
   Effort: Low (5 minutes)
   Risk: Low
   Priority: 1 - DO FIRST

2. Move 'Revenue Share' to end of chain
   Impact: High (70% time reduction)
   Effort: Medium (restructure 10 metrics)
   Risk: Medium (dependencies)
   Priority: 2 - DO SECOND

3. Reduce Product dimension cardinality
   Impact: Medium (30% time reduction)
   Effort: High (data model change)
   Risk: High (affects many metrics)
   Priority: 3 - DO THIRD
```

## Step 5: Implement Changes

### Objective

Apply optimizations systematically and safely.

### Implementation Process

**5.1 Start with Quick Wins**

- Implement high-impact, low-effort changes first
- Verify each change before moving to the next
- Profile after each change

**5.2 Make One Change at a Time**

- Don't combine multiple optimizations
- Isolate the impact of each change
- Easier to debug if something breaks

**5.3 Test in Development First**

- Don't optimize in production directly
- Test changes thoroughly
- Verify correctness, not just speed

**5.4 Document Changes**

- Record what was changed
- Note the reason for the change
- Document expected impact

**5.5 Communicate Changes**

- Inform affected users
- Explain what was optimized
- Set expectations for improvement

### Implementation Checklist

Before implementing:

- [ ] Baseline metrics recorded
- [ ] Change is well-understood
- [ ] Testing plan defined
- [ ] Rollback plan prepared

During implementation:

- [ ] Change made in development
- [ ] Formula correctness verified
- [ ] Performance improvement measured
- [ ] No unintended side effects

After implementation:

- [ ] Change documented
- [ ] Users notified
- [ ] Monitoring in place

## Step 6: Verify Improvements

### Objective

Confirm that optimizations achieved the desired performance improvement.

### Verification Process

**6.1 Re-Profile the Operation**

- Perform the same operation as baseline
- Open the profiler
- Compare to baseline metrics

**6.2 Measure Improvement**

Calculate:

- Time reduction (seconds)
- Percentage improvement
- Scope improvement (X/Y notation)

**6.3 Verify Correctness**

Confirm:

- Results are identical to before
- No data discrepancies
- No broken dependencies
- Users see correct data

**6.4 Check for Regressions**

Test:

- Other operations still work
- No new slow operations introduced
- Downstream metrics unaffected

**6.5 Gather User Feedback**

Ask users:

- Is the operation faster?
- Any issues noticed?
- Satisfaction with improvement?

### Verification Documentation Template

```
Optimization: [What was changed]

Before:
  - Computation time: [X seconds]
  - Scope: [X/Y]
  - User feedback: [Slow/Acceptable/Fast]

After:
  - Computation time: [Y seconds]
  - Scope: [A/B]
  - User feedback: [Slow/Acceptable/Fast]

Improvement:
  - Time reduction: [X - Y seconds]
  - Percentage: [(X-Y)/X * 100%]
  - Scope improvement: [Description]

Correctness verified: [Yes/No]
Regressions found: [None/List]
User satisfaction: [Improved/Same/Worse]
```

## Step 7: Document Findings

### Objective

Create a record of the performance issue, analysis, and resolution for future reference.

### Documentation Components

**7.1 Issue Summary**

- What was slow
- When it was reported
- Who was affected
- Impact on users

**7.2 Analysis Summary**

- Root causes identified
- Bottlenecks found
- Contributing factors

**7.3 Optimization Summary**

- Changes implemented
- Rationale for each change
- Implementation details

**7.4 Results Summary**

- Performance improvements
- Time savings
- User feedback

**7.5 Lessons Learned**

- What worked well
- What didn't work
- Patterns to avoid in future
- Best practices reinforced

**7.6 Recommendations**

- Future optimization opportunities
- Preventive measures
- Monitoring suggestions

### Documentation Template

```
# Performance Optimization Report

## Issue
[Description of the performance problem]

## Analysis
### Bottlenecks Identified
1. [Bottleneck 1]
2. [Bottleneck 2]

### Root Causes
1. [Root cause 1]
2. [Root cause 2]

## Optimizations Implemented
### Change 1: [Name]
- Description: [What was changed]
- Rationale: [Why this change]
- Impact: [Expected improvement]

### Change 2: [Name]
- Description: [What was changed]
- Rationale: [Why this change]
- Impact: [Expected improvement]

## Results
- Baseline: [X seconds]
- After optimization: [Y seconds]
- Improvement: [Z%]
- User feedback: [Positive/Negative]

## Lessons Learned
- [Lesson 1]
- [Lesson 2]

## Recommendations
- [Recommendation 1]
- [Recommendation 2]
```

## Common Performance Troubleshooting Scenarios

### Scenario 1: Slow Board Load

**Symptoms**: Board takes >10 seconds to load.

**Investigation steps**:

1. Identify which views are slow
2. Check metrics in those views
3. Profile metric calculations
4. Look for scope loss or dense metrics

**Common causes**:

- Complex aggregations in views
- Dense boolean metrics
- Unnecessary dimensions
- Missing filters

### Scenario 2: Slow Input Response

**Symptoms**: After user input, calculation takes >5 seconds.

**Investigation steps**:

1. Profile the input operation
2. Trace the computation chain
3. Identify where scope is lost
4. Check for iterative calculations

**Common causes**:

- Early scope loss
- Long iterative calculations
- Unnecessary REMOVE operations
- Complex AR rules

### Scenario 3: Timeout Errors

**Symptoms**: Calculations timeout and don't complete.

**Investigation steps**:

1. Identify which operation times out
2. Check metric dimensions and cardinality
3. Look for iterative calculations
4. Check for very large datasets

**Common causes**:

- Iterative calculations over long horizons
- Very high-cardinality dimensions
- Dense metrics with many dimensions
- Unoptimized formulas

### Scenario 4: Inconsistent Performance

**Symptoms**: Sometimes fast, sometimes slow.

**Investigation steps**:

1. Identify pattern (which users, when, what data)
2. Check for user-specific factors (AR)
3. Look for data-dependent performance
4. Check for concurrent operations

**Common causes**:

- Complex AR rules
- Data-dependent sparsity
- Concurrent user operations
- Caching effects

## Performance Troubleshooting Checklist

### Initial Assessment

- [ ] Baseline metrics recorded
- [ ] Issue reproduced
- [ ] Profiler output captured
- [ ] User impact assessed

### Analysis

- [ ] Bottlenecks identified
- [ ] Root causes analyzed
- [ ] Scope propagation checked
- [ ] Sparsity assessed
- [ ] Formula structure reviewed

### Optimization

- [ ] Changes prioritized
- [ ] Quick wins implemented first
- [ ] One change at a time
- [ ] Testing completed

### Verification

- [ ] Performance improvement measured
- [ ] Correctness verified
- [ ] No regressions found
- [ ] User feedback collected

### Documentation

- [ ] Issue documented
- [ ] Analysis recorded
- [ ] Changes documented
- [ ] Lessons learned captured

## Best Practices for Performance Troubleshooting

1. **Always profile first** - Don't guess where the bottleneck is
2. **Establish baseline** - Measure before optimizing
3. **One change at a time** - Isolate the impact of each optimization
4. **Verify correctness** - Speed is useless if results are wrong
5. **Document everything** - Future you will thank present you
6. **Communicate changes** - Keep users informed
7. **Monitor after changes** - Watch for regressions
8. **Learn from patterns** - Build institutional knowledge

## See Also

- [Performance Profiler Usage](./performance_profiler_usage.md) - How to use the profiler
- [Performance Scoping Patterns](./performance_scoping_patterns.md) - Scope optimization
- [Performance Formula Optimization](./performance_formula_optimization.md) - Formula-level optimization
- [Performance Sparsity Deep Dive](./performance_sparsity_deep_dive.md) - Sparsity management
