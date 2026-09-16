# Logical Functions

---

## IF

`IF(Condition, ValueIfTrue [, ValueIfFalse])`

- Third arg optional; omitting returns BLANK (not FALSE or 0)
- Condition resolving to BLANK takes the false/blank branch
- Result dims = union of all argument dimensions (auto-allocated)
- Both branches always evaluated (not short-circuit)

```pigment
IF('Revenue' > 1000000, "High", "Low")
IF('Score' >= 90, "A", IF('Score' >= 80, "B", "C"))
IF('Actual' > 'Budget', 'Actual' - 'Budget')
IF(Country = SET_Selected_Country, 'Local Rate', 'Default Rate')
```

**Sparsity**: `IF(cond, TRUE, FALSE)` densifies. `IF(cond, TRUE)` is sparse. Prefer `Revenue[FILTER: CurrentValue > 1000]` over `IF(Revenue > 1000, Revenue)` (evaluates once vs twice).

---

## SWITCH

`SWITCH(Expression, Case1, Result1 [, CaseN, ResultN] [, Default])`

- Case values must be **scalar** (no dimensions)
- Returns first matching case
- BLANK/non-matching Expression → Default if provided, else BLANK
- Case values must be dimension item refs or constants, not string literals. Prefer Dimension-typed input metrics over hard-coded items (e.g. `SET_Target_Region` instead of `Region."EU"`)

```pigment
// Numeric cases — text return values (not dimension items; fine)
SWITCH('Score', 90, "A", 80, "B", 70, "C", "F")

// Dimension-item cases — SWITCH requires literal items; for dynamic dispatch use IF with Dimension-typed input metrics
SWITCH('Status', Status."Active", 1, Status."Inactive", 0, Status."Pending", 0.5, 0)
SWITCH('Product'.'Category', Category."Electronics", 'Price' * 1.2, Category."Clothing", 'Price' * 1.1, 'Price')
```

**Hard-coding caveat**: SWITCH cases must be scalar constants, so dimension item literals are unavoidable here. This is one of the rare acceptable uses. For dynamic dispatch (where the set of cases may change), prefer chained `IF` with Dimension-typed input metrics instead.

**Sparsity**: Without Default → sparse (unmatched = BLANK). With Default → potentially densifying.

---

## AND / OR (Infix Operators)

**Pigment uses infix syntax, NOT function calls.**

```pigment
'Revenue' > 1000 AND 'Profit' > 0
'Status' IN (SET_Active_Status, SET_Pending_Status)
```

### Three-valued blank truth tables

| AND | TRUE | FALSE | BLANK |
| --- | --- | --- | --- |
| **TRUE** | TRUE | FALSE | BLANK |
| **FALSE** | FALSE | FALSE | BLANK |
| **BLANK** | BLANK | BLANK | BLANK |

| OR | TRUE | FALSE | BLANK |
| --- | --- | --- | --- |
| **TRUE** | TRUE | TRUE | TRUE |
| **FALSE** | TRUE | FALSE | FALSE |
| **BLANK** | TRUE | FALSE | BLANK |

**Critical**: `BLANK AND TRUE → BLANK` (not TRUE). Pigment uses three-valued logic; blanks propagate.

---

## NOT (Prefix Operator)

`NOT condition`

```pigment
NOT 'Employee'.'IsActive'
NOT Month IN (SET_Excluded_Month_1, SET_Excluded_Month_2)
```

**Critical**: `NOT(BLANK)` does **NOT** evaluate to TRUE. Only `NOT(FALSE)` returns TRUE.

---

## TRUE / FALSE

Bare constants; no parentheses (unlike Excel's `TRUE()` / `FALSE()`).

```pigment
IF(condition, TRUE)       // sparse: BLANK where condition fails
IF(condition, TRUE, FALSE) // dense: stores FALSE everywhere
```

**Sparsity**: Storing FALSE densifies. Use BLANK instead of FALSE when semantically equivalent.

---

## Blank Handling (Critical for Sparsity)

### Preference order: IFDEFINED > IFBLANK > IF(ISBLANK())

### ISDEFINED

`ISDEFINED(Value)` → TRUE where defined, **BLANK** (not FALSE) elsewhere. **Sparse.**

```pigment
ISDEFINED('Price')
```

"isNotDefined" does not exist. Use `IFDEFINED(X, BLANK, TRUE)` instead.

### IFDEFINED

`IFDEFINED(Block, ValueIfDefined [, ValueIfBlank])` → third arg defaults to BLANK. **Sparse.**

```pigment
IFDEFINED('Price', 'Price' * 'Quantity', BLANK)
IFDEFINED('Exchange Rate', 'Amount' * 'Exchange Rate', 'Amount')
IFDEFINED(User, 'Confidential Data', BLANK)
```

**Exception**: Never use `IFDEFINED(actual_metric, actual_value, forecast_formula)` for actual/plan layering. Use `IF(Is_Actual, ...)`.

### IFBLANK

`IFBLANK(MetricToFill, Value)` → returns first arg if defined, second arg otherwise.

```pigment
IFBLANK('Price', 0)
IFBLANK('Discount', 0.1)
IFBLANK(A, IFBLANK(B, C))   // priority-based merge
```

**Gotchas**:

- Evaluated at granular level **before** modifiers: `IFBLANK('Wk sales data', 1)[REMOVE SUM: City, Product]` fills ones before sum
- Constant fill value densifies: `IFBLANK(SOURCE_METRIC, 20)` fills all blanks
- Combine with IF to limit scope: `IF('Active Employee', IFBLANK(Bonus, 10))`

### ISBLANK / ISNOTBLANK — Avoid Unless Necessary

`ISBLANK(Value)` → TRUE if blank, FALSE otherwise. **Always dense.**
`ISNOTBLANK(Value)` → TRUE if defined, FALSE if blank. **Always dense.**

| Instead of | Use | Why |
| --- | --- | --- |
| `ISBLANK(A)` | `ISDEFINED(A)` | Avoids densifying |
| `IF(ISBLANK(A), B, A)` | `IFBLANK(A, B)` | Cleaner, simpler |
| `IF(ISNOTBLANK(A), A*1.1)` | `IFDEFINED(A, A*1.1)` | Sparse |
| `IF(A AND ISBLANK(B), TRUE)` | `IF(A [EXCLUDE: B], TRUE)` | Sparse |

### Blank vs FALSE

| Term | Stored | Notes |
| --- | --- | --- |
| BLANK / not defined | **No** | Sparse; no memory used |
| FALSE | **Yes** | Dense; occupies memory |

### Blank behavior with operators

| Operator group | Operators | Blank + value = |
| --- | --- | --- |
| Additive | `+`, `-`, `OR` | value (blank = identity) |
| Multiplicative | `*`, `/`, `AND` | BLANK (blank propagates) |

Metric + constant: blanks stay blank (`'Revenue' + 100` keeps blank cells blank).

---

## ANYOF / ALLOF

`ANYOF(BooleanBlock)` → TRUE if **any** cell is TRUE. Returns scalar (no dimensions).
`ALLOF(BooleanBlock)` → TRUE if **all** cells are TRUE. Returns scalar (no dimensions).

```pigment
ANYOF('Product'.'Include Category')
ALLOF('IsActiveProduct')
```

Note: ALLOF exists in product but has no KB documentation page.

---

## IN (Infix Operator)

**Items**: `Block IN (Item1, Item2, ..., ItemN)`
**Range** (inclusive both ends): `Block IN (lower : upper)`

```pigment
Country IN (SET_Primary_Country, SET_Secondary_Country)
NOT Month IN (SET_Excluded_Month_1, SET_Excluded_Month_2)
'Switchover Date'[ADD: Year] IN (Year.'Start Date' : Year.'End Date')
```

Prefer Dimension-typed input metrics over `Dimension."Item"` string literals.

---

## Idiomatic Conditional Composition

**Prefer IFBLANK precedence chains** over nested IF for override/merge patterns. When branching is driven by data presence (not value comparisons), chain `IFBLANK(Expr1, IFBLANK(Expr2, Fallback))` rather than `IF(ISBLANK(...), ..., IF(...))`.

**Factor shared modifiers outside IF branches.** When both branches apply the same FILTER/EXCLUDE, move it out and vary only the scalar:

```pigment
// ✅ Factor shared EXCLUDE; IF only on the varying multiplier
'Revenue'[EXCLUDE: Customers.'Exclude from Report']
* IF(IsForecast, 1.10, 1)

// ❌ Repeating the same EXCLUDE in both branches
IF(IsForecast,
  'Revenue'[EXCLUDE: Customers.'Exclude from Report'] * 1.10,
  'Revenue'[EXCLUDE: Customers.'Exclude from Report'])
```

**Lift modifiers when 3+ branches repeat them.** If the same `[FILTER: X][EXCLUDE: Y]` appears on 3+ IF branches, extract modifiers into a shared expression or an intermediary metric. Deeply nested IF with repeated modifiers is a performance anti-pattern.

**IFDEFINED + EXCLUDE on fallback branch.** When the fallback requires scoping (e.g. excluding archived items), wrap in IFDEFINED:

```pigment
IFDEFINED(
  'Context_Specific_Input',
  'Context_Specific_Input',
  'Fallback_Expression'[EXCLUDE: 'Entity'.'Excluded_Flag']
)
```

---

## Division by Zero/BLANK

Pigment handles div/0 natively: returns BLANK. No IF check needed.

```pigment
'Numerator' / 'Denominator'    // safe: BLANK if denominator is 0 or BLANK
```
