---
name: using-mapped-dimensions
description: "Execution skill. Use when modeling dynamic or time-dependent hierarchies, card metrics for flexible reporting (e.g. headcount by team/function via BY-arrow mappings), or any scenario where reporting breakdowns must differ from the structural dimensions on the metric."
metadata:
  skill_path: /skills/using-mapped-dimensions/SKILL.md
  base_directory: /skills/using-mapped-dimensions
---

# Using Mapped Dimensions

Mapped dimensions (time-dependent hierarchies, slowly changing dimensions) model parent-child relationships that **change across periods**. Past periods keep the old parent; future ones reflect the new assignment.

## Choose Between Static Properties and Mapped Dimensions

| Criterion | Dimension-Type Property (static) | Mapped Dimension (dynamic) |
| --- | --- | --- |
| Relationship changes? | Rarely or never | Changes over time or by version |
| History preservation | Updating the parent moves **all history** with the child | Each period retains its own parent assignment |
| Where it works | Metric formulas, aggregation, views | **Views only** (Joined Pivot); core metrics stay lean |
| Examples | Product to Category, Store to Region | Employee to Team (monthly), Cost Center to Department (re-org), Product to Promotion (seasonal) |

**Rule of thumb**: if the parent-child mapping is the same for all periods, use a dimension-type property. If it varies by Month, Quarter, or Version, use a mapped dimension.

## Implement a Mapped Dimension

### Step 1 — Create the Mapping Metric

Create a metric with:

- **Data type**: Dimension (referencing the target parent dimension).
- **Structure**: the source (child) dimension, plus the dimension along which the relationship changes (typically Month; sometimes Version).

Example: `Employee Team Mapping` structured on `Employee x Month`, data type `Dimension` referencing `Team`. Each cell holds the Team assignment for that employee in that month.

Use `tool:create_metric` to create the mapping metric.

### Step 2 — Populate the Mapping

- **Formula**: derive the assignment from other data.

```pigment
IF(Month >= 'Employee'.'Transfer Date', 'Employee'.'New Team', 'Employee'.'Original Team')
```

- **Manual input**: use `tool:set_metric_input`.
- **Import**: no public MCP bulk-import for metrics; ask the user to load mapping values in the Pigment UI, or use `tool:set_metric_input` for small datasets.

### Step 3 — Use Joined Pivot in Views

Use `tool:create_view` then `tool:update_view_pivots` to add the mapped dimension as a **Joined Pivot**. Do **not** add the parent dimension to every underlying metric's structure.

When users view a metric like `Salary` (structured on `Employee x Month`), the Joined Pivot through `Employee Team Mapping` aggregates salaries by Team with each month reflecting the correct assignment.

## Scope and Limitations

- Mapped dimensions work in **Views only**. Cannot reference them in formulas.
- Mapping metric consumes storage proportional to `child items x periods`. Keep manageable.
- Parent dimension does not become part of metric structure — keeps metrics lean, avoids dimension explosion.
- BLANK mapping cells mean the child has no parent for that period and won't appear under any parent group in the View.

## Dimensional Transformation Interaction

- **Case 0 (no modification)**: mapping metric has same dimensions as source metric. No transformation needed.
- **Case 1 (aggregation)**: View aggregates child-level data to mapped parent level. Aggregation operator defined on View pivot.
- **Case 2 (allocation via properties)**: use static property for fixed allocation; mapped dimension when allocation parent changes over time.
- **Case 3 (add/remove dimensions)**: mapped dimensions do not add structural dimensions. Parent appears only in the View.

## Best Practices

- Use only when the relationship genuinely varies over time. Static hierarchies are simpler and more performant.
- Name mapping metrics clearly: `[Child] [Parent] Mapping` (e.g. `Employee Team Mapping`).
- Keep time granularity as coarse as possible (Quarter instead of Month if changes are quarterly).
- Add Version to mapping metric structure when mapping varies by Version as well.
- Document re-org dates so users understand why aggregations shift.

## Compare with Alternative Approaches

| Approach | Pros | Cons |
| --- | --- | --- |
| **Dimension-type property** | Works everywhere (formulas, Views); simple | Overwrites history when parent changes |
| **Mapped dimension** | Preserves history; no metric restructuring | View-only; requires a mapping metric |
| **Separate dimension per period** | Full flexibility | Dimension explosion; hard to maintain |
| **Boolean flag property** | Simple filtering | No aggregation to parent |

Mapped dimensions are the standard solution for time-dependent aggregation without restructuring metrics or losing historical assignments.
