# Pigment FX: Currency Conversion Design & Usage Guide

## Purpose

FX conversion is a **centralized, Version-aware engine** in the Hub app (`00. Hub`). The Hub owns FX dimensions, rate inputs, entity mapping, and the consolidated rate output. Bespoke apps (Nexus, OPEX, Workforce, etc.) **never implement FX logic**; they PULL the Hub rate metric and multiply local amounts.

**When to use:** building or connecting a multi-currency layer across several applications; setting up FX in a Hub app; wiring a consuming app to convert local amounts to reporting currency.

**When not to use:** single-currency workspace with no multi-entity FX needs; ad-hoc conversion without a shared Hub engine.

---

## What Lives Where

| Block type | Hub app | Bespoke / consuming apps |
| --- | --- | --- |
| **Dimension lists** | `Currency`, `FX Rate Types`, `Reporting Currency`, `Entity`, `Version`, Calendar (`Month`, etc.) | Activate Hub Library; use shared dimensions in metric structures (normal PascalCase names, no PUSH_/PULL_) |
| **Input / calc metrics** | `INPUT_FX_01_Rates`, `CALC_FX_02_Rates_Spread`, `MAP_FX_Entity_Currencies` | None; do not recreate FX inputs or mapping |
| **Shared output metric** | `PUSH_DH_FX_Rates` (one metric only) | `PULL_DH_FX_Rates` referencing `'00. Hub'::'PUSH_DH_FX_Rates'` |
| **Reporting metrics** | None | Local-amount metrics × `PULL_DH_FX_Rates` (e.g. `REP_PnL_Data` in Nexus) |

**Hard rules:**

- **Dimensions** are shared as dimension lists via Library. They keep PascalCase names (`Currency`, `Entity`, `FX Rate Types`). No PUSH_/PULL_ prefix, no wrapper metrics.
- **Metrics** crossing app boundaries use PUSH_ (Hub) / PULL_ (consumer). Only `PUSH_DH_FX_Rates` leaves the Hub.
- **Entity mapping** (`MAP_FX_Entity_Currencies`) is a Hub-internal metric, not shared. Consumers get the mapping baked into `PUSH_DH_FX_Rates`.

`DH` in metric names = Data Hub (the source app acronym), same pattern as `PULL_CR_*`, `PULL_WF_*`, `PULL_OP_*`.

---

## Dimensions (Hub-owned; shared via Library)

These are **dimension lists**, not metrics. Create them in the Hub and share with `tool:batch_share_blocks` (unlocked by `skill:sharing-data-between-applications`). Consuming apps activate the Hub Library and include them in metric structures as needed.

| Dimension | Role |
| --- | --- |
| **Currency** | Universe of currencies (USD, EUR, GBP, CNY). Property: Name (Text). |
| **FX Rate Types** | Rate context selector. Standard items: **AVG** (P&L, flow items), **END** (Balance Sheet, stock items). |
| **Reporting Currency** | Target currency for output. Standard items: **Local** (entity functional currency), **Group** (consolidation currency). This is a planning axis, not the Currency dimension itself. |
| **Entity** | Org structure; each entity has a functional currency resolved via `MAP_FX_Entity_Currencies`. |
| **Version** | Planning cycles; each Version carries its own FX rate series. Keep in Hub for synchronized cycles across apps. |
| **Month** (and Calendar) | Time axis for rate lookup. |

### Seed Dimension Items

After creating dimension lists, populate them with initial items. Ask the user which currencies, entities, and rate types to include. Do NOT leave dimension lists empty; every `tool:create_list` for a dimension MUST be followed by `tool:add_list_items` (both unlocked by `skill:creating-dimensions-and-hierarchies`).

**Local vs transactional:** "Local" on Reporting Currency means the entity's **functional** currency, not the transactional currency of individual line items.

**Triangulation:** optional intermediate calc in the Hub when the rate source does not cover all currency pairs directly (e.g. CNY→USD and USD→EUR but no CNY→EUR). Most models do not need this.

---

## Hub Metric Pipeline

All FX **metrics** below live only in the Hub. Only the last one is shared.

```
INPUT_FX_01_Rates          raw FX rate input (by Version)
  ↓
CALC_FX_02_Rates_Spread    FILLFORWARD over Month when INPUT has gaps (skip if every month populated)
  ↓
MAP_FX_Entity_Currencies   entity → functional currency mapping (Hub-internal)
  ↓
PUSH_DH_FX_Rates           consolidated rate; shared via Library
```

### INPUT_FX_01_Rates

Dims: `Currency × FX Rate Types × Version × Month`. Raw input store. Rates are entered per Version from the start (Budget, Reforecast Q1, etc. each carry their own series). When creating a new Version, clone rates from an existing Version and adjust.

### CALC_FX_02_Rates_Spread _(only if needed)_

Dims: `Currency × FX Rate Types × Version × Month`. Same structure as `INPUT_FX_01_Rates`.

**When to add this layer:** source rates are loaded periodically and leave blanks on intermediate months (quarterly rates, annual budget rates, mid-year new currency with no backfill). **Skip it** when every month has an explicit rate in `INPUT_FX_01_Rates`.

**Formula:** propagate the last known rate forward along `Month` for each `(Currency, FX Rate Types, Version)` combination. Use `FILLFORWARD`, not `PREVIOUS` or `IFBLANK` + `PREVIOUS`; it is non-iterative and the right fit for "carry last value forward" (see `skill:using-formula-functions`, `skill:writing-pigment-formulas`).

```pigment
FILLFORWARD('INPUT_FX_01_Rates', Month)
```

Keep this as a **separate metric** so `INPUT_FX_01_Rates` stays the untouched raw input. Downstream Hub metrics (`MAP_FX_Entity_Currencies`, `PUSH_DH_FX_Rates`) reference `CALC_FX_02_Rates_Spread` when it exists, or `INPUT_FX_01_Rates` directly when every month is populated.

**Behaviour:** months before the first non-blank rate stay blank; from the first entered rate onward, blanks inherit the most recent non-blank value. Each Version's series is filled independently because `Version` remains in the metric structure.

### MAP_FX_Entity_Currencies _(Hub-internal)_

Dims: `Entity × Reporting Currency` → value is a **Currency** dimension member. Static mapping (e.g. French entity + Reporting Currency."Local" → EUR). No Version dimension. Feeds `PUSH_DH_FX_Rates`; never shared outside the Hub.

### PUSH_DH_FX_Rates _(only metric shared from the Hub)_

Dims: `FX Rate Types × Version × Entity × Month × Reporting Currency`. Consolidated conversion rate: "multiply a local amount by this rate to express it in the selected Reporting Currency, for this Version, Entity, Month, and Rate Type." Share with `tool:batch_share_blocks`.

Use `[BY CONSTANT: Currency -> 'MAP_FX_Entity_Currencies']` (not `[BY: ...]`) for the entity-to-currency lookup in the final rate metric. The PUSH metric must return 1 when Reporting Currency = "Local".

---

## Consuming App Integration

In each bespoke app (Nexus, OPEX, Workforce, etc.):

1. **Activate the Hub Library** with `tool:enable_application_library` (unlocked by `skill:sharing-data-between-applications`).
2. **Use shared dimensions** (`Entity`, `Version`, `Month`, `FX Rate Types`, `Reporting Currency`) in local metric structures. Do not duplicate them.
3. **Create one PULL\_ metric** referencing the Hub output:

```
PULL_DH_FX_Rates = '00. Hub'::'PUSH_DH_FX_Rates'
```

4. **Keep amounts in local/functional currency** until the reporting step, then multiply:

```
REP_PnL_Data =
  'PnL_Nexus_99_Actual_Plan_Data'
  * 'PULL_DH_FX_Rates'[SELECT: 'FX Rate Types'."AVG"]
```

Use **AVG** for P&L flow items. Use **END** for Balance Sheet stock items. The Rate Type dimension selects the correct rate without separate FX metrics per statement.

---

## Pitfalls

- **Do not mix dimensions and metrics.** `Currency`, `FX Rate Types`, `Reporting Currency` are dimension lists. `PUSH_DH_FX_Rates` / `PULL_DH_FX_Rates` are metrics. Never prefix dimensions with PUSH_/PULL_.
- **Do not duplicate FX logic in consuming apps.** No local FX inputs, no local entity-currency mapping, no direct reference to Hub internal metrics (`INPUT_FX_01_*`, `CALC_FX_02_*`, `MAP_FX_*`).
- **Do not reference Hub PUSH\_ directly.** Always wrap in `PULL_DH_FX_Rates` in the consuming app.
- **Rates are per Version.** Each Version owns its rate series from `INPUT_FX_01_Rates` onward; clone between Versions, do not share one unversioned series.
- **AVG for flow, END for stock.** Establish at setup and enforce consistently.

---

Related skills: [Centralizing Financial Reporting Metric (Nexus Pattern)](./finance_nexus_financial_statements.md) · [OPEX Planning – Architecture & Patterns](./opex_planning_application_architecture.md) · `skill:sharing-data-between-applications` · `skill:architecting-multi-application-solutions`
