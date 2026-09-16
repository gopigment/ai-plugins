---
name: solving-workforce-planning
description: "Planning skill. Workforce planning use cases: headcount tracking, hiring pipeline, compensation planning, employee transfers, HRIS integration. Each use case has inline guidance; load sub-files only for advanced patterns."
metadata:
  skill_path: /skills/solving-workforce-planning/SKILL.md
  base_directory: /skills/solving-workforce-planning
  includes:
    - "*.md"
---

# Workforce Planning

> Prerequisites: Version dimension exists (`skill:building-versions-and-planning-cycles`). Calendar configured (`skill:setting-up-calendar`).

---

## Headcount & FTE Tracking

**When:** track existing employees, count headcount and FTE by department, plan simple hires.

**Dimensions:** Version, Month, Employee (or Position list), Department. Add Entity if multi-entity. Employee dimension stays lean: Employee ID (created when the Roster load creates missing items) and Name (pulled from the Roster TL). Department, Entity, Hire Date, Term Date, Salary, FTE live as properties on the Roster transaction list (dimensioned by Employee), not on the Employee dimension itself.

**Structure:** Headcount metric based on Roster TL hire/term dates and Month comparison. FTE derived from headcount × FTE ratio (use PRORATA for partial months). Aggregate by Department via view grouping or REMOVE.

**Forecasting:** planned hires = new Employee items with future Hire Dates. Salary increases = growth rate metric or direct input per Version. No TBH dimension, no card/stats layers.

**Reporting:** Department (rows) × Month (columns). Department and Entity are structural dimensions when tracking a single population (employees only); no BY-arrow mappings needed. **If planned hires exist in a separate list, go to the Hiring Pipeline section below** — the combined model needs a unifying dimension and BY-arrow card metrics.

---

## Hiring Pipeline (To-Be-Hired)

**When:** separate hiring requests from existing employees, **whenever existing employees and planned hires must be reported as one total**, or with validation workflows.

**Dimensions:** adds TBH Requests list alongside the Employee list. TBH Requests carries no properties beyond its default — Department, Entity, Hire Date, FTE, Salary, and Validation Status are `TBH_Req_Input_*` metrics dimensioned by TBH Requests × Version, and even its Workforce mapping is a metric (`TBH_Workforce`, at TBH Requests grain), never a list property. Workforce dimension unifies both populations.

**Structure:** TBH requests go through a validation workflow (Draft → Submitted → Approved). Only approved requests feed planning metrics. Existing employees and TBH are lifted into a single Workforce dimension via card metrics.

**Consolidation pattern:** `WF_Stats_Headcount = EE_Stats_Headcount[BY: EE_Workforce] + TBH_Stats_Headcount[BY: TBH_Workforce]`. `EE_Workforce` and `TBH_Workforce` are metrics (Employee grain and TBH Requests grain respectively) mapping each item to its Workforce entry — not list properties. This requires a Workforce dimension that unifies both populations.

**Reporting:** unified headcount and FTE across existing + planned hires. Breakdowns by Department or Entity use BY-arrow mappings from card metrics, not structural dimensions.

**Extend to full architecture when:** the model needs all five layers (Data → Card → Stats → Comp → Push/KPI) with naming conventions and override-first staging. Load [Architecture & Patterns](./workforce_planning_architecture_patterns.md) + [Cards & BY-Arrow Mappings](./workforce_planning_cards_mapped_dimensions.md).

---

## Compensation Planning

**When:** plan salaries, bonuses, benefits, taxes, and total cost per employee or position.

**Simple approach:** Monthly salary = annual salary / 12 × headcount. Total cost = salary × (1 + benefits rate). Keep in one or two metrics; no comp layer needed.

**Advanced approach:** multi-component compensation (base + bonus + benefits + taxes + merit cycles). Each component in a separate metric. Merit assumptions by grade or position. Map to P&L accounts for financial reporting.

**Extend to full comp layers when:** 4+ compensation components, merit cycle governance, or PnL/BS mapping required. Load [Architecture & Patterns](./workforce_planning_architecture_patterns.md).

---

## Employee Transfers & Overrides

**When:** users submit change requests (department transfer, salary update, term date change) with effective dates and approval workflows.

**Simple approach:** direct edits on metrics. No changelog dimension needed.

**Advanced approach:** Changelog dimension carries no properties beyond its default — each item is one change request, and every attribute (target employee, effective date, workflow, validation status, audit) is a `CL_*` metric at Changelog grain; the new values themselves (new department, new salary, new term date, etc.) are `CL_Input_*` metrics dimensioned by Changelog × Version. Changes are projected into override metrics at planning grain. Staging uses override-first logic (IFDEFINED: if override exists use it, otherwise use source).

**Extend to changelog pattern when:** formal change-request workflow with approval, effective dates, and version-close governance. Load [Changelog to Override Metrics](./workforce_planning_changelog_overrides.md).

---

## HRIS Integration (Snapshot Data)

**When:** source data comes from periodic HRIS exports (one snapshot per load date) rather than clean monthly data.

**Problem:** HRIS provides sparse snapshots (e.g. one export per quarter), but planning needs a dense grid (every employee × every month). Gaps between snapshots must be filled.

**Structure:** spread layer selects which snapshot applies to each (Version, Employee, Month), propagates attributes forward with FILLFORWARD, and toggles between history and plan behavior. All downstream metrics reference the spread layer, never the raw snapshots (e.g. the Roster TL) directly.

**Skip this when:** source data is already at monthly grain or loaded cleanly per employee per month.

**Load pattern:** [Snapshot Spread Logic](./workforce_planning_snapshot_spread.md).
