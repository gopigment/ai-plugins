---
name: building-a-full-application
description: "Planning skill. Use when building a complete Pigment application from scratch or when the request spans multiple modeling phases. This skill, together with any other planning skill the request calls for, is sufficient to produce the full architecture spec; do not load execution skills until the spec is approved and you begin building blocks."
metadata:
  skill_path: /skills/building-a-full-application/SKILL.md
  base_directory: /skills/building-a-full-application
---

# Building a Full Application

When building a complete Pigment application (or a significant portion of one), follow the two-stage workflow below.


---

## Stage 1 — Plan

During Stage 1, do NOT call any modeling tool and do NOT load any execution skill. Loading another planning skill is allowed. Use the phase outlines below to produce the full architecture and spec.

**Safe Modeling prerequisite** — When the build involves destructive or hard-to-reverse changes on a live application, the spec must say so: clone first, make changes on the clone, review the diff, and ask for approval before deploying back.

### Spec outline by phase

Phases are ordered by dependency; respect the order in the spec and during execution.

#### Phase 0 — Scope and Architecture

- **Single application or multi-app?** If the solution spans multiple teams, security boundaries, or planning cadences, decide whether a Hub-and-spoke layout is needed. Put shared dimensions, versions, exchange rates, and reusable reporting outputs in the Hub.
- **What planning domain?** Identify the domain (FP&A, Workforce Planning, Sales Planning, Supply Chain, etc.) and list the main planning outputs, source data, and shared assumptions.
- **Other planning skills cover this stage** — requirements discovery, multi-application architecture, and each planning domain each have their own planning skill, recognizable by a description starting with "Planning skill".

#### Phase 1 — Calendar

- Choose calendar type, fiscal year start, date range, and enabled time dimensions.
- Date range must cover historical data and planning horizon; enable Year, Quarter, and Month at minimum for FP&A.
- Never recreate Month or Year lists manually; always use the built-in calendar.

#### Phase 2 — Naming and Folder Structure

- Choose a clear, sortable naming convention for the application, folders, lists, metrics, and tables.
- Choose the application name with a sortable prefix, or confirm the target application already exists.
- Include the standard folder hierarchy (`0. Settings`, `1. Dimensions`, etc.).

#### Phase 3 — Dimensions and Hierarchies

- Identify dimension lists, parent-child hierarchies, dimension properties, and subsets.
- Order by dependency: top-level parents first, then children with Dimension-type properties.
- Identify unique properties needed for import readiness.
- Use mapped dimensions when mappings change over time (e.g. employee moving cost centers).
- Access rights drive structure — if access is controlled at a given level (e.g. country), that level must be a dimension, not just a property.

#### Phase 4 — Versions and Planning Cycles

Version is a dimension. Dimensions and Version can be created in either order; both must exist before Metrics.

**This phase is mandatory in all use cases.** Every application with metrics gets a Version dimension. Do not skip, defer, or reason about whether the use case needs it.

- Specify a Version dimension with one item per planning cycle (e.g. Actual, Budget FY25, Forecast Q2), including at minimum an `Actual` item.
- If imported data has no version info, tag all imported data as `Actual`.
- Time-based partitioning (e.g. "actuals are Jan-Jun, plan is Jul-Dec") is NOT a replacement for a Version dimension.
- Plan switchover properties (e.g. Switchover Month, Start Month, End Month) so each version can separate actuals from plan.
- Place the Version dimension in the Hub when shared across apps.
- Decide how completed versions are protected: snapshots (data slices) freeze numbers but cannot serve as basis for replanning; keep a live version alongside for that purpose.
- Decide how new versions are initialized (import from another version, bulk copy, clone data, re-import from scenario, or start from scratch).
- Max ~10 live versions recommended; every live version carries ongoing calculation cost.

#### Phase 5 — Metrics and Transaction Lists

- Decide which data belongs in transaction lists, input metrics, imported metrics, calculated metrics, and tables.
- Transaction lists for event-level source data (GL entries, orders, HRIS events).
- Input metrics for manual entry or imported values.
- Calculated metrics in formula dependency order.
- Tables to combine related metrics (e.g., P&L).
- Challenge any metric with more than 5 dimensions.
- Do not skip intermediate derivations. If the final output requires multiple logical steps, plan an explicit metric for each step.
- Do NOT create a metric for a parent-level aggregation when a view can group by a property (e.g. Country → Region). Use view grouping instead.
- Prefer view calculated items for between-item comparisons (e.g. Actual vs Forecast) and "show value as" for display transformations (YTD, cumulative, offsets, ratios) over dedicated metrics.

#### Phase 6 — Formulas

- List formula dependency order and identify where dimension alignment, aggregation, lookup, time logic, forecasting, or finance functions are needed.
- Use explicit intermediate metrics for multi-step logic instead of hiding many steps in one formula.
- Preserve sparsity: prefer BLANK for non-applicable cells, scope calculations before aggregation, avoid unnecessary dense outputs.

#### Phase 7 — Data Import

- Identify source files, mapping keys, and whether each import targets a dimension list, transaction list, or metric.
- Specify which CSV imports can be performed by the agent and which P2P imports or scheduled import configurations are UI-only.
- Validate imported data against dimension items.
- Import master data (customers, products, employees) into **dimensions**. Import transactional data (orders, sales, movements) into **transaction lists**.
- Historical outputs or aggregated budgets can be loaded directly into metrics when detail level does not justify a transaction list.
- Apply minimum transformation in Pigment — it is not an ETL tool.
- Only load required data and metadata; never load properties "just in case."
- Separate metadata loading from transactional data loading.

#### Phase 8 — Formatting and Highlighting

- Decide number formats, metric display names, units, and visual conventions for all user-facing metrics.
- Conditional highlighting is UI-only (metric settings → Conditional highlighting). Ask the user to configure it if requested.

#### Phase 9 — Views

- Define one View per distinct analysis perspective, with clear pivots, filters, sort rules, aggregators, and display modes.
- Avoid overloading a single View with too many pivots.
- Plan meaningful default filters so users see relevant data immediately.
- Use properties for aggregations (report level ≠ structure level); do not add a dimension just to group in reports — use view grouping on the property instead.

#### Phase 10 — Boards

- Decide Board audiences, pages, sections, widget layout, and the narrative each dashboard should support.
- Group related Views on the same Board page; use separate pages for different audiences or topics.
- Add text widgets for section headers or instructions when needed.

#### Phase 11 — Cross-Application Sharing

If other applications need data from this one (or vice versa):

- Identify which dimensions, metrics, and lists should be shared through Libraries.
- Keep intermediate calculations internal.

#### Phase 12 — Security

- Identify roles, readable and writable areas, access-right dimensions, and access-right metrics.
- Plan access-right metrics after the data model is stable. Apply rules are UI-only; ask the user to configure them.

#### Phase 13 — Performance Review

- Review formula sparsity, metric dimensionality, heavy aggregations, transaction-list volumes, and slow calculations.
- Re-check formulas for sparsity and scope violations before delivery.

### Write the spec

The spec is a single document you write in your reply, ordered by phase, covering only the phases in scope.

Close the spec with two short sections: the **open questions** you could not resolve from context, and the **UI-only steps** the user must finish by hand (conditional highlighting, access-right apply rules, connectors and scheduled imports).

---

## Stage 2 — Execute

### Execution principles

1. **Respect dependency order** — a phase's blocks must not be created before the blocks they depend on exist (e.g. dimensions before metrics that use them, metrics before views that display them). The Stage 1 phase outline is ordered by dependency; execute in that order.
2. **Parallelize within and across phases** — batch independent tool calls together. Creating 5 dimensions that don't reference each other? One batch to create dimensio, then create properties in order. Creating metrics that only depend on already-existing dimensions? Batch them alongside other independent work. Two phases with no dependency between them (e.g. Phase 8 Formatting and Phase 9 Views on different metrics) can run concurrently.
3. **Do not re-read what you just created.** Trust tool responses. Do not call `tool:search_metrics_and_lists`, `tool:search_folders`, `tool:search_tables`, `tool:get_board`, `tool:get_views`, or other read tools to re-fetch a block you just created in the same session.
4. **Combine dependent steps.** When the next step depends on IDs from the current step (e.g. `tool:add_list_items` after `tool:create_list`), extract IDs from tool responses and emit the dependent calls in the immediately next turn.
5. **Load skills as needed** — pick each phase's execution skill from the skills library headers. You may load several at once if several phases will execute in the same batch. Do not artificially serialize skill loading.

---

## Verification checklist

Before telling the user the build is done:

- [ ] **Spec coverage** — every block the approved spec listed exists; nothing was silently skipped or renamed
- [ ] **Formula health** — no formula errors on the application (confirm via Pigment UI Issues or user-provided error list)
- [ ] **No empty dimensions** — every dimension list created has items (`tool:get_list_items`)
- [ ] **Version dimension** — it exists with at least an `Actual` item, and the metrics holding plan data carry it
- [ ] **Values plausible** — spot-check the primary output metrics with `tool:fetch_metric_data`; a wrong order of magnitude means a wrong formula, not a display problem
- [ ] **Folder placement** — no blocks left in "No Folder"
- [ ] **Views and boards** — every planned View renders and every Board page shows the intended widgets
- [ ] **Handover** — restate the UI-only steps from the spec so the user knows what is left to do by hand
