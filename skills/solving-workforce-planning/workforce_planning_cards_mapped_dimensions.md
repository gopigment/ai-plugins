# Workforce Planning – Workforce Cards & BY-Arrow Mappings

## Purpose

> **Terminology note:** BY-arrow formulas (`BY: -> WF_Card_*`) are for formula-level dimension mapping. Distinct from Pigment's mapped dimensions feature (Joined Pivot in Views); see `skill:using-mapped-dimensions` for the View-based approach.

The **4.0 layer** of workforce planning: Workforce Cards provide a unified view at Version × Month × Workforce, and BY-arrow mapping lets you report by Entity, Department, PnL, etc. without adding those dimensions to every core metric. Core model stays on Employee × Month × Version (EE) and TBH Requests × Month × Version (TBH); the Workforce layer lifts both into a single dimension and attaches attributes via card metrics; reporting uses `BY: -> Card_Attribute` to project metrics along those attributes.

**When to use:** two populations (EE and TBH) reported together as one workforce; flexible reporting by Entity/Department without hard-coding into every stats/comp metric; keeping core metrics on minimal driver dimensions.

**When not to use:** single-population models may not need a unified Workforce dimension. If every report always needs the same fixed axes, you might embed them — but BY-arrow pattern is still usually preferable.

**Why unify into one Workforce dimension:**

- Single set of total metrics (WF_Stats_Headcount, FTE, comp) already combining EE + TBH
- Consistent events and KPIs from one workforce tree, not two parallel trees
- Unified UX: one grid at Version × Month × Workforce or by BY-arrow mappings
- Security mapping once at Workforce level via `SEC_WF_Card_*`

---

## Core Concepts

- **Workforce dimension**: unifies two populations. Each item is employee-backed (linked via a Workforce-side property to Employee) or TBH-backed (linked to TBH Requests). Mapping: Employee → Workforce via Workforce[BY FIRSTNONBLANK: Workforce.Employee]; TBH → Workforce via Workforce[BY FIRSTNONBLANK: Workforce.TBH]. (`Workforce.Employee` / `Workforce.TBH` are properties on the Workforce dimension — Employee and TBH Requests carry no properties themselves.)
- **Reverse mapping (`EE_Workforce`, `TBH_Workforce`)**: to redistribute EE_Stats_*/TBH_Stats_* into Workforce (`[BY: EE_Workforce]` / `[BY: TBH_Workforce]`), the mapping must exist at Employee/TBH Requests grain. Since those dimensions carry no properties, `EE_Workforce` and `TBH_Workforce` are metrics (Dimension-typed → Workforce) — the reverse of `Workforce.Employee` / `Workforce.TBH` above.
- **Workforce Cards**: metrics at Version × Month × Workforce holding one attribute per item (Entity, Department, Currency, etc.). Calculated: for employee-backed rows, pull from `EE_Card_*` [BY CONSTANT: Workforce.Employee]; for TBH-backed rows, use TBH-side card. Workforce is "decorated" by attributes without adding Entity/Department as structural dimensions.
- **BY-arrow mappings**: reporting axes not in core metric structure. Get them by `Core_metric [BY: -> WF_Card_Department, WF_Card_Entity]`. Redistributes the metric along card values.
- **Core stays lean**: stats and comp at Workforce only (and Version, Month, Validation Status). Entity/Department appear only in derived metrics via BY: -> cards.

---

## Pipeline

```text
EE (Version × Employee × Month) + TBH (Version × TBH Requests × Month)
  → Lift to Workforce:
      WF_Stats_* = EE_Stats_*[BY: EE_Workforce] + TBH_Stats_*[BY: TBH_Workforce]
  → Build WF_Card_* (Entity, Department, Currency) at Version × Month × Workforce
  → Core stats/comp stay at Workforce only
  → Breakdowns via BY-arrow: WF_Stats_Headcount [BY: -> WF_Card_Department, WF_Card_Entity]
  → Push to P&L / financial: WF_Comp_Plan_Data [BY: Workforce -> WF_Card_Entity, WF_Card_Department]
```

---

## Patterns

### Workforce Cards (4.0 layer)

For any (Version, Month, Workforce item), answer: "What is this item's Entity / Department / Currency / …?" so downstream never needs to know whether the row is EE or TBH.

**Unified attribute pattern (e.g. Entity):**

- **WF_Card_Entity** = IFDEFINED(EE_Card_Entity[BY CONSTANT: Workforce.Employee], EE_Card_Entity[BY CONSTANT: Workforce.Employee], WF_Card_Entity_TBH)
- **WF_Card_Entity_TBH** = IFDEFINED(TBH_Stats_FTE[REMOVE: Validation Status][BY: TBH_Workforce], TBH_Req_Input_Entity[BY CONSTANT: Workforce.TBH])

**Department pattern:** same idea: IFDEFINED(EE_Card_Department[BY CONSTANT: Workforce.Employee], …, IFDEFINED(WF_Stats_FTE[REMOVE: Validation Status], TBH_Req_Input_Department[BY CONSTANT: Workforce.TBH])).

`TBH_Req_Input_Entity` and `TBH_Req_Input_Department` are metrics at TBH Requests × Version (not TBH Requests list properties) — see **Workforce Planning – Application Architecture & Patterns**.

For any Workforce item, a single set of card metrics (Entity, Department, Currency, …) gives its attributes for that Version × Month. All "decoration" is computed, not structural.

### BY-arrow mappings in practice

**Headcount by Dept & Entity:**

- WF_Stats_Headcount_Dep_Entity = WF_Stats_Headcount [BY: -> WF_Card_Department, WF_Card_Entity]

**Push to P&L / financial:**

- PUSH_WF_Workforce_Plan_Data = WF_Comp_Plan_Data [BY: Workforce -> WF_Card_Entity, WF_Card_Department]

**Why not structural dimensions everywhere:** adding Entity × Department to every stats/comp metric causes large sparsity, bigger formulas, more complex filters/access rights, harder refactoring. BY-arrow mappings project only when needed.

---

## Pitfalls

- **Do not add Entity, Department, or other "rich" axes as structural dimensions** to `WF_Stats_*` or `WF_Comp_*`. Use `BY: -> WF_Card_*` for breakdowns.
- **Every Employee and TBH Request must map to exactly one Workforce item** (BY FIRSTNONBLANK). Duplicate or missing mappings silently break totals.
- **`WF_Card_*` metrics must cover all Workforce items** (both EE- and TBH-backed). Blank TBH-side cards cause mapped breakdowns to lose those rows.
- **Security cards (`SEC_WF_Card_*`) must mirror `WF_Card_*`.** Divergence means access rights won't match actual data groupings.
- **Employee and TBH Requests carry no custom properties.** `EE_Workforce` / `TBH_Workforce` (the mapping used for `BY: EE_Workforce` / `BY: TBH_Workforce`) are metrics, not list properties, on those dimensions.

---

For Library activation and sharing mechanics, see `skill:sharing-data-between-applications`.

**Related skills:** [Workforce Planning – Application Architecture & Patterns](./workforce_planning_architecture_patterns.md); [Workforce Planning – Changelog to Override Metrics](./workforce_planning_changelog_overrides.md); [Workforce Planning – Snapshot Spread Logic](./workforce_planning_snapshot_spread.md).
