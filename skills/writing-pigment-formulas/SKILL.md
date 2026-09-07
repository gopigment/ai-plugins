---
name: writing-pigment-formulas
description: Always use this skill when writing, editing, or debugging Pigment formulas — including conditional logic, blank handling, date-range logic, aggregation, prior-period lookups, and dimensional transformations. Pigment uses a proprietary formula language — NEVER assume you know the syntax, and ALWAYS read the documentation before writing any formula. Covers data types, modifiers, functions, calculation patterns, and performance trade-offs (calibrated by formula complexity). This skill includes supporting files in this directory; explore as needed.
metadata:
  skill_path: /skills/writing-pigment-formulas/SKILL.md
  base_directory: /skills/writing-pigment-formulas
  includes:
    - "*.md"
---

# Writing Pigment Formulas

This skill provides comprehensive guidance for writing formulas in Pigment's multidimensional formula language, including formula builder tools for validation and generation.
Pigment uses proprietary formula language which should never be confused with other language.
Never mix Pigment formula language with other language, and never assume you know the language before reading this documentation.

**CRITICAL - ABSOLUTE PROHIBITION**: Pigment has its own unique formula language.
You MUST NEVER write code or functions using another language being Excel, SQL, Python, JavaScript, MDX, DAX, or ANY other programming or query language.
ONLY Pigment syntax exists when writing formulas.

## When to Use This Skill

- **Write formulas** - Creating calculations for metrics and list properties
- **Aggregate data** - Rolling up from transaction lists or detailed dimensions
- **Perform time-series calculations** - YTD, rolling averages, sequential logic
- **Use functions** - CUMULATE, SHIFT, ITEM, MATCH, TIMEDIM, etc.
- **Use modifiers** - BY, ADD, REMOVE, SELECT, FILTER, EXCLUDE, TOPARENTLIST, TOSUBSET
- **Debug syntax** - Troubleshooting formula errors
- **Test and validate formulas** - Verifying formula correctness and expected behavior
- **Transform dimensions** - Changing dimensional structure of calculations
- **Allocate data** - Distributing values across dimensions
- **Match and lookup** - Finding data across dimensions

---

## Syntax Fundamental

**Quoting Rules - MUST FOLLOW:**

| Element         | Syntax                        | Example                                           |
| --------------- | ----------------------------- | ------------------------------------------------- |
| Metric names    | Single quotes                 | `'Revenue'`, `'Total Sales'`                      |
| Dimension names | Single quotes                 | `'Product'`, `'Country'`                          |
| Property access | Dot notation with quotes, chainable | `'Product'.'Category'`, `City.Country.Currency` |
| Dimension items | Double quotes after dimension — **MP02:** literal only in `VAR_` default | `Month."Jan 25"` only when setting a `VAR_` metric default |
| String values   | Double quotes                 | `"Active"`, `"Completed"`                         |

**Cross-app references:** 
- A block from another application can only be referenced if it has been shared through a Library and that Library is activated in the current application. 
- The syntax is `'APPLICATION_NAME'::'BLOCK_NAME'`. 
- If the block name is unique across all activated libraries the application prefix may be omitted, but always use the full form for clarity.

**No hard-coding (MP02 — hard constraint):** See [modeling_principles §4](../modeling-pigment-applications/modeling_principles.md). Before member-specific or time-bounded formulas, read [formula_writing_workflow.md](./formula_writing_workflow.md) Step 2 and [formula_modifiers.md](./formula_modifiers.md) (FILTER, SELECT, BY CONSTANT).

**Common Mistakes:**

- ❌ `Revenue` → ✅ `'Revenue'` (missing quotes)
- ❌ `Product.Category` → ✅ `'Product'.'Category'` (missing quotes)
- ❌ `Month.'Jan 25'` → ✅ `Month."Jan 25"` (items use double quotes; in formulas use a `VAR_` metric per MP02)
- ❌ `Country.Name = "France"` → ✅ `Country = VAR_Selected_Country` (same hard-coding as `Country."France"`)

---

## Performance Patterns

**Apply this checklist proportionally to formula complexity.** Simple arithmetic between existing same-dimensioned metrics (e.g. `'A' + 'B'`, `'A' * 'B'`, `'A' / 'B'`) needs no performance wrapping — deliver as-is. Use the checklist as a review gate for formulas that introduce conditionals, dimensional changes, date-range logic, or that target large/sparse metrics.

Read [formula_performance_patterns.md](./formula_performance_patterns.md) and verify:

**Always check (universal):**

- [ ] Identifiers are correctly quoted (single quotes for names, double quotes for items)
- [ ] Dimensions are aligned — no unintended ADD or dimension mismatch
- [ ] Scoping clauses appear FIRST (FILTER, EXCLUDE, IFDEFINED)
- [ ] Aggregations (REMOVE, BY) appear AFTER calculations

**Check when conditionals are present:**

- [ ] Using IFDEFINED instead of IF(ISBLANK()) for existence checks
- [ ] Using IFBLANK instead of IF(ISBLANK(...), default, ...) for defaults
- [ ] Conditional creation: use IF (not ADD + FILTER); subsetting a computed expression: use FILTER: CurrentValue (not IF(expr, expr, BLANK))

**Check when date ranges are defined by Start/End:**

- [ ] Avoid multi-conditional IFs (`Date >= Start AND Date < End`) when PRORATA semantics apply
- [ ] Prefer `PRORATA()` to express "active within a date range" and derive booleans or numeric flags from `PRORATA()` using ISDEFINED/IFDEFINED

**Check when prior period lookups are needed:**

- [ ] Using SELECT for prior period lookups (NOT PREVIOUS)

**Check when dimensional changes or mappings are involved:**

- [ ] Using BY instead of ADD where mapping exists
- [ ] If you BY on a dimension-typed metric, do not add IF/ISBLANK guards; BY respects that metric's sparsity

**Check when the metric is large/sparse or involves access rights:**

- [ ] Avoid ISBLANK/ISNOTBLANK on large sparse metrics — use ISDEFINED/IFDEFINED
- [ ] Use BLANK instead of 0 for empty values (see exception below for meaningful zeros)
- [ ] Use BLANK instead of FALSE for boolean flags (FALSE is stored, BLANK is not)
- [ ] Access rights wrapped in IFDEFINED(User, ...)
- [ ] **MP02:** No `Dimension."Item"` in formulas; no `DATE(...)` for planning bounds; relative metric names only — see [formula_writing_workflow.md](./formula_writing_workflow.md) Step 6 checklist

For the full date-range presence pattern (PRORATA worked examples, ISDEFINED/IFDEFINED derivation, when simple IF is acceptable), see **Pattern 11** in [formula_performance_patterns.md](./formula_performance_patterns.md).

---

## Formula Writing Process

**Key phases**: Understand Context → Search Documentation → Design → Build → Optimize → Validate → Deliver

**Follow the complete 8-step workflow**: [./formula_writing_workflow.md](./formula_writing_workflow.md)

- **Critical**: Always search documentation first before writing
- **Governance check (MP02 — required):** [modeling_principles §4](../modeling-pigment-applications/modeling_principles.md); Version patterns: `skill:planning-cycles-pigment-applications`.
- **Validation & Delivery**: Use Formula Builder Tools to validate and deliver formulas

---

## Formula Validation and Building Tools

**Important**: These tools are for **validation and implementation** when working with real formulas.

### Quick Validation

- `tool:validate_formula` - Validate formula syntax WITHOUT applying it to any block
  - Use for: Checking syntax before passing a formula to `tool:create_list_property` or `tool:update_list_property`
  - Use for: Ensuring formula syntax is correct before including in user messages
  - Use for: Re-checking a downstream metric's formula against its own dimensions after a structural
    dimension change elsewhere in the formula chain (pass `target`)
  - Input: `formula` (the Pigment formula text), `target` (the block the formula is for — see the tool's
    own description for when to pass it)
  - Returns: Validation result with error highlighting and hints if invalid
  - **Limitations**:
    - Do NOT use with formulas containing `Previous` or `PreviousOf` functions

**Recommended Workflow**:

1. **Draft formula** - Write your formula based on requirements
2. **Validate** - Use `tool:validate_formula` to check syntax
3. **Fix errors** - Iterate until formula is valid
4. **Apply** - Use `tool:create_metric`/`tool:update_metric` or `tool:create_list_property`/`tool:update_list_property`

**How to apply**: After validation, use:

- Metrics: `tool:create_metric` (with `formula` field) or `tool:update_metric` (with `formula` field), to set correct default formatting on metrics see `skill:formatting-and-highlighting`
- List properties: `tool:create_list_property` or `tool:update_list_property` with the formula

### Structural Dimension Changes: Check Downstream Formulas

Adding or removing a structural dimension on a metric does not automatically propagate to metrics that
reference it. If a downstream metric's formula doesn't already carry the new dimension, the compiler will
silently broadcast (or collapse) values to align with the target instead of failing — this can produce
wrong numbers with no visible error.

**Trigger — do not skip this**: `tool:update_metric` (when changing `dimension_ids`/`target_type`) and
`tool:update_metric` (with `formula`) both return a `hints` list in their response. If it contains an "automatic
formula dimensions adjustment" hint, stop before declaring the change done and work through the steps below.
This can fire on the metric you just changed (its own formula no longer matches its new dimensions) even
before you look at any downstream metric.

Before declaring a structural dimension change done:

1. Identify metrics whose formulas reference the changed metric: use `tool:get_data_dependency_tree`
   with direction `Usages` on the changed metric to find its downstream formula consumers.
2. For each one, call `tool:validate_formula` with that metric's `formula` and `target.metric_id` set to it.
3. If the "automatic formula dimensions adjustment" hint appears, the formula's dimensions don't match the
   metric's own dimensions — don't accept the implicit broadcast/collapse as correct. Confirm with the user
   whether the dimension should genuinely propagate to that metric (and how — the change may need to be
   threaded further upstream instead, e.g. onto the intermediary metric that actually carries the source
   data), or if the mismatch is intentional.

---

## Prerequisites

This skill focuses on formula **implementation**. Before writing formulas, understand foundational concepts from the **modeling-pigment-applications** skill:

- Core platform knowledge (multidimensional engine, dimensions vs properties, sparsity principles)
- Pigment Modeling Best Practices standards (sparsity preservation, dimension alignment, formatting)
- Dimensional design concepts (source-to-target relationships, transformation cases)
- Modifier concepts

### Type Considerations

Formulas produce results that must match the target metric or property type:

- **Number**: Arithmetic operations, aggregations, most calculations
- **Date**: Date functions (DATE, DATEVALUE, EDATE), TIMEDIM conversions
- **Text**: String operations, concatenation, TEXT() conversion
- **Dimension**: ITEM, MATCH lookups returning dimension references
- **Boolean**: Logical operations (AND, OR, comparisons)

**Type conversions**: Use TEXT() to convert to text, VALUE() to convert to number, TIMEDIM() to convert dates to calendar dimensions. See [functions_text.md](./functions_text.md) and [functions_lookup.md](./functions_lookup.md).

**Reference**: For detailed type selection guidance, see modeling-pigment-applications skill.

---

## Quick Reference

| Topic                             | File                                                                 |
| --------------------------------- | -------------------------------------------------------------------- |
| Formula Writing Process           | [formula_writing_workflow.md](./formula_writing_workflow.md)         |
| **Conditionals style (IFBLANK, FILTER/EXCLUDE vs IF)** | [formula_conditionals_style.md](./formula_conditionals_style.md) |
| Modifiers (BY, ADD, FILTER, TOPARENTLIST, TOSUBSET, etc.) | [formula_modifiers.md](./formula_modifiers.md)                       |
| BY with mapping metrics (->)      | [formula_by_mapping_arrow.md](./formula_by_mapping_arrow.md)         |
| Lookup Functions                  | [functions_lookup.md](./functions_lookup.md)                         |
| Numeric Functions                 | [functions_numeric.md](./functions_numeric.md)                       |
| Time and Date Functions           | [functions_time_and_date.md](./functions_time_and_date.md)           |
| Iterative Calculation (PREVIOUS & PREVIOUSOF) | [functions_iterative_calculation.md](./functions_iterative_calculation.md) |
| Logical Functions                 | [functions_logical.md](./functions_logical.md)                       |
| Text Functions                    | [functions_text.md](./functions_text.md)                             |
| Performance Patterns              | [formula_performance_patterns.md](./formula_performance_patterns.md) |

---

## Function Reference

### Most Common Functions & Modifiers

- **BY** → [./formula_modifiers.md](./formula_modifiers.md) - Aggregate or allocate; **BY with mapping metrics (`->`)** → [./formula_by_mapping_arrow.md](./formula_by_mapping_arrow.md)
- **TOPARENTLIST** → [./formula_modifiers.md](./formula_modifiers.md#toparentlist-and-tosubset-list-subsets) — subset dimension → parent (1:1 remap; parent items outside the subset are blank)
- **TOSUBSET** → [./formula_modifiers.md](./formula_modifiers.md#toparentlist-and-tosubset-list-subsets) — parent dimension → subset (1:1 remap; parent rows outside the subset are dropped)
- **CUMULATE** → [./functions_numeric.md](./functions_numeric.md) - Running totals (use instead of PREVIOUSOF + value)
- **FILTER** → [./formula_modifiers.md](./formula_modifiers.md) - Include data by condition
- **EXCLUDE** → [./formula_modifiers.md](./formula_modifiers.md) - Remove data by condition
- **FILLFORWARD** → [./functions_time_and_date.md](./functions_time_and_date.md) - Fill blanks (use instead of IFBLANK + PREVIOUS)
- **IF** → [./functions_logical.md](./functions_logical.md) - Conditional logic
- **IFDEFINED** → [./functions_logical.md](./functions_logical.md) - Sparsity-preserving conditionals
- **ITEM** → [./functions_lookup.md](./functions_lookup.md) - Lookup by unique property
- **MATCH** → [./functions_lookup.md](./functions_lookup.md) - Lookup by non-unique property
- **MOVINGSUM** → [./functions_numeric.md](./functions_numeric.md) - Rolling sums
- **MOVINGAVERAGE** → [./functions_numeric.md](./functions_numeric.md) - Rolling averages
- **SELECT with time offset** → [./formula_modifiers.md](./formula_modifiers.md) - Month-12 (prior year same month), Month-1 (prior month, formulas only); MoM reporting → Show Value As
- **PREVIOUS/PREVIOUSOF** → [./functions_iterative_calculation.md](./functions_iterative_calculation.md) - Iterative calculations (circular dependencies, configuration, syntax); see also [functions_time_and_date.md](./functions_time_and_date.md) for SELECT vs PREVIOUS
- **SHIFT** → [./functions_lookup.md](./functions_lookup.md) - Shift dimension-typed properties
- **SWITCH** → [./functions_logical.md](./functions_logical.md) - Multi-way branching
- **TIMEDIM** → [./functions_lookup.md](./functions_lookup.md) - Date to time dimension

### By Category

**Lookup Functions**: [./functions_lookup.md](./functions_lookup.md) - ITEM, MATCH, SHIFT, TIMEDIM

**Numeric Functions**: [./functions_numeric.md](./functions_numeric.md) - CUMULATE, DECUMULATE, MOVINGSUM, MOVINGAVERAGE, ABS, SIGN, EXP, LN, LOG, SIN, COS, SQRT, MIN, MAX, MOD, QUOTIENT, POWER, ROUND, ROUNDUP, ROUNDDOWN, TRUNC, CEILING, FLOOR, RANK, SPREAD

**Time and Date Functions**: [./functions_time_and_date.md](./functions_time_and_date.md) - DATE, DATEVALUE, DAY, MONTH, YEAR, DAYS, NETWORKDAYS, WEEKDAY, STARTOFMONTH, EOMONTH, EDATE, INPERIOD, DAYSINPERIOD, PRORATA, MONTHDIF, FILLFORWARD, YEARTODATE, QUARTERTODATE, MONTHTODATE

**Iterative Calculation**: [./functions_iterative_calculation.md](./functions_iterative_calculation.md) - PREVIOUS, PREVIOUSOF (full spec: circular dependencies, configuration, performance, debugging)

**Text Functions**: [./functions_text.md](./functions_text.md) - TEXT, VALUE, LEN, LEFT, MID, RIGHT, LOWER, UPPER, PROPER, TRIM, CONTAINS, STARTSWITH, ENDSWITH, FIND, SUBSTITUTE, & (concatenation)

**Logical Functions**: [./functions_logical.md](./functions_logical.md) - AND, OR, NOT, TRUE, FALSE, ANYOF, ALLOF, ISBLANK, ISNOTBLANK, ISDEFINED, IFDEFINED, IF, SWITCH, IN, IFBLANK

**Basic Aggregation Functions**: [./functions_basic_aggregations.md](./functions_basic_aggregations.md) - AVGOF, COUNTALLOF, COUNTBLANKOF, COUNTUNIQUEOF, SUMOF, MINOF, MAXOF, COUNTOF

**Finance Functions**: [./functions_finance.md](./functions_finance.md) - NPV, XNPV, IRR, XIRR

**Forecasting Functions**: [./functions_forecasting.md](./functions_forecasting.md) - FORECAST_ETS, FORECAST_LINEAR, SIMPLE_EXPONENTIAL_SMOOTHING, DOUBLE_EXPONENTIAL_SMOOTHING, SEASONAL_LINEAR_REGRESSION, STANDARD_NORMAL_DISTRIBUTION

**Security Functions**: [./functions_security.md](./functions_security.md) - ACCESSRIGHTS, RESETACCESSRIGHTS

**List subset ↔ parent remap (1:1, no aggregator)**: [./formula_modifiers.md](./formula_modifiers.md#toparentlist-and-tosubset-list-subsets) — TOPARENTLIST, TOSUBSET

---

## Cross-References

**Before formula writing**: modeling-pigment-applications (core concepts, Pigment Modeling Best Practices standards, dimensional design)

**Related skills**: optimizing-pigment-performance (formula optimization, sparsity management)

---

## Critical Notes

- **ABSOLUTE: Pigment syntax ONLY**: You MUST NEVER write functions in other languages like Excel, SQL, Python, JavaScript, MDX, DAX, or ANY other language. Think ONLY in Pigment terms.
- **Search first**: Always search documentation to discover functions and patterns before writing
- **Follow workflow**: Complete the 8-step process in [./formula_writing_workflow.md](./formula_writing_workflow.md)
- **Review performance patterns**: Formulas with conditionals, dimensional changes, date ranges, or large/sparse targets must pass the checklist in [formula_performance_patterns.md](./formula_performance_patterns.md) before delivery. Simple arithmetic between same-dimensioned metrics does not require this review.
- **Prerequisites matter**: Understand modeling concepts from modeling-pigment-applications skill first
- **Document your work**: List which files you consulted for transparency

---

## Formula Commenting Standard

All generated formulas must include `//` comments for readability and maintainability.

**Top-level comment (required):**

- One `//` comment on its own line(s) immediately above the first line of the formula
- Explains the formula's **purpose** (what it computes and why)
- Use the same language as the block name

**Part-level comments (for non-trivial formulas only):**

- Add when the formula has multiple logical steps (several operations, functions, modifiers)
- Each comment on its **own line**, below the formula segment it describes
- One blank line between a part-level comment and the next formula segment
- Skip for one-liners or very obvious formulas

If comments are already present, try to maintain or enhance them. Replace them completely only if a formula update made them wrong or misleading.

**Example (multi-step):**

```pigment
// Final revenue: actual revenue for active scenarios plus budget adjustments by category

'Revenue'[FILTER: 'Scenario'.'Active' = TRUE]
// Filter revenue to active scenarios only

+ 'Budget Adjustment'[BY: 'Product'.'Category']
// Add budget adjustments mapped by product category
```

**Example (simple):**

```pigment
// Total cost: sum of fixed and variable costs
'Fixed Cost' + 'Variable Cost'
```

Comments must be included in the formula string passed to `tool:create_metric`, `tool:update_metric`, `tool:create_list_property`, or `tool:update_list_property`.

---

## Key Rules Summary

**Syntax:**

- Single quotes for identifiers: `'Revenue'`, `'Product'.'Category'`
- Double quotes for dimension items: literal form only when setting a `VAR_` metric default (MP02 — see [modeling_principles §4](../modeling-pigment-applications/modeling_principles.md)).
- Double quotes for string values: `"Active"`, `"Completed"`

**Modifiers:**

- **BY only changes the dimension you specify** - use REMOVE to eliminate dimensions
- **In BY, list only dimensions whose grain is changing** - do not re-list dimensions that are already on the metric (avoid over-explicit BY). For normalization ratios, use double BY in the denominator with only the changing dimension — see [formula_modifiers.md](./formula_modifiers.md).
- **Never chain BY on transaction lists** - use single BY with comma-separated expressions
- **BY > ADD** - BY is sparse (uses mappings), ADD is dense (all combinations)
- **SELECT = FILTER + REMOVE** - removes dimension after filtering, but only safe when the RHS carries no dimensions of its own (literal item or scalar VAR_ metric); if RHS is a metric with its own dimensions, those get removed too — use FILTER+REMOVE explicitly (see [formula_modifiers.md](./formula_modifiers.md))

**Transaction Lists in Metrics:**

- Must aggregate: `'List'.'Property'[BY: ...]`
- Use list column names: `'Orders'.'Customer'` not just `Customer`
- Text to dimension: `ITEM('List'.'TextCol', 'Dimension'.'Property')`
- Date to time: `TIMEDIM('List'.'DateCol', Month)`

**Sparsity:**

- Avoid ISBLANK/ISNOTBLANK on large sparse metrics — use ISDEFINED instead (returns TRUE/BLANK, not TRUE/FALSE). For small already-dense metrics or where explicit TRUE/FALSE output is required (e.g. data-completeness exports), ISBLANK is acceptable; see [functions_logical.md](./functions_logical.md) allow-list.
- If you BY on a dimension-typed metric, its sparsity is respected automatically; do not add IF/ISBLANK guards.
- Use IFBLANK(A, B) instead of IF(ISBLANK(A), B, A) - cleaner and doesn't densify
- Use BLANK for empty values when the cell genuinely has no data. Use 0 (not BLANK) when zero is a meaningful business value (zero variance, zero balance, zero growth) that must be displayed or that participates in downstream multiplication.
- EXCLUDE, not FILTER: NOT — see [formula_conditionals_style.md](./formula_conditionals_style.md)
- See [functions_logical.md](./functions_logical.md) for detailed blank handling guidance
