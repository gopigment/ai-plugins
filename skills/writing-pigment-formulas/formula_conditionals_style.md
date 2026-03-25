# Formula Conditionals Style (Sparsity-First)

**When to use this doc**: You are writing conditional logic, inclusion/exclusion rules, or filtering data — either with IF/SWITCH or with modifiers (FILTER, EXCLUDE). This document defines the **canonical Pigment-native style** for conditionals so formulas stay sparse, readable, and consistent.

**MUST read this doc when** the formula involves any of: nested IF chains, IFBLANK/ISBLANK patterns, FILTER vs IF tradeoffs, EXCLUDE vs NOT, boolean logic with many AND/OR branches, repeated condition blocks that can be normalized, or optimization of conditional logic for performance/sparsity.

**Goal**: Write Pigment-native formulas, not Excel-style formulas in Pigment syntax. Prefer sparsity and modifiers (FILTER, EXCLUDE, ADD) as first-class tools; use IFBLANK / IFDEFINED / ISDEFINED as the primary way to branch on data presence; avoid deeply nested IF/AND/OR and unnecessary densification.

**Related**: Syntax and function reference → [functions_logical.md](./functions_logical.md), [formula_modifiers.md](./formula_modifiers.md). Performance (scoping, densification) → [formula_performance_patterns.md](./formula_performance_patterns.md).

---

## Quick Reference: When to Use What

| Situation | Prefer | Avoid |
| --------- | ------ | ----- |
| "First non-blank wins" (override hierarchy) | `IFBLANK(Expr1, IFBLANK(Expr2, ...))` | Nested `IF(cond1, Expr1, IF(cond2, Expr2, ...))` |
| Override when present, else base | `IFBLANK('Override', 'Base')` | `IF(ISBLANK('Override'), 'Base', 'Override')` |
| Branch on "is this input defined?" | `IFBLANK(ISDEFINED('Input'), 'Fallback'[EXCLUDE: ...])` | Nested IF with AND/OR on flags |
| Mutually exclusive cases (e.g. Case A vs Case B) | `IFBLANK(Expr_A[FILTER: condA], Expr_B[EXCLUDE: condA])` | `IF(condA, Expr_A, Expr_B)` with dense condition |
| "Which rows should this apply to?" | `'Metric'[FILTER: ...]` or `[EXCLUDE: ...]` | `IF(condition, 'Metric', BLANK)` when ELSE is BLANK |
| Exclude a flagged subset | `'Metric'[EXCLUDE: 'Entity'.'Is_Archived']` | `'Metric'[FILTER: NOT 'Entity'.'Is_Archived']` |
| Sparse boolean "tag where condition holds" | `IF(condition, TRUE)` or `IF(condition, TRUE, BLANK)` | `condition` alone (densifies to TRUE/FALSE) |
| Both IF branches share same expression (e.g. same metric + modifiers) | Factor out: `Expr * IF(cond, a, b)` or `Expr + IF(cond, x, y)` | Repeating `Expr` in both branches |
| Multiple branches repeat same FILTER/EXCLUDE conditions (4+ similar blocks) | Nested IF that factors common expression; vary only the differing part | IFBLANK with repeated FILTER/EXCLUDE on each branch (can degrade performance) |

---

## 1. IFBLANK as the Primary Tool for Conditionals

IFBLANK is the preferred building block for:

- Precedence / case chains ("first non-blank wins")
- Override vs default logic
- Sparsity-driven branching based on data presence
- Case-style branching where each case is scoped with FILTER / EXCLUDE

Prefer it over deeply nested IF + AND/OR trees.

### 1.1 Precedence chains ("first non-blank wins")

**Goal**: Choose the first expression that returns a value; if blank, try the next.

**Canonical pattern**:

```pigment
// First non-blank among Expr1, Expr2, Expr3, Expr4
IFBLANK(
  Expr1,
  IFBLANK(
    Expr2,
    IFBLANK(
      Expr3,
      Expr4
    )
  )
)
```

- If Expr1 has a value → use it; else Expr2; else Expr3; else Expr4.
- Use for: override hierarchy, multi-source chaining (e.g. plan → actual → baseline → constant).
- If any expression is complex, move it into a helper metric and reference it in the chain.

### 1.2 Override vs base (no explicit ISDEFINED)

**Goal**: Use an override when present; otherwise use a base value.

**Canonical pattern**:

```pigment
IFBLANK('Override_Metric', 'Base_Metric')
```

**Avoid**:

```pigment
IF(ISBLANK('Override_Metric'), 'Base_Metric', 'Override_Metric')
```

### 1.3 Sparsity-driven branching on definedness

**Goal**: When the driver for branching is "does this input exist?", not its value.

**Canonical pattern**:

```pigment
IFBLANK(
  ISDEFINED('Context_Specific_Input'),
  'Fallback_Expression'[EXCLUDE: 'Entity'.'Excluded_Flag'] < 'Context'.'Cutoff'
)
```

- First argument: test signal (TRUE where context input exists, BLANK where it doesn’t).
- Second argument: fallback expression when there is no context-specific input.
- Use EXCLUDE (or FILTER) to scope the fallback by row, not AND/OR inside the expression.

### 1.4 Case-style branching with FILTER/EXCLUDE-scoped expressions

**Goal**: Mutually exclusive cases (e.g. TBH vs in-seat) without one giant IF with AND/OR.

**Canonical pattern**:

```pigment
IFBLANK(
  'TBH Comp Calc'[FILTER: 'Roster'.'Is_TBH'],
  'Inseat Comp Calc'[EXCLUDE: 'Roster'.'Is_TBH']
)
```

- Each branch is scoped by FILTER or EXCLUDE; IFBLANK just picks "first non-blank".
- For 3+ cases, nest IFBLANK and scope the default with EXCLUDE for the union of case conditions.

**Avoid**: One IF with a dense boolean condition and two scalar expressions.

### 1.5 Replacing deep IF trees with IFBLANK

**Avoid**:

```pigment
IF(cond1, Expr1, IF(cond2, Expr2, IF(cond3, Expr3, Expr4)))
```

**Prefer** one of:

- Precedence chain: `IFBLANK(Expr1, IFBLANK(Expr2, IFBLANK(Expr3, Expr4)))` (when "first defined wins").
- Override vs base: `IFBLANK('Override', 'Base')`.
- Signal-based: `IFBLANK(ISDEFINED('Input'), 'Fallback'[EXCLUDE: ...])`.
- Case-style: `IFBLANK(Expr_A[FILTER: condA], Expr_B[EXCLUDE: condA])`.

---

## 2. FILTER and EXCLUDE: When and How

FILTER and EXCLUDE control **which dimensional combinations exist** in an expression. Use them when the question is "which rows/coordinates should this apply to?", not "which of two scalar expressions do I pick?".

### 2.1 Use separate FILTERs (and EXCLUDEs)

**Canonical**:

```pigment
'Metric'
[FILTER: 'Entity'.'Status' = "Active"]
[FILTER: 'Entity'.'Region' = "EMEA"]
```

**Less preferred** (one big AND):

```pigment
'Metric'[FILTER: 'Entity'.'Status' = "Active" AND 'Entity'.'Region' = "EMEA"]
```

Separate modifiers are easier to read, debug, and mix with EXCLUDE.

### 2.2 Filter by expression value: use CURRENTVALUE

**Canonical**:

```pigment
'Revenue'[FILTER: CURRENTVALUE > 0]
'Margin %'[FILTER: CURRENTVALUE >= 0.20][FILTER: 'Entity'.'Status' = "Live"]
```

This avoids repeating the expression and keeps value rules and property rules separate.

### 2.3 Exclude a flagged subset: use EXCLUDE, not FILTER NOT

**Canonical**:

```pigment
'Metric'[EXCLUDE: 'Entity'.'Is_Discontinued']
'Metric'[EXCLUDE: 'Entity'.'Is_Archived'][EXCLUDE: 'Entity'.'Is_Test_Data']
```

**Avoid**:

```pigment
'Metric'[FILTER: NOT 'Entity'.'Is_Archived']
```

EXCLUDE keeps semantics positive ("drop these rows") and preserves sparsity; NOT over a boolean that can be BLANK densifies. See section 3.

### 2.4 Replace IF(condition, expr, BLANK) with FILTER when ELSE is BLANK

When the else branch is BLANK and the condition is "which rows should be present", use FILTER (or EXCLUDE) instead of IF.

**Anti-pattern**:

```pigment
IF('Entity'.'Status' = "Active", 'Metric', BLANK)
```

**Canonical**:

```pigment
'Metric'[FILTER: 'Entity'.'Status' = "Active"]
```

Same for multiple conditions: prefer stacked FILTER/EXCLUDE over IF with AND.

**Anti-pattern**:

```pigment
IF('Entity'.'Status' = "Active" AND NOT 'Entity'.'Is_Archived', 'Metric', BLANK)
```

**Canonical**:

```pigment
'Metric'[EXCLUDE: 'Entity'.'Is_Archived'][FILTER: 'Entity'.'Status' = "Active"]
```

### 2.5 Heuristic: when to choose FILTER/EXCLUDE vs IF

- **Use FILTER/EXCLUDE** when:
  - The else branch is BLANK (or empty).
  - The condition is about which rows exist, not which of two scalar values to choose.
  - You have mutually exclusive cases that can be written as expressions scoped with FILTER/EXCLUDE.
- **Use IF** when:
  - You are choosing between two scalar expressions with one or two simple conditions.
  - You are doing conditional creation (no existing expression to subset) — see [formula_performance_patterns.md](./formula_performance_patterns.md) Pattern 4.

### 2.6 Factorize common subexpressions in IF branches

When **both branches** of an IF use the **same expression** (e.g. the same metric with the same FILTER/EXCLUDE), prefer **factoring it out** so the IF only chooses the differing part (multiplier, constant, etc.). This keeps expressions small and composable (DRY).

**Less idiomatic** (repeated EXCLUDE and base expression in both branches):

```pigment
IF(
  Month.'Period type' = 'Period type'.Name."Forecast"
    AND ( ... segment test ... ),
  'Revenue'[EXCLUDE: Customers.'ARR Segment' = 'ARR Segment'.Name."Test"
    OR Customers.'ARR Segment' = 'ARR Segment'.Name."Internal"],
  'Revenue'[EXCLUDE: Customers.'ARR Segment' = 'ARR Segment'.Name."Test"
    OR Customers.'ARR Segment' = 'ARR Segment'.Name."Internal"]
)
```

**More idiomatic** (one EXCLUDE, IF only on the multiplier):

```pigment
'Revenue'[EXCLUDE:
  Customers.'ARR Segment' = 'ARR Segment'.Name."Test"
    OR Customers.'ARR Segment' = 'ARR Segment'.Name."Internal"
]
* IF(
    Month.'Period type' = 'Period type'.Name."Forecast"
      AND ( ... segment test ... ),
    1.10,
    1
  )
```

Apply the same idea when the only difference between branches is an additive constant, a divisor, or another scalar: factor the common expression once and use IF for the varying part.

### 2.7 When nested IF is acceptable (repeated FILTER/EXCLUDE)

When **multiple branches** (e.g. 4+) use the **same FILTER/EXCLUDE conditions** with only a varying expression or multiplier, the IFBLANK/FILTER pattern can degrade performance. Each branch evaluates its own scoped expression; repeated modifiers across many branches add overhead.

**Prefer** a nested IF that factors out the common logic and uses IF only for the varying part:

```pigment
// Multiple branches with same Approved?, Merit Effective, Backfill conditions
// Prefer: factor common conditions, vary only the salary expression
IF(
  'Headcount ID'.Approved? AND (
    'WFP_Elect to Backfill?' = FALSE
    OR ISBLANK('WFP_Elect to Backfill?')
    OR ('WFP_Elect to Backfill?' AND Month < TIMEDIM('WFP_Backfill Date', Month))
  ),
  'WFP_Salary'
  * IF(Month >= 'WFP_Merit Effective', 1, 1 - 'Global Merit Adjustment'[BY: 'Headcount ID'.Geo])
  * 1/12 * 'FTE in Dept Calc',
  BLANK
)
```

**Avoid** when the same FILTER/EXCLUDE stack repeats on many IFBLANK branches — benchmarks show nested IF can be ~40% faster in such cases.

**Rule of thumb**: If you would write the same `[FILTER: X][FILTER: Y][EXCLUDE: Z]` on 4+ branches, consider a nested IF that factors the common conditions instead.

---

## 3. Logical Operators and 3-State Booleans

### 3.1 Pigment booleans are 3-state

- **TRUE** → dense
- **FALSE** → dense
- **BLANK** → sparse (no value at that coordinate)

Prefer patterns that preserve BLANK as a distinct state (IFBLANK, ISDEFINED, FILTER, EXCLUDE). Avoid turning BLANK into TRUE or FALSE unless necessary.

### 3.2 Avoid NOT in favor of EXCLUDE / positive conditions

**NOT** flips TRUE↔FALSE. For BLANK, NOT has no meaningful opposite; applying NOT to a boolean that can be BLANK densifies (BLANKs become TRUE or FALSE). Prefer EXCLUDE or a positive condition.

**Rule**: Do not write `[FILTER: NOT 'Flag']` or `[FILTER: NOT 'Entity'.'Is_Archived']`. Use:

```pigment
[EXCLUDE: 'Flag']
[EXCLUDE: 'Entity'.'Is_Archived']
```

Same for IF: avoid `IF(NOT 'Entity'.'Flag', 'Metric', BLANK)`. Use `'Metric'[EXCLUDE: 'Entity'.'Flag']`.

### 3.3 Sparse boolean masks: IF(condition, TRUE, BLANK)

A bare comparison like `'Metric' = 'Other'` yields a **dense** boolean (TRUE/FALSE everywhere). To keep a **sparse** boolean (TRUE only where the condition holds, BLANK elsewhere):

**Canonical**:

```pigment
IF('Metric' = 'Something Else', TRUE)
// or explicitly
IF('Metric' = 'Something Else', TRUE, BLANK)
```

Use this when you need a sparse "tag" of cells that meet a condition, without densifying the rest to FALSE.

---

## 4. Densification: When ADD + FILTER Is Acceptable

- **Small dimensional spaces**: `TRUE[ADD: Entity, Version][FILTER: condition]` can be acceptable and readable.
- **Large dimensional spaces**: Densifying a scalar with ADD then FILTER is a performance anti-pattern; prefer IF(condition, expr, BLANK) or FILTER on an expression that already has the right dimensions.

See [formula_performance_patterns.md](./formula_performance_patterns.md) Pattern 4 (conditional creation vs subsetting) and Pattern 6 (BY over ADD). When in doubt, prefer IF or FILTER on an existing expression over ADD + FILTER on a scalar.

---

## Summary Checklist for Conditionals

- [ ] Prefer IFBLANK for override/default, precedence chains, and case-style logic with FILTER/EXCLUDE.
- [ ] Use FILTER/EXCLUDE when the else branch is BLANK and the condition is "which rows apply".
- [ ] Use separate FILTER (and EXCLUDE) modifiers instead of one big AND.
- [ ] Use EXCLUDE for exclusions; do not use FILTER: NOT.
- [ ] Use IF(condition, TRUE, BLANK) when you need a sparse boolean mask.
- [ ] Avoid NOT on booleans that can be BLANK; use EXCLUDE or positive conditions instead.
- [ ] When both IF branches share the same expression (e.g. same metric with same modifiers), factor it out so the IF only chooses the differing part (multiplier, constant).
