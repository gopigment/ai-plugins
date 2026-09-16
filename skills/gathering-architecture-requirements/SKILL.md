---
name: gathering-architecture-requirements
description: "Planning skill. Use before building anything to gather requirements through structured discovery questions. Covers dimensions, UX, data, governance, and planning cycles. Load this skill during Phase 0 of building-a-full-application."
metadata:
  skill_path: /skills/gathering-architecture-requirements/SKILL.md
  base_directory: /skills/gathering-architecture-requirements
---

# Gathering Architecture Requirements

Work through each area systematically, gathering requirements and making decisions before proceeding to the spec.

---

## Dimensions and Structure

- At what granularity will users input data for each business dimension?
- At what granularity will calculations occur?
- Do dimension item mappings change over time (employee moving cost centers)?
- What level drives user access rights?
- Are there organizational hierarchies with varying granularity needs?
- Which dimensions are required in the structure? Can any be replaced with properties or mapped dimensions?

---

## End User UX

- Can you provide wireframes or mockups of desired input screens?
- Can you share examples of current reports before Pigment?
- On which dimensions do users need to input?
- What are the key final reports (P&L, etc.)?
- What variations need to appear as columns in reports?
- What KPIs or ratios are required, and how are they calculated?
- Do certain calculations require separate metrics vs calculated items?
- What workflow steps exist, and do access rights need to change between steps?

---

## Data Sources and Cycle

- What are all data sources (ERPs, HR systems, CRM, etc.)?
- At what granularity is data available for each dimension?
- At what granularity do users need to plan?
- What is the gap between available data and planning granularity?
- Where will data transformation occur (before or within Pigment)?
- What metadata is available for each dimension?
- What historical data needs to be loaded, and is it transactional or aggregated?

---

## Governance and Security

- Which teams will manage which use cases?
- What data is sensitive and requires restricted access?
- Do any use cases involve individual-level data?
- Which dimensions are shared across multiple use cases?
- Are any dimensions created by business logic within a specific use case?
- Which users need access to which data?
- Do users need to navigate between applications?

---

## Planning Cycles and Versions

- What versions/scenarios are needed?
- How many live versions concurrently (max ~10 recommended)?
- How should completed plans be protected from changes?
- Do users need to replan based on historical budgets?
- When loading new data, should it impact historical plans?
- What variations need to appear as columns in reports?
- How will new planning cycles be initialized?
- Do transaction lists need effective dating?

---

## Validation Checklist

Before finalizing architecture, validate:

- Can the dimensional structure support all required inputs and calculations?
- Does the structure enable the desired UX?
- Does data granularity match structural requirements?
- Are governance and security requirements met?
- Is version management adequate for the planning cycle?
- Are there fewer than ~10 live versions?
- Have all dimensions been challenged for necessity?
