---
name: creating-transaction-lists
description: "Execution skill. Use when creating a transaction list, and whenever you aggregate rows from a transaction list into a metric — including a list that already exists in the application."
metadata:
  skill_path: /skills/creating-transaction-lists/SKILL.md
  base_directory: /skills/creating-transaction-lists
---

# Creating Transaction Lists

## Recognize What a Transaction List Is

A **transaction list** stores high-volume, row-based event data. Each row is one transaction or event.

- Items **do not need to be unique** (many rows can share the same product or date).
- Pigment auto-generates unique row IDs — source data does not need one.
- Represents granular events: orders, GL entries, bookings, movements, invoices.
- **Cannot** be used in a metric's structure — only referenced in formulas.

Transaction lists hold **transactional data**; dimension lists hold **metadata** (products, accounts, employees).

For deciding whether data belongs in a metric or a transaction list, see `skill:creating-metrics-and-tables`.

## Aggregate Transaction Lists into Metrics

Transaction list data reaches metrics through aggregation:

```pigment
'TransactionList'.'NumericProperty'[BY SUM: ...]
```

**Date properties** must map to calendar dimensions through a stored property, never through an inline conversion. Before writing any formula that aggregates a transaction list over time:

1. `tool:create_list_property` — name `Month`, type Dimension, `referencedDimensionId` = the calendar Month list.
2. `tool:update_list_property` — formula `TIMEDIM('Orders'.'Date', Month)`.

Then reference the property in BY:

```pigment
'Orders'.'Amount'[BY SUM: 'Orders'.'Month', 'Orders'.'Product']
```

This applies equally to a list you just created and to one that already exists in the application: an imported transaction list carrying a `Date` property has no time dimension until you add this property. Writing `[BY SUM: TIMEDIM('Orders'.'Date', Month)]` instead returns the right numbers but leaves the mapping buried in every formula that touches the list, so it is not acceptable.

**Text properties** that reference dimension items require conversion:

```pigment
'Orders'.'Amount'[BY SUM: ITEM('Orders'.'ProductCode', 'Product'.'Code'), Month]
```

Workflow:

1. Load raw events into the transaction list.
2. Map every date driving the aggregation to the calendar with a Month property, as above.
3. Create or identify target metric with desired dimensions.
4. Write formula aggregating transaction properties onto metric dimensions.
5. Use the metric (not the transaction list) in reports and downstream calculations.

## Distribute Date-Bounded Values Across Months

When a record has start/end dates (employees with hire/term dates, contracts, leases, subscriptions), use PRORATA to distribute values across active months rather than dumping the full value at a single point.

The stored Month property described above applies here too: map the driving date to the calendar on the list, then reference that property in the formula. Do not use `[ADD: Month]` with manual arithmetic (e.g., dividing annual salary by 12) as this ignores actual active periods.

When a record has no end date, treat it as open-ended (active through the planning horizon).

## Preserve Source Granularity in First-Level Aggregation

Keep all business-relevant dimensions on the intermediate metric, even if the final report will aggregate some away. Removing a dimension during initial aggregation is irreversible. Create the first-level aggregation at full business grain, then create summary metrics that REMOVE dimensions as needed.

This matters when later calculations (cohort analysis, variance, ranking, drill-down) need a dimension that seemed unnecessary at the initial aggregation step.

## How to Create Transaction Lists

Use `tool:create_list` with the transaction list type. Use `tool:create_list_property` to add properties (amount, date, product reference, etc.). Use `tool:add_list_items` to populate it.
