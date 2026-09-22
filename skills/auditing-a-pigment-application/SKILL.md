---
name: auditing-a-pigment-application
description: "Execution skill. Use when auditing, reviewing, or health-checking a Pigment application. Covers the structured audit checklist, severity classification, and output format. Also triggers on requests to clean up, find unused blocks, assess model quality, or find what is wrong with an application."
metadata:
  skill_path: /skills/auditing-a-pigment-application/SKILL.md
  base_directory: /skills/auditing-a-pigment-application
---

# Auditing a Pigment Application

Run a structured, read-only audit. Do not modify anything unless the user explicitly asks for fixes after reviewing the report.

## CRITICAL -- Collect Formula Health Evidence First

Start every audit by reviewing formula computation errors (ask the user to check **Issues & errors** in the Pigment UI or share an export). These are highest-severity because they produce wrong or missing outputs.

For each errored block, retrieve its formula with `tool:search_metrics_and_lists` (`show_details: true`) and identify the root cause (type mismatch, missing reference, scenario-specific failure). Use `tool:get_metric_dependencies` on errored blocks to trace downstream impact and distinguish root causes from cascading symptoms.

## CRITICAL -- Trace Dependency Fragility

Use `tool:get_metric_dependencies` on critical output metrics (cashflow totals, P&L summaries, KPIs).

Flag:

- Long chains where a single upstream error cascades to many outputs
- Missing intermediate metrics (too much logic in one formula)
- Fragile roll-forward patterns (e.g. `CUMULATE` + `SELECT: Month-1` without clear beginning/ending balance separation)

## HIGH -- Scan Formula Patterns

Use `tool:search_metrics_and_lists` with `formula_regex_search` pattern queries. Do not rewrite formulas during the audit. For the full anti-pattern reference, see `skill:writing-performant-formulas`. For AR anti-patterns, see `skill:securing-with-access-rights`.

Flag these patterns (highest priority first):

- **Unguarded `ACCESSRIGHTS`**: no `IFDEFINED('Users roles', ...)` guard — AR computed for every user
- **`ACCESSRIGHTS` guarded only by `IFDEFINED(User, ...)`**: flag for review — not a substitute for `'Users roles'`
- **`ACCESSRIGHTS(x, FALSE)`**: densifies denied cells; should use BLANK
- **`ISBLANK` / `ISNOTBLANK`**: densifies; should use `ISDEFINED` / `IFDEFINED`
- **`[ADD: X][FILTER: ...]`**: densify-then-subset; use `IF(condition, value)`
- **`[FILTER: NOT ...]`**: use `[EXCLUDE: ...]`
- **Heavy `PREVIOUS` chains / multiple `PREVIOUS(...)` calls**: check if `CUMULATE`, `FILLFORWARD`, or `SELECT` fits
- **`IFBLANK(X, PREVIOUS(...))` or `IFBLANK(X, PREVIOUSOF(...))`**: use `FILLFORWARD`
- **`[ADD: X][REMOVE: X]` / `[REMOVE: X][ADD: Y]` round-trips**: use `BY` with mapping
- **Chained `[BY:][BY:]` on Transaction List**: drops properties; use single `BY` with comma-separated mappings
- **`[REMOVE: Version]`**: collapses scenario meaning; verify intentional
- **`REMOVE` in formulas**: scope loss; check with `tool:get_metric_dependencies`
- **2-arg `IF(cond, expr)` on 6+ dim metrics** with non-trivial second arg: use `expr[FILTER: cond]`
- **Hard-coded year / `DATE(YYYY, ...)`**: use Date-typed or Dimension-typed input metrics
- **Periods or colons in block names**: breaks formula references
- **Unresolved `Copy of X` default names**: leftover from duplication
- **Formula metrics with input overrides enabled**: audit risk if untracked

## HIGH -- Check Cycles and Versions

Use `tool:list_cycles` to check planning cycles. Use `tool:search_metrics_and_lists` with `kind: ["Dimension"]` and a name regex to verify a Version dimension exists. A missing Version dimension is a high-severity structural gap; see `skill:building-versions-and-planning-cycles` for fixes.

## HIGH -- Verify Calendar Coverage

Use `tool:calendar_get` to read the current configuration. Flag:

- Calendar end date before the next fiscal year (blocks future planning)
- Missing Quarter dimension when the app is FP&A-oriented
- Daily calendar with a large range (performance risk for iterative formulas)

For fixes, see `skill:setting-up-calendar`.

## MEDIUM -- Review Scenario Governance

Use `tool:list_scenarios`. Flag:

- Duplicate or near-duplicate names (case differences like `Budget` vs `budget`)
- Non-descriptive names (`child`, `test`, `scenario1`)
- Total count above 5-6 in a non-enterprise app (each scenario increases recalculation cost)
- Scenarios with formula errors (cross-reference with formula health findings)

## MEDIUM -- Audit Library and Shared Content

Use `tool:list_application_libraries`. Flag:

- Libraries enabled but providing no actively used blocks
- Old or versioned library names (e.g. `FY24` library on an `FY25` app)
- Excessive shared surface exposing internal blocks
- Shared metrics consumed locally but not through the library mechanism (direct cross-app references instead of PULL/PUSH)

## MEDIUM -- Check Modeling Governance

Use `tool:search_metrics_and_lists` and `tool:get_metric_dependencies` to identify structural debt:

- **Repeated logic across metrics**: Same subexpression duplicated in multiple formulas instead of centralized in a shared intermediary metric
- **Multi-purpose metrics**: A single metric doing aggregation, filtering, and business logic; should be split into focused layers
- **Hidden-metric access gaps**: Metrics with AR Apply rules that reference dimensions not in the metric's own structure, or metrics consuming AR-protected data without their own AR coverage
- **Access rights enforced only via Board visibility**: Data is hidden on boards but has no AR metric backing it; users with Block Explorer access can see everything. See `skill:securing-with-access-rights`

## LOW -- Inspect Naming Hygiene

Use `tool:search_metrics_and_lists` with `friendly_name_regex_search` to find blocks matching test/copy/temporary patterns:

- **`copy`, `Copy`, `COPY`**: Leftover duplicates
- **`test`, `Test`, `TEST`**: Experimental blocks not cleaned up
- **`new metric`, `New Metric`, `NewBlock`**: Default names never renamed
- **`tmp`, `TMP`, `ZZ`, `TBD`**: Explicitly temporary blocks
- **Hex-dash sequences (UUID-like)**: Auto-generated names

Count them, note their folder, flag as cleanup candidates.

## LOW -- Find Unused Blocks

Use `tool:search_metrics_and_lists` with `kind: ["Metric"]` to page through all metrics, then `tool:get_metric_dependencies` on each to find orphans (zero usages). Batch dependency checks in parallel.

**Safe to delete**: no formula references, no views, no boards, no automations, no ARM. **Not safe**: displayed on a board/view, shared via library (other apps may consume), input metrics with user data, metrics with overrides, or settings/parameters folder items. Always present the list to the user before deleting; group by folder; highlight input and shared metrics separately.

## LOW -- Check Folder Organization

Use `tool:search_metrics_and_lists` with `parent_folder_regex_search: ["^/$"]` or an empty-path filter to identify blocks in root / no folder. Check whether a standard folder structure exists (numbered prefixes like `0. Settings`, `1. Dimensions`). Mixed-purpose or flat layouts are a maintenance risk. For fixes, see `skill:naming-and-organizing-applications`.

## LOW -- Evaluate Board Quality

Use `tool:search_boards` and `tool:get_board` for suspicious boards. Flag:

- Total board count above 20
- UUID-like or meaningless board names
- Duplicate boards (same or very similar names)
- Homepage action cards pointing to wrong destinations
- Boards with more than 20 widgets (load-time risk)

## Report Structure

1. **Executive summary**: 2-3 sentences on model health and top priorities
2. **Findings by severity**: CRITICAL (active errors) → HIGH (structural risks) → MEDIUM (governance debt) → LOW (cosmetic). Name specific blocks with `mention:` syntax.
3. **Remediation priority**: numbered list; errors first, then structural, then hygiene
4. **Next steps**: offer cleanup plan, fixes, or deeper performance analysis

## Audit Checklist

Before delivering the report, confirm all checks ran:

- [ ] Formula health (UI Issues review + dependency tracing)
- [ ] Dependency fragility (critical output chains)
- [ ] Formula patterns (sparsity, ACCESSRIGHTS guards, iterative, structural, dimension-aware, naming)
- [ ] Cycles and versions (`tool:list_cycles`)
- [ ] Calendar coverage (`tool:calendar_get`)
- [ ] Scenario governance (`tool:list_scenarios`)
- [ ] Library surface (`tool:list_application_libraries`)
- [ ] Naming hygiene (test/copy/tmp/UUID patterns)
- [ ] Unused blocks (`tool:get_metric_dependencies` zero usages)
- [ ] Modeling governance (repeated logic, multi-purpose metrics, hidden-metric access gaps)
- [ ] Folder organization (blocks in root)
- [ ] Board quality (`tool:get_board`)
