---
name: diagnosing-performance-issues
description: "Execution skill. Use when a Pigment application has slow calculations, timeouts, or when diagnosing performance bottlenecks. Covers profiler interpretation, scope chip reading, systematic troubleshooting workflow, and iterative calculation optimization."
metadata:
  skill_path: /skills/diagnosing-performance-issues/SKILL.md
  base_directory: /skills/diagnosing-performance-issues
---

# Diagnose Pigment Performance Issues

Use when calculations are slow, timing out, or the user asks to find bottlenecks.

## Open the Profiler on the Slow Metric

1. Ask the user to reproduce the slow action (input change, import, scenario switch) so a fresh update exists in History. Updates are profiled for three days only.
2. Open the Profiler via:
   - **Application History:** Sidebar > History > hover the update > **Profile update**
   - **Block updates:** Expand the Block on a Board > Block updates icon > three-dot menu > profile
   - **Block Explorer:** Open the Metric > profiling from the formula bar menu
3. User must have Builder account type or above.
4. In the Profiling pane, note **total computation time** and **number of Blocks revised**.
5. Sort by **Computation Duration** (descending). Use **Search** to locate a specific Metric.
6. Optionally filter **Filter on path to this object** to isolate upstream dependencies.

## Read Scope Chips and M/N Notation

Each computation step shows a **Scope** value and colored chip.

### Scope notation (M/N)

- **M/N** = M Dimensions scoped out of N total in the Metric structure
- **2/2** = full scope (ideal -- engine recalculates only impacted cells)
- **2/3** = partial scope; investigate which Dimension lost scope
- **0/N** = no scope; full recomputation. Primary performance red flag.

Hover the Scope fraction to see which Dimension Items are scoped and which Dimensions pass to the next step. **Bold** M/N means scope passed downstream differs from the current step.

### Scope chip colors

| Chip | Meaning | Action |
| --- | --- | --- |
| **Black** | Scope preserved and passed downstream | Good — no action needed |
| **Blue** | New scope introduced (dimension added or newly constrained) | Expected after filters/selects; verify it is intentional |
| **Gray** | Computation triggered but output unchanged | Wasted work — investigate why the step ran |

Goal: maximize scoped Dimensions (approach N/N) throughout the chain.

## Trace Computation Chains

1. Start at the slowest step (highest Computation Duration).
2. Check its Scope chip and M/N value.
3. If **0/N**, walk upstream until you find where scope was lost.
4. Hover each step to see dependency lists.
5. For iterative calculations, the Profiler shows cycle-level timing; check each Metric individually.
6. Distinguish **Scheduled** (gray bar) vs **Execution** (blue bar) time. Long gray bars indicate access-rights overhead.

### Known scope-loss triggers

- Assigns a constant value across Dimensions
- Uses **ADD** (introduces unscoped Dimension)
- Uses **REMOVE** on a scoped Dimension
- Uses **CUMULATE**, **PREVIOUS**, **PREVIOUSOF**, **PRORATA**, **MOVINGAVERAGE**, or **MOVINGSUM**
- Uses **BY CONSTANT**

## Systematic Troubleshooting Workflow

1. **Profile** the slow Metric after reproducing the user action.
2. **Identify the bottleneck**: sort by Computation Duration; record the slowest step and time.
3. **Check scope chips** on the slow step and its inputs. Scope loss (especially 0/N) is the most common culprit.
4. **Classify the cause**: scope loss, densification, unnecessary REMOVE, cross-join from ADD, iterative calculation, or access-rights recomputation.
5. **Apply a fix** (see patterns below). Change one thing at a time.
6. **Re-profile** and compare before/after timing.
7. **Document** original duration, fix applied, and new duration.

If scope is healthy but execution is still slow, investigate iterative calculations or large Dimension cardinality.

## Fix Common Bottleneck Patterns

### Scope loss after REMOVE

- **Symptom:** Scope drops after REMOVE; downstream shows 0/N or partial scope.
- **Fix:** Only REMOVE when aggregation is truly needed. Check if **BY** with a mapping Metric achieves the same result while preserving scope. Push filtering earlier with SELECT or FILTER.

### Dense metrics from ISBLANK checks

- **Symptom:** High calculated size / density; slow steps on ISBLANK or ISNOTBLANK formulas.
- **Fix:** Replace with **ISDEFINED** / **IFDEFINED** to preserve sparsity.

### Long iterative calculations (PREVIOUS over many periods)

- **Symptom:** Slow execution proportional to time Dimension length.
- **Fix:** Subset the iteration scope, replace CUMULATE/FILLFORWARD where possible, reduce dimensionality, and group expressions.

### Large cross-joins from ADD

- **Symptom:** Blue scope chip after ADD; M/N drops; execution time spikes.
- **Fix:** Replace ADD with **BY** and a mapping Metric to allocate without introducing an unscoped Dimension.

### Access rights recomputing for all users

- **Symptom:** Long scheduled (gray) bars; AR Metrics appear early with 0/N scope.
- **Fix:** Wrap AR formulas in **`IFDEFINED('Users roles', ...)`** — this is the mandatory guard that skips AR computation for users without role assignments. Optionally add **`IFDEFINED(User, ...)`** as an additional performance trim (not a substitute). See `skill:writing-performant-formulas` and `skill:securing-with-access-rights`.

## Manage Calendar Dimension Performance

Daily calendars over many years amplify every iterative and unscoped calculation.

- Prefer **monthly grain** unless daily resolution is a business requirement.
- If daily is required, **subset** the Calendar to the relevant date range.
- Flag Metrics dimensioned by full daily Calendar with PREVIOUS or CUMULATE as high-risk for timeouts (3-minute execution limit).

## Tell the User When UI Action Is Required

The agent cannot perform these steps:

- **Performance Insights** (avg/max/total execution time, density): Block Explorer > All Blocks > Metrics tab > **Insights** toggle.
- **Dependency diagram exploration**: Block Explorer or Metric settings > dependency diagram.
- **Profiler access**: User must click Profile update in History or Block updates.
- **Re-profile after fix**: User must reproduce the action and profile the new update.

## Report Findings

Structure every diagnosis as:

1. **Bottleneck:** Metric name, step duration, scope (M/N), chip color
2. **Root cause:** scope loss / iterative / cross-join / densification / access rights
3. **Recommended fix:** specific formula change (one change at a time)
4. **Verification:** ask user to re-profile and share before/after timing

Prioritize by Computation Duration impact. Fix scope loss before tuning iterative patterns.

When a profile marks an execution as having impacted data Members were viewing, treat those executions first: they gate what Members see refreshing. The others are deliberately deferred by the engine, so their extra contention alone is not a defect. Say "impacted viewed data" to the user, never the raw field name.
