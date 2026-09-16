# Iterative Calculation (PREVIOUS & PREVIOUSOF)

> **PREVIOUSBASE() is deprecated** (removal Q2 2026). Use PREVIOUSOF instead.

> See also: [Iterating with Previous and Cycles](../iterating-with-previous-and-cycles/SKILL.md) for optimization strategies, subsetting, FILLFORWARD vs PREVIOUS.

---

## When to Use

Use PREVIOUS/PREVIOUSOF **ONLY** when current period's calculated result depends on prior period's calculated result (true circular dependency at metric level).

For everything else:

- Time shift (non-circular) → `[SELECT: Month-N]`
- Simple running totals (no per-period logic) → `CUMULATE()`
- Single-metric balance roll-forward (one metric only) → `PREVIOUS(Month)` with `IFDEFINED` seed
- Multi-metric balance roll-forward (inventory, cash balances) → `PREVIOUSOF` + iterative calculation cycle
- Fill blanks → `FILLFORWARD()`
- MoM display → Show Value As

---

## PREVIOUS — Single Block

`PREVIOUS(IterationDimension [, Offset])`

- Returns the previous cell of the **current Metric** in the iteration dimension
- **Takes the dimension, NOT the metric**: `PREVIOUS(Month)` not `PREVIOUS(Metric)`
- Default offset: 1; negative offset → BLANK
- Integer offsets faster than Metric offsets
- Engine computes full formula before moving to next cell; post-processing in a **separate metric**

```pigment
Cash = PREVIOUS(Month) + Income - Expense
End Inventory = PREVIOUS(Month) + Incoming - Outgoing
PREVIOUS(Month, 2) + 1
```

### Constraints

- Max 10,000 items in iterating dimension
- Cannot combine multiple PREVIOUS on different dimensions in one formula
- **Incompatible with**: IRR, XIRR, NPV, forecast/smoothing functions, DAYSINPERIOD, PRORATA (including when called with Working Days/Holidays args)
- **Densifies** the model → performance impact
- Modifiers (BY, SELECT, REMOVE, FILTER) supported **if they do not reference the iterating dimension**

---

## PREVIOUSOF — Multiple Blocks

`PREVIOUSOF(Metric [, Offset])`

- Returns referenced Metric value shifted by one period along iteration dimension
- Equivalent to `Metric[SELECT: IterationDimension - 1]` but allows circular references
- Metric must be in iterative calculation config (cycle)
- Metric cannot be stored in the Security folder
- Does not support Access Rights inheritance

### Setup workflow

1. `tool:list_cycles` — check existing cycles
2. Identify all metrics in dependency chain and iteration dimension
3. Create missing metrics first
4. `tool:create_cycle` with name, iteration dimension ID, all metric IDs
5. Write PREVIOUSOF formulas
6. `tool:update_cycle` to add/remove metrics

### Constraints

- Max 10 metrics per cycle
- All metrics must include the iteration dimension
- Max 10,000 items in iteration dimension
- Cannot combine or link two cycles

```pigment
// Balance roll-forward
Opening Balance = PREVIOUSOF('Ending Balance') + 'Inflow' - 'Outflow'
Ending Balance = 'Opening Balance' + 'Changes'
```

---

## Workaround — Avoid PREVIOUSOF via PREVIOUS + Offsets

Use only when the model needs a single balance metric and no separate Beginning metric. Do not use `[SELECT: Month - 1]` as a substitute for PREVIOUSOF when Beginning and Ending are separate metrics in a cycle.

```pigment
// Single-metric pattern only
'Ending Balance' = IFDEFINED(PREVIOUS(Month), PREVIOUS(Month), 'Opening Balance') + 'Movements'
```

---

## Reduction Heuristic

Collapse into PREVIOUS only when separate Beginning/Ending metrics are not required for reporting or downstream logic. When both metrics exist in the model, keep PREVIOUSOF + cycle.

---

## Compatibility

| Feature | PREVIOUS | PREVIOUSOF |
| --- | --- | --- |
| Arithmetic, IF/IFDEFINED | Yes | Yes |
| Modifiers (not iterating dim) | Yes | Yes |
| Window functions (CUMULATE, etc.) | Yes | Yes |
| Modifiers on iterating dim | No | No |
| Max iterating dimension | 10,000 | 10,000 |
| Max metrics | 1 (self) | 10 per cycle |
| Access Rights inheritance | Yes | No |
