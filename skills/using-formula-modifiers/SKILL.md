---
name: using-formula-modifiers
description: "Execution skill. Use when a formula's source metric dimensions differ from the target metric — apply BY, ADD, REMOVE, KEEP, SELECT, FILTER, EXCLUDE, TOPARENTLIST, or TOSUBSET. Also covers arrow-mapping syntax (BY ... ->) and tiered lookup patterns."
metadata:
  skill_path: /skills/using-formula-modifiers/SKILL.md
  base_directory: /skills/using-formula-modifiers
---

# Apply Formula Modifiers

Modifiers appear in square brackets after an expression: `expression[MODIFIER: argument]`. They change dimensionality or filter data so a source metric aligns with the target metric.

## Distinguish Source from Target

- **Source** — the metric or expression you reference (right of `=`).
- **Target** — the metric whose formula you are writing; its dimension list defines the output shape.
  Every modifier maps source dimensions toward target dimensions. Check target dimensions first, then pick modifiers.

## Dimension Relationship Types

| Source→Target | Type | Modifier | Methods |
| --- | --- | --- | --- |
| **N→1** | Aggregation | `BY` | SUM, AVG, MIN, MAX, COUNT |
| **N→none** | Aggregation | `REMOVE` | SUM, AVG, MIN, MAX, COUNT |
| **N→none (filtered)** | Conditional agg. | `SELECT` | SUM, AVG, MIN, MAX, COUNT |
| **1→N** | Allocation | `BY` | CONSTANT, SPLIT |
| **none→N** | Allocation | `ADD` | CONSTANT, SPLIT |
| **Defaults**: SUM for aggregation, CONSTANT for allocation. |

**Before choosing a BY method**: check which dimension the metric *currently* has, not just which pair appears in an example below. Dimension already present → aggregating (SUM/AVG/...). Dimension missing and being added → allocating (CONSTANT/SPLIT).

## Aggregate with BY (N→1)

Roll up from a finer dimension to a coarser one via a mapping property:

```pigment
'Employee Headcount'[BY SUM: 'Employee'.'Department']
'Warehouse Capacity'[BY AVG: 'Month'.'Quarter']
```

`[BY: dim.prop]` removes `dim` and adds `prop`'s dimension — no separate REMOVE needed. Multi-level hierarchies chain BY: `'Revenue'[BY SUM: 'Month'.'Quarter'][BY SUM: 'Quarter'.'Year']`.
When a dimension-typed metric defines the mapping, BY respects its sparsity automatically — do not add IF(ISBLANK(...)) guards.

## Allocate with BY (1→N)

```pigment
'Region Cap'[BY CONSTANT: 'Country'.'Region']    // same value to every country
'Region Revenue'[BY SPLIT: 'Country'.'Region']   // total divided equally across countries
```

`CONSTANT` copies the same value to each child. `SPLIT` divides equally.
If the request says "split," "distribute," "divide equally," or "spread" — use SPLIT, not the CONSTANT default.

## BY with Arrow Syntax

Use arrow (`->`) when the mapping lives in a dimension-typed metric or you need explicit control over which dimensions are replaced, kept, or added.
**Syntax**: `Source[BY Method: SourceDim1, SourceDim2 -> MappingMetric, ExtraDim]`

- **Left of `->` (replace)** — dimensions removed and replaced by the mapping's target dimension.
- **Right of `->` (mapping/add)** — mapping metrics/properties defining new dimensions, plus extra grouping dims.
- **Empty left** (`BY CONSTANT: -> 'Map'`) — adds a dimension without replacing any.
- **Unlisted source dims** — silently aggregated using the BY method. Typical "at risk" dims: Version, Time, Scenario, Currency.
  Use **arrow** when the mapping is a dimension-typed metric or multiple dims/properties in one BY. Use **plain property** (`'Country'.'Region'`) for simple hierarchy lookups.

### Over-Aggregation Pitfall

```pigment
// Source dims = Account, Version. Mapping dims = Account, Version, type = Segment.
// WRONG — Version silently aggregated:
'Metric_X'[BY COUNT: 'Account to Segment Map', 'Account'.'Market']
// CORRECT — Account replaced, Version preserved:
'Metric_X'[BY COUNT: Account -> 'Account to Segment Map', 'Account'.'Market']
```

### Arrow Checklist

1. List all source dimensions (source metric + mapping metrics).
2. Decide for each: replace (before `->`)| keep (ensure present) | add (after `->`).
3. Watch for silent dims (Version, Time, Scenario, Currency).
4. Ask: "What are the final dimensions?" If an expected axis is missing, revisit rules 1-3.

## Add a Dimension with ADD (none→N, dense)

`ADD` introduces a dimension at full cardinality — every combination. This densifies data.

```pigment
'Annual Budget'[ADD: 'Month']
'Total Budget'[ADD SPLIT: 'Country']
```

**Methods**: CONSTANT (default), SPLIT. Prefer `BY` when a mapping exists — BY is sparse; ADD is dense. See `skill:writing-performant-formulas`.

## Remove a Dimension with REMOVE (N→none)

```pigment
'Revenue'[REMOVE SUM: 'Country']
'Revenue'[REMOVE FIRSTNONBLANK: 'Version']
```

Default is SUM. Removes the dimension entirely. Use REMOVE to drop an axis; `[BY SUM: Dim]` on an existing `Dim` does not remove it.
**Tiered/banded lookup** — assign each item to a band based on thresholds:

```pigment
IF(
  'DATA_Value' > 'INPUT_Tier_Threshold',
  Tier
)[REMOVE FIRSTNONBLANK: Tier]
```

Evaluates over Account × Tier, picks the first qualifying tier per Account. Use `>` for floor-based, `<` for ceiling-based. Dimension item order determines FIRSTNONBLANK vs LASTNONBLANK. Define a catch-all tier with threshold 0 as fallback.

## Keep Only Listed Dimensions with KEEP

```pigment
'Revenue'[KEEP: 'Country', 'Month']
'Revenue'[KEEP FIRSTNONBLANK ON RANK(Product): 'Country', 'Month']
```

Removes every dimension not listed. **Methods**: SUM (default), AVG, MIN, MAX, FIRST, FIRSTNONBLANK. FIRST/FIRSTNONBLANK with multiple dimensions require `ON RANK(Dimension)`.

## Filter and Remove Dimensions — SELECT

```pigment
'Revenue'[SELECT: 'Country' = SET_Selected_Country]
```

SELECT filters then removes the filtered dimension(s). **Methods**: SUM (default), AVG, MIN, MAX, COUNT.

** If the target metric still needs this dimension, do not use SELECT.** SELECT removing a dimension and a later `[ADD CONSTANT: Dim]`/`[BY CONSTANT: Dim]` re-adding it are **not inverses** — CONSTANT broadcasts the one surviving value to *every* item on that dimension instead of restoring it only to its original item. Use FILTER instead: it keeps the dimension and blanks every non-matching item.

### SELECT Modes

| Mode | Syntax | Behavior |
| --- | --- | --- |
| **Filter & remove** | `Metric[SELECT: Dim = item]` | Filters to matching rows, removes the dimension |
| **1:1 mapping lookup** | `Metric[SELECT: DimA.TargetItem]` | Reads value at the mapped target item |
| **Aggregate & allocate** | `Revenue[SELECT: Month.Quarter]` | Aggregates up then replicates to each source item |
| **Offset** | `Metric[SELECT: Month - 12]` | Shifts dimension value; **keeps** the dimension |

```pigment
'Revenue'[SELECT: Month - 12]              // Same month last year (keeps Month)
'Revenue'[SELECT: 'Month'.'Quarter']       // Quarterly total replicated to each month
```

Use **Filter & remove** mode only when the target metric genuinely drops the dimension. If the target keeps the dimension, use FILTER — never SELECT followed by ADD/BY CONSTANT to put the dimension back.

**Performance**: SELECT is fast (parallel). PREVIOUS/PREVIOUSOF are slow (iterative) — use only when current value depends on prior. See `skill:using-formula-functions`.

## Filter Without Removing Dimensions — FILTER

```pigment
'Revenue'[FILTER: 'Country' = SET_Selected_Country]
'Revenue'[FILTER: CurrentValue > 1000]
'Revenue'[FILTER: 'Is Active']
```

FILTER applies a boolean condition but keeps all original dimensions. Non-matching cells become BLANK. Shorthand: `'Revenue'[CurrentValue > 1000]`.

## Exclude Matching Rows — EXCLUDE

```pigment
'Revenue'[EXCLUDE: 'Country' = SET_Excluded_Country]
'Revenue'[EXCLUDE: 'Country'.'Exclude from Report']
```

Removes rows where the condition is true. Opposite of FILTER. **FILTER vs EXCLUDE blank semantics**: FILTER blanks are excluded (cells outside filter become BLANK). EXCLUDE blanks are included (kept). Always prefer EXCLUDE for exclusions — `FILTER: NOT(...)` over a boolean that can be BLANK densifies the result.

## Remap Subset ↔ Parent List

```pigment
'Revenue'[TOPARENTLIST: 'ActiveProducts']   // subset → parent (1:1)
'Revenue'[TOSUBSET: 'ActiveProducts']       // parent → subset (1:1)
```

Use when source and target differ only by subset vs full list, not aggregation.

## Transaction Lists — Always Use BY

Transaction lists cannot structure a metric directly. Aggregate into target dimensions with a single BY using comma-separated mapping expressions. Never chain multiple BY — it loses list properties:

```pigment
// WRONG — chained BY loses list properties
'Orders'.'Amount'[BY SUM: 'Orders'.'Month'][BY SUM: 'Orders'.'Product']
// CORRECT — single BY, comma-separated
'Orders'.'Amount'[BY SUM: 'Orders'.'Customer', 'Orders'.'Product', 'Orders'.'Month']
```

- **Dimension-typed columns**: use directly — `'Orders'.'Customer'`
- **Text columns referencing dims**: convert with ITEM() — `ITEM('Orders'.'ProductCode', 'Products'.'Code')`
- **Date columns**: reference a stored Month property, `'Orders'.'Month'`. If the list does not have one yet, add it first (type: Dimension targeting Month, formula `TIMEDIM('Orders'.'Date', Month)`) — see `skill:creating-transaction-lists`. Do not inline `TIMEDIM(...)` in the BY modifier.

## Method Catalog

| Context | Number/Integer | Date | Boolean | Text | All types |
| --- | --- | --- | --- | --- | --- |
| **Aggregation** (BY, REMOVE, KEEP, SELECT) | SUM*, AVG, MEDIAN, STDEVS, STDEVP, FIRSTNONZERO, LASTNONZERO | MIN, MAX | ANY, ALL | TEXTLIST | FIRST, LAST, FIRSTNONBLANK, LASTNONBLANK, COUNT, COUNTBLANK, COUNTALL, COUNTUNIQUE |
| **Allocation** (BY, ADD) | CONSTANT*, SPLIT | — | — | — | — |
| \* = default method. |

**Earliest/first event per entity**: use `Date[BY MIN: 'TL'.'Entity']`. Prefer `BY MIN` over `BY FIRST` — `BY FIRST` only returns the earliest date if the dimension happens to be chronologically ordered, which is not guaranteed.

## Common Patterns

```pigment
// Weighted average
('Revenue' * 'Margin')[BY SUM: Country.Region] / 'Revenue'[BY SUM: Country.Region]

// Ratio to year total — only Month grain changes
'Metric' / 'Metric'[BY SUM: Month.'Year'][BY CONSTANT: Month.'Year']

// Percentage of total
'Country Revenue' / 'Country Revenue'[REMOVE: Country]

// Aggregate a transaction list to Year via the stored Month property
'Orders'.'Amount'[BY SUM: 'Orders'.'Month'.'Year']

// Filter a transaction list before aggregating
'Orders'.'Amount'[SELECT: 'Orders'.'Date' >= DATE(2024,1,1)][BY: 'Orders'.'Month']
```

## Modifier Chaining

Two equivalent syntaxes: `Block[mod1][mod2]` or `Block[mod1; mod2]` (semicolon). **Caution**: chained non-commutative aggregators (e.g. `[BY AVG: dim1][BY AVG: dim2]`) can yield different results than `[BY AVG: dim1, dim2]` when blanks exist (average of averages ≠ average of all).

## Implicit Modifier Behavior

- **Fewer target dims** → implicit REMOVE SUM.
- **More target dims** → implicit ADD CONSTANT.
- **Different dims (no mapping)** → implicit REMOVE SUM + ADD CONSTANT.
- Use explicit modifiers when changing grain. "Explicit" means specifying the transformation, not repeating unchanged dimensions.

## Critical Rules

1. **BY changes only the dimension you specify** — list only dimensions whose grain is changing.
2. **Never chain BY on transaction lists** — one BY, comma-separated mappings.
3. **BY before ADD** — when a mapping exists, use BY (sparse); use ADD only for full dense cardinality.
4. **Match target dimensions** — the final expression dimensions must equal the target metric's dimensions.
5. **Default methods** — SUM for aggregation, CONSTANT for allocation. Specify explicitly for AVG, MIN, MAX, COUNT, SPLIT.
6. **Arrow syntax: account for all dimensions** — any source dimension not before `->` and not added after `->` is silently aggregated away.
7. **EXCLUDE over FILTER NOT** — prefer EXCLUDE for exclusions to avoid densifying blanks.
8. **SELECT offset keeps the dimension** — unlike other SELECT modes that remove the filtered dimension.
9. **Identifier quoting** — use single-quoted names: `'Country'.'Region'`, `'My Subset'`.
10. **Transaction list column key rule** — always use the list column name (`'Orders'.'Customer'`), not just the dimension name.
