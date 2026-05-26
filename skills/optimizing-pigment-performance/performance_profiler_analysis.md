# Analyzing a Change Profile Response

## Introduction

When the `performance_profile_change` tool returns a profile, it arrives as formatted markdown text. This guide explains how to read that text, derive scope chips and scope notation, identify bottlenecks, and produce a user-friendly analysis.

---

## Actual Text Format Received

The tool returns text in this exact shape:

```
Change profiled successfully. N execution(s) found.

[Note: some executions may be hidden due to permission restrictions.]

Executions:
1. **{job_type}**
   - Id: {execution_uuid}
   - Blocks: {block_id_1}, {block_id_2}, ...
   - Dimensions: {uuid_1}, {uuid_2}, ...
   - Ready at: {time_schedule_ms}ms, Executed at: {executed_at_ms}ms, Duration: {time_execution_ms}ms
   - Effective scope: {scope_text}
   - Output scope: {scope_text}
   - Depends on: {execution_uuid_a}, {execution_uuid_b}, ...    ← only present if there are ancestors

2. **{job_type}**
   ...
```

---

## Reading Each Field

| Text field | What it represents | User-facing label |
|---|---|---|
| `Ready at: Xms` | Time until all dependencies finished | Ready at |
| `Executed at: Yms` | Time until execution actually started (= ready + contention) | Executed at |
| `Duration: Zms` | Actual computation time | Duration |
| `Effective scope:` | Scope used during computation | Effective scope |
| `Output scope:` | Scope of changed cells passed to downstream executions | Output scope |
| `Depends on: uuid_a, uuid_b, ...` | Full list of upstream execution IDs this one depended on | Dependencies |

### Block ids

Each block in the `Blocks:` line is formatted as:

| Text | Meaning |
|---|---|
| `Metric(\`uuid\`)` | Metric block |
| `List(\`uuid\`)` | List block |
| `Table(\`uuid\`)` | Table block |
| `Cycle(\`uuid\`)` | Cycle block (iterative calculation) |
| `Block(app:\`uuid\`)` | Unknown block, application ID shown |

### Scope text

The `Effective scope:` and `Output scope:` lines use one of these forms:

| Text | Meaning |
|---|---|
| `no change` | No cells changed in this execution |
| `no scope, full computation` | All cells recomputed (no dimension restriction) |
| `dim:uuid1 (N modalities), dim:uuid2 (M modalities)` | Scoped: only N/M modalities per dimension |

---

## Scope Notation X/Y

Derive the X/Y notation per execution from the text:

- **Y** = count of comma-separated UUIDs in the `Dimensions:` line
- **X** = count of `dim:uuid` entries in the `Effective scope:` text

**Examples**:

```
Dimensions: uuid1, uuid2, uuid3                             → Y = 3
Scope: no scope, full computation                           → X = 0 → "0/3"
Scope: [dim:uuid1 (2 modalities), dim:uuid2 (5 modalities)] → X = 2 → "2/3"
```

The target is X = Y (fully scoped, best performance).

---

## Time Analysis

### Identifying slow executions

Sort by `Duration` descending. Flag executions where:
- Duration > 10000 ms (10 seconds)
- Duration is disproportionate relative to the chain depth or block count

### Identifying contention

Contention = `Executed at` - `Ready at`. If the gap is large compared to Duration, the system was overloaded and many executions were queued. This is a workload issue, not a formula issue.

### Total wall time

The overall change wall time is approximately the maximum `Executed at + Duration` value across all executions.

---

## Dependency Chain Analysis

Each execution's `Depends on:` line lists the full UUIDs of its direct ancestors. Cross-reference these with the `Id:` lines of other executions to build the exact dependency graph:

1. **Match by ID** - the UUIDs in `Depends on:` are the `Id:` values of upstream executions.
2. **Execution order** - executions are listed in scheduling order, so ancestors always appear before their dependents.
3. **Ready at values** - an execution's `Ready at` reflects when its slowest direct ancestor completed.

### Tracing scope loss

Read executions top-to-bottom. For each `Effective scope:` line:

```
"no change"                     → gray, no output
"no scope, full computation"    → X = 0, full recomputation
"[dim:... ]"                    → X > 0, scoped

First execution where X drops to 0 → scope loss origin
```

The block at that execution is the candidate for optimization (look for REMOVE, CUMULATE, time functions, RANK in its formula).

---

## Pattern Recognition and Recommendations

### Cascading scope loss (most common)

**Signature**: first executions show `[dim:...]` scope text, then one drops to `no scope, full computation`, all subsequent ones stay at `no scope, full computation` or `no change`.

**How to spot it**: first execution where `Effective scope:` switches to `no scope, full computation`.

**Recommendation**: Check that block's formula for REMOVE, CUMULATE, PREVIOUS, YEARTODATE, RANK, or any aggregation requiring all cells. If the aggregation is only needed for reporting, move it to the end of the chain.

### No change with long duration

**Signature**: `Output scope: no change` but Duration is still high (>500ms).

**How to spot it**: `Output scope: no change` AND Duration > 500ms.

**Recommendation**: Computation ran but had no effect. Add earlier filtering in the formula to prevent triggering this execution for changes that won't affect it.

### High contention

**Signature**: `Executed at` is much larger than `Ready at` across many executions (large contention gap).

**Recommendation**: The change triggered too many parallel executions. This may indicate overly broad scope or many independent branches in the computation graph.

---

## Producing a User-Friendly Summary

### What to include

1. **Overview**: total executions, approximate total wall time, whether results are filtered by permissions.
2. **Execution chain**: for each execution -
   - Block label(s) using natural names (Metric, List, Table...)
   - Job type in natural language ("formula recalculation", not "Formula")
   - Ready at / Executed at / Duration
   - Scope notation (X/Y)
3. **Bottleneck callout**: slowest execution(s) and the first scope loss.
4. **Recommendations**: based on the pattern identified above.

### Vocabulary rules

- Never say "execution_id", "ancestor_execution_ids", "time_schedule_ms", "effective_scope", "output_scope", "clauses", "criteria".
- Say "ready at", "executed at", "duration", "scope", "dependency", "metric", "list".
- "3/3" notation is user-friendly. "Fully scoped" is also fine.

### Example narrative

Given this profile text:

```
Change profiled successfully. 5 execution(s) found.

Executions:

1. **Input**
   - Blocks: Metric(`aaa`)
   - Dimensions: d1, d2
   - Ready at: 0ms, Executed at: 0ms, Duration: 10ms
   - Effective scope: [dim:d1 (1 modalities), dim:d2 (1 modalities)]
   - Output scope: [dim:d1 (1 modalities), dim:d2 (1 modalities)]

2. **Formula**
   - Id: bbb-exec-uuid
   - Blocks: Metric(`bbb`)
   - Dimensions: d1, d2
   - Ready at: 10ms, Executed at: 10ms, Duration: 150ms
   - Effective scope: [dim:d1 (1 modalities), dim:d2 (1 modalities)]
   - Output scope: [dim:d1 (1 modalities), dim:d2 (1 modalities)]
   - Depends on: aaa-exec-uuid

3. **Formula**
   - Id: ccc-exec-uuid
   - Blocks: Metric(`ccc`)
   - Dimensions: d1, d2, d3
   - Ready at: 160ms, Executed at: 160ms, Duration: 5200ms
   - Effective scope: no scope, full computation
   - Output scope: no scope, full computation
   - Depends on: bbb-exec-uuid

4. **Formula**
   - Id: ddd-exec-uuid
   - Blocks: Metric(`ddd`)
   - Dimensions: d1, d2, d3
   - Ready at: 5360ms, Executed at: 5360ms, Duration: 4800ms
   - Effective scope: no scope, full computation
   - Output scope: no change
   - Depends on: ccc-exec-uuid

5. **Formula**
   - Id: eee-exec-uuid
   - Blocks: Metric(`eee`)
   - Dimensions: d1, d2, d3
   - Ready at: 10160ms, Executed at: 10160ms, Duration: 3100ms
   - Effective scope: no scope, full computation
   - Output scope: no scope, full computation
   - Depends on: ccc-exec-uuid
```

**Analysis**:
- Execution 1: scope 2/2 (input, fast)
- Execution 2: scope 2/2 (formula, fast)
- Execution 3: scope 0/3 - **scope loss origin** (5.2s)
- Execution 4: scope 0/3 (ran but no output change, 4.8s wasted)
- Execution 5: scope 0/3 (3.1s)

**Narrative to produce**:

> The change triggered 5 computations with a total duration of ~13 seconds.
>
> The first two steps ran efficiently with full scope (2/2) - only the changed cells were recalculated. However, execution 3 lost its scope and required full recomputation across all 3 dimensions (5.2s). This was the scope loss origin. Executions 4 and 5 inherited this scope loss.
>
> Execution 4 (4.8s) ran fully but produced no output change - this time was wasted.
>
> **Recommendation**: Investigate execution 3's formula for REMOVE or aggregation operators. If the scope-losing calculation is only needed for reporting, move it to the end of the chain. Fixing this could reduce total time by ~57%.

---

## See Also

- [./performance_profiler_usage.md](./performance_profiler_usage.md) - Profiler concepts, scope chips, notation
- [./performance_scoping_patterns.md](./performance_scoping_patterns.md) - How scope propagates and when it's lost
- [./performance_formula_optimization.md](./performance_formula_optimization.md) - Formula-level fixes for scope loss
- [./performance_troubleshooting_workflow.md](./performance_troubleshooting_workflow.md) - End-to-end audit methodology
