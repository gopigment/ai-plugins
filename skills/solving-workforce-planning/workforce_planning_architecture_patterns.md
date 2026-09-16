# Workforce Planning – Application Architecture & Patterns

## Purpose

Reusable pattern for an employee-based workforce planning application: layered metric architecture, dimension roles, data flows (Existing Employees + To-Be-Hired), override and validation patterns, and naming conventions.

**When to use:** building or extending a workforce planning app combining existing employees (HRIS actuals, spread logic, cards, events, compensation), To-Be-Hired (request list, FTE/salary/dates, validation), consolidated workforce (headcount, FTE, transfers, comp at Workforce × Dept × Entity × Version × Month), overrides (Changelog → override metrics → staging), governance (validation, version close, access rights, merit cycles), and financial alignment (PnL/BS mapping, FX, merit and tax assumptions).

**When not to use:** purely headcount-only models with no compensation or TBH may not need full layering; apps without version/scenario dimensions or validation workflows skip scenario and governance patterns.

---

## Core Concepts

- **Layered metric architecture**: Data (load/staging) → Card (canonical attributes per entity/month) → Stats (events: hires, terms, transfers) → Comp (salary, bonus, benefits, taxes) → Push/KPI (export, counts) and Security (ARM/ARC). Each layer consumes the layer below; no skipping.
- **Two populations**: **Existing Employees (EE)** — Employee dimension holds only Employee ID and Name; every other attribute comes from the Roster load (Roster TL) + Changelog overrides. Changelog itself carries no properties beyond its default: identity/timing/workflow/audit are `CL_*` metrics (Changelog grain), and new values are `CL_Input_*` metrics (Changelog × Version). **To-Be-Hired (TBH)** — TBH Requests list carries no properties beyond its default; Department, Entity, Hire Date, FTE, Salary, Validation Status are `TBH_Req_Input_*` metrics at TBH Requests × Version. Both feed Workforce-level cards and stats.
- **Reverse Workforce mapping**: since Employee and TBH Requests carry no custom properties, the link each uses to redistribute its own stats into Workforce is a metric, not a property: `EE_Workforce` (Employee grain, Dimension-typed → Workforce) and `TBH_Workforce` (TBH Requests grain, Dimension-typed → Workforce). Used as `EE_Stats_*[BY: EE_Workforce]` / `TBH_Stats_*[BY: TBH_Workforce]`.
- **Data → Card → Stats**: Data resolves raw sources to consistent grain (Version × Entity × Month). Card fixes attributes at a single "planning load" reference. Stats derive events by comparing cards across months.
- **Override-first staging**: EE staging uses IFDEFINED(Override, Override, HRIS_logic). Overrides come from Changelog projection (see "Changelog to Override Metrics"). Cards and stats only see post-override view.
- **Validation dimension**: Validation Status (Draft/Submitted/Approved/Rejected) tags rows or versions. Plan metrics filter to Approved only; access rights control who can change status.
- **Version & scenario dimensions**: Version holds window start/end, close date, last actuals month. Data Type (Actual/Budget/Forecast) can drive FX or display. Metrics and access rights respect version close and windows.
- **Workforce Cards & BY-arrow mappings**: The 4.0 layer unifies EE + TBH at Version × Month × Workforce. `WF_Card_*` metrics hold each item's attributes; core stats/comp stay at Workforce only. Breakdowns use `BY: -> WF_Card_*` so Entity/Department are mapped, not structural. See **Workforce Planning – Workforce Cards & BY-Arrow Mappings**.
- **Naming conventions**:
  - Prefix by domain: `EE_` (existing employee), `TBH_` (to-be-hired), `WF_` (workforce total), `ASM_` (assumptions), `PUSH_` (export), `KPI_`, `ARM_`/`ARC_` (access rights)
  - Suffix by layer: `_Data`, `_Card`, `_Stats`, `_Comp`, `_Input`, `_Calc`, `_Req_`, `_Rep_`, `_View`
- **Dimension roles**:
  - Structural: Entity, Department, Workforce, Employee, Job Position, Grade
  - Scenario: Version (window start/end, close date, last actuals month), Data Type (Actual/Budget/Forecast)
  - Time: Month, Quarter
  - Financial: PnL_Account, BS_Account, Currency, FX Rate Types
  - Validation: Validation Status (Draft/Submitted/Approved/Rejected)
  - Security: User (Name, Email, ID)
  - Metrics rarely use Pigment native Scenarios; Version and Data Type dimensions model scenarios

---

## Pipeline

**Existing Employees**

```text
HRIS load list (e.g. Roster TL) → Data layer (EE_Data_*: resolve by snapshot, history/plan toggle, override-first)
  → Spread logic (effective snapshot month, FILLFORWARD dates, version window)
  → Override metrics (EEO_* from Changelog)
  → Card layer (EE_Card_*: canonical attributes at Version × Employee × Month)
  → Stats (EE_Stats_*: new hires, terminations, transfer in/out via card comparison)
  → Workforce aggregation (EE_Stats_* [BY: EE_Workforce] → WF_Stats_*)
```

**To-Be-Hired**

```pigment
TBH request list (TBH ID only) + TBH_Req_Input_* (Department, Entity, Hire Date, FTE, Salary, Validation Status; metrics at TBH Requests x Version) → Validation status
  → TBH_Stats_* (headcount, FTE, terminations; PRORATA by hire/term; filter Approved, plan months; exclude hired)
  → Workforce mapping (TBH_Stats_* [BY: TBH_Workforce] → WF_Card_*, WF_Stats_*)
```

**Consolidation & output (4.0 Workforce layer)**

```pigment
WF_Card_* (unified attributes at Version × Month × Workforce: EE_Card_* or TBH-side cards)
  → WF_Stats_* at Workforce only (EE_Stats_* [BY: EE_Workforce] + TBH_Stats_* [BY: TBH_Workforce])
  → Breakdowns via BY-arrow mappings: WF_Stats_Headcount [BY: -> WF_Card_Department, WF_Card_Entity], etc.
  → WF_Comp_* → WF_Comp_Plan_Data → PUSH_* (map to Entity/Dept/PnL via cards)
  → KPI_* (export, filled TBH count, changelog count)
```

See **Workforce Planning – Workforce Cards & BY-Arrow Mappings** for why core metrics stay at Workforce and how mapping works.

**Governance:** Validation Status = Approved; version close excluded; ARM/ARC for edit and approval; merit/tax assumptions and FX in dedicated layers.

---

## Patterns

### Data layer (`EE_Data_*`, resolution to Version × Employee × Month)

Resolve HRIS (and spread) to planning grain; apply history vs plan; apply override-first.

```text
IF(SET_Populate_History,
  Source_list.'Attribute'
    [BY LASTNONBLANK: Entity_Mapping, Snapshot_Date]
    [ADD: Version][FILTER: Is_Actual]
    [BY CONSTANT: Spd_Effective_Snapshot_History],
  Source_list.'Attribute'
    [BY LASTNONBLANK: Entity_Mapping, Snapshot_Date]
    [ADD: Version][FILTER: Is_Actual]
    [BY CONSTANT: Spd_Effective_Snapshot_Plan]
)
```

Override-first when attribute can be overridden: IFDEFINED(`EEO_*`, `EEO_*`, <above>). See "Snapshot Spread Logic" and "Changelog to Override Metrics" skills.

### Card layer (`EE_Card_*`, `WF_Card_*`)

Canonical attributes at Version × Entity × Month (EE) or Version × Month × Workforce (WF). EE cards fix to one "planning load" snapshot; WF cards unify EE + TBH.

- **EE pattern:** Card_Attribute = Data_Attribute [BY CONSTANT: Spd_Effective_Snapshot_Month]
- **WF pattern:** WF_Card_Attribute = IFDEFINED(EE_Card_Attribute[BY CONSTANT: Workforce.Employee], same, WF_Card_Attribute_TBH). Breakdowns use core metric [BY: -> WF_Card_Department, WF_Card_Entity]. Full pattern: **Workforce Planning – Workforce Cards & BY-Arrow Mappings**.

### Stats layer (`EE_Stats_*`, `TBH_Stats_*`, `WF_Stats_*`)

Events (new hire, termination, transfer in/out) and levels (headcount, FTE).

**EE events:** compare cards across months. Example transfer out:

```text
IF(
  (EE_Card_Entity <> EE_Card_Entity[SELECT: Month + 1]) OR
  (EE_Card_Department <> EE_Card_Department[SELECT: Month + 1]),
  -1
)
```

New hire/termination: presence in current month but not previous (or reverse). Use +1/-1 flags then aggregate at Workforce.

**TBH headcount presence:** `PRORATA(Month, STARTOFMONTH(TBH_Req_Input_HireDate), STARTOFMONTH(TBH_Req_Input_TermDate + 1)) * ROUNDUP(TBH_Req_Input_FTE, 0) [SET_Plan_Month_View] [BY: -> TBH_Req_Input_ValidationStatus]` — yields 0/1 headcount per month. `TBH_Req_Input_*` are metrics at TBH Requests × Version, not list properties. Filter to approved status via a SET_ input metric or a boolean property. Exclude rows linked to a hired employee.

**TBH fractional FTE:** `PRORATA(Month, TBH_Req_Input_HireDate, TBH_Req_Input_TermDate + 1) * TBH_Req_Input_FTE [SET_Plan_Month_View] [BY: -> TBH_Req_Input_ValidationStatus]` — raw dates, no `STARTOFMONTH`; returns 0–1 fractional value per month. Use this form when you need partial-month proration, not 0/1 presence.

**WF stats:** `EE_Stats_… [BY: EE_Workforce] [FILTER: Validation Status = Approved] + TBH_Stats_… [BY: TBH_Workforce]`. Keep at Workforce only. Breakdowns: `WF_Stats_Headcount_Dep_Entity = WF_Stats_Headcount [BY: -> WF_Card_Department, WF_Card_Entity]`.

### Comp layer (`WF_Comp_*`, `TBH_Comp_*`)

Salary, bonus, benefits, taxes by Workforce (or TBH) × Version × Month; merit and tax assumptions; map to PnL/BS.

**PnL export pattern:** `(WF_Comp_01_Salary[BY: -> MAP_Comp_to_PnL] + WF_Comp_01_Bonus[BY: -> MAP_Comp_to_PnL] + …) [SELECT: 'Validation Status' = SET_Approved_Status] [PUSH_DH_View_Load_Plan]`. Use a mapping metric (`MAP_Comp_to_PnL`) instead of hard-coding account codes; use a Dimension-typed input metric for the status filter.

### KPI & Security layers

- **KPI:** counts (filled TBH per Dept/Entity; pending changelog per Employee). IFDEFINED(…) or IFBLANK(Changelog[BY: …][FILTER: …][REMOVE COUNT: Changelog], 0).
- **Security:** `ARM_*` (read/write by User, optionally Department/Entity). `ARC_*` (input flags for approval). `MAP_*` (merit cycle: editable only when cycle month matches and version not closed). ACCESSRIGHTS(TRUE, TRUE) / ACCESSRIGHTS(TRUE, BLANK) with IF(Admin, …, IF(ARC_Input, …, BLANK)). For access-rights fundamentals, see `skill:securing-with-access-rights`.

---

## Pitfalls

- **Do not skip layers.** Data → Card → Stats → Comp → Push/KPI. Jumping from Data to Stats bypasses overrides and card canonicalization.
- **Override-first belongs in staging only.** Cards and stats never reference Changelog or override metrics directly.
- **Keep WF_Stats and WF_Comp at Workforce only.** Use `BY: -> WF_Card_*` for breakdowns; do not add Entity or Department as structural dimensions to core metrics.
- **Naming discipline.** Consistent prefixes and suffixes prevent confusion as the model grows.
- **Version close.** All relevant metrics and access rights must respect version close.
- **Driver-based events.** Derive hires, terms, transfers from card comparison (SELECT: Month + 1), not manual event flags.
- **Headcount presence (0/1).** `PRORATA(Month, STARTOFMONTH(TBH_Req_Input_HireDate), STARTOFMONTH(TBH_Req_Input_TermDate + 1))` — yields 0 or 1 (present on last day of month); use TIMEDIM for date-period alignment.
- **Fractional FTE.** Use `PRORATA(Month, TBH_Req_Input_HireDate, TBH_Req_Input_TermDate + 1)` — raw dates, no `STARTOFMONTH`. Returns 0–1 fractional value. Do NOT use the STARTOFMONTH form when you need a fractional FTE.
- **Validation filtering.** All plan-facing metrics and exports filter to Validation Status = Approved.
- **Access rights by cycle/version.** Merit inputs editable only when Month of Year matches merit cycle and Version has no Close Date.
- **Do not store load/request attributes as list properties.** Employee dimension = ID + Name only (from the Roster load); Changelog and TBH Requests dimensions carry no properties beyond their default. Department, Entity, Hire Date, Term Date, Salary, FTE, Validation Status live in the Roster TL (for EE) or `TBH_Req_Input_*` metrics at TBH Requests × Version (for TBH); Changelog's own identity/timing/workflow/audit fields are `CL_*` metrics and its new values are `CL_Input_*` metrics. Even the Employee ↔ Workforce and TBH Requests ↔ Workforce links are metrics (`EE_Workforce`, `TBH_Workforce`), not properties. This keeps every attribute composing with formulas, access rights, and version scoping like the rest of the model.

---

**Related skills:** [Workforce Planning – Snapshot Spread Logic](./workforce_planning_snapshot_spread.md); [Workforce Planning – Changelog to Override Metrics](./workforce_planning_changelog_overrides.md); [Workforce Planning – Workforce Cards & BY-Arrow Mappings](./workforce_planning_cards_mapped_dimensions.md).
