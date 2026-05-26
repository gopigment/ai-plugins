---
name: planning-cycles-pigment-applications
description: Always use this skill whenever the user mentions or implies versions, Budget, Actual, Forecast, Reforecast, switchover, scenarios, snapshots, planning cycles, or Actual/Plan layering. Covers Version Dimensions (foundational to all planning applications), Native Scenarios (used for what-if), and Snapshots setup (used to freeze data).
metadata:
  skill_path: /planning-cycles-pigment-applications/SKILL.md
  base_directory: /planning-cycles-pigment-applications
  includes:
    - "*.md"
---

# Planning Cycles in Pigment

Three features handle planning cycles, scenarios, and lifecycle in Pigment. They are **complementary, not alternatives**. Read first to pick the right one. Jump to the linked deep dives for the procedures.

## When to Use This Skill

Read this skill whenever the user touches metric structure or mentions:

- Versions, Budget, Actual, Forecast, Reforecast, Rolling Forecast
- Switchover, lock, layering Actual with Plan, Is Actual / Is Plan / Is Version
- Native Scenarios, what-if, Optimistic / Pessimistic / Stress
- Snapshots, archiving, closing a planning cycle

---

## Mental Model

Planning lifecycle in Pigment is three orthogonal features at the application level:

- **Version Dimension** -- custom structural dim on metrics, modeler built
  - Items: `Budget FY26`, `Reforecast Q2 FY26`, ... (Actual is optional)
  - Properties: Start / End Month, Switchover Month, Active, Lock, Version Type
  - Booleans: Is Version, Is Actual, Is Plan (gate Actual vs Plan layering)
- **Native Scenarios** -- app-level overlay, not a dimension
  - Optimistic, Pessimistic, Stress
  - Formula Groups for safe formula trials
- **Snapshot** -- point-in-time copy of the app, used for closed cycles and archives

Invariants:

- The **Version Dimension is a custom dimension built by the modeler.** It sits in metric structures and drives cross-version formulas, locking, and AR.
- **Native Scenarios are not a dimension**. They are an application-level overlay for ad-hoc sensitivity on top of an existing plan.
- **Snapshots are copies of an app**. They never replace a Version. They freeze a state.

---

## Three Distinct Features

| Feature | Use it for | Identity | Read |
|---|---|---|---|
| **Version Dimension** | Modeling planning cycles (Budget, Forecast, Reforecast, Rolling Forecast). Cross-version formulas, governance, locking. | Regular Dimension created by the modeler. Part of the Metric structure. | [./planning_cycles_versions.md](./planning_cycles_versions.md) |
| **Native Scenario** | Quick what-if sensitivity on top of an existing model. Safe trialing of formula changes via Formula Groups. | Application-level feature. Not a Dimension. | [./planning_cycles_scenarios.md](./planning_cycles_scenarios.md) |
| **Snapshot** | Freezing the state of an Application. Closing planning cycles. Archiving. | Point-in-time copy of an Application. | [./planning_cycles_snapshots.md](./planning_cycles_snapshots.md) |

---

## Decisions in Order

1. **Identify the intent.** Structured plan with governance and cross-plan formulas, ad-hoc sensitivity, or archiving a state. The mental model above maps each intent to one feature.
2. **Build a Version Dimension** for any real planning cycle (Budget, Reforecast, Forecast, Rolling Forecast).
3. **Pick the Version Items.** Include creation year or window span in the name (`Budget FY26`, `Reforecast Q2 FY25 Pessimistic`). Decide whether to add an `Actual` item (optional, see below).
4. **Add mandatory properties:** `Start Month`, `End Month`, `Switchover Month` (or `Switchover Year` for year-granular cycles), `Active Version` (Bool), `Lock Version` (Bool).
5. **Add `Version Type` (optional, recommended).** Dimension-typed property categorizing each version (`Budget`, `Reforecast`, `Rolling Forecast`, `Long Range Planning`, `Actual`). Use it to reference the Actual version in formulas without hard-coding an item (MP02 / T&D safe).
6. **Build the three Boolean Metrics:** `Is Version` (window), `Is Actual` (up to Switchover Month inclusive), `Is Plan` (after Switchover Month). They gate Actual vs Plan layering inside formulas.
7. **Wire Access Rights** from `Active Version` and `Lock Version`: locked Versions are read-only, active Versions are open for edit. See `skill:securing-pigment-applications`.
8. **Use Native Scenarios only for overlays** (Optimistic, Pessimistic, Stress) or for trialing formula changes in a Formula Group.
9. **Snapshot at lifecycle boundaries.** Closing a Budget cycle, year-end, or before a structural rework.
10. **Govern the live set.** Regularly review and clean up the Version list. Archive locked or outdated Versions via Snapshots.

---

## Versions vs Native Scenarios: Decision

Use a **Version Dimension** for any structured plan, any cross-plan formula reference, and any per-plan access control.

Use a **Native Scenario** only for ad-hoc sensitivity (Optimistic, Pessimistic, Stress) on top of an existing plan, or for trialing a new formula in a Formula Group before porting it back to the main model.

**MG12:** model planning cycles as a Dimension, never as a Native Scenario.

---

## Calendars vs Versions: Do Not Confuse Them

**Calendars** define the time structure of an application: Month, Quarter, Year, fiscal year, date range. They are configured once via the calendar tools.

**Versions** are a modeling pattern. A Dimension you create to hold Budget, Actual, Forecast, etc., with Switchover properties and Boolean metrics that gate Actual vs Plan data.

Do **not** use Calendar tools (`calendar_create`, `calendar_expand`, `calendar_add_time_dimension`, etc.) to implement planning cycles. They configure time, not planning cycles. For calendar setup see [`../modeling-pigment-applications/modeling_time_and_calendars.md`](../modeling-pigment-applications/modeling_time_and_calendars.md).

---

## Glossary

- **Version Item**: one row of the Version Dimension (e.g. `Budget FY26`, `Reforecast Q2 FY26`). The `Actual` item is optional.
- **Switchover Month / Year**: per-version property marking the last month (or year) of actual data. Plan picks up after it. Only `Month` or `Year`; not Quarter or Date.
- **Start Month / End Month**: per-version properties defining the planning window of that Version.
- **Active Version**: Boolean property flagging Versions currently displayed for input or reporting.
- **Lock Version**: Boolean property flagging Versions locked from edits once approved. Drives the read-only AR rule.
- **Version Type**: optional Dimension-typed property categorizing each Version (`Budget`, `Reforecast`, `Rolling Forecast`, `Long Range Planning`, `Actual`). The T&D / MP02-safe way to reference the Actual version.
- **Is Version / Is Actual / Is Plan**: three Boolean metrics over Version x Time. `Is Version` = inside the window. `Is Actual` = inside the window up to Switchover Month inclusive. `Is Plan` = inside the window after Switchover Month.
- **Layering**: combining Actuals up to Switchover Month with Plan beyond it, inside a single metric.
- **Formula Group**: a set of formula overrides scoped to a Native Scenario, used to trial alternative logic without touching the base model.
- **Shared vs Local Scenario**: scope of a Native Scenario (shared across users, or private to one user).
- **Live Version**: a Version Item currently in active use. Regularly clean up locked or outdated ones.
- **Snapshot**: point-in-time copy of an Application. Read-only by default.

---

## Critical Rules

- **Always read the matching document before building.** Do not rely on this SKILL.md summary alone.
- **Never use Calendar tools to implement versioning.**
- **Never model Budget, Actual or Forecast as Native Scenarios.** Use a Version Dimension.
- **Never hard-code Version Items in formulas.** Use an input Metric of type Dimension.
- **Never use `REMOVE` on Version.** Use `FILTER` or `SELECT`.
- **Keep the Version Dimension lean.** Only keep active Versions in use and locked Versions needed for reference. Regularly review and clean up; archive older Versions via Snapshots.

---

## Deeper Dives

| Need | Doc |
|---|---|
| Version Dimension setup, switchover, mandatory Boolean metrics, layering, live formula changes, multi-app | [./planning_cycles_versions.md](./planning_cycles_versions.md) |
| Native Scenarios: when to use, Shared vs Local, anti-patterns, combining with Versions | [./planning_cycles_scenarios.md](./planning_cycles_scenarios.md) |
| Snapshots and lifecycle: when to snapshot, cycle workflow, performance budget | [./planning_cycles_snapshots.md](./planning_cycles_snapshots.md) |
| Calendar setup (fiscal year, granularity, date range) | [`../modeling-pigment-applications/modeling_time_and_calendars.md`](../modeling-pigment-applications/modeling_time_and_calendars.md) |
| Modeling foundations (mental model, core concepts) | `skill:modeling-pigment-applications` |
| Architecture (Version Dimension in Hub app) | [`../modeling-pigment-applications/modeling_architecture_design.md`](../modeling-pigment-applications/modeling_architecture_design.md) |
| Access Rights on locked versions | `skill:securing-pigment-applications` |
| Formula patterns (cross-version, layering, MP02) | `skill:writing-pigment-formulas` |
