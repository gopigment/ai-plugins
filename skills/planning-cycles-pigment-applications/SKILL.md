---
name: planning-cycles-pigment-applications
description: Always use for any Pigment modeling task. The Version Dimension is foundational to almost every Pigment metric -- consult this skill before designing any metric structure, and whenever the user mentions or implies versions, Budget, Actual, Forecast, Reforecast, Rolling Forecast, switchover, scenarios, snapshots, planning cycles, locked plans, layering Actuals with Plan, or Is Actual / Is Plan / Is Version Boolean metrics. Covers Version Dimension setup (switchover, lock, mandatory Boolean metrics, Actual/Plan layering), Native Scenarios for ad-hoc sensitivity, Snapshots for archiving live cycles, lifecycle, and anti-patterns. Calendars are NOT planning cycles; never use calendar tools to implement versioning.
metadata:
  skill_path: /planning-cycles-pigment-applications/SKILL.md
  base_directory: /planning-cycles-pigment-applications
  includes:
    - "*.md"
---

# How to Use This Skill

**Progressive Disclosure Pattern**: This `SKILL.md` provides an overview. Pick the file that matches the user's intent and read it before producing anything.

**Required workflow**:

1. **Read this file first** -- understand which of the three documents applies.
2. **Read the matching document fully** before designing or building.
3. **Explore as needed** with `tool:read_file`, `tool:grep`, or `tool:glob`.

# Planning Cycles in Pigment

This skill covers the three Pigment features that handle planning cycles, scenarios, and lifecycle. They are **complementary, not alternatives**.

## Three Distinct Features

| Feature | Use it for | Identity | Read |
| --- | --- | --- | --- |
| **Version Dimension** | Modeling planning cycles (Budget, Forecast, Reforecast, Rolling Forecast). Cross-version formulas, governance, locking. | Regular Dimension created by the modeler. Part of the Metric structure. | [planning_cycles_versions.md](./planning_cycles_versions.md) |
| **Native Scenario** | Quick "what-if" sensitivity on top of an existing model. Safe trialing of formula changes via Formula Groups. | Application-level feature. Not a Dimension. | [planning_cycles_scenarios.md](./planning_cycles_scenarios.md) |
| **Snapshot** | Freezing the state of an Application. Closing planning cycles. Archiving. | Point-in-time copy of an Application. | [planning_cycles_snapshots.md](./planning_cycles_snapshots.md) |

## When to Use This Skill

Read this skill when the user asks to:

- Build a **Version Dimension** or planning cycle from scratch -> [planning_cycles_versions.md](./planning_cycles_versions.md)
- Track **Budget vs Actual** with variance -> [planning_cycles_versions.md](./planning_cycles_versions.md)
- Set up **Forecast**, **Reforecast**, or **Rolling Forecast** -> [planning_cycles_versions.md](./planning_cycles_versions.md)
- Configure a **Switchover** date / month / year -> [planning_cycles_versions.md](./planning_cycles_versions.md)
- Layer **Actuals** with **Plan** data in a single metric -> [planning_cycles_versions.md](./planning_cycles_versions.md)
- Use the `Is Version` / `Is Actual` / `Is Plan` Boolean metrics -> [planning_cycles_versions.md](./planning_cycles_versions.md)
- Run **ad-hoc what-if** sensitivity (Optimistic / Pessimistic / Stress) -> [planning_cycles_scenarios.md](./planning_cycles_scenarios.md)
- Configure **Shared vs Local** Scenarios -> [planning_cycles_scenarios.md](./planning_cycles_scenarios.md)
- **Combine** Versions and Native Scenarios -> [planning_cycles_scenarios.md](./planning_cycles_scenarios.md)
- Manage **Snapshots** for closing planning cycles or archiving -> [planning_cycles_snapshots.md](./planning_cycles_snapshots.md)
- Understand the **planning-cycle lifecycle** (snapshot, clone, switchover) -> [planning_cycles_snapshots.md](./planning_cycles_snapshots.md)

**Always read the matching document before designing or modifying any planning cycle, version, or scenario.**

---

## Versions vs Native Scenarios: Decision

Use a **Version Dimension** for any structured plan, any cross-plan formula reference, and any per-plan access control.

Use a **Native Scenario** only for ad-hoc sensitivity (Optimistic, Pessimistic, Stress test) on top of an existing plan, or for trialing a new formula in a Formula Group before porting it back to the main model.

**MG12:** model planning cycles as a Dimension, never as a Native Scenario.

---

## Calendars vs Versions: Do Not Confuse Them

**Calendars** define the time structure of an application: Month, Quarter, Year, fiscal year, date range. They are configured once via the calendar tools.

**Versions** are a modeling pattern: a Dimension you create to hold Budget, Actual, Forecast, etc., with Switchover properties and Boolean metrics that gate Actual vs Plan data.

Do **not** use Calendar tools (`calendar_create`, `calendar_expand`, `calendar_add_time_dimension`, etc.) to implement planning cycles. They configure time, not planning cycles. For calendar setup see [`../modeling-pigment-applications/modeling_time_and_calendars.md`](../modeling-pigment-applications/modeling_time_and_calendars.md).

---

## Documentation Files

- [./planning_cycles_versions.md](./planning_cycles_versions.md) -- Version Dimension setup: switchover, mandatory properties, Boolean metrics, layering, live formula changes, multi-app.
- [./planning_cycles_scenarios.md](./planning_cycles_scenarios.md) -- Native Scenarios: when to use, constraints (Shared vs Local), anti-patterns, combining with Versions.
- [./planning_cycles_snapshots.md](./planning_cycles_snapshots.md) -- Snapshots and lifecycle: when to snapshot, cycle workflow, performance budget.

---

## Cross-References

- **Calendars and time dimensions**: [`../modeling-pigment-applications/modeling_time_and_calendars.md`](../modeling-pigment-applications/modeling_time_and_calendars.md)
- **Modeling foundations**: `skill:modeling-pigment-applications`
- **Access Rights for locked versions**: see `modeling_access_rights.md` in `skill:modeling-pigment-applications`
- **Architecture (Version Dimension in Hub app)**: [`../modeling-pigment-applications/modeling_architecture_design.md`](../modeling-pigment-applications/modeling_architecture_design.md)
- **Modeling principles (MG12, MP02)**: [`../modeling-pigment-applications/modeling_principles.md`](../modeling-pigment-applications/modeling_principles.md)
- **Formula patterns**: `skill:writing-pigment-formulas`

---

## Critical Notes

- **Always read the matching document before building.** Do not rely on this SKILL.md summary alone.
- **Never use Calendar tools to implement versioning.**
- **Never model Budget, Actual or Forecast as Native Scenarios.** Use a Version Dimension.
- **Never hard-code Version Items in formulas.** Use an input Metric of type Dimension.
- **Never use `REMOVE` on Version.** Use `FILTER` or `SELECT`.
- **Limit live Versions** to ~6; archive older ones via Snapshots.
