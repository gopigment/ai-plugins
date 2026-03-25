---
name: writing-pigment-formulas
description: Always use this skill when writing, editing, or debugging Pigment formulas. Pigment uses a proprietary formula language — NEVER assume you know the syntax, and ALWAYS read the full documentation before writing any formula. Covers data types, modifiers, functions, calculation patterns, and formula performance best practices. Pigment has its own formula syntax, NEVER write formulas in any other language. This skill includes supporting files in this directory - explore as needed.
metadata:
  skill_path: /writing-pigment-formulas/SKILL.md
  base_directory: /writing-pigment-formulas
  includes:
    - "*.md"
---

# How to Use This Skill

**Progressive Disclosure Pattern**: This `SKILL.md` provides an overview. Most details live in supporting files.

**This file alone is often not sufficient**

**Required workflow**:

1. **Read this file first** - Understand available resources and when to use them
2. **Identify relevant topics** - Match your task to any of the supporting documents
3. **Read supporting files** - Use `read_file` or `grep` to access detailed documentation
4. **Explore as needed** - Use `ls`, `grep`, or `glob` to discover additional resources in this directory (some might not be explicitly mentioned in this file)

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
- **Use modifiers** - BY, ADD, REMOVE, SELECT, FILTER, EXCLUDE
- **Debug syntax** - Troubleshooting formula errors
- **Test and validate formulas** - Verifying formula correctness and expected behavior
- **Transform dimensions** - Changing dimensional structure of calculations
- **Allocate data** - Distributing values across dimensions
- **Match and lookup** - Finding data across dimensions

### Required reading for conditional logic

**MUST read [formula_conditionals_style.md](./formula_conditionals_style.md) when** the task involves any of:

- Nested IF chains or deep IF/AND/OR trees
- IFBLANK, ISBLANK, or IF(ISBLANK(...)) patterns
- FILTER vs IF tradeoffs (when to use each)
- EXCLUDE vs NOT on booleans
- Boolean logic with many AND/OR branches
- Repeated condition blocks across branches
- Optimizing conditional logic for performance or sparsity

Do not rely on SKILL.md summaries alone — open and read the full document for conditional design decisions.

---

## Syntax Fundamental

**Quoting Rules - MUST FOLLOW:**

| Element         | Syntax                        | Example                                           |
| --------------- | ----------------------------- | ------------------------------------------------- |
| Metric names    | Single quotes                 | `'Revenue'`, `'Total Sales'`                      |
| Dimension names | Single quotes                 | `'Product'`, `'Country'`                          |
| Property access | Dot notation with quotes      | `'Product'.'Category'`, `'Employee'.'Department'` |
| Dimension items | Double quotes after dimension | `Month."Jan 25"`, `Country."France"`              |
| String values   | Double quotes                 | `"Active"`, `"Completed"`                         |

**Dimension items and default property:**

- `Dimension."Item"` is shorthand for `Dimension.<DefaultDisplayProperty>."Item"`.
- When the default property is `Name`, `Country."France"` and `Country.Name."France"` are equivalent; both are valid.
- The explicit form `Dimension.Property."Value"` is valid when that property is the identifier used to resolve the item (e.g. `Country.Code."FR"` if the dimension uses Code as identifier).

**Best practice (MP02):** Hard-coding dimension items in formulas (e.g. `Country."France"`, `Version."Budget"`) is discouraged. Prefer an input metric of type Dimension (e.g. VAR_Budget_Version) or other structural references. See [modeling_principles – Formula Best Practices and MP02](../modeling-pigment-applications/modeling_principles.md).

**Common Mistakes:**

- ❌ `Revenue` → ✅ `'Revenue'` (missing quotes)
- ❌ `Product.Category` → ✅ `'Product'.'Category'` (missing quotes)
- ❌ `Month.'Jan 25'` → ✅ `Month."Jan 25"` (items use double quotes)

---

## Performance Patterns

**Every formula MUST apply performance patterns before delivery.**

Read [formula_performance_patterns.md](./formula_performance_patterns.md) and verify:

- [ ] Scoping clauses appear FIRST (FILTER, EXCLUDE, IFDEFINED)
- [ ] Aggregations (REMOVE, BY) appear AFTER calculations
- [ ] Using IFDEFINED instead of IF(ISBLANK())
- [ ] Using IFBLANK instead of IF(ISBLANK(...), default, ...)
- [ ] Conditional creation: use IF (not ADD + FILTER); subsetting a computed expression: use FILTER: CurrentValue (not IF(expr, expr, BLANK))
- [ ] Prefer sparsity-first conditionals (IFBLANK, FILTER/EXCLUDE, EXCLUDE not NOT) — **MUST read** [formula_conditionals_style.md](./formula_conditionals_style.md) when conditionals are present
- [ ] Using SELECT for prior period lookups (NOT PREVIOUS)
- [ ] Using BY instead of ADD where mapping exists
- [ ] Using BLANK instead of 0 for empty values
- [ ] Using BLANK instead of FALSE for boolean flags (FALSE is stored, BLANK is not)
- [ ] Access rights wrapped in IFDEFINED(User, ...)
- [ ] For continuous date ranges defined by Start/End Date, avoid multi-conditional IFs (`Date >= Start AND Date < End`).
- [ ] Prefer `PRORATA()` to express "active within a date range" and derive booleans or numeric flags from `PRORATA()` instead of using ISBLANK/ISNOTBLANK.
- [ ] If you BY on a dimension-typed metric, do not add IF/ISBLANK guards; BY respects that metric's sparsity.

### Date Range Presence (Prefer PRORATA over multi-conditional IF)

**Anti-pattern (to avoid):**

Using IF with multiple date comparisons to flag whether something is active between two dates:

```pigment
// Presence flag on Day (anti-pattern)
IF(
  Day >= 'Start Date'
  AND Day <= 'End Date',
  1,
  BLANK
)
```

This is:

- Verbose and harder to read
- Easy to get wrong at boundaries (inclusive vs exclusive)
- Duplicates logic that PRORATA already encodes

**Preferred Pattern: PRORATA for presence**

For any continuous interval defined by Start/End Date, use `PRORATA()` as the source of truth for presence.

**Step 1 — Numeric presence factor on a time dimension:**

```pigment
// Each active day = 1, outside range = BLANK
PRORATA(Day, 'Start Date', 'End Date' + 1)

// Each active month gets the proportional factor of active days in that month
PRORATA(Month, 'Start Date', 'End Date' + 1)
```

- Start Date is included
- End Date is excluded, so use End Date + 1 to make it inclusive
- On the Day dimension, this behaves as a sparse numeric flag (1 or BLANK)

**Step 2 — Boolean or numeric presence derived from PRORATA:**

Instead of using ISBLANK / ISNOTBLANK (which densify), derive presence as:

```pigment
// Boolean presence: TRUE when active, FALSE otherwise
ISDEFINED(
  PRORATA(Day, 'Start Date', 'End Date' + 1)
)
```

If you explicitly need a numeric 1/BLANK flag:

```pigment
IFDEFINED(
  PRORATA(Day, 'Start Date', 'End Date' + 1),
  1
)
```

**Guideline:** Encode the date-range logic once with `PRORATA(TimeDim, Start, End + 1)`. Derive booleans or numeric flags from `PRORATA()` using ISDEFINED / IFDEFINED. Do not use multi-conditional IFs (>= Start AND <= End) or ISBLANK / ISNOTBLANK for this pattern.

### Pre-delivery checklist (governance)

Before delivering any formula:

- **Dimension item literals:** Does the formula contain a literal dimension item (e.g. `Month."Jan 25"`, `Version."Budget"`, `Country."France"`)? If **yes**: have you applied MP02 (input metric of type Dimension or other structural reference)? If not, do not deliver the formula as-is; propose the MP02-compliant solution. See [modeling_principles](../modeling-pigment-applications/modeling_principles.md) section 4 and MP02.

---

## Formula Writing Process

**Key phases**: Understand Context → Search Documentation → Design → Build → Optimize → Validate → Deliver

**Follow the complete 8-step workflow**: [./formula_writing_workflow.md](./formula_writing_workflow.md)

- **Critical**: Always search documentation first before writing
- **Governance check (MP02):** If the formula will reference a specific dimension item (e.g. a month, version, country), read [modeling_principles](../modeling-pigment-applications/modeling_principles.md) section 4 and MP02 **before** writing the formula. Do not hard-code `Dimension."Item"`; propose or use an input metric of type Dimension (e.g. VAR_Selected_Month) unless the user explicitly accepts a one-off hard-coded reference.
- **Validation & Delivery**: Use Formula Builder Tools to validate and deliver formulas

---

## Formula Validation and Building Tools

**Important**: These tools are for **validation and implementation** when working with real formulas.

### Quick Validation

- `validate_formula` - Validate formula syntax WITHOUT applying it to any block
  - Use for: Checking syntax before calling `generate_metric_formula` or `create_or_update_dimension_formula`
  - Use for: Ensuring formula syntax is correct before including in user messages
  - Input: `formula` (the Pigment formula text)
  - Returns: Validation result with error highlighting and hints if invalid
  - **Limitations**:
    - Do NOT use with formulas containing `Previous` or `PreviousOf` functions

### Formula Generation (Expert System)

- `generate_metric_formula` - Generate/validate formulas for metrics (requires `metric_id` and `prompt`)
- `generate_list_property_formula` - Generate/validate formulas for list properties (requires `list_id` and `prompt`)

**Recommended Workflow**:

1. **Draft formula** - Write your formula based on requirements
2. **Validate** - Use `validate_formula` to check syntax
3. **Fix errors** - Iterate until formula is valid
4. **Apply** - Use `create_or_update_formula` or `update_list_property_formula`

**Alternative**: For complex formulas, use `generate_metric_formula` or `generate_list_property_formula` which combine generation and validation.

**How to apply**: After validation, use:

- Metrics: `create_or_update_formula` with the formula
- List properties: `update_list_property_formula` with the formula

---

## Prerequisites

This skill focuses on formula **implementation**. Before writing formulas, understand foundational concepts from the **modeling-pigment-applications** skill:

- Core platform knowledge (multidimensional engine, dimensions vs properties, sparsity principles)
- Pigment Modeling Best Practices standards (sparsity preservation, dimension alignment, formatting)
- Dimensional design concepts (source-to-target relationships, transformation cases)
- Modifier concepts
- **Governance (MP02):** Before writing any formula that filters or selects on a dimension member (e.g. "revenue for January 25", "revenue for Budget version"), you **must** read [modeling_principles](../modeling-pigment-applications/modeling_principles.md) section 4 and MP02 and apply the input-metric-of-type-Dimension pattern; do not output a formula with hard-coded `Dimension."Item"` without having checked MP02.

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
| Modifiers (BY, ADD, FILTER, etc.) | [formula_modifiers.md](./formula_modifiers.md)                       |
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
- **SELECT with time offset** → [./formula_modifiers.md](./formula_modifiers.md) - Prior period lookup: `'Revenue'[SELECT: Month-1]`
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

---

## Cross-References

**Before formula writing**: modeling-pigment-applications (core concepts, Pigment Modeling Best Practices standards, dimensional design)

**Related skills**: optimizing-pigment-performance (formula optimization, sparsity management)

---

## Critical Notes

- **ABSOLUTE: Pigment syntax ONLY**: You MUST NEVER write functions in other languages like Excel, SQL, Python, JavaScript, MDX, DAX, or ANY other language. Think ONLY in Pigment terms.
- **Search first**: Always search documentation to discover functions and patterns before writing
- **Follow workflow**: Complete the 8-step process in [./formula_writing_workflow.md](./formula_writing_workflow.md)
- **Apply performance patterns**: Every formula must pass the checklist in [formula_performance_patterns.md](./formula_performance_patterns.md) before delivery
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

Comments must be included in the formula string passed to `create_or_update_formula` or `generate_metric_formula`.

---

## Key Rules Summary

**Syntax:**

- Single quotes for identifiers: `'Revenue'`, `'Product'.'Category'`
- Double quotes for dimension items: `Country."France"`, `Month."Jan 25"` (short form); `Country.Name."France"` is also valid when `Name` is the dimension's default property. In general, avoid hard-coding items; prefer input metrics or structural references (see modeling_principles MP02).
- Double quotes for string values: `"Active"`, `"Completed"`

**Modifiers:**

- **BY only changes the dimension you specify** - use REMOVE to eliminate dimensions
- **In BY, list only dimensions whose grain is changing** - do not re-list dimensions that are already on the metric (avoid over-explicit BY). For normalization ratios, use double BY in the denominator with only the changing dimension — see [formula_modifiers.md](./formula_modifiers.md).
- **Never chain BY on transaction lists** - use single BY with comma-separated expressions
- **BY > ADD** - BY is sparse (uses mappings), ADD is dense (all combinations)
- **SELECT = FILTER + REMOVE** - removes dimension after filtering

**Transaction Lists in Metrics:**

- Must aggregate: `'List'.'Property'[BY: ...]`
- Use list column names: `'Orders'.'Customer'` not just `Customer`
- Text to dimension: `ITEM('List'.'TextCol', 'Dimension'.'Property')`
- Date to time: `TIMEDIM('List'.'DateCol', Month)`

**Sparsity:**

- NEVER use ISBLANK/ISNOTBLANK — use ISDEFINED (returns TRUE/BLANK, not TRUE/FALSE); they are densifying. See allow list and anti-patterns in [functions_logical.md](./functions_logical.md).
- If you BY on a dimension-typed metric, its sparsity is respected automatically; do not add IF/ISBLANK guards.
- Use IFBLANK(A, B) instead of IF(ISBLANK(A), B, A) - cleaner and doesn't densify
- Use BLANK, not 0 or FALSE for empty values
- Prefer sparsity-first conditionals: IFBLANK for override/precedence/case-style, FILTER/EXCLUDE when else is BLANK, EXCLUDE not NOT. See [formula_conditionals_style.md](./formula_conditionals_style.md).
- See [functions_logical.md](./functions_logical.md) for detailed blank handling guidance
