---
name: using-list-subsets
description: "Execution skill. Use when creating, modifying, or reasoning about List Subsets — mirror dimensions, same-dimension-on-two-axes, cohort modeling, restricted dropdowns, iterative performance optimization, or any request involving subsets."
metadata:
  skill_path: /skills/using-list-subsets/SKILL.md
  base_directory: /skills/using-list-subsets
---

# Using List Subsets

A **subset** is a selected group of items from a parent dimension list, usable as its own dimension in metric structure. Use only when they deliver clear modeling or performance benefits.

## Subset Mental Model

A subset behaves as a **separate dimension** in structures and formulas:

- Metrics dimensioned by the subset and by the parent are **different shapes** — not interchangeable without explicit mapping.
- Subsets inherit items, properties, order, and sharing behavior from the parent list.
- Deselecting an item from a subset **permanently deletes** associated datapoints in metrics dimensioned by that subset. Affects all scenarios; not reversed if re-added.

## Decision Checklist

Go through these five gates before recommending a subset:

1. **Security or filtering goal?** Use Access Rights or view filters/Boolean properties. Do not propose subsets.
2. **Smaller input list?** If membership changes and there is no iterative/performance need, prefer a separate regular list.
3. **Iterative calculation on a subset of members?** If PREVIOUS/PREVIOUSOF iterates over a large dimension but only a few members matter, a subset reduces compute. Remap results to parent afterward.
4. **Same dimension needed twice in one metric (mirror dimension)?** Subsets are the standard solution.
5. **Manual inputs with changing membership?** Use Pattern A (STORE/CALC) or avoid subsets entirely.

Only proceed if gate 3 or 4 is clearly met and risks are manageable.

## When to Recommend Subsets

**Strongly recommend:**

- **Mirror dimensions** (same list twice in one metric). Examples: cohort modeling (Month as Time + Month as Cohort), intercompany elimination (Company in rows + columns).
- **Restricted dropdown UX** when parent list is very large. Use subset as selection dimension; store data on parent via TOPARENTLIST (Pattern B).

**Recommend with caveats (performance only):**

- **Iterative calculation optimization** where parent list is large and subset materially reduces the iterating dimension. Remap results to parent downstream (Pattern C).

## When NOT to Recommend Subsets

- **Focused analysis** (active entities, top countries) — use view filters or Boolean properties. Data-loss overhead rarely worth it.
- **Time subsets** (forecast months, open periods) — same data-loss risk. Prefer filters or helper metrics (`Is Forecast Month` Boolean). Use only in specific performance/mirror scenarios with explicit risk acknowledgement.
- **Changing membership without review** — data loss is likely.

Prefer a regular list when membership changes often, users input data at that level, and there is no mirror-dimension or iterative need.

## Three Mandatory Warnings

### Warning 1 — Data Loss Is Irreversible

Deselecting an item permanently deletes datapoints in metrics dimensioned by that subset. Affects all scenarios; not reversed on re-add. Especially dangerous when subset membership is formula-driven and users enter manual inputs. Suggest Pattern A (STORE/CALC) or store on parent.

### Warning 2 — Subset and Parent Are Different Dimensions

A metric on the subset is not interchangeable with one on the parent without remapping:

- **TOPARENTLIST**: maps subset expression to parent (items not in subset are BLANK).
- **TOSUBSET**: maps parent expression to subset (items outside subset are dropped).
- When the compiler rejects subset modifiers or multiple subsets feed one block, use mapping properties and `[BY: ...]`.

### Warning 3 — Operational Overhead

Safe subset usage requires storage/helper metrics, imports, board workflows, and owners who understand membership changes. Do not propose subsets if governance is lacking.

## Safe Implementation Patterns

### Pattern A — Safe Formula-Driven Subset (STORE/CALC)

Use when subset membership logic may change and there are manual inputs in metrics dimensioned by the subset.

1. Use `tool:create_metric` to create two metrics on the **parent list**:
   - `CALC_Subset` (Boolean, formula): determines if an item should be in the subset now.
   - `STORE_Subset` (Boolean, manual/storage): linked to subset membership.
2. In `CALC_Subset`, implement selection logic excluding already-stored members:

```pigment
IF(
  'Parent'.'Some Property' = ...,
  TRUE,
  FALSE
)
[EXCLUDE: 'STORE_Subset']
```

3. Use `tool:create_metric_copy_config` to create a Metric-to-Metric copy from `CALC_Subset` to `STORE_Subset` with **Clear Values OFF**. Board button creation is UI-only; ask the user to configure it in the Pigment UI.
4. No agent tool available for linking subset membership to a metric. Ask the user to link subset membership to `STORE_Subset` in the Pigment UI (list settings → Subset membership).

New candidates are added automatically; existing members stay unless explicitly removed.

When not needed: if all metrics using the subset are fully formula-driven with no manual inputs, drive membership directly from a formula metric.

### Pattern B — Restricted Dropdown UX (Store on Parent)

Use when you want a curated subset for user selections but data should live on the parent dimension.

1. Create the subset (e.g. `Supplier_Subset`) from the parent using `tool:create_sublist`.
2. Use the subset as the selection dimension in tables or forms.
3. Remap back to the parent:

```pigment
'Selected Supplier on Parent' =
  'Selected Supplier (Subset)'[TOPARENTLIST: 'Supplier_Subset']
```

Most persistent data lives on the parent list, less likely to have items removed.

### Pattern C — Remap Subset to Parent (General)

Use when you compute at subset level but need results at parent level.

```pigment
'Metric on Parent' =
  'Metric on Subset'[TOPARENTLIST: 'Subset']
```

For different subsets mapping into the same parent, centralize mappings and use `[BY: ...]` consistently. Prefer TOPARENTLIST/TOSUBSET for straight remaps; use reusable mapping metrics/properties for multi-subset shapes.

### Pattern D: Cohort Mirror (Month as Time + Month as Cohort)

When a metric needs the same dimension on two axes (e.g. MRR by reporting Month and by Cohort Month), create a List Subset:

1. `tool:create_sublist` on the Month dimension to create a "Cohort Month" subset
2. Add a Cohort property on the entity dimension (e.g. Customer) typed to the Cohort Month subset
3. Formula: `'Invoice Date'[BY MIN: 'Invoices'.'Customer']` mapped to the subset via `[-> 'Cohort Month']`
4. Cohort metric: `'MRR'[BY SUM: 'Customer'.'Cohort Month']`

**Immediately after `tool:create_sublist`:** populate subset membership. No agent tool is available for linking subset membership programmatically. Ask the user to link subset membership in the Pigment UI. Do NOT skip this step — an empty subset will produce blank values for all formulas that reference it.

Do NOT hand-build a mirror list with `tool:add_list_items` manually populating items. Use `tool:create_sublist` instead.

## Follow CRUD Order

Create subsets using `tool:create_sublist` only after the parent dimension is stable. Before deleting a parent list, remove its subsets first.
