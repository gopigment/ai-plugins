---
name: writing-pigment-formulas
description: "Execution skill. Use when writing, editing, or debugging Pigment metric formulas — quoting, references, data types, BLANK, and comments."
metadata:
  skill_path: /skills/writing-pigment-formulas/SKILL.md
  base_directory: /skills/writing-pigment-formulas
---

# Write Pigment Formulas

**Pigment syntax only.** Never write Excel, SQL, Python, DAX, or any other language.

Formulas operate on metrics (multidimensional grids). Each cell = intersection of one item per dimension. Modifiers control dimension alignment.

---

## Stage 1 — Identify Inputs

Before writing any formula:

1. **Confirm the target metric** — name, dimensions, type (Number / Date / Text / Dimension / Boolean). The formula result type must match.
2. **Confirm source blocks** — metrics, lists, properties referenced in the formula. Use `tool:search_metrics_and_lists` (or `tool:semantic_search` when names are unclear) to find exact names and spot duplicates.
3. **Map dimensions** — compare source dimensions vs target dimensions. If they differ, you will need modifiers (Stage 2.4).
4. **Check for circular dependencies** — if the formula references its own metric or forms a cycle, you need PREVIOUS / PREVIOUSOF with a declared cycle.

---

## Stage 2 — Write the Formula

### 2.1 Start with a comment

Every medium-to-complex formula should be explained with a comment: above the formula, restate its logic in Structured English, in evaluation order, using business terms and friendly names. A reader who doesn't know Pigment must be able to understand it.

Guidance on commenting:
- the comment should not be much longer than the formula
- if the formula is simple, short and self-explanatory (one FUNCTION(), simple IF with friendly metric names), don't comment
- do not use Metric names within, because they can change and your comments will be obsolete
- do not comment within the formula, only before it
- `//` only; no block comments.

```pigment
// IF the period is an actual period
// then aggregate the actuals from the data
// else bring the planning numbers from the other applications
```

### 2.2 Apply quoting rules

| Reference type | Quotes | Example |
| --- | --- | --- |
| Metric / dimension / property / list name | Single `'...'` | `'Revenue'`, `'Product'.'Category'` |
| Dimension item | Double `"..."` after dim | `Type."Revenue"`, `Status."Active"` |
| Text literal | Double `"..."` | `"Active"`, `"Completed"` |
| Cross-app block (via Library) | `'App'::'Block'` | `'Finance'::'Revenue'` |
| List property | `'List'.'Property'` | `'Employee'.'Start Date'` |

Full item path: `'List'.'Property'."Item"` — default property can be omitted: `Status."Active"`.

**⛔ Never hard-code dimension items or dates in formulas.**
`Month."Jun 24"`, `Version."Actual"`, `Country."France"` are all forbidden.

- For a specific member → create a Dimension-typed input metric and reference that.
- For "last actual month" → use `[SELECT LASTNONBLANK: 'Is Actual']` or equivalent flag metric.
- For a fixed date → create a Date-typed input metric.
  The only exceptions are stable structural items (e.g. `Version."Actual"` in a Version-aware
  application where that item is guaranteed to always exist and never be renamed).

**Always quote identifiers** even when Pigment allows omitting quotes on simple names.

### 2.3 Write the core calculation

Build the expression without modifiers first. Pass only the function arguments the requirement calls for — do not add extra optional parameters speculatively. Use the correct operators for the data type:

| Type | Operators | Notes |
| --- | --- | --- |
| Number | `+ - * / = <> < > <= >=` | BLANK propagates on `*`, `/`, `AND`; division by zero → BLANK |
| Date | `= <> < > <= >= + -` | `+`/`-` for day offsets; use date functions for month/year logic |
| Text | `= <> &` | `&` concatenation; wrap numbers with `TEXT()` |
| Dimension | `=` | From dim-typed properties or mapping metrics |
| Boolean | `AND OR NOT = <>` | 3-state: TRUE / FALSE / BLANK |

A bare comparison (e.g. `Day = Department.'Start Date'`) is **dense**: FALSE everywhere it doesn't hold, not BLANK. When comparing against a sparse property, wrap it — `IF(Day = Department.'Start Date', TRUE, BLANK)` — to keep the result sparse.

### 2.4 Add modifiers for dimension alignment

Syntax: `Block[MODIFIER Method: arguments]`

| Need | Modifier | Effect |
| --- | --- | --- |
| Aggregate child → parent | `[BY SUM: Dim.Parent]` | Replaces dim |
| Allocate parent → child, same value | `[BY CONSTANT: Dim.Parent]` | Replicates value to each child |
| Allocate parent → child, divided | `[BY SPLIT: Dim.Parent]` | Divides value equally across children |
| Drop a dimension | `[REMOVE SUM: Dim]` | Removes dim |
| Filter, keep dimension | `[FILTER: condition]` | Sparse subset |
| Exclude rows | `[EXCLUDE: condition]` | Opposite of FILTER; preserves sparsity |

**Allocation method defaults to CONSTANT** — if the request says "split," "distribute," "divide equally," or "spread," use `SPLIT` explicitly instead.
**Prefer BY over ADD when a mapping exists** — `BY` is sparse (only allocates where the mapping is defined); `ADD` is dense (full cross-product) and should be reserved for cases with no mapping.

`CurrentValue` = value of the expression the modifier applies to: `'Revenue'[FILTER: CurrentValue > 1000]`

**Transaction lists:** always use a single BY with all dimension mappings. Never chain BY (properties are lost after first aggregation).

```pigment
// ✅ Single BY with all mappings — use stored Month property (not inline TIMEDIM)
'Orders'.'Amount'[BY SUM: 'Orders'.'Month', 'Orders'.'Product']
// Month property on the TL has formula: TIMEDIM('Orders'.'Date', Month)
```

Full modifier reference: `skill:using-formula-modifiers`

### 2.5 Use the right functions

Full reference with syntax → `skill:using-formula-functions`. Key categories:

| Category | Functions | Use when |
| --- | --- | --- |
| Conditional | `IF`, `SWITCH`, `IFBLANK`, `IFDEFINED` | Branching; override chains; sparsity guards |
| Aggregation | `SUMOF`, `AVGOF`, `COUNTOF`, `MINOF`, `MAXOF` | Aggregate a list property without BY |
| Text | `LEFT`, `RIGHT`, `MID`, `FIND`, `SUBSTITUTE`, `TEXT`, `VALUE`, `&`, `CONTAINS` | String manipulation; type conversion; `CONTAINS(substring, haystack)` — substring first |
| Date / Time | `DATE`, `YEAR`, `MONTH`, `DAY`, `NETWORKDAYS`, `DAYSINPERIOD`, `PRORATA` | Date arithmetic; period coverage |
| Numeric | `ROUND`, `ABS`, `MIN`, `MAX`, `RANK`, `MOVINGSUM`, `CUMULATE` | Rounding; ranking; rolling windows |
| Lookup | `MATCH`, `ITEM` | Find items in lists; convert text to dimension |
| Iterative | `PREVIOUS`, `PREVIOUSOF`, `FILLFORWARD` | Time-series accumulation; gap filling (require cycles); see below |
| Utility | `TIMEDIM`, `ISDEFINED`, `IN`, `SHIFT` | Date→dim conversion; existence check; set membership; dimension offset |
| Finance | `NPV`, `IRR`, `XNPV`, `XIRR` | Discounted cash flow; internal rate of return |
| Forecasting | `FORECAST_ETS`, `FORECAST_LINEAR` | Statistical forecasting |

**Never invent functions.** Only use documented Pigment functions from the reference above.

**Iterative patterns:**

- **PREVIOUS** — each period builds on prior (requires a declared cycle): `IFDEFINED(PREVIOUS(Month), PREVIOUS(Month) * (1 + 'Growth Rate'), 'Seed Value')`
- **FILLFORWARD** — fill gaps in time series (use a separate cleaning metric): `FILLFORWARD('FX Rate Input', Month)`
- **PREVIOUSOF** — cross-metric cycles (beginning/ending balance, inventory roll-forwards) → `skill:iterating-with-previous-and-cycles`

### 2.6 Handle BLANK correctly

`BLANK` = not stored (sparse). `0` / `FALSE` = stored (dense).

| Expression | Result | Why |
| --- | --- | --- |
| `BLANK + 5` | `5` | Additive identity |
| `BLANK * 5` | `BLANK` | Multiplicative propagation |
| `X / 0` or `X / BLANK` | `BLANK` | Auto-handled; no guard needed |
| `NOT(BLANK)` | `BLANK` | NOT does not flip BLANK to TRUE |

**Rules:**

- Return `BLANK` where no value should exist; never substitute `0` or `FALSE`
- Division by zero/BLANK is auto-handled; no guard needed

Sparsity preference rules (IFDEFINED vs ISBLANK), performance anti-patterns: `skill:writing-performant-formulas`

---

## Stage 3 — Validate Before Applying

**You MUST call `tool:validate_formula` before passing a formula to `tool:create_metric`, `tool:update_metric`, `tool:create_list_property`, or `tool:update_list_property`.** The only exception is PREVIOUS/PREVIOUSOF formulas without a target metric — those cannot be validated standalone; pass `target.metric_id` for an existing metric to validate them normally. Applying an unvalidated formula puts the metric or property into an error state.

Run through this checklist first:

1. [ ] Single quotes on all identifiers; double quotes on items and strings
2. [ ] BLANK (not 0/FALSE) where no value should exist
3. [ ] No ISBLANK / ISNOTBLANK; use IFDEFINED / IFBLANK / ISDEFINED
4. [ ] No hard-coded dimension items or dates
5. [ ] Transaction list: single BY with all dim mappings; never chain BY
6. [ ] Allocation: BY (not ADD) when a mapping exists; SPLIT explicitly requested where the ask is "split"/"distribute"/"divide equally"/"spread" — not left on the CONSTANT default
7. [ ] Formula result type matches target metric type

### Structural Changes: Apply Before Validating

If the task requires both a structural change (adding/removing dimensions, changing type) **and** a formula
update on the same metric or list property, **apply the structural change first**, then call
`tool:validate_formula`. The validator checks against the block's current live structure — validating before
applying the structural change tests the formula against the old structure and produces misleading results.

### Structural Dimension Changes: Check Downstream Formulas

Adding or removing a structural dimension on a metric does not automatically propagate to the metrics that reference it. If a downstream formula does not already carry the new dimension, the compiler silently broadcasts or collapses values to align with the target instead of failing — wrong numbers, no visible error.

Do NOT assume a referencing formula "just passes the dimension through". Before declaring a structural dimension change done:

1. Identify metrics whose formulas reference the changed metric (search for references to its name).
2. For each one, call `tool:validate_formula` with that metric's `formula` **and** `target.metric_id` set to it — the target is what surfaces the mismatch.
3. If the "automatic formula dimensions adjustment" hint appears, the formula's dimensions do not match the metric's own dimensions. Do not accept the implicit broadcast/collapse as correct. Confirm with the user whether the dimension should genuinely propagate to that metric (and how — the change may need to be threaded further upstream instead), or whether the mismatch is intentional.

This applies once per changed metric in the chain, not once for the whole change: validate each metric after its own dimension and formula update, before moving on to the next one.

---

## Related Skills

- `skill:using-formula-modifiers` — modifier mechanics, arrow-mapping syntax, aggregation methods
- `skill:using-formula-functions` — all function signatures and examples
- `skill:writing-performant-formulas` — sparsity rules, anti-patterns, pre-delivery checklist
- `skill:choosing-formula-patterns` — business requirement → pattern mapping
- `skill:iterating-with-previous-and-cycles` — PREVIOUS/PREVIOUSOF cycles
