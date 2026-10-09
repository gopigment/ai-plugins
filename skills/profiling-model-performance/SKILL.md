---
name: profiling-model-performance
description: "Execution skill. Use when investigating slow inputs, slow calculations, or timeouts with the Performance Insights tools (performance_profile_change, performance_profile_change_scope_details, performance_slowest_changes, performance_changes_statistics, get_top_blocks_by_performance). Covers which tool to call, the includeAllExecutions default, reading change profiles (X/Y scope, contention, impacted viewed data), drilling into the scoped dimensions of some executions, locating the bottleneck, and comparing timings before/after a fix. Always profile before formula changes; never optimize from assumptions."
metadata:
  skill_path: /skills/profiling-model-performance/SKILL.md
  base_directory: /skills/profiling-model-performance
---

# Profiling Model Performance

Measure compute performance with the Performance Insights tools, locate the bottleneck, and verify fixes. This skill covers the tools and their output only:

- Formula fixes: `skill:writing-performant-formulas`; iterative horizons: `skill:iterating-with-previous-and-cycles`.
- Troubleshooting loop and reporting structure, and the in-product Profiler when these tools are unavailable: `skill:diagnosing-performance-issues`.

## Choose the Tool

| Tool | Use when |
|---|---|
| `tool:get_top_blocks_by_performance` | Hotspot block unknown; rank blocks app-wide over a time window |
| `tool:get_blocks_performance` | Blocks already known; get their stats without re-ranking the model |
| `tool:performance_slowest_changes` | No `changeId` known; rank the slowest user changes of an application over a time window (maximum 7 days) |
| `tool:performance_profile_change` | Slow action reproduced and you have its `changeId` (UUID) from the audit trail or a slowest-changes result; returns the execution chain with a high-level overview of scopes (counts only) |
| `tool:performance_profile_change_scope_details` | Profile already read; you need to know *which* dimensions an execution has and which ones its scopes restrict, for a few executions of that change |
| `tool:performance_changes_statistics` | `changeId`s known; compare aggregated timings across changes (e.g. before/after a fix) without full profiles |

## Workflow

1. **Triage (if the hotspot is unknown).** `tool:get_top_blocks_by_performance` with `range_start`, `range_end`, `top_n`, and `criteria`: `ExecutionTimeSumMs` (total cost), `ExecutionTimeAvgMs` (typical cost), `ExecutionCount` (churn), `CombinedCardinality` (data volume, no execution history needed). To rank whole user actions instead of blocks, `tool:performance_slowest_changes` with `applicationId`, `maxChangesCount`, and optionally `rangeStart`/`rangeEnd`.
2. **Reproduce** the slow input or formula change and capture its `changeId`.
3. **Profile.** `tool:performance_profile_change` with `changeId`, leaving `includeAllExecutions` omitted.
4. **Fork board-render vs compute.** Low total execution Duration but a slow board means rendering (`skill:designing-boards-and-views`), not formula work.
5. **Locate the bottleneck** using the sections below, then map its `Blocks:` line to metrics and read their formulas (`tool:search_metrics_and_lists` with `show_details: true`). When the scope matters (scope-loss origin, partial scope), get the scoped dimensions of the suspect executions and their direct ancestors with `tool:performance_profile_change_scope_details`.
6. **Verify each fix.** One change at a time: new `changeId`, re-profile, compare Duration and scope. For a quick before/after comparison, `tool:performance_changes_statistics` with both `changeId`s.

## Set `includeAllExecutions`

All three change tools take this flag: optional and `false` by default on `tool:performance_profile_change`, required on `tool:performance_slowest_changes` and `tool:performance_changes_statistics`.

**Never set it to `true` unless the user explicitly asks about all executions:** omit it on `tool:performance_profile_change`, pass `false` on the two others.

- With `false`, only executions that impacted data Members were viewing when the change was made are covered, i.e. the executions that gated what Members saw refreshing.
- With `true`, deliberately deferred executions are counted too, which inflates timings with work that delayed nobody.

A change that comes back with no execution at all is not necessarily an empty change: re-run with `true` before concluding.

## Read Top Blocks Output

`tool:get_top_blocks_by_performance` and `tool:get_blocks_performance` emit the same block lines, under their own header (`Top N blocks ranked by performance:` / `Performance summary for N block(s):`):

```
- {block_id} ({block_type}) — cardinality={n}, executions={count}, avg_ms={avg}, sum_ms={sum}, full_executions={count}, full_avg_ms={avg}, full_sum_ms={sum}, scoped_executions_pct={pct}
```

| Field | Meaning |
|---|---|
| `executions`, `avg_ms`, `sum_ms` | **All executions** of the block over the window, whatever their scope |
| `full_executions`, `full_avg_ms`, `full_sum_ms` | Same stats restricted to **full executions**: the ones that recomputed the whole block because they ran with no scope |
| `scoped_executions_pct` | `(executions - full_executions) / executions * 100` — share of executions that ran scoped |

Missing stats → widen the time window or switch `criteria`. Then profile a change on the suspect block.

**Always analyze on the all-executions fields** (`executions`, `avg_ms`, `sum_ms`) unless the user explicitly asks about full executions. They are the real cost of the block. The full-executions fields exist only to compute `scoped_executions_pct`; never report `full_sum_ms` as the block's cost.

### Reading `scoped_executions_pct` (scope-loss signal)

A **low** `scoped_executions_pct` means the block is almost always recomputed in full, i.e. it is usually executed unscoped. Before concluding, compare it against the other blocks of the same run:

| Situation | Read | Action |
|---|---|---|
| Low on a **minority** of blocks, the rest healthy | Those blocks' own formulas lose the scope | Strong remodeling candidate: inspect the formula; see `skill:writing-performant-formulas` |
| Low on the **majority** of blocks | The user actions themselves were unscoped, not the formulas | Do not recommend formula work on that basis; look at what triggers the recomputes (imports, full data loads, structural changes) |

This is a triage signal, not a diagnosis: confirm with `tool:performance_profile_change` on a change touching the block (first `no scope, full computation` in the chain) before recommending a remodeling.

## Read Slowest Changes and Changes Statistics Output

Both tools return one entry per change with aggregated timings over the covered executions:

| Field | Meaning |
|---|---|
| `durationMs` | Wall-clock time of the change, from the user action to the end of the last covered execution |
| `sumExecutionTimesMs` | Total compute time |
| `sumContentionTimesMs` | Total queue wait (previous tentative execution times counted as contention) |
| `executionsCount` | Number of covered executions |
| `averageExecutionTimeMs` / `medianExecutionTimeMs` / `maxExecutionTimeMs` | Distribution of the covered execution times |

`tool:performance_slowest_changes` also returns `date`; to identify the user action behind a change, look up its `changeId` in the audit trail.

Rank and report by `durationMs` unless the user asks about total compute. Changes without any profiled execution are omitted from statistics. Profile the retained change with `tool:performance_profile_change` before recommending formula work.

## Read Change Profile Output

```
Change profiled successfully. N execution(s) found, covering {all executions|only executions that impacted data Members were viewing}.

Executions:
1. **{jobType}**
   - Id: {uuid}
   - Blocks: Metric(`uuid`), ...
   - Dimensions: {count}
   - Ready at: Xms, Executed at: Yms, Duration: Zms
   - Effective scope: {text}
   - Output scope: {text}
   - Impacted data Members were viewing: yes|no   ← only with includeAllExecutions=true
   - Contention while impacting data Members were viewing: {n}ms   ← optional
   - Depends on: uuid, ...   ← optional
```

The profile is a **high-level overview of the scope**: to keep it small on big changes, it only counts the dimensions of each execution and the dimensions its scopes restrict. It never says which dimensions they are, nor how many items each is restricted to. For that, see [Read Scope Details Output](#read-scope-details-output).

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
| `X scoped dimension(s)` | X dimensions restricted to a subset of their items |

### X/Y scope notation

- **Y** = count on the `Dimensions:` line
- **X** = count of scoped dimensions in `Effective scope:`
- Target **X = Y** on hot paths

| Effective | Output | Interpretation |
|---|---|---|
| `N scoped dimension(s)` | Same count | Scope preserved, as far as counts tell |
| `N scoped dimension(s)` | Higher count | Scope introduced downstream |
| any | `no change` | Ran, no output (still check Duration) |
| `no scope, full computation` | — | Scope-loss origin candidate |

The first `no scope, full computation` in the chain is the scope-loss origin: inspect that block's formula.

Counts cannot tell two different dimension sets apart: equal counts on consecutive executions do not prove the same dimensions stayed scoped, and a partial X/Y does not say which dimension lost its scope. Whenever the conclusion depends on which dimensions are scoped, check with `tool:performance_profile_change_scope_details` before reporting.

## Read Scope Details Output

`tool:performance_profile_change_scope_details` takes the `changeId` and the `executionIds` (at most 100) to inspect, as listed on the `Id:` lines of the profile. It returns the dimensions and scopes of those executions in full:

```
Scope details of N execution(s):
- Execution `{uuid}`
  - Dimensions: uuid1, uuid2, ... | none
  - Effective scope: {text}
  - Output scope: {text}
```

Scope text is `no change`, `no scope, full computation`, or `dim:uuid (N modalities), ...`: one entry per scoped dimension, restricted to N items. Dimensions of `Dimensions:` missing from `Effective scope:` are the unscoped ones (Y − X).

- **Request only what you need:** the scope-loss origin and its direct ancestors (`Depends on:`), or the few steps whose X/Y you report. Do not request every execution of a big change.
- Report dimensions by name, never by UUID.

### Time, contention, and dependencies

- **Optimization order:** with the default `includeAllExecutions=false`, every listed execution gated what Members saw refreshing, so work through them by Duration. With `true`, focus first on executions with `Impacted data Members were viewing: yes`, then the others. `no` executions are deliberately deferred, so their extra contention alone is not a defect.
- Sort by **Duration**; flag > 1000 ms or a dominant wall-time share.
- **Contention** = Executed at − Ready at. Large relative to Duration means workload or queueing, not formula.
- **Contention split.** `Contention while impacting data Members were viewing` is the tail of that same wait, ending when the execution ran; the earlier part elapsed while the execution could still be deferred. The line appears only for executions that ended up impacting viewed data. Compare it against the whole contention:
  - **Much smaller than the contention** → the execution sat in the queue while no Member was waiting on it, then one opened something depending on it and it ran shortly after. Give this explanation whenever an execution appears far down the timeline yet is marked as impacting viewed data: for almost all of its wait, it was delaying nobody. Neither the wait nor that block's formula is the problem.
  - **Close to the contention** → it waited while already delaying Members. That is a queueing problem, still not a formula problem.
  - **Line absent** → with `includeAllExecutions=true`, the execution never impacted viewed data: its whole wait was deferrable and is not a defect on its own. With the default `false`, it simply never waited while impacting viewed data.
- **Wall time** ≈ max(Executed at + Duration) across executions.
- Match `Depends on` UUIDs to upstream `Id:` lines; ancestors appear earlier in the list. A `Depends on` UUID with no matching `Id:` is an ancestor that was not covered (deferrable, or hidden by permissions), not a broken chain.

### Profile patterns

| Pattern | Signature | Action |
|---|---|---|
| Cascading scope loss | Scoped runs, then a first `no scope, full computation`, rest full or no change | Fix the scope-loss origin's formula |
| No change, high Duration | `Output scope: no change` and Duration > 500 ms | Scope the formula earlier |
| High contention | Executed at ≫ Ready at on many rows | Broad scope or too many parallel branches |
| Late execution impacting viewed data | `Impacted data Members were viewing: yes`, Executed at ≫ Ready at, and contention while impacting viewed data far below the contention | Explain the late pickup (a Member opened something depending on it mid-change); do not chase that block's formula |

## Report to the User

On top of the findings structure in `skill:diagnosing-performance-issues`, include: execution count, what the profile covered (executions that impacted viewed data, or all of them), approximate wall time, a note that results are filtered by the caller's permissions, the chain with natural block names, X/Y per step, and the scope-loss origin, naming the dimension that lost its scope when checked with `tool:performance_profile_change_scope_details`. When a step impacted viewed data only for the tail of its wait, say why it appears late in the timeline rather than presenting it as a slow step.

**Vocabulary:** say ready at, executed at, duration, scope, dependency, impacted viewed data, contention while impacting viewed data. Never say executionId, timeScheduleMs, effectiveScope, effectiveScopedDimensionsCount, clauses, criteria, impactedViewedData, timeContentionWhileImpactingViewedDataMs.
