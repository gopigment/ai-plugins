# Workforce Planning – Snapshot Spread Logic

## Purpose

> **Terminology note:** "Snapshot" here = periodic source-data loads (e.g. HRIS exports at specific dates), not Pigment application Snapshots (immutable point-in-time archives; see `skill:building-versions-and-planning-cycles`).

How to bridge **snapshot-based source data** (e.g. HRIS loads with a "load month" and sparse events) to a **regular planning grid** (Version × Employee × Month). The spread layer centralizes snapshot selection, propagation of point-in-time attributes over time, and history-vs-plan behavior so downstream metrics stay simple.

**When to use:** source provides as-of snapshots (one row per employee per "HRIS Load Month"); planning requires a dense grid on a regular time dimension (Month); need a single consistent rule for "which snapshot applies to each (scenario, entity, period)".

**When not to use:** client provides historical data already at planning grain (no backward/forward snapshot mapping needed). Backward spreading is never 100% correct; it is best-effort for history visualization, not audit/legal accuracy. Forward spreading is mandatory for forecast/plan.

---

## Core Concepts

- **Snapshot source**: list or metric with raw snapshot data (e.g. the Roster transaction list), a snapshot date dimension (e.g. Load Month), and an entity mapping (e.g. Employee Mapping). The Employee dimension itself stays lean (ID + Name); every other attribute is read from this source, never copied onto Employee as a list property.
- **Planning grain**: dimensions for planning/reporting (e.g. Version × Employee × Month).
- **Snapshot selection**: for each (Version, Entity, Planning Period), which snapshot date to use. Encoded in one or a few metrics.
- **Backward propagation (history)**: for past periods: use the snapshot actually valid at that time (last load month ≤ this month). Approximates historical view from snapshots.
- **Forward propagation (plan)**: for plan/future periods: use a fixed or forward-carried snapshot (e.g. latest known load month). Mandatory for forecasting.
- **Spread layer**: metrics that: (1) define effective snapshot date per (Version, Entity, Planning Period) for history and plan, (2) spread point-in-time attributes over the planning time dimension, (3) provide an active mask (which Entity/Period are in scope), (4) apply version windows.
- **History vs plan**: global toggle (e.g. "Populate history?") and optionally a cutover month per version. Centralizing in the spread layer avoids repeating `IF(SET_Populate_History, ..., ...)` in every staging metric.

---

## Pipeline

| Layer | Role | Typical content |
| --- | --- | --- |
| **1. Snapshot load** | Raw import | List/metric with snapshot date dimension and entity mapping; sparse attributes (hire date, term date, entity, department, etc.). |
| **2. Spread logic** | Bridge | Helpers: effective snapshot date (history + plan), spreaded dates (e.g. FILLFORWARD of hire/term), active mask, plan-month filter; all with version window applied once here. |
| **3. Data staging** | Map attributes to planning grain | For each attribute: pull from snapshot source using **BY CONSTANT: [effective snapshot metric]** (history or plan depending on toggle). No repeated FILLFORWARD or window logic. |
| **4. Overrides** (optional) | User overrides | Override staging values where users can edit (e.g. plan overrides). |
| **5. Cards / final attributes** | Output for reporting | IFDEFINED(Override, Staging value), still using spread helpers to anchor to the right snapshot. |
| **6. Stats & aggregations** | Headcount, FTE, events | Built on top of cards; no direct snapshot logic. |

The **spread layer (2)** is the only place that implements snapshot selection, date spreading, history vs plan, and version windows. All other layers reference its outputs.

---

## Patterns

### Spreading a point-in-time attribute (e.g. hire date)

Have the attribute available for every (Version, Entity, Month) where relevant (e.g. every month after hire). Use FILLFORWARD over planning time dimension, restrict to version window.

```text
IFBLANK(
  'Source_Attribute',           // from staging or load, already at planning grain where defined
  FILLFORWARD('Source_Attribute', Planning_Time_Dimension)
)
[FILTER: Version.'Window End' >= Planning_Time_Dimension]
```

Downstream metrics reference this "spreaded" metric for every month in scope without re-implementing fill or window logic.

### Choosing snapshot for history vs plan

Build two helper metrics: one backward (last load month ≤ this month), one forward (forward-carried load month for plan). Combine with active mask and plan-month filter. Expose a single "effective snapshot date" for staging.

**In staging (data) metrics:**

```text
IF(
  SET_Populate_History,   // global toggle
  Source_List.'Attribute'
    [BY LASTNONBLANK: Entity_Mapping, Snapshot_Date_Dimension]
    [ADD: Version][FILTER: Is_Actual]
    [BY CONSTANT: 'Spd_Effective_Snapshot_History'],
  Source_List.'Attribute'
    [BY LASTNONBLANK: Entity_Mapping, Snapshot_Date_Dimension]
    [ADD: Version][FILTER: Is_Actual]
    [BY CONSTANT: 'Spd_Effective_Snapshot_Plan']
)
```

All staging metrics use the same pattern; only the attribute and source change.

### Discover blocks and properties

Template names (e.g. `EE_Load_HRIS`, `SET_Populate_History`) are illustrative. Before implementing:

- Use `tool:search_metrics_and_lists` (and `tool:semantic_search` when names are unclear) to locate snapshot source list, planning dimensions, and history/plan toggles
- Search for Scenario/Version and confirm which properties encode the version window
- After building staging metrics, verify reporting/headcount metrics reference spread-layer metrics only, not the raw snapshot list

---

## Pitfalls

- **Backward spread is approximate:** good for visualizing history from snapshots; not guaranteed correct for audit/legal.
- **Forward spread is required for planning:** always define a clear rule for which snapshot drives plan/future periods.
- **Do not duplicate logic:** centralize "which snapshot?" and "history vs plan?" in the spread layer.
- **Version windows:** apply "only months in version window" once in spread layer so downstream metrics do not repeat version filters.

---

**Related skills:** [Workforce Planning – Application Architecture & Patterns](./workforce_planning_architecture_patterns.md); [Workforce Planning – Workforce Cards & BY-Arrow Mappings](./workforce_planning_cards_mapped_dimensions.md); [Workforce Planning – Changelog to Override Metrics](./workforce_planning_changelog_overrides.md).
