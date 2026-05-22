# Version Dimension: Planning Cycles in Pigment

A **Version Dimension** is a regular Pigment Dimension created by the modeler to hold planning cycles (Budget, Forecast, Reforecast, Rolling Forecast). It is part of the Metric structure, supports cross-version formula references, and powers governance and locking. It is the canonical way to model planning cycles in Pigment.

**MG12:** model planning cycles as a Dimension, never as a Native Scenario.

## 1. When to Use a Version Dimension

Use a Version Dimension for:

- Any structured plan (Budget, Forecast, Reforecast, Rolling Forecast).
- Any cross-plan formula reference (e.g. `Reforecast Q2 FY26` references `Reforecast Q1 FY26`).
- Any per-plan access control (lock past plans).

### Examples

- **Budget vs Actual with variance.** Track Budget and Actual side by side and compute `Variance = Actual - Budget` per Account and Month.
- **Rolling Forecast.** Quarterly rolling forecast where each cycle builds on the previous one (e.g. `Reforecast Q2 FY26` uses `Reforecast Q1 FY26` Actuals plus a growth factor). Pattern: Version Items `Reforecast Q1 FY26`, `Reforecast Q2 FY26`, `Reforecast Q3 FY26`, `Reforecast Q4 FY26`; advance `Switchover Month` by one quarter per cycle; reference the previous Version via an input Metric of type Dimension `VAR_Previous_Reforecast_Version` (MP02): `'Revenue'[FILTER: Version = VAR_Previous_Reforecast_Version] * (1 + Growth Factor)`.

For ad-hoc what-if sensitivity that does not require cross-version references, see [planning_cycles_scenarios.md](./planning_cycles_scenarios.md).

## 2. Mandatory Setup

When the user asks to set up versioning, a planning cycle, a Forecast or a Budget, produce **all** of the following.

### 2.1 Version Dimension and Properties

Create an `LST_Versions` Dimension List. One Item per planning cycle (e.g. `Budget FY26`, `Reforecast Q2 FY26 Pessimistic`). Include the creation year or the window span in the name. Stay consistent across the Dimension.

**Mandatory Properties:**

- `Switchover Month` (Dimension referencing `Month`). Use `Switchover Year` if the planning grain is Year.
- `Start Month`, `End Month` (Dimension referencing `Month`).
- `Active Version` (Boolean): TRUE for Versions in active use.
- `Lock Version` (Boolean): drives the Access Rights Metric. Locked Versions are read-only.

**Optional Properties:**

- `Version Type` (Dimension): `Budget`, `Reforecast`, `Rolling Forecast`, `Long Range Planning`. Reference it in formulas to apply different rules per Version Type. Test & Deploy safe and follows MP02.
- `Version #` (Number): incremental for repeated occurrences of the same Version.

### 2.2 Switchover Semantics

`Switchover Month` is the **last month of actual data** for that Version. Months strictly after it are Plan.

- `Budget FY26` with `Switchover Month = December 2025`: Actuals up to Dec 2025; Jan to Dec 2026 are Plan.
- `Reforecast Q1 FY26` with `Switchover Month = March 2026`: Actuals Jan to Mar 2026; Apr to Dec 2026 are Plan.

### 2.3 Three Mandatory Boolean Metrics at `Version × Month`

- `Is Version`: TRUE for months between `Start Month` and `End Month`.
- `Is Actual`: TRUE for months inside the Version window **up to and including** `Switchover Month`.
- `Is Plan`: TRUE for months inside the Version window **after** `Switchover Month`.

### 2.4 Layering Actuals and Plan

For each measure, create three Metrics at `<Driver Dimensions> × Version × Month`:

- `<Measure>_Actual` gated by `Is Actual`.
- `<Measure>_Plan` from forward-looking assumptions.
- `<Measure>` (final): `IF(Is Actual, <Measure>_Actual, <Measure>_Plan)`.

### 2.5 Optional Dedicated `Actual` Version

Include a single-Item `Actuals` Version only if the model needs to isolate Actuals independently of any plan cycle, or if Reforecast cycles leave Actuals coverage gaps. Update its `Switchover Month` each month. Never create one Item per year.

## 3. Anti-Patterns

1. **Modeling Budget, Actual or Forecast as Native Scenarios.** Use a Version Dimension.
2. **Hard-coding Version Items in formulas** (`Version."Budget"`). Use an input Metric of type Dimension (e.g. `VAR_Current_Budget_Version`) or the `Version Type` Property. See MP02.
3. **Adding Version to every Metric.** Skip Metrics that hold only Actuals or pure reference data.
4. **Using `REMOVE` on Version.** It mixes Actual, Budget and Forecast into one value. Use `FILTER` or `SELECT` instead.
5. **Editing a formula in a live Version without flagging it.** Use the Boolean logic flag pattern (section 4).
6. **Keeping more than ~6 live Versions.** Archive older ones via Snapshots (see [planning_cycles_snapshots.md](./planning_cycles_snapshots.md)).
7. **Proposing a Version Dimension setup missing any of:** `Switchover Month` (or Year), the four mandatory Properties, or the three Boolean Metrics.

## 4. Boolean Logic Flag for Live Formula Changes

When a Version is live, never edit a formula directly. Add a Boolean Property on the Version Dimension (e.g. `Y+1 Logic Changes`):

```
IF(Y+1 Logic Changes, New formula, Old formula)
```

Set TRUE only on the Versions adopting the new logic. This makes transitions explicit and auditable.

## 5. Multi-Application

Create the Version Dimension in the **Hub Application** and share it with the spoke Apps. Reuse the same Version Dimension across as many Applications as possible. Create a separate one only when a clear business need requires a different cadence (e.g. Sales needing 10x more Versions). See [`../modeling-pigment-applications/modeling_architecture_design.md`](../modeling-pigment-applications/modeling_architecture_design.md).

## See Also

- [planning_cycles_scenarios.md](./planning_cycles_scenarios.md): Native Scenarios for ad-hoc sensitivity on top of Versions.
- [planning_cycles_snapshots.md](./planning_cycles_snapshots.md): Snapshots for archiving live Versions.
- [`../modeling-pigment-applications/modeling_principles.md`](../modeling-pigment-applications/modeling_principles.md): MG12 (planning cycles), MP02 (no hard-coding of Dimension Items).
- [`../modeling-pigment-applications/modeling_application_auditing.md`](../modeling-pigment-applications/modeling_application_auditing.md): reviewing live Versions.
- Pigment KB: [Versions and Scenarios](https://kb.pigment.com/docs/versions-scenarios), [Implement Versions and plans](https://kb.pigment.com/docs/managing-versions-in-pigment).
