# Versions and Scenarios in Pigment

## 1. Three distinct features

| Feature | Use it for | Identity |
| --- | --- | --- |
| **Version Dimension** | Modeling planning cycles (Budget, Forecast, Reforecast, Rolling Forecast). Cross-version formulas, governance, locking. | Regular Dimension created by the modeler. Part of the Metric structure. |
| **Native Scenario** | Quick "what-if" sensitivity on top of an existing model. Safe trialing of formula changes via Formula Groups. | Application-level feature. Not a Dimension. |
| **Snapshot** | Freezing the state of an Application. Closing planning cycles. Archiving. | Point-in-time copy of an Application. |

These features are complementary, not alternatives.

## 2. Decision: Version Dimension vs Native Scenario

Use a **Version Dimension** for any structured plan, any cross-plan formula reference, and any per-plan access control.

Use a **Native Scenario** only for ad-hoc sensitivity (Optimistic, Pessimistic, Stress test) on top of an existing plan, or for trialing a new formula in a Formula Group before porting it back to the main model.

**MG12:** model planning cycles as a Dimension, never as a Native Scenario.

### Examples

- **Simple what-if analysis → Use Native Scenarios.** Compare three independent revenue projections (Base, Optimistic, Pessimistic) with their own assumptions and no need to reference each other.
- **Budget vs Actual with variance → Use Version Dimension.** Track Budget and Actual side by side and compute `Variance = Actual - Budget` per Account and Month.
- **Rolling Forecast → Use Version Dimension.** Quarterly rolling forecast where each cycle builds on the previous one (e.g. `Reforecast Q2 FY26` uses `Reforecast Q1 FY26` Actuals plus a growth factor). Native Scenarios cannot express the cross-version reference. Pattern: Version Items `Reforecast Q1 FY26`, `Reforecast Q2 FY26`, `Reforecast Q3 FY26`, `Reforecast Q4 FY26`; advance `Switchover Month` by one quarter per cycle; reference the previous Version via an input Metric of type Dimension `VAR_Previous_Reforecast_Version` (MP02): `'Revenue'[FILTER: Version = VAR_Previous_Reforecast_Version] * (1 + Growth Factor)`.

## 3. Building a versioning system: switchover, properties, data layering

When the user asks to set up versioning, a planning cycle, a Forecast or a Budget, produce **all** of the following.

### 3.1 Version Dimension and Properties

Create an `LST_Versions` Dimension List. One Item per planning cycle (e.g. `Budget FY26`, `Reforecast Q2 FY26 Pessimistic`). Include the creation year or the window span in the name. Stay consistent across the Dimension.

**Mandatory Properties:**

- `Switchover Month` (Dimension referencing `Month`). Use `Switchover Year` if the planning grain is Year.
- `Start Month`, `End Month` (Dimension referencing `Month`).
- `Active Version` (Boolean): TRUE for Versions in active use.
- `Lock Version` (Boolean): drives the Access Rights Metric. Locked Versions are read-only.

**Optional Properties:**

- `Version Type` (Dimension): `Budget`, `Reforecast`, `Rolling Forecast`, `Long Range Planning`. Reference it in formulas to apply different rules per Version Type. Test & Deploy safe and follows MP02.
- `Version #` (Number): incremental for repeated occurrences of the same Version.

### 3.2 Switchover semantics

`Switchover Month` is the **last month of actual data** for that Version. Months strictly after it are Plan.

- `Budget FY26` with `Switchover Month = December 2025`: Actuals up to Dec 2025; Jan to Dec 2026 are Plan.
- `Reforecast Q1 FY26` with `Switchover Month = March 2026`: Actuals Jan to Mar 2026; Apr to Dec 2026 are Plan.

### 3.3 Three mandatory Boolean Metrics at `Version × Month`

- `Is Version`: TRUE for months between `Start Month` and `End Month`.
- `Is Actual`: TRUE for months inside the Version window **up to and including** `Switchover Month`.
- `Is Plan`: TRUE for months inside the Version window **after** `Switchover Month`.

### 3.4 Layering Actuals and Plan

For each measure, create three Metrics at `<Driver Dimensions> × Version × Month`:

- `<Measure>_Actual` gated by `Is Actual`.
- `<Measure>_Plan` from forward-looking assumptions.
- `<Measure>` (final): `IF(Is Actual, <Measure>_Actual, <Measure>_Plan)`.

### 3.5 Optional dedicated `Actual` Version

Include a single-Item `Actuals` Version only if the model needs to isolate Actuals independently of any plan cycle, or if Reforecast cycles leave Actuals coverage gaps. Update its `Switchover Month` each month. Never create one Item per year.

## 4. Anti-patterns: never produce these

1. **Modeling Budget, Actual or Forecast as Native Scenarios.** Use a Version Dimension.
2. **Hard-coding Version Items in formulas** (`Version."Budget"`). Use an input Metric of type Dimension (e.g. `VAR_Current_Budget_Version`) or the `Version Type` Property. See MP02.
3. **Adding Version to every Metric.** Skip Metrics that hold only Actuals or pure reference data.
4. **Using `REMOVE` on Version.** It mixes Actual, Budget and Forecast into one value. Use `FILTER` or `SELECT` instead.
5. **Editing a formula in a live Version without flagging it.** Use the Boolean logic flag pattern (section 6).
6. **Keeping more than ~6 live Versions.** Archive older ones via Snapshots.
7. **Proposing a Version Dimension setup missing any of:** `Switchover Month` (or Year), the four mandatory Properties, or the three Boolean Metrics.
8. **Treating Shared vs Local Scenarios as switchable.** The choice is irreversible.

## 5. Snapshots and lifecycle

Pigment is a live calculation engine. The only way to freeze state is a Snapshot.

Each cycle:

1. Snapshot the Application at the end of the cycle (or monthly for Rolling Forecasting).
2. Add a new Version Item. Use **Clone data to** to copy assumptions from the previous Version. Update `Switchover Month`.

**Performance budget:** ~6 live Versions max. Archive older ones via Snapshots. Live Versions inflate recalculation cost.

## 6. Boolean logic flag for live formula changes

When a Version is live, never edit a formula directly. Add a Boolean Property on the Version Dimension (e.g. `Y+1 Logic Changes`):

```
IF(Y+1 Logic Changes, New formula, Old formula)
```

Set TRUE only on the Versions adopting the new logic. This makes transitions explicit and auditable.

## 7. Native Scenario: constraints when proposing one

- Independent calculation environment with its own inputs.
- **Cannot reference another Scenario's data** in formulas.
- Lists must be consistent across Scenarios.
- Not a Dimension. Cannot enter a Metric structure or pivot on a Page.
- **Shared vs Local** is set at creation and cannot be reverted. Pick **Shared** if Shared Blocks must carry Scenario-specific assumptions across Applications. Otherwise **Local**.
- Combine with Versions: keep the structured plan in the Version Dimension, layer Native Scenarios on top for sensitivity or formula trials.

## 8. Multi-Application

Create the Version Dimension in the **Hub Application** and share it with the spoke Apps. Reuse the same Version Dimension across as many Applications as possible. Create a separate one only when a clear business need requires a different cadence (e.g. Sales needing 10x more Versions). See `modeling_architecture_design.md`.

## See also

- `modeling_principles.md`: MG12 (planning cycles), MP02 (no hard-coding of Dimension Items).
- `modeling_architecture_design.md`: Hub and multi-Application strategy.
- `modeling_application_auditing.md`: reviewing live Versions and Scenarios.
- `../designing-pigment-boards/board_design_rules.md`: Scenario Planning Board pattern.
- Pigment KB: [Versions and Scenarios](https://kb.pigment.com/docs/versions-scenarios), [Implement Versions and plans](https://kb.pigment.com/docs/managing-versions-in-pigment), [Get started with Scenarios](https://kb.pigment.com/docs/get-started-scenarios), [Compare Data with Data slices](https://kb.pigment.com/docs/compare-versions-with-data-slices).
