# Performance Profiling (Modeler Agent)

Measure compute performance with profiling tools, parse their output, and report findings to the user. For scope mechanics and formula fixes, see [./performance_scoping_patterns.md](./performance_scoping_patterns.md). For the full performance loop, see [./performance_troubleshooting_workflow.md](./performance_troubleshooting_workflow.md).

## Prerequisites

| Tool | Use when |
|---|---|
| `tool:get_top_blocks_by_performance` | Hotspot block unknown; rank blocks app-wide over a time window |
| `tool:performance_profile_change` | Slow action reproduced; you have `change_id` from audit trail |

## Workflow

1. **Triage (optional).** `tool:get_top_blocks_by_performance` with `scenario_id`, `range_start`, `range_end`, `top_n`, and `criteria`: `ExecutionTimeSumMs` (total cost), `ExecutionTimeAvgMs` (typical cost), `ExecutionCount` (churn), `CombinedCardinality` (data volume, no execution history needed).
2. **Reproduce** the slow input or formula change.
3. **Profile.** `tool:performance_profile_change` with `change_id` (UUID).
4. **Analyze** using sections below; map `Blocks:` to formulas.
5. **Fix one change at a time.** New `change_id` → re-profile → compare `Duration` and scope.

**Board-render fork:** Low total execution time but slow board load → `skill:designing-boards`, not formula work.

---

## Top blocks output

```
Top N blocks ranked by performance:
- {block_id} ({block_type}) — cardinality={n}, executions={count}, avg_ms={avg}, sum_ms={sum}
```

Missing `job_profile` → widen the time window or switch `criteria`. Then profile a change on the suspect block.

---

## Change profile output

```
Change profiled successfully. N execution(s) found.

Executions:
1. **{job_type}**
   - Id: {uuid}
   - Blocks: Metric(`uuid`), ...
   - Dimensions: uuid1, uuid2, ...
   - Ready at: Xms, Executed at: Yms, Duration: Zms
   - Effective scope: {text}
   - Output scope: {text}
   - Impacted data Members were viewing: yes|no   ← optional
   - Contention while impacting data Members were viewing: {n}ms   ← optional
   - Depends on: uuid, ...   ← optional
```

| Field | Meaning | Tell the user |
|---|---|---|
| Ready at | Wait for dependencies | Ready at |
| Executed at | Ready + queue contention | Executed at |
| Duration | Compute time | Duration |
| Effective scope | Scope used | Effective scope |
| Output scope | Scope passed downstream | Output scope |
| Impacted data Members were viewing | Recomputed (directly or indirectly) data a Member was viewing at change time | Impacted viewed data |
| Contention while impacting data Members were viewing | Tail of the wait that delayed what Members saw; the rest of the wait was deferrable | Contention while impacting viewed data |
| Depends on | Upstream execution IDs | Dependencies |

**Block labels:** `Metric(...)`, `List(...)`, `Table(...)`, `Cycle(...)`, `Block(app:...)`.

**Scope text:**

| Text | Meaning |
|---|---|
| `no change` | No cells written |
| `no scope, full computation` | Full recompute (X = 0) |
| `dim:uuid (N modalities), ...` | Scoped to N modalities per dimension |

### X/Y notation

- **Y** = count on `Dimensions:` line
- **X** = count of `dim:` entries in `Effective scope:`
- Target **X = Y** on hot paths

| Effective | Output | Interpretation |
|---|---|---|
| `dim:...` | Same | Scope preserved |
| `dim:...` | More dims | Scope introduced downstream |
| any | `no change` | Ran, no output (still check Duration) |
| `no scope, full computation` | — | Scope-loss origin candidate |

First `no scope, full computation` → inspect that block's formula (`REMOVE`, `CUMULATE`, `PREVIOUS`, `RANK`).

### Time and dependencies

- **Optimization order:** focus first on executions with `Impacted data Members were viewing: yes` — they gate what Members see refreshing — then the others. `no` executions are deliberately deferred, so their extra contention alone is not a defect.
- Sort by **Duration**; flag > 1000 ms or dominant wall-time share.
- **Contention** = Executed at − Ready at (large vs Duration → workload/queueing, not formula).
- **Contention split.** `Contention while impacting data Members were viewing` is the tail of that same wait, ending when the execution ran; the earlier part elapsed while the execution could still be deferred. The line appears only for executions that ended up impacting viewed data. Compare it against the whole contention:
  - **Much smaller than the contention** → the execution sat in the queue while no Member was waiting on it, then one opened something depending on it and it ran shortly after. This is the explanation to give whenever an execution appears far down the timeline yet is marked as impacting viewed data: for almost all of its wait, it was delaying nobody. Neither the wait nor that block's formula is the problem.
  - **Close to the contention** → it waited while already delaying Members. That is a queueing problem, still not a formula problem.
  - **Line absent** → the execution never impacted viewed data; its whole wait was deferrable and is not a defect on its own.
- **Wall time** ≈ max(Executed at + Duration) across executions.
- Match `Depends on` UUIDs to upstream `Id:` lines; ancestors appear earlier in the list.

### Patterns

| Pattern | Signature | Action |
|---|---|---|
| Cascading scope loss | Scoped runs, then first `no scope, full computation`, rest full/no change | Defer `REMOVE`/aggregations; see scoping patterns doc |
| No change, high Duration | `Output scope: no change` and Duration > 500 ms | Add earlier `FILTER`/`EXCLUDE` |
| High contention | Executed at ≫ Ready at on many rows | Broad scope or too many parallel branches |
| Late execution impacting viewed data | `Impacted data Members were viewing: yes`, Executed at ≫ Ready at, and `Contention while impacting data Members were viewing` far below the contention | Explain the late pickup (Member opened something depending on it mid-change); do not chase that block's formula |

---

## Report to the user

Include: execution count, approximate wall time, permission filter note, chain with natural block names, X/Y per step, slowest step, scope-loss origin, one recommendation (prioritize steps that impacted data Members were viewing). When a step impacted viewed data only for the tail of its wait, say why it appears late in the timeline rather than presenting it as a slow step.

**Vocabulary:** Say ready at, executed at, duration, scope, dependency, impacted viewed data, contention while impacting viewed data. Do not say execution_id, time_schedule_ms, effective_scope, clauses, impacted_viewed_data, time_contention_while_impacting_viewed_data_ms.

---

## See Also

- [./performance_scoping_patterns.md](./performance_scoping_patterns.md)
- [./performance_formula_optimization.md](./performance_formula_optimization.md)
- [./performance_troubleshooting_workflow.md](./performance_troubleshooting_workflow.md)
