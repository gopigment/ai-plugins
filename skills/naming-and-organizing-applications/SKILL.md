---
name: naming-and-organizing-applications
description: "Execution skill. Use when naming or organizing Pigment Applications, folders, and blocks so models stay navigable, formula-safe, and consistent across teams."
metadata:
  skill_path: /skills/naming-and-organizing-applications/SKILL.md
  base_directory: /skills/naming-and-organizing-applications
---

# Apply Naming and Folder Conventions Before Creating Blocks

Before adding any block, scan the target Application for existing naming patterns. Match existing style for consistency. Only introduce standard conventions on greenfield Applications or when the user explicitly requests cleanup.

## Choose the Three-Tier Name for Every Block

Every block has three name layers. Set all three deliberately:

| Layer | Purpose | Where it appears |
| --- | --- | --- |
| **Technical Name** | System identifier used in formulas, APIs, and Test & Deploy | Formula bar, `'Block Name'` references |
| **Friendly Name** | Default label shown in the UI when no Display Name is set | Block sidebar, Views, Boards |
| **Display Name** | Optional localizable label (EN/FR per member language) | Everywhere except formulas and uniqueness contexts |

Rules:

- Technical Name is the source of truth for formulas — keep it stable once referenced.
- Use Display Name for end-user readability without renaming the Technical Name.
- Dimensions appear as-is across Boards — keep Technical Names short and readable (PascalCase, no prefix).

## Never Use Periods or Colons in Block or Folder Names

Periods (`.`) and colons (`:`) break formula references and autocomplete. Application names may use a period in the sortable prefix (e.g. `00. Hub`) since applications are not referenced in formulas. Avoid ambiguous spaces inside names.

## Name Applications with a Sortable Prefix

Use a two-digit numeric prefix plus a descriptive title so Applications sort predictably in the Workspace:

| Pattern | Example | Use when |
| --- | --- | --- |
| `00.` prefix | `00. Hub` | Central shared-data Application (FX, dimensions, admin) |
| `01.`–`09.` | `01. Core Reporting` | Use-case Applications in logical process order |
| `ZZ_` prefix | `ZZ_POC Revenue Planning` | Retired or unused Applications kept temporarily |

Application titles use PascalCase or plain descriptive names — no metric-style prefixes.

## Name Folders with Numeric Prefixes for Sort Order

Pigment sorts folders numerically (`1.` before `10.`). Top-level: `N. Name`. Subfolders: `N.M Name`.

### Standard Top-Level Folder Structure

Use `tool:create_folder` to create folders. Place every block in an explicit folder using `tool:move_blocks`. **Never leave blocks in "No Folder".**

Organize folders by **functional area**, not by block type. Each folder groups all blocks (dimensions, metrics, transaction lists, tables) that belong to the same business process or domain topic.

- **`0. Settings`**: Configuration metrics, variables, mapping metrics, admin blocks
- **`1+` functional area folders**: All blocks for a specific business domain or process step

Example for a Workforce Planning Application:

- **`0. Settings`**: Admin variables, calendar config, mapping metrics
- **`1. Headcount`**: Employee dimension, `LOAD_Employee_Events`, `DATA_Employee_Count`, `CALC_FTE`
- **`2. Compensation`**: `INPUT_Salary_Grid`, `ASM_Raise_Rate`, `CALC_Total_Comp`
- **`3. Recruiting`**: `INPUT_Open_Positions`, `CALC_Hiring_Plan`, `OUTPUT_Recruiting_Budget`
- **`4. Reporting`**: `RES_PnL_Summary`, `[TBL] Headcount_Summary`, output metrics

Adapt numbering to complexity. Functional folders follow the business process: domain-specific steps → outputs.

## Block-Type Naming Rules

### Dimensions -- PascalCase, No Prefix

```
Department
CostCenter
GLAccount
Product
```

Keep names concise -- dimensions render directly on Boards and in pivot axes.

### Metrics -- Snake_Case with Type Prefix

Combine an optional process prefix (Prefix A) with a utilization prefix (Prefix B), separated by underscores:

```
INPUT_Budget_Amount
CALC_Net_Revenue
DATA_Employee_Count
ASM_Churn_Rate
MAP_Country_To_Region
PUSH_WF_Staff_Costs
PULL_Hub_FX_Rates
OUTPUT_Total_Revenue
RES_PnL_Summary
```

### Common Metric Prefixes

| Prefix | Meaning | Block role |
| --- | --- | --- |
| `INPUT_` | Manual end-user input | Assumptions, requests, overrides |
| `CALC_` | Intermediary calculation | Derived from other blocks |
| `OUTPUT_` / `RES_` | Final result | End outputs for reporting or sharing |
| `DATA_` | Aggregated transaction data | Simple transforms from transaction lists |
| `ASM_` | Assumption | Model parameters and drivers |
| `MAP_` | Mapping / allocation | Dimension or value mapping |
| `PUSH_` | Shared outward | Sanitized metric copy published to other Applications (see `skill:sharing-data-between-applications`; does not apply to shared dimension lists) |
| `PULL_` | Received inward | Reference to another Application's PUSH_ metric (see `skill:sharing-data-between-applications`; does not apply to shared dimension lists) |
| `LOAD_` | (Transaction lists only) | External data load target |
| `ADM_` / `SET_` | Admin / settings | Application configuration |
| `VAR_` | Variable | Application-level variables |
| `FIL_` | Filter | Table filter metrics |

Add process-specific Prefix A acronyms when the Application covers multiple domains (e.g., `EE_` for existing employees, `REV_` for revenue).

### Transaction Lists -- Snake_Case with LOAD_ Prefix

```
LOAD_Sales_Orders
LOAD_Employee_Events
LOAD_GL_Journal_Entries
```

Optionally append the source system: `LOAD_Employee_HRIS`.

### Properties -- Snake_Case, No Block Prefix

```
Sort_Order
Account_Type
Is_Active
External_ID
```

### Tables -- Snake_Case, Optional [TBL] Prefix

```
[TBL] Variance_Analysis
[TBL] PnL_Summary
Headcount_Summary
```

## Validate Names Before Saving

1. **No forbidden characters**: no `.` or `:` in Technical Name or folder name (Application sortable prefixes like `00.` are allowed).
2. **Correct case pattern**: PascalCase for dimensions; Snake_Case with prefix for metrics, transaction lists, properties, tables.
3. **Explicit folder**: block is placed in a numbered folder, not "No Folder". Use `tool:move_blocks`.
4. **Prefix reflects role**: metric prefix matches usage (input vs calc vs output vs load).
5. **Display Name set**: if end users need a friendlier label, use `tool:update_metric` or `tool:update_list` to set Display Name without changing the Technical Name.
6. **Consistent with Application**: if existing blocks use a different convention, follow the established pattern.
