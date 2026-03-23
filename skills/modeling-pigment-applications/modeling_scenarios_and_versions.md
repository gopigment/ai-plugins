# Scenarios vs Versions: When to Use Which

## Overview

Pigment offers two approaches for managing planning cycles, "what-if" analyses, and multiple versions of data:

1. **Native Scenario Feature** - A built-in Pigment feature (like Calendar) for what-ifs scenario and snapshots comparison
2. **Version Dimension** - A regular dimension you create for flexible planning cycle management

**Key Decision Rule:** Use **Native Scenarios ONLY for simple what-if analysis** with independent scenarios. Use **Version Dimension for all planning cycles** (Budget, Actual, Forecast) and virtually all other use cases. Native Scenarios are designed for cases where each scenario is completely independent, when there is no need to reference or calculate across scenarios in formulas. If you need any interaction, calculation, or data referencing between scenarios, use a Version Dimension instead.

Understanding when to use each approach is crucial for effective modeling. This guide helps you make the right choice for your use case.

## Key Concepts

### Planning Cycle Use Cases

Both approaches address common planning needs:

- **Actual vs Budget vs Forecast** - Comparing different planning phases
- **What-If Analysis** - Testing different assumptions and scenarios
- **Version Control** - Managing multiple versions of plans
- **Access Control** - Restricting who can edit which versions/scenarios
- **Historical Tracking** - Maintaining snapshots of past plans

### The Fundamental Difference

**Native Scenarios** are snapshots that exist independently. You cannot reference data from one scenario in another scenario's formulas.

**Version Dimensions** are part of your metric structure. You can reference data across versions in formulas, enabling cross-version calculations and comparisons.

---

## Native Scenario Feature

### What It Is

The **Scenario** feature is a built-in Pigment capability, similar to how Calendar is a native time dimension. You cannot create it manually - it's automatically available in your application.

### How It Works

- Each scenario is a **snapshot** of your data at a point in time
- Scenarios are **independent** - they don't share data or calculations
- You can **compare scenarios** side-by-side in views and boards
- Scenarios support **live data** comparisons (comparing a scenario snapshot to current live data)

### Key Characteristics

✅ **Strengths:**

- Built-in feature - no setup required
- Clean separation between scenarios
- Easy scenario switching in views
- Powerful snapshot comparisons
- Good for "what-if" analyses where scenarios are independent

❌ **Limitations:**

- **Cannot reference other scenarios in formulas** - This is the critical limitation
- No cross-scenario calculations
- No formula-based comparisons between scenarios
- Less flexible than version dimensions
- Cannot use in metric structure for calculations

### When to Use Native Scenarios

**Use native scenarios ONLY for simple what-if analysis:**

1. **Simple What-If Analysis** - Testing different independent assumptions/scenarios
2. **Independent Scenarios** - Each scenario is completely independent with no need to reference other scenarios
3. **No Cross-Scenario Calculations** - You don't need formulas that reference multiple scenarios
4. **Side-by-Side Comparison** - Main use case is comparing independent scenarios

**Example Use Cases:**

- Comparing "Base Case" vs "Optimistic" vs "Pessimistic" scenarios where each is independently calculated
- Simple sensitivity analysis where scenarios don't depend on each other
- Testing different growth rate assumptions independently

**Note:** For virtually all other use cases (planning cycles, Budget vs Actual, Forecast, etc.), use Version Dimension instead.

---

## Version Dimension

### What It Is

A **Version Dimension** is a regular dimension you create (like Product, Region, or Employee) that represents different versions or scenarios of your plan. You add it to your metric structure just like any other dimension.

### How It Works

- Version is a **dimension in your metric structure** (e.g., `Account × Version × Month`)
- You can **reference versions in formulas** - Calculate across versions, compare versions, allocate between versions
- Versions share the **same calculation logic** - Formulas work across all versions
- You can use **Clone data to** functionality to copy data between versions
- You can configure **access rights** per version to control who can edit what

### Key Characteristics

✅ **Strengths:**

- **Full formula flexibility** - Can reference any version in formulas
- Cross-version calculations and comparisons
- Shared calculation logic across versions
- Flexible access control per version
- Can use in metric structure for calculations
- Supports complex planning workflows

❌ **Considerations:**

- Requires creating and managing a dimension
- Adds a dimension to your metric structure (performance consideration)
- More setup and configuration required
- Need to manage version items manually

### When to Use Version Dimension

**Use a version dimension for all planning cycles and most use cases:**

1. **Planning Cycles** - Budget, Actual, Forecast, Plan (all standard planning cycles)
2. **Cross-Version Calculations** - You need formulas that reference multiple versions
3. **Version Comparisons** - Calculate variances, differences, or ratios between versions
4. **Interdependent Versions** - Versions depend on each other or need shared calculations
5. **Flexible Planning Workflows** - Copy data between versions, allocate across versions, complex version-based logic
6. **Access Control Requirements** - Granular control over who can edit which versions
7. **Version as Input Dimension** - Users input data directly into different versions
8. **Rolling Forecasts** - Forecasts that reference actuals or previous forecasts
9. **Historical Tracking** - Maintaining versions with calculations or comparisons

**Example Use Cases:**

- Budget vs Actual vs Forecast (standard planning cycles)
- Rolling forecasts where each version builds on previous versions
- Multi-version planning where versions reference each other
- Version-based allocations (e.g., allocate corporate targets to versions)
- Variance analysis requiring formula-based comparisons
- Any planning cycle where you need formula flexibility

**General Rule:** If it's not simple, independent what-if analysis, use Version Dimension.

---

## Comparison Table

| Aspect                          | Native Scenarios                    | Version Dimension                       |
| ------------------------------- | ----------------------------------- | --------------------------------------- |
| **Setup**                       | Built-in, no setup                  | Create dimension, add to structure      |
| **Formula Flexibility**         | ❌ Cannot reference other scenarios | ✅ Can reference any version            |
| **Cross-Scenario Calculations** | ❌ Not supported                    | ✅ Fully supported                      |
| **Snapshot Comparisons**        | ✅ Built-in comparison views        | ✅ Can compare in views                 |
| **Live Data Comparisons**       | ✅ Compare snapshot to live         | ✅ Can compare versions to live         |
| **Access Control**              | Limited                             | ✅ Granular per version                 |
| **Data Copying**                | Manual snapshots                    | ✅ Clone data to functionality          |
| **Metric Structure**            | ❌ Cannot add to structure          | ✅ Part of metric structure             |
| **Performance**                 | Independent snapshots               | Adds dimension (sparsity consideration) |
| **Use Case Fit**                | Independent scenarios               | Interdependent versions                 |
| **Complexity**                  | Simple                              | More complex setup                      |

---

## Decision Framework

Use this simple decision rule:

### Primary Rule: Use Native Scenarios Only for What-If Analysis

**Use Native Scenarios** only when:

- You need **simple what-if analysis** with independent scenarios
- Scenarios are completely independent (no cross-scenario dependencies)
- You don't need cross-scenario calculations in formulas
- Main use case is comparing independent "what-if" scenarios side-by-side

**Use Version Dimension** for everything else:

- Planning cycles (Budget, Actual, Forecast)
- Cross-version calculations and comparisons
- Version-based allocations or transformations
- Access control per version
- Any scenario/version that needs to reference other scenarios/versions in formulas
- Rolling forecasts or interdependent planning cycles
- Historical tracking with calculations

### Quick Decision Guide

| Use Case                                        | Approach          |
| ----------------------------------------------- | ----------------- |
| Simple what-if analysis (independent scenarios) | Native Scenarios  |
| Budget vs Actual                                | Version Dimension |
| Forecast that references Actual                 | Version Dimension |
| Variance calculations                           | Version Dimension |
| Cross-scenario formulas                         | Version Dimension |
| Planning cycles                                 | Version Dimension |
| Version-based access control                    | Version Dimension |
| Historical snapshots with comparisons           | Version Dimension |
| Independent "what-if" scenarios                 | Native Scenarios  |

**General Recommendation:** Unless you're doing simple, independent what-if analysis, use **Version Dimension**. It provides the flexibility needed for most planning use cases.

---

## Examples

### Example 1: Simple What-If Analysis → Use Native Scenarios

**Use Case:** Compare three independent revenue scenarios (Base, Optimistic, Pessimistic) based on different growth assumptions.

**Why Native Scenarios:**

- Scenarios are independent (no cross-scenario calculations)
- Main need is comparison
- Simple setup

**Implementation:**

- Create three scenarios: "Base Case", "Optimistic", "Pessimistic"
- Input data independently into each scenario
- Compare scenarios side-by-side in views

### Example 2: Budget vs Actual with Variance → Use Version Dimension

**Use Case:** Track Budget and Actual, calculate variance, and have Actual affect future Budget calculations.

**Why Version Dimension:**

- Need formula: `Variance = Actual - Budget`
- Need formula: `Next Period Budget = IF(Actual > Budget, Actual * 1.1, Budget)`
- Versions are interdependent

**Implementation:**

- Create Version dimension with items: "Budget", "Actual", "Forecast"
- Add Version to metric structure: `Account × Version × Month`
- Write formulas that reference versions: `'Revenue'[FILTER: Version = Version."Actual"] - 'Revenue'[FILTER: Version = Version."Budget"]`

### Example 3: Rolling Forecast → Use Version Dimension

**Use Case:** Quarterly rolling forecast where Q2 Forecast references Q1 Actual.

**Why Version Dimension:**

- Need cross-version references: `Q2 Forecast = Q1 Actual * Growth Factor`
- Versions build on each other
- Complex planning logic

**Implementation:**

- Create Version dimension: "Q1 Actual", "Q2 Forecast", "Q3 Forecast", etc.
- Add Version to metric structure
- Write formulas: `IF(Version = Version."Q2 Forecast", 'Revenue'[FILTER: Version = Version."Q1 Actual"] * 1.05, ...)`

### Example 4: Historical Tracking → Use Version Dimension

**Use Case:** Maintain versions of plans at different points in time for historical tracking and comparison.

**Why Version Dimension:**

- Even for historical tracking, you typically need to compare versions or calculate changes
- More flexible for future needs (calculations, comparisons)
- Better access control per version
- Can calculate variances between historical versions

**Implementation:**

- Create Version dimension: "Plan Jan 1", "Plan Feb 1", "Plan Mar 1"
- Add Version to metric structure: `Account × Version × Month`
- Can calculate changes: `'Plan Feb 1' - 'Plan Jan 1'`
- Compare versions in views

**Note:** If snapshots are truly independent with no comparison needs, Native Scenarios could work, but Version Dimension is generally preferred for flexibility.

### Example 5: Multi-Version Planning with Allocations → Use Version Dimension

**Use Case:** Corporate sets targets at "Corporate" version, allocate to "Division" versions, then aggregate to "Consolidated" version.

**Why Version Dimension:**

- Need cross-version calculations
- Versions are part of planning workflow
- Complex version-based logic

**Implementation:**

- Create Version dimension: "Corporate", "Division A", "Division B", "Consolidated"
- Add Version to metric structure
- Write formulas: `IF(Version = 'Consolidated', SUM('Revenue'[BY: Version WHERE Version IN ('Division A', 'Division B')]), ...)`

---

## Implementation Guidance

### Implementing Native Scenarios

1. **No setup required** - Scenarios are built-in
2. **Create scenarios** through the Pigment UI
3. **Input data** into each scenario independently
4. **Compare scenarios** in views using scenario filters
5. **Take snapshots** to preserve scenario state

**Best Practices:**

- Use clear, descriptive scenario names
- Document scenario assumptions
- Take snapshots before major changes
- Use consistent naming conventions

### Implementing Version Dimension

1. **Create Version dimension** in your application (typically in a hub application)
2. **Add version items** (e.g., "Budget", "Actual", "Forecast", "Plan")
3. **Add Version to metric structure** - Include in metrics that need versioning
4. **Write formulas** that reference versions using FILTER or SELECT modifiers
5. **Configure access rights** per version to control editing permissions
6. **Use Clone data to** to copy data between versions

For the full step-by-step (switchover, properties, key metrics, and actual/plan layering), see **Building a versioning system** below.

**Best Practices:**

- **Create in Hub Application**: If using multi-app architecture, create Version dimension in hub app and share it
- **Naming Convention**: Use clear names (e.g., "FY25 Budget", "FY25 Actual", "FY25 Forecast")
- **Structure Decision**: Only add Version to metrics that need versioning (don't add unnecessarily)
- **Access Rights**: Configure read/write access per version for proper governance
- **Clone Strategy**: Use "Clone data to" to initialize new versions from existing ones
- **Live Version**: Maintain one "live" version that's regularly updated

**Example Structure:**

```
Metric: Revenue
Structure: Account × Version × Month × Region

Version Dimension Items:
- FY25 Budget
- FY25 Actual
- FY25 Forecast
- FY26 Plan
```

**Example Formula with Versions:**

```
'Revenue'[FILTER: Version = Version."FY25 Actual"] - 'Revenue'[FILTER: Version = Version."FY25 Budget"]
```

---

## Building a versioning system: switchover, properties, and data layering

This section describes how to implement a full versioning system in Pigment: types of plans and data, switchover logic, version properties, key boolean metrics, and how to layer actuals with plan data in formulas. Use it when you have chosen a Version Dimension (see above) and need the concrete implementation pattern.

### Types of plans and types of data

**Common types of plans:**

- **Budget**: A fixed annual plan, usually built at the start of the fiscal year. Serves as a target and baseline for performance. Budgets are typically not adjusted once finalised.
- **Reforecasts**: Periodic updates to the original forecast or budget (e.g. Q1, Q2, Q3 Reforecasts). Used to reassess performance mid-year and adjust expectations.
- **Live / Rolling forecast**: An evolving plan that combines actuals to date with forecasted data for future months, regularly updated.

**Important:** Budgets are static targets. Forecasts and Reforecasts are dynamic expectations that evolve with reality.

**Types of data in a plan:**

- **Actuals**: Historical data from past months, imported from business systems (e.g. GL, CRM, HRIS).
- **Plan data**: Future-looking data from projections, assumptions, and calculations.

The way actuals and plan data are combined is **per version** and is controlled by **Switchover logic** (see below).

### Version dimension and naming

- Create a **Version dimension** (standard Dimension list) and add **one item per planning cycle** (e.g. "Budget FY26", "Reforecast Q2 FY25 Pessimistic").
- **Naming:** Include either the **creation year** or the **version window span** in the name. Remain consistent; naming can be automated with formulas on Version dimension properties (e.g. from Version Type, Scenario, Planning Years).
- **Why:** Improves clarity, keeps a chronological record, and simplifies version creation: use **Clone data to** to copy assumptions from a previous cycle into a new version.

### Switchover logic

- Add a **Switchover Month** property (dimension type Month) on the Version dimension. It defines the **last month of actual data** that applies to that version. Months after the switchover are plan.
- **Effect:** Each version has its own cut-off. For example:
  - **Budget FY26**: Actuals up to December 2025; January–December 2026 are plan.
  - **Reforecast Q1 FY26**: Actuals for Jan–Mar 2026; April–December 2026 are plan (recalculated).
- If planning is at **Year** granularity, use **Switchover Year** instead; the approach is the same.

### Optional "Actual" version

Including a dedicated **"Actual"** version in the Version dimension is **optional**.

**Benefits:** Simplifies references in formulas (e.g. explicit "Actual" for comparisons); useful when Reforecast cycles leave gaps (e.g. need May actuals before Q2 Reforecast starts).

**Drawbacks:** Actuals are often already included in each plan version via switchover logic, so a separate Actual version can mean redundant calculation and extra maintenance.

**Recommendation:** Use an "Actual" version only if you need to isolate actuals in formulas or reports, or if Reforecast cycles leave gaps in coverage. Otherwise, rely on switchover logic within each version.

**If you do include an "Actual" version:** Use a **single** item (e.g. "Actuals"), not one per year ("Actuals 2023", "Actuals 2024"). Update the **Switchover Month** each month to bring in the latest actuals. This keeps the dimension lean.

### Version dimension properties

**Mandatory properties:**

- **Start Month** (Dimension → Month): Starting month of the version window.
- **End Month** (Dimension → Month): Ending month of the version window.
- **Active Version** (Boolean): Identifies versions currently used for input or reporting.
- **Lock Version** (Boolean): Locks versions from edits once approved. Used with Access Rights (e.g. Actual version always locked; others locked after approval).

**Optional properties:**

- **Version Type** (Dimension): Categorise versions (e.g. "Budget", "Reforecast", "Rolling Forecast", "Long Range Planning"). Useful to reference by type in formulas (e.g. for Test & Deploy compatibility) and to apply different calculation rules per type.
- **Scenario** (Dimension): Categorise scenarios within a version (e.g. "Expected", "Optimistic", "Pessimistic").
- **Version #** (Number): Incremental number for multiple occurrences of a version (e.g. for comparison); clean up to keep a single validated version for reference.

Version names can be automated via formulas using Version Type, Planning Years (from Switchover Month + 1), Creation Year, Scenario, and Version #.

**Hygiene:** Only keep **active** versions (used for planning) and **locked** versions (needed for reference). Regularly review and remove outdated items.

### Key version metrics (boolean)

Create three **Boolean metrics** that define the month range and actual vs plan for each version:

1. **Is Version**: TRUE for months between the version's Start Month and End Month. Defines the **version window**.
2. **Is Actual**: TRUE for months inside the version window **up to and including** the Switchover Month. These months use actual data.
3. **Is Plan**: TRUE for months inside the version window **after** the Switchover Month. These months use plan/forecast data.

Use these metrics in formulas to:

- Layer data correctly by version (bring actuals into each version according to its Switchover Month).
- Apply different assumptions per version in calculations and reports.

### Layering actuals and plan in formulas

- Create a **metric for actuals by version** (e.g. "Revenue Actual" at Account × Version × Month), fed from your data source and gated so it only appears where **Is Actual** is TRUE (or equivalent logic).
- Create a **forecast / plan metric by version** that uses assumptions and calculations for future months.
- Build a **final metric** that combines both: for each Version × Month, use actuals when **Is Actual** is TRUE and plan when **Is Plan** is TRUE. Pattern:

  - `IF(Is Actual, Revenue_Actual, Revenue_Plan)` at the same grain (e.g. Account × Version × Month), or equivalent logic using the boolean metrics to select the right source.

- The result is one metric containing both actual and plan data per version, ready for reporting and variance analysis.

### Optional: Data Type for display

- Create a **Data Type** dimension with items such as "Actual" and "Plan".
- Create a **metric by Version × Month** (and other dimensions as needed) that returns the appropriate Data Type item (e.g. "Actual" when Is Actual is TRUE, "Plan" when Is Plan is TRUE).
- Use **Mapped dimensions** in the view so that the Data Type appears in the table or board, making it easy to see which cells are actual vs plan.

### Summary of implementation steps

1. Create Version dimension; add items (naming: creation year or window span).
2. Add Switchover Month (or Year) and mandatory properties (Start Month, End Month, Active Version, Lock Version); add optional ones if needed.
3. Create key metrics: Is Version, Is Actual, Is Plan.
4. Build actual metric by version and plan/forecast metric by version; combine them in a final metric using Is Actual / Is Plan.
5. Use Clone data to initialize new versions from existing ones.
6. Optionally add Data Type metric and Mapped Dimension for display.
7. Keep only active and locked versions; clean up regularly.

---

## Pigment Modeling Best Practices Rule Reference

This topic relates to **MG12 - Planning Cycle: Use Version Dimension** from the Pigment Modeling Best Practices:

> The Scenario feature that Pigment offers natively, as opposed to a classical Dimension that would be called Scenario or Version, is a robust tool designed to facilitate "What if?" analyses. It allows you to model each planning cycle exercise (Actual, Budget, Forecast) as separate Scenarios. It doesn't support cross-Scenario calculations or data referencing through formulas, but it allows for powerful comparisons between Scenario Snapshots and live data.
>
> **Recommendation:** For more flexibility and to fully accommodate your planning needs, using a normal **Dimension** to model your planning cycle is recommended.

**Key Takeaway:** While native scenarios have their place, version dimensions are generally recommended for most planning use cases due to their flexibility and formula support.

---

## Common Mistakes to Avoid

### Mistake 1: Using Native Scenarios When You Need Cross-Scenario Calculations

**Problem:** Trying to calculate variance between scenarios using formulas.

**Solution:** Use Version Dimension instead, which supports formula-based comparisons.

### Mistake 2: Creating a "Scenario" Dimension When Native Scenarios Exist

**Problem:** Creating a dimension called "Scenario" when you could use native scenarios.

**Solution:** If you don't need cross-scenario calculations, use native scenarios. If you do need them, use a Version dimension (not named "Scenario" to avoid confusion).

### Mistake 3: Adding Version to All Metrics Unnecessarily

**Problem:** Adding Version dimension to every metric, even those that don't need versioning.

**Solution:** Only add Version to metrics that require versioning. This improves performance and simplifies the model.

### Mistake 4: Not Using Clone Data To

**Problem:** Manually copying data between versions.

**Solution:** Use the "Clone data to" functionality to efficiently copy data between versions.

### Mistake 5: Using REMOVE on Version

**Problem:** Using `[REMOVE: Version]` (or otherwise aggregating over Version) in a formula. That mixes all scenarios (Actual, Budget, Forecast, etc.) into one value and breaks version-specific meaning, comparisons, and reporting.

**Solution:** Keep Version in the metric structure. When you need a single version, use FILTER or SELECT on Version, not REMOVE. Version is a scenario axis and must not be aggregated away.

---

## Summary

**Choose Native Scenarios when:**

- **Only for simple what-if analysis** with independent scenarios
- Scenarios are completely independent (no dependencies)
- You don't need cross-scenario calculations
- Main use case is comparing independent "what-if" scenarios

**Choose Version Dimension when:**

- **For all planning cycles** (Budget, Actual, Forecast)
- You need cross-version calculations in formulas
- Versions are interdependent
- You need flexible planning workflows
- You require granular access control
- You need version-based allocations or transformations
- **For virtually all real-world planning use cases**

**General Recommendation:** Use **Version Dimension** for almost all planning use cases. Use Native Scenarios **only** for simple, independent what-if analysis where scenarios don't need to reference each other.

---

## See Also

- [Modeling Principles](./modeling_principles.md) - MG12 rule on planning cycles
- [Modeling Fundamentals](./modeling_fundamentals.md) - Example 4: Scenario Planning
- [Board Design Patterns](../designing-pigment-boards/board_design_rules.md) - Scenario Planning Board pattern
