# Workforce Planning – Changelog to Override Metrics

## Purpose

How to model **change requests** (employee transfers, salary updates, term dates) as rows in a **Changelog dimension**, then project them into **override metrics** at planning grain (Version × Employee × Month). The Changelog dimension itself carries no properties beyond its default (label) property — every attribute (identity/target, timing, workflow, audit, and the new values themselves) is a metric. Staging uses **override-first** (IFDEFINED(Override, Override, Source)), so cards and stats consume a single consistent view regardless of whether data came from changelog or primary source (e.g. HRIS).

**When to use:** users request discrete changes to master data (new department, entity, job position, salary, term date) with an effective period; separating data collection/workflow from planning impact; governance needed (only approved changes apply, version close respected, inputs locked after submission).

**When not to use:** all master data edited directly on the source with no approval workflow; bulk imports without per-row effective dates or workflow (simpler override/replace logic may suffice).

### Discover blocks in this workspace

Template names are illustrative. Before creating blocks:

- Use `tool:search_metrics_and_lists` to find existing Worker, Employee, Scenario/Version, Department, and primary-source blocks; use `kind` or `regexp` when names are similar; use `tool:semantic_search` when names are unknown.
- Confirm freeze/validation and effective-date metrics on Scenario and Changelog (`CL_*` metrics, not list properties).
- Prefer extending existing lists and metrics over introducing duplicate blocks.

---

## Core Concepts

- **Changelog dimension**: identity only. One item = one change request. The list carries **no properties beyond its default** — every attribute below is a metric dimensioned by Changelog (some also by Version), never a list property.
- **`CL_*` identity/workflow metrics** (grain: Changelog only — one value per change request, does not vary by Version): `CL_Target_Entity` (Dimension-typed, e.g. → Employee), `CL_Target_Version` (Dimension-typed → Version), `CL_Active_As_Of` (the effective Month), `CL_Send_For_Validation` (Boolean), `CL_Validation_Status` (Dimension-typed → Validation Status), `CL_Created_On`, `CL_Created_By`.
- **`CL_Input_*` metrics** (grain: Changelog × Version): one metric per overridable attribute (e.g. `CL_Input_New_Department`, `CL_Input_New_Salary`, `CL_Input_New_Term_Date`). Holds the new value entered for that change request, for a given Version. Keeping these — and the `CL_*` metrics above — as metrics rather than list properties means every attribute inherits standard metric behavior (formatting, access rights, IFDEFINED/aggregation) consistently with the rest of the model.
- **Effective period**: when the change applies, from `CL_Active_As_Of` onward. Term date attributes may have different projection logic.
- **Override metrics**: at planning grain (Version × Entity × Month). Hold new value from `CL_Input_*` where a change applies, blank elsewhere. Built by projecting Changelog rows: filter approved, exclude after version close, ADD Month, EXCLUDE months before effective, BY LASTNONBLANK per entity for latest applicable change.
- **Projection**: turning dimension rows (events) into time-series overrides: extend each row over months from `CL_Active_As_Of` onward, pick latest change per entity per month.
- **Override-first staging**: IFDEFINED(Override_metric, Override_metric, Primary_source_logic). Primary source used only when no override.
- **Governance**: only approved rows contribute (`CL_Validation_Status` = Approved); rows after version close excluded (FIL_Changelog_Exclude_Version); inputs locked after `CL_Send_For_Validation` is TRUE; only certain roles set `CL_Validation_Status`.

---

## Pipeline

| Layer | Role | Typical content |
| --- | --- | --- |
| **Changelog dimension** | Change requests (identity only) | No properties beyond the list's default. One item per change request; everything else lives in metrics below. |
| **`CL_*` metrics** | Identity, timing, workflow, audit | `CL_Target_Entity`, `CL_Target_Version`, `CL_Active_As_Of`, `CL_Send_For_Validation`, `CL_Validation_Status`, `CL_Created_On`, `CL_Created_By` — all at Changelog grain (not Changelog × Version, since they don't vary by scenario). |
| **`CL_Input_*` metrics** | New-value inputs | One metric per overridable attribute (department, entity, job position, salary, term date, etc.), dimensioned by Changelog × Version. Entered directly on the change request; never a list property. |
| **Governance metrics** | Filter & security | Which rows are valid (approved, not after version close); who can edit vs approve; lock after submit. |
| **Override metrics** | Projection to planning grain | One metric per overridable attribute. Formula: `CL_Input_New_X` [FILTER: Approved] [EXCLUDE: after version close] [ADD: Month] [EXCLUDE: Month < Active as of] [BY LASTNONBLANK: Entity]. Result at Version × Entity × Month. |
| **Staging** | Override-first | IFDEFINED(Override, Override, Primary_source). Primary source = e.g. HRIS or spread logic. |
| **Cards** | Canonical attributes | Built from staging (and spread helpers if needed). Already contain overrides. |
| **Stats & aggregations** | Transfers, headcount, FTE, etc. | Use only cards (and derived metrics). They automatically respect overrides. |

---

## Patterns

### Changelog metrics structure

Each item = one change request. The Changelog dimension itself carries no properties beyond its default; every attribute is a metric:

- **Identity & target (Changelog grain):** `CL_Target_Entity` (Dimension-typed, e.g. → Employee), `CL_Target_Version` (Dimension-typed → Version)
- **Effective timing (Changelog grain):** `CL_Active_As_Of` (Month; optionally derive an effective date from the month's start)
- **Workflow (Changelog grain):** `CL_Send_For_Validation` (Boolean), `CL_Validation_Status` (Pending/Approved/Rejected)
- **Audit (Changelog grain):** `CL_Created_On`, `CL_Created_By` (optionally a "current state at creation" metric for context)
- **New values (Changelog × Version grain):** one `CL_Input_*` metric per overridable attribute (`CL_Input_New_Department`, `CL_Input_New_Entity`, `CL_Input_New_Job_Position`, `CL_Input_New_Salary`, `CL_Input_New_Term_Date`, etc.), entered directly on the change request for the relevant Version.

### Projecting changes to planning grain

For each (Version, Entity, Month), get the latest approved change effective in that month (`CL_Active_As_Of` ≤ Month).

**Generic pattern for "from month X onward" attributes** (entity, department, job position, salary):

```pigment
CL_Input_New_[Attribute]   // metric at Changelog x Version, not a Changelog property
  [FILTER: CL_Validation_Status = Validation_Status."Approved"]  // CL_Validation_Status is a metric, not a Changelog property
  [BY: -> CL_Target_Version]
    [EXCLUDE: FIL_Changelog_Exclude_Version]     // exclude rows created after version close
  [ADD: Month]
    [EXCLUDE: Month < CL_Active_As_Of]   // only from effective month onward
  [BY LASTNONBLANK ON CL_Created_On: CL_Target_Entity]   // or ON the Changelog item's own identity
```

- **FILTER:** only approved rows (`CL_Validation_Status` = Approved)
- **EXCLUDE FIL_Changelog_Exclude_Version:** TRUE when `CL_Created_On` > Version.'Close Date' (evaluated via `BY: -> CL_Target_Version`)
- **ADD Month then EXCLUDE Month < CL_Active_As_Of:** each change extended over months from effective month onward
- **BY LASTNONBLANK:** for each (Version, Entity, Month), keep latest change (by `CL_Created_On` or the Changelog item's own identity)

**Single-date attributes (e.g. term date):** use LASTNONBLANK by `CL_Created_On`/identity to pick latest approved term-date change per employee, then map to Month as needed (e.g. BY LASTNONBLANK on TIMEDIM(CL_Created_On, Month)).

### Override-first in staging

```pigment
IFDEFINED(
  Override_metric,      // e.g. EEO_Entity
  Override_metric,
  Primary_source_logic  // e.g. HRIS or spread-layer attribute
)
```

Cards and stats reference staging. They always see post-override values; no need to reference Changelog or override metrics directly.

> **IFDEFINED vs IFBLANK:** this pattern uses `IFDEFINED` because a defined override (even zero) should take precedence. For manual-input overrides where BLANK = "no override", `IFBLANK` is equally valid (see OPEX patterns in `skill:solving-financial-planning`).

### Governance

- **Which rows apply:** `CL_Validation_Status` = Approved. FIL_Changelog_Exclude_Version: exclude rows where `CL_Created_On` > Version.'Close Date'.
- **Lock after submit:** when `CL_Send_For_Validation` is TRUE, set write access to BLANK.
- **Who can approve:** separate metric (e.g. ARC_Allow_Validation_Write) grants write on `CL_Validation_Status` only to Admins or approval roles.
- **Monitoring:** KPI per entity (count of pending changelog rows) helps users see what awaits action.

Implement once; override and staging formulas reference the result (FILTER Approved, EXCLUDE FIL_Changelog_Exclude_Version), not the full workflow logic.

---

## Pitfalls

- **Do not duplicate governance in every formula.** Filter to approved and exclude-after-close once in override metrics.
- **Override-first in one place.** Staging is the only layer choosing override vs primary source.
- **Single-date vs from-month-onward.** Most attributes apply from month X onward; term date may need different projection (latest event per entity, then expose at Month grain).
- **Version close.** Exclude changelog rows created after version close so frozen versions are not modified by late entries.
- **Changelog has no custom properties.** Target entity/version, effective timing, workflow, validation status, and audit fields are all `CL_*` metrics at Changelog grain — the list itself carries nothing beyond its default property.
- **New values are Changelog × Version metrics.** Do not model New Department / New Salary / New Term Date etc. as Changelog properties or as Changelog-only metrics. They are `CL_Input_*` metrics at Changelog × Version, so they compose with formatting, access rights, and version scoping like the rest of the model.

---

**Related skills:** [Workforce Planning – Application Architecture & Patterns](./workforce_planning_architecture_patterns.md); [Workforce Planning – Workforce Cards & BY-Arrow Mappings](./workforce_planning_cards_mapped_dimensions.md); [Workforce Planning – Snapshot Spread Logic](./workforce_planning_snapshot_spread.md).
