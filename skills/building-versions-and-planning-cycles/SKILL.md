---
name: building-versions-and-planning-cycles
description: "Execution skill. Use when implementing versions, planning cycles, or actuals vs forecast separation, and whenever realized data is extended into a plan or budget — loading actuals then planning the remaining periods, forecasting forward from actuals, Actual/Budget/Plan layering, plan vs actual variance — even when the request never uses the word version. Covers Version Dimensions, Switchover patterns, Actual/Forecast layering, Native Scenarios, and Snapshots."
metadata:
  skill_path: /skills/building-versions-and-planning-cycles/SKILL.md
  base_directory: /skills/building-versions-and-planning-cycles
---

# Building Versions and Planning Cycles

## Distinguish the Three Orthogonal Planning Features

Pigment offers three independent mechanisms. Never conflate them:

| Feature | Purpose | Mutability |
| --- | --- | --- |
| **Version Dimension** | Structural planning cycles (Budget, Forecast, Actual) integrated into metric structure and formulas | Live; editable until locked |
| **Native Scenarios** | What-if branching with independent inputs and formula groups | Live; temporary alternative assumptions |
| **Snapshots** | Point-in-time freeze of application data | Immutable once created |

All three can coexist. A forecast Version can have Optimistic and Pessimistic Scenarios; a Snapshot can capture a closed cycle for audit.

## Choose Version Dimension vs Scenario vs Snapshot

**Use Version Dimension when:**

- Planning is recurring, auditable, and must appear in formulas (Budget FY25, Reforecast Q2)
- You need switchover logic blending actuals and plan per version
- You initialize new cycles via Clone Data from prior versions
- Variance reporting compares structural plans over time

**Use Native Scenarios when:**

- You need rapid what-if analysis (Optimistic / Pessimistic / Stress test)
- Assumptions or formulas differ temporarily without adding version items
- Compare Scenarios side-by-side for decision support
- Scenarios do not replace Version Dimension — they branch within it

**Use Snapshots when:**

- Closing a planning cycle or month-end close requires a frozen record
- Audit trail or before/after analysis against live data
- Baseline for Data Slices (e.g., "Last year forecast") without maintaining extra versions
- Data must not change after capture

Limit active version items to ~6 for performance. Take Snapshots at cycle end or monthly for rolling forecasts.

## Build a Version Dimension Step by Step

### Step 0 — Version Dimension Is Mandatory

**Every application with metrics MUST have a Version dimension.** The only exception is a pure reference data hub with zero metrics. Create Version before any other metric. If imported data lacks version information, tag all as `Actual`. Retrofitting later (restructuring every metric, rewriting every layering formula) is expensive.

### Step 1 — Create the Version Dimension

Use `tool:create_list` to create a Dimension List named `Version`. Use `tool:add_list_items` to add items:

- `Actual`
- `Budget FY25`
- `Forecast Q2`

Include creation year or window span in names for clarity and Clone Data workflows.

### Step 2 — Add Required Properties

Use `tool:create_list_property` to add these properties on the Version dimension:

| Property | Type | Purpose |
| --- | --- | --- |
| `Switchover Month` (or `Switchover Year`) | Month (or Year) | Last month of actual data for this version |
| `Start Month` | Month | First month in the version window |
| `End Month` | Month | Last month in the version window |
| `Is_Active` | Boolean | Marks versions open for input or reporting |
| `Is_Locked` | Boolean | Prevents edits once approved |

Switchover defines where actuals end and plan begins. Different versions can use different switchover dates.

> **Property naming:** Domain templates may use different names (e.g. `Last Actuals Month` instead of `Switchover Month`). Semantics are the same; adapt to existing application conventions.

### Step 3 — Create Boolean Version Metrics

**All three metrics are mandatory, delivered in one pass.** They are never deferred to a later phase and never treated as optional. Use `tool:create_metric` with the `formula` field to create them dimensioned by `Version` × `Month` and set their formulas in a single call:

- [ ] **`Is_Version`** at `Version × Month`. Formula: `'Version'.'Start Month' <= Month AND Month <= 'Version'.'End Month'`
- [ ] **`Is_Actual`** at `Version × Month`. Formula: `'Is_Version' AND Month <= 'Version'.'Switchover Month'`
- [ ] **`Is_Plan`** at `Version × Month`. Formula: `'Is_Version' AND Month > 'Version'.'Switchover Month'`

These drive layering logic in downstream formulas. A model that separates actuals from plan without them is incomplete.

### Step 4 — Add Version to Metric Structures

When creating metrics with `tool:create_metric`, include Version in structure only for metrics needing per-version data (inputs, layered outputs, version-specific assumptions). Do not add Version to reference or lookup metrics unnecessarily.

**Hard rule**: any metric that combines actuals with a forward plan or forecast always carries Version in its structure, alongside its driver dimensions. "Adding Version would multiply the data unnecessarily" is not a reason to omit it there — without Version the metric cannot hold two planning cycles with different switchovers.

### Step 5 — Build Layering Formulas

Separate actual and plan source metrics, then combine using the version flags from Step 3:

```pigment
IF('Is_Actual', 'Revenue Actual', IF('Is_Plan', 'Revenue Forecast', BLANK))
```

The result is a single output metric containing actuals through switchover and plan thereafter.

**You MUST gate the split on `Is_Actual` / `Is_Plan`.** Comparing `Month` against a switchover value directly inside the layering formula bypasses Version: it reduces the split to one global cutoff, so a second planning cycle with its own switchover cannot coexist.

### Actuals-to-Forecast Recipe

When the model combines actuals with a forward plan or forecast, follow this exact build order:

1. Calendar setup (Month, Quarter, Year with properties)
2. Version bootstrap — Steps 1 to 3 above, all of them, no exceptions
3. `<Measure> Actual` metric gated by `Is_Actual`
4. `<Measure> Plan` metric from forward-looking assumptions
5. `<Measure>` (unified output) at `<Driver Dimensions> × Version × Month`: `IF('Is_Actual', '<Measure> Actual', '<Measure> Plan')`

**WRONG patterns (observed in traces):**

```pigment
IFDEFINED('Units Actual', 'Units Actual', PREVIOUS(Month) * (1 + 'Monthly Growth Rate'))
IF(Month <= 'Switchover Month', 'Units Actual', 'Units Plan')
IF(Month <= 'Actuals End Month', 'Units Actual', PREVIOUS(Month) * (1 + 'Monthly Growth Rate'))
```

All three skip the Version Dimension. `IFDEFINED` infers the split from whether actual data happens to exist. The other two read a switchover value from a scalar metric or straight from the property, at driver grain rather than `Version × Month`. None of them survives a switchover that differs per version, and none can carry a second planning cycle.

**Forbidden reasoning:** rationales such as "kept simple with a single combined output metric", "keeping it lean", "avoids unnecessary cardinality expansion", or "no Version Dimension needed here" are wrong when actuals and forecast coexist. Parameterising the cutoff into an input metric does not substitute for the Version Dimension. The Version Dimension is always required in that scenario.

**Never use the calendar's Actual vs Forecast toggle** for this split: pass `actual_vs_forecast_enabled: false` to `tool:calendar_create`. It gives one global switchover that cannot vary per Version. See `skill:setting-up-calendar`.

> **Alternative gating:** Templates may apply the flags differently (e.g. `[EXCLUDE:]` on flag metrics). That variation is about how the flags are consumed, not about whether Version and the flags exist — Steps 1 to 3 are required either way.

## Implement Rolling Forecasts

1. Add a new version item (e.g., `Forecast M+1`) using `tool:add_list_items`
2. Clone data from prior forecast. No agent tool for Clone Data; ask the user (Application menu → Clone data).
3. Update Switchover Month on new version to current closing month using `tool:set_metric_input`
4. Snapshot the prior version for audit. No agent tool for Snapshots; ask the user in Pigment UI.

Repeat monthly. Keep Version dimension lean — retire superseded items after Snapshot.

## Compare Versions with Data Slices

No public MCP tool creates Data Slices. Ask the user to configure slices in the Pigment UI for cross-version reporting (Budget vs Actual variance). You can still create a slicing dimension list with `tool:create_list` / `tool:add_list_items` and wire views once slices exist.

## Configure Native Scenarios

Ask the user to create scenario branches in the Pigment UI when needed. Use `tool:list_scenarios` to inspect existing scenarios:

- **Shared Scenarios**: visible across applications; required when changing assumptions on shared blocks from Libraries. Cannot convert to Local after creation.
- **Local Scenarios**: restricted to one application; shared block data comes from nearest Shared Scenario.

Each scenario can override input values independently. Formula Groups and Compare Scenarios are UI-only; ask the user to configure in Pigment UI.

Scenarios combine with Version Dimension (Version × Scenario). Do not use Scenarios as a substitute for version planning.

## Capture and Use Snapshots

No agent tool available for Snapshot creation. Ask the user in Pigment UI. Snapshots freeze all blocks and data at a point in time:

- Use after planning cycle approval or before major structural changes
- Read-only; switchover dates cannot be adjusted inside a Snapshot
- Compare live data to Snapshot for before/after analysis
- Include selected Scenarios when snapshotting if scenario data must be preserved

Snapshots serve archiving and audit. They do not participate in live formula logic.

## Validate Version System

1. Each active version has correct Start Month, End Month, and Switchover Month
2. `Is_Actual` + `Is_Plan` cover the version window without gaps or overlaps
3. Layering formula returns actuals through switchover and plan after
4. Locked versions (`Is_Locked = TRUE`) prevent user inputs
5. Only active versions appear for input; historical versions remain for reporting
6. Rolling forecast workflow documented (clone → update switchover → snapshot)
