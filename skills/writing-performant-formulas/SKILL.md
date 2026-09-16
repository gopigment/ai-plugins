---
name: writing-performant-formulas
description: "Execution skill. Use when reviewing or finalizing Pigment formulas before delivery. Provides the mandatory pre-delivery checklist, sparsity preservation rules, scope-first patterns, and anti-patterns to avoid."
metadata:
  skill_path: /skills/writing-performant-formulas/SKILL.md
  base_directory: /skills/writing-performant-formulas
---

# Writing Performant Pigment Formulas

Apply this skill to every formula before delivery. Pigment is a sparse multidimensional engine: only defined cells are stored. Performance depends on keeping formulas sparse and scoped.

Simple same-dimension arithmetic (e.g. `'A' + 'B'`) needs no special performance wrapping. Always review the checklist; deep rewrite when any item fails. For profiler-based troubleshooting after delivery, see `skill:diagnosing-performance-issues`.

## Pre-Delivery Checklist

Before delivering any formula, verify each item. Rewrite until all pass.

1. **Identifiers**: Correctly quoted — single quotes for names (`'Revenue'`), double quotes for items and text literals.
2. **Dimension alignment**: No unintended `ADD` or dimension mismatch.
3. **Sparsity**: No unnecessary `0`, `FALSE`, or `TRUE` where `BLANK` suffices.
4. **Existence checks**: Prefer `ISDEFINED` / `IFDEFINED` / `IFBLANK` over `ISBLANK` / `ISNOTBLANK` (see `skill:using-formula-functions` for rare valid exceptions).
5. **Scope-first**: `FILTER`, `EXCLUDE`, or `IFDEFINED` appear before calculations, not after.
6. **Aggregations last**: `REMOVE`, `BY`, and other aggregations come after the core calculation.
7. **Prior periods**: Use `SELECT` with offset — not `PREVIOUS` unless true iteration is required.
8. **Allocation**: Use `BY` with a mapping when one exists — not `ADD`.
9. **Date ranges**: Use `PRORATA` — not `IF(Date >= Start AND Date <= End, ...)`.
10. **BY guards**: No `IF` / `ISBLANK` wrappers on `BY` when the source is dimension-typed.
11. **Negation**: Prefer `EXCLUDE: condition` over `FILTER: NOT(condition)`.
12. **Access rights**: Wrap in `IFDEFINED('Users roles', ...)`. Optionally add `IFDEFINED(User, ...)` as an additional performance trim.
13. **Conditional output**: Use `IF(condition, value)` (sparse) — not `IF(condition, value, 0)` or `IF(condition, TRUE, FALSE)`.
14. **Subsetting**: Use `FILTER: CurrentValue` — not `IF(expr, expr, BLANK)`.

## Treat BLANK as Absence, Not Zero or False

| State | Stored? | Meaning |
| --- | --- | --- |
| `BLANK` / undefined | No | Cell does not exist |
| `0` | Yes | Explicit numeric zero |
| `FALSE` / `TRUE` | Yes | Explicit boolean |

Example: 1000 Products x 12 Months = 12,000 possible cells. If only 500 have values, sparse metric stores 500 cells (~4%). Densifying to 12,000 multiplies storage and computation.

**Rule**: Represent "no value" with `BLANK`. Never substitute `0` for empty numbers or `FALSE` for empty booleans unless downstream logic explicitly requires stored values.

**Meaningful zero exception**: Explicit `0` IS correct when zero is a business value (zero variance, zero balance, zero growth rate, inactive line item contributing `0` to a total). In these cases `BLANK` would incorrectly omit the line from aggregations.

## Prefer ISDEFINED Over ISBLANK

`ISBLANK(A)` returns `TRUE` or `FALSE` for **every** cell (dense). `ISDEFINED(A)` returns `TRUE` only where a value exists, `BLANK` elsewhere (sparse). Reserve `ISBLANK` / `ISNOTBLANK` only when every cell must hold an explicit boolean — see `skill:using-formula-functions` for the rare-valid-exception allow-list.

```pigment
// WRONG — densifies
IF(ISBLANK('Revenue'), 'Default', 'Revenue')

// CORRECT — sparse
IFBLANK('Revenue', 'Default')
IFDEFINED('Revenue', 'Revenue')
```

## Structure Formulas Scope-First, Aggregations Last

Put narrowing modifiers and guards **before** calculations; aggregations **after**.

```pigment
// WRONG — computes on all cells, then filters
('Revenue' * 'Rate')[FILTER: 'Region' = SET_Selected_Region]

// CORRECT — scope first, then calculate, then aggregate
'Revenue'
  [FILTER: 'Region' = SET_Selected_Region]
  * 'Rate'
  [REMOVE: Product]
```

Pattern: `Source [scope modifiers] → calculation → [REMOVE / BY aggregation]`.

## Model Date Ranges with PRORATA

`PRORATA(TimeDimension, StartDate, EndDate)` returns the fraction of each period within `[StartDate, EndDate)`. Start date is **included**; end date is **excluded**; add 1 for inclusive end.

```pigment
// WRONG — verbose, error-prone, poor sparsity
IF('Date' >= 'Start Date' AND 'Date' <= 'End Date', 1, BLANK)

// CORRECT — sparse presence factor
PRORATA(Month, 'Start Date', 'End Date' + 1)
```

Derive presence booleans with `ISDEFINED(PRORATA(...))`, not `ISBLANK(PRORATA(...))`.

## Prior Period Lookups

Use `SELECT` with offset for simple lags — not `PREVIOUS` (iterative, expensive).

`PREVIOUS(Month)` is correct only for true iterative calculations within a single metric (e.g. cumulative balance, cash roll-forward, inventory carry-forward). For multi-metric iteration (opening/closing inventory across metrics), use `PREVIOUSOF(...)` with a cycle.

## Allocation: BY Over ADD

`BY` follows a mapping (sparse); `ADD` creates all combinations (dense).

- Use `BY CONSTANT` with a mapping for value replication instead of `ADD CONSTANT` (dense).
- Do not wrap `BY` with `IF(ISBLANK(...))` guards on dimension-typed metrics.
- See anti-pattern table for `ADD+FILTER` and round-trip patterns.

## Access Rights

`IFDEFINED('Users roles', ...)` wrapping is **mandatory** for access rights formulas. `IFDEFINED(User, ...)` is an optional additional trim — **not** a substitute for `IFDEFINED('Users roles', ...)`.

```pigment
// Mandatory wrapping
IFDEFINED('Users roles', 'Can Edit'[BY: User])

// Optional additional trim layered on top
IFDEFINED(User, IFDEFINED('Users roles', 'Can Edit'[BY: User]))
```

Use `BLANK` (not `FALSE`) for denied access to preserve sparsity.

## Conditional Creation and Subsetting

Use `IF(condition, value)` to create values sparsely — `[ADD: X][FILTER: condition]` densifies first, then subsets.

- Use `FILTER` only on already-computed values; prefer `[FILTER: CurrentValue]` over `IF(expr, expr, BLANK)`.
- Use `EXCLUDE` instead of `FILTER: NOT(...)`.
- When the goal is to pick one dimension item and drop the dimension, use `SELECT` (aggregates it away) rather than `FILTER` (keeps dimension in output).
- **Division by zero**: Pigment handles it natively (returns `BLANK`); do not guard with `IF(x <> 0, a / x)`.

## Fix These Anti-Patterns Before Delivery

Scan every formula for these patterns. Each row is a rewrite trigger.

| Anti-Pattern | Why Bad | Fix |
| --- | --- | --- |
| `ISBLANK`/`ISNOTBLANK` for existence checks | Densifies (TRUE/FALSE stored everywhere) | `ISDEFINED`/`IFDEFINED`/`IFBLANK`; e.g. `IFBLANK(A, B)` not `IF(ISBLANK(A), B, A)` |
| `ISBLANK`/`ISNOTBLANK` in `AND`/`OR` chains | Densifies entire expression via blank-presence | `EXCLUDE` to remove blank rows, or nested `IFDEFINED` guards |
| `0`/`FALSE` for empty numeric/boolean | Stores explicit value, destroys sparsity | `BLANK` (absence = not stored) |
| Calculations before scoping | Computes irrelevant cells | `FILTER` / `EXCLUDE` / `IFDEFINED` first |
| `PREVIOUS` for simple lag; `SELECT` for iterative calc | Lag: iterative overhead; Iterative: circular ref | Lag: `[SELECT: Month - 1]`; same-metric: `PREVIOUS(Month)`; multi-metric: `PREVIOUSOF(...)` + cycle |
| `ADD`/`ADD CONSTANT`/`ADD+FILTER` when mapping or `IF` fits | Dense cross-product or dense replication | `BY` with mapping; `BY CONSTANT` for replication; `IF(condition, value)` for conditional rows |
| `IF`/`ISBLANK` guard on `BY` with dim-typed metric | Redundant, densifies | Remove guard |
| `IF(Date >= Start AND Date <= End, 1, BLANK)` | Verbose, error-prone | `PRORATA(TimeDim, Start, End + 1)` |
| `ISBLANK(PRORATA(...))` for presence | Densifies | `ISDEFINED(PRORATA(...))` |
| `FILTER: NOT(condition)` | Less sparse-friendly | `EXCLUDE: condition` |
| `IF(cond, val, 0)` or `IF(cond, TRUE, FALSE)` | Densifies | `IF(cond, val)` or `IFBLANK` |
| `IF(expr, expr, BLANK)` for subsetting | Redundant branching | `[FILTER: CurrentValue]` |
| `FILTER` to aggregate away a dimension | Keeps dimension in output | `SELECT` to aggregate; `FILTER` to keep dimension |
| `IF(x<>0, a/x)` for safe division | Unnecessary guard | Just divide — Pigment returns `BLANK` for division by zero |
| Access rights without `IFDEFINED('Users roles', ...)` | Densifies across users | `IFDEFINED('Users roles', ...)` mandatory; `IFDEFINED(User, ...)` is a trim, not a substitute |
| `[ADD: X][REMOVE: X]` or `[REMOVE: X][ADD: Y]` round-trips | Unnecessary expand/collapse | `BY` with a mapping |
| Chained `[BY:][BY:]` on Transaction Lists | Silently drops list properties between steps | Single `BY` with comma-separated mappings |
| `[REMOVE: Version]` without verification | Collapses scenario meaning | Verify collapse is intentional |
| `CUMULATE`/`MOVINGSUM`/`FIND`/`SUBSTITUTE` on large lists | Expensive per-cell | Subset with `FILTER` first |
| Multiple `PREVIOUS(...)` in one formula | Multiplies iterative passes | Single `PREVIOUS` call |
| `IFBLANK(X, PREVIOUS(Dim))` or `IFBLANK(X, PREVIOUSOF(...))` | Verbose, triggers iteration | `FILLFORWARD` |
| `ACCESSRIGHTS(x, FALSE)` for denied access | Densifies with explicit FALSE | `ACCESSRIGHTS(x, BLANK)` |
| 2-arg `IF(condition, expr)` on 6+ dim metric | Expands scope across all intersections | `expr[FILTER: condition]` |
| Hard-coded year literals or `DATE(YYYY, ...)` | Breaks across fiscal years | Date-typed or Dimension-typed input metrics |

**Remember**: `BLANK` = undefined = not stored. `FALSE` ≠ `BLANK` (stored, densifies). `0` ≠ `BLANK` (stored). Always prefer absence over explicit empty values unless business logic demands a stored zero or boolean.

## Related Skills

- `skill:diagnosing-performance-issues` — profiler-based troubleshooting of delivered formulas
- `skill:using-formula-functions` — `ISBLANK` rare-valid-exception allow-list
