---
name: modeling-pigment-applications
description: Always use this skill when designing or modifying Pigment applications. Covers core concepts, architectural decisions, governance standards and modeling rules, implementation of dimensions/metrics/transaction lists/tables/calendars, list subsets (when to use, data-loss risks, safe patterns), access rights design, application auditing, application cleaning, architecture design, and Test & Deploy safe modeling. This skill includes supporting files in this directory - explore as needed.
metadata:
  skill_path: /modeling-pigment-applications/SKILL.md
  base_directory: /modeling-pigment-applications
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

# Modeling Pigment Applications

This skill provides guidance for designing and architecting Pigment applications.

## When to Use This Skill

Read this skill when:

- Understanding **core modeling concepts** (blocks, metric and property types, metric vs transaction list, sparsity, etc.)
- Making **architectural decisions** (application structure, folder organization, data sharing)
- Applying **governance standards** (best practices, naming conventions)
- **Creating blocks and components** (dimensions, metrics, transaction lists, tables, calendars)
- Designing **Centralized Reporting Metrics** (final aggregation for P&L, Balance Sheet, reporting)
- Designing and applying **Access Rights** (AR metrics, rules, scope, Role vs User, debugging)
- **Auditing an app** (modeling, formula hygiene, folders, boards, governance, cleanup)
- **Cleaning an application** (deletion of unused objects, boards by usage)
- Designing a complete **Pigment architecture** (dimensional structure, UX, data, governance, planning cycle)
- Ensuring **Test & Deploy safety** (formula item references, dimension connectivity, deployment-safe modeling)
- **Creating or using List Subsets** (when to use vs filters/other list, data-loss risks, mirror dimensions, safe patterns)

**Always read this skill before designing or modifying Pigment application structure.**

---

## Modeling Workflow

### Step 1: Understand Intent

- [ ] Understand what you're building (dimension, metric, table, calendar, etc.)
- [ ] Clarify business purpose and usage
- [ ] Identify obvious dependencies (e.g., metrics need dimension lists first)
- [ ] Check if prerequisites exist (dimensions, properties, source metrics)

**If unclear** → Ask user for clarification

### Step 2: Search & Read Documentation

- [ ] Search this SKILL.md for relevant section
- [ ] Read documentation files listed
- [ ] Review applicable Pigment Modeling Best Practices rules
- [ ] Read Core Concepts if unfamiliar with fundamental concepts
- [ ] Check dimensional relationships for multi-dimensional structures

### Step 3: Model with CRUD in Mind

- [ ] **Choose the target folder** before creating the block: list existing Blocks folders, pick the folder that matches the block type and purpose (see [Working with Folders](./modeling_working_with_folders.md) – Placing new blocks). Never create in "No Folder".
- [ ] For **structural or risky changes** (e.g. rewriting a formula, changing metric dimensions, changing an existing board): consider a copy-first approach (duplicate metric or board, or create new property with " test" suffix). Offer the user the choice when the change is high-impact.
- [ ] Create prerequisite components first (e.g., dimension lists before metrics)
- [ ] Follow Pigment Modeling Best Practices standards
- [ ] Apply naming conventions from the Quick Naming Rules section below
- [ ] Validate against standards after creation

---

## Core Principle: Think CRUD

Always "think CRUD" - create prerequisites first:

- Dimension lists before metrics that use them
- Source metrics before calculated metrics that reference them
- Properties before formulas that reference them

---

## Quick Naming Rules (Apply When Creating Blocks)

**Apply these conventions when creating blocks.** If the application already uses
a different naming style (e.g. Snake_Case for dimensions), prefer consistency with
that style.

| Element              | Style                               | Example                                                          |
| -------------------- | ----------------------------------- | ---------------------------------------------------------------- |
| **Metric**           | Snake_Case with type prefix         | `CALC_Net_Revenue`, `INPUT_Budget_Amount`, `DATA_Employee_Count` |
| **Dimension**        | PascalCase, no prefix               | `Department`, `CostCenter`, `GLAccount`                          |
| **Transaction List** | Snake_Case with `LOAD_` prefix      | `LOAD_Sales_Orders`, `LOAD_Employee_Events`                      |
| **Property**         | Snake_Case                          | `Sort_Order`, `Account_Type`, `Is_Active`                        |
| **Folder**           | Numeric prefix `#.` or `##.`        | `0. Settings`, `1. Dimensions`, `10. Data Loads`                 |
| **Table**            | Snake_Case, optional `[TBL] ` prefix | `[TBL] Variance_Analysis`, `Cash_Flow_Summary`                 |

**Common Metric Prefixes:**

- `INPUT_` - User-entered data
- `CALC_` - Intermediate calculations
- `OUTPUT_` / `RES_` - Final results
- `DATA_` - Aggregated data from transaction lists
- `ASM_` - Assumptions
- `MAP_` - Lookups/mappings
- `PUSH_` / `PULL_` - Cross-app sharing

**Forbidden Characters:**

- **Never use periods `.`** - breaks formula references
- **Never use colons `:`** - breaks formula references

**Full conventions:** [./modeling_naming_conventions.md](./modeling_naming_conventions.md)

---

## Essential Foundation

### 1. Core Concepts (Read First)

**[./modeling_fundamentals.md](./modeling_fundamentals.md)** - Essential platform knowledge

**Covers**:

- Pigment as multidimensional engine (4 core principles)
- Dimensions vs properties (strategic usage, hierarchies)
- Metric and property types (Number/Date/Text/Dimension/Boolean)
- Metrics vs transaction lists (when to use each)
- Sparsity principles and performance implications
- IFDEFINED vs ISBLANK patterns

**Critical** - These are building blocks of everything in Pigment

### 2. Pigment Modeling Best Practices Standards and Principles

**[./modeling_principles.md](./modeling_principles.md)** - Complete governance guide

**Covers**:

- Folder structure (OX folders: Settings, Dimensions, Library, Assumptions)
- Data and themed folder organization
- Library folder for data sharing (Push/Pull patterns)
- Formula best practices
- **28 Modeling Rules** (MG: General, MS: Speed, MP: Posterity)

**Critical** - Ensures models are compliant, performant, maintainable

### 3. Naming Conventions

**[./modeling_naming_conventions.md](./modeling_naming_conventions.md)** - Comprehensive naming guide

**Covers**:

- Character rules and sort order
- Two-prefix system for metrics
- Conventions for all element types (applications, folders, dimensions, metrics, tables, boards, views)
- Anti-patterns to avoid

**Critical** - Consistent naming improves auditability and simplifies formulas

---

## Task-Based Routing

### 1. Understanding Platform Fundamentals (Start Here)

**Core Concepts**:

- "What are dimensions, transaction lists, properties, metrics, tables?"
- "What metric or property type should I use (Number/Date/Text/Dimension/Boolean)?"
- "When to use metric vs transaction list?"
- "How does sparsity work?"

**Read**: [./modeling_fundamentals.md](./modeling_fundamentals.md)

**Dimensions and Hierarchies**:

- "How to create hierarchies with dimension-type properties?"
- "Multi-level hierarchies (4+ levels)?"
- "Handle ragged or unbalanced hierarchies?"
- "Should this be a dimension or property?"
- "Add to metric structure or use property?"
- "Avoid dimension explosion?"

**Read**: [./modeling_dimensions_and_hierarchies.md](./modeling_dimensions_and_hierarchies.md)

### 2. Designing Application Structure

**Tasks**:

- "How to organize folders?"
- "What are OX folders?"
- "Single or multiple apps?"
- "What is Hub pattern?"
- "Share data between apps?"
- "What's Push vs Pull?"

**Read**: [./modeling_principles.md](./modeling_principles.md) - Sections 1, 2, and 6

### 3. Applying Governance Standards

**Naming Conventions**:

- "How to name applications?"
- "What prefix conventions for metrics?"
- "How to name folders?"
- "What characters are allowed in names?"
- "How to use the two-prefix system?"

**Read**: [./modeling_naming_conventions.md](./modeling_naming_conventions.md)

**Best Practices Rules**:

- "What are Pigment Modeling Best Practices rules?"
- "Formula formatting standards?"
- "Ensure maintainability?"

**Read**: [./modeling_principles.md](./modeling_principles.md) - Section 8 (28 Modeling Rules)

### 4. Implementing Dimensional Models

**Designing Dimensional Models**:

Planning metric structures, understanding multi-dimensional relationships, designing data flows, choosing aggregation/allocation patterns.

**Read**: [./modeling_dimensions_and_hierarchies.md](./modeling_dimensions_and_hierarchies.md) - Dimensional concepts and hierarchy patterns

**For modifier syntax**: See `../writing-pigment-formulas/formula_modifiers.md` - BY, ADD, REMOVE, SELECT, KEEP modifiers

**Planning Data Loading Strategy**:

- "Load into transaction list or metric?"
- "When to use dimensions vs transaction lists?"
- "Handle granular transactional data?"

**Read**: [./modeling_principles.md](./modeling_principles.md) - Section 5

Also see: [../integrating-pigment-data/data_import_csv.md](../integrating-pigment-data/data_import_csv.md)

**Setting Up Planning Cycles**:

- "Use native scenarios or version dimension?"
- "Calculate variance between scenarios?"
- "Set up Budget vs Actual?"

**Read**: [./modeling_scenarios_and_versions.md](./modeling_scenarios_and_versions.md)

**Setting Up Calendars and Time**:

**Read**: [./modeling_time_and_calendars.md](./modeling_time_and_calendars.md) - Calendar configuration and time dimensions

### 5. Performance Optimization

**Tasks**:

- "How to optimize model performance?"
- "Avoid slow calculations?"
- "Reduce memory usage?"

**Read**: [./modeling_performance_considerations.md](./modeling_performance_considerations.md)

Also see: [./modeling_principles.md](./modeling_principles.md) - Section 8 (MS rules for Speed)

### 6. Centralized Reporting Metrics

**Tasks**:

- "What is a Centralized Reporting Metric (Nexus)?"
- "How do I design a P&L or Balance Sheet aggregation metric?"
- "Where should the final reporting metric live?"
- "Push/Pull and Centralized Reporting Metrics?"

### 7. Access Rights (Design and Rules)

**Tasks**:

- "How do I set up Access Rights by Country/Region/Department?"
- "What's the difference between building an AR metric and applying it?"
- "Apply vs Ignore rules?"
- "Role-based vs user-based AR?"
- "Why can't this user see data?" / "Why can they see data they shouldn't?"

**Read**: [./modeling_access_rights.md](./modeling_access_rights.md)

For formula syntax (ACCESSRIGHTS, IFDEFINED): [../writing-pigment-formulas/functions_security.md](../writing-pigment-formulas/functions_security.md). For AR performance: [../optimizing-pigment-performance/performance_access_rights.md](../optimizing-pigment-performance/performance_access_rights.md).

### 8. Auditing a Pigment App

**Tasks**:

- "Audit this app for modeling and governance issues"
- "Review formulas, folders, and boards for best practices"
- "Find unused metrics, temporary metrics, or cleanup opportunities"
- "Check board size, naming, and folder structure"

**Read**: [./modeling_application_auditing.md](./modeling_application_auditing.md)

For detailed formula checks use writing-pigment-formulas; for performance use optimizing-pigment-performance; for boards use designing-pigment-boards.

### 9. Application Cleaning

**Tasks**:

- "Clean up unused metrics, dimensions, tables, or properties"
- "Identify and delete dead boards"
- "What is the deletion order for unused objects?"
- "How do I define 'unused' for a metric or dimension?"

**Read**: [./modeling_application_cleaning.md](./modeling_application_cleaning.md)

### 10. Architecture Design

**Tasks**:

- "Design a complete Pigment architecture from scratch"
- "How many applications do I need?"
- "What dimensions should go in the metric structure?"
- "How do I design for UX, data flow, governance, and planning cycles?"

**Read**: [./modeling_architecture_design.md](./modeling_architecture_design.md)

### 11. List Subsets (sublists)

**Tasks**:

- "Should I use a subset for X?" / "Create a subset of this list"
- "Same dimension twice" (e.g. Time + Cohort month, Company rows + columns)
- "Limit the list for input" / "Only show active items" / "Restrict dropdown"
- Performance: "Use a subset to speed up iterative calc?"

**Read**: [./modeling_subsets.md](./modeling_subsets.md) - When to recommend vs avoid, data-loss warnings, decision checklist, safe patterns (STORE/CALC, dropdown UX on parent, remap to parent). Tool: create_sublist exists; update/delete not yet exposed—design guidance applies regardless.

### 12. Test & Deploy (when used)

**Tasks**:

- "How do I make my formulas safe for Test & Deploy?"
- "Can I reference dimension items directly in a T&D context?"
- "Which dimensions are managed vs non managed?"
- "How to avoid deployment failures from item references?"

**Read**: [./modeling_principles.md](./modeling_principles.md) - Section 4 (baseline: no direct item reference) and Section 9 (When Test & Deploy is used: Rules 1 & 2, context, enforcement).

### 13. Segmentation (tiered / banded lookup)

**Tasks**:

- "Assign each item to a tier/band based on thresholds" (account segmentation, salary bands, discount brackets)
- "Avoid nested IFs or SWITCH for tiering when thresholds live on a dimension"
- "Lookup which band matches a value" (similar to Excel XMATCH/XLOOKUP match_mode -1 or 1)

**Read**: [./modeling_segmentation_tiered_lookup.md](./modeling_segmentation_tiered_lookup.md) - Pattern with IF + REMOVE FIRSTNONBLANK/LASTNONBLANK, floor/ceiling variants, dimension order, best practices.

For modifier syntax (REMOVE, FIRSTNONBLANK, LASTNONBLANK): [../writing-pigment-formulas/formula_modifiers.md](../writing-pigment-formulas/formula_modifiers.md).

---

## Moving to Formula Implementation

Once model designed, move to formula writing.

**Use writing-pigment-formulas skill for**:

- Writing and debugging formula syntax
- Detailed function documentation (CUMULATE, SHIFT, ITEM, etc.)
- Detailed modifier documentation (BY, ADD, REMOVE, SELECT, FILTER)
- Formula writing workflow
- Technical implementation of aggregation, allocation, transformation

---

## Quick Reference

| Topic                                    | File                                                                                                         |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Fundamentals                             | [modeling_fundamentals.md](./modeling_fundamentals.md)                                                       |
| Dimensions & Hierarchies                 | [modeling_dimensions_and_hierarchies.md](./modeling_dimensions_and_hierarchies.md)                           |
| Folder Structure                         | [modeling_principles.md](./modeling_principles.md) - Section 1                                               |
| Naming Conventions                       | [modeling_naming_conventions.md](./modeling_naming_conventions.md)                                           |
| Best Practices Rules (28)                | [modeling_principles.md](./modeling_principles.md) - Section 8                                               |
| Time & Calendars                         | [modeling_time_and_calendars.md](./modeling_time_and_calendars.md)                                           |
| Modifiers (syntax)                       | [formula_modifiers.md](../writing-pigment-formulas/formula_modifiers.md)                                     |
| Scenarios vs Versions                    | [modeling_scenarios_and_versions.md](./modeling_scenarios_and_versions.md)                                   |
| Performance                              | [modeling_performance_considerations.md](./modeling_performance_considerations.md)                           |
| Centralized Reporting Metrics            | See section 6 above                                                                                         |
| Access Rights (design, rules, debug)     | [modeling_access_rights.md](./modeling_access_rights.md)                                                     |
| App audit (modeling, UX, governance)     | [modeling_application_auditing.md](./modeling_application_auditing.md)                                       |
| Application cleaning (deletion workflow) | [modeling_application_cleaning.md](./modeling_application_cleaning.md)                                       |
| Architecture design (5 pillars)          | [modeling_architecture_design.md](./modeling_architecture_design.md)                                         |
| Test & Deploy (when used)               | [modeling_principles.md](./modeling_principles.md) - Section 9                                               |
| List Subsets (when to use, risks, patterns) | [modeling_subsets.md](./modeling_subsets.md)                                                             |
| Segmentation (tiered / banded lookup)   | [modeling_segmentation_tiered_lookup.md](./modeling_segmentation_tiered_lookup.md)                         |
| What the agent cannot do (UI only)      | [SKILL.md](../agent-capabilities-and-behavior/SKILL.md)                                                    |

---

## Documentation Files

- [./modeling_fundamentals.md](./modeling_fundamentals.md) - Platform fundamentals
- [./modeling_dimensions_and_hierarchies.md](./modeling_dimensions_and_hierarchies.md) - Dimensional modeling, hierarchies, structure decisions
- [./modeling_naming_conventions.md](./modeling_naming_conventions.md) - Comprehensive naming conventions for all Pigment elements
- [./modeling_principles.md](./modeling_principles.md) - Folder structure, best practices (28 rules)
- [./modeling_time_and_calendars.md](./modeling_time_and_calendars.md) - Calendar configuration and time dimensions
- [./modeling_scenarios_and_versions.md](./modeling_scenarios_and_versions.md) - Planning cycles
- [./modeling_performance_considerations.md](./modeling_performance_considerations.md) - Performance
- [./modeling_access_rights.md](./modeling_access_rights.md) - Access Rights design, Apply/Ignore rules, patterns, debugging, governance
- [./modeling_application_auditing.md](./modeling_application_auditing.md) - Audit app (modeling, formulas, folders, boards, governance, cleanup) and report with severity
- [./modeling_application_cleaning.md](./modeling_application_cleaning.md) - Deletion-only application cleaning, unused definitions, mandatory order, boards by usage
- [./modeling_architecture_design.md](./modeling_architecture_design.md) - Pigment architecture design across 5 pillars
- [./modeling_subsets.md](./modeling_subsets.md) - List subsets: when to use vs avoid, data-loss risks, decision checklist, safe patterns (STORE/CALC, dropdown on parent, remap to parent)
- [./modeling_segmentation_tiered_lookup.md](./modeling_segmentation_tiered_lookup.md) - Tiered/banded lookup: assign items to bands/tiers from thresholds on a dimension (IF + REMOVE FIRSTNONBLANK/LASTNONBLANK)
---

## Cross-References

**After Modeling**: writing-pigment-formulas (formulas), designing-pigment-boards (boards), integrating-pigment-data (data import), optimizing-pigment-performance (optimization)

---

## Critical Notes

- **Only dimension lists can be used in metric structure**
- **Start with understanding intent** - Clarify what you're building and why
- **Think CRUD** - Create prerequisites before dependent components
- **Follow Pigment Modeling Best Practices standards** - Reference [modeling_principles.md](./modeling_principles.md) regularly
- **Always apply naming conventions when creating blocks** - See Quick Naming Rules section above
- **Dimension vs property decision is critical** - Wrong choice causes rework
- **Type selection is critical** - Use Dimension for categorical/hierarchical, Number for quantitative, Date for temporal, Text for descriptive, Boolean for flags
- **Metric vs transaction list decision is critical** - Impacts performance and design
- **Centralized Reporting Metric = final aggregation for reporting** - Use one centralizing metric per financial view; keep detailed logic upstream and use Push/Pull for security
- **AR: Build != Apply** - Create AR metric then create Apply rule in Data access rights; use BLANK not FALSE; guard with IFDEFINED('Users roles', ...)
- **App audit** - Use modeling_application_auditing for full audits; delegate formula/perf details to writing-pigment-formulas and optimizing-pigment-performance; always validate cleanup with user
- **Architecture first** - Dimensional structure is the most critical decision; design architecture before building; challenge any metric structure with more than 5 dimensions
- **Test & Deploy safety** - See Modeling Principles section 4 (baseline: no direct item reference) and section 9 (when T&D is used: both rules, context, enforcement)
- **Folder placement** - Never create blocks in "No Folder". Always choose the appropriate folder from the application's Blocks folders (see [Working with Folders](./modeling_working_with_folders.md) – Placing new blocks); use block type and purpose (e.g. dimension → Dimensions folder, PUSH_/PULL_ → Library, DATA_ → Data Loads or themed Data) to decide.
- **Safe modeling** - For structural or risky changes (formula rewrite, metric structure, existing board), prefer copy-first: duplicate metric or board, or create new property with " test" suffix. Boards have no undo — when changing an existing board, ask if the user wants a copy first.
- **Tool coverage** - Not all Pigment features are available in the agent's tools (e.g. access rights config, permissions, automations, list duplicate, formula update). Check [What the agent cannot do](../agent-capabilities-and-behavior/SKILL.md) and direct the user to the UI when needed.
- **List Subsets** - Use only when they add clear value (mirror dimensions, targeted iterative perf, restricted dropdown with data on parent). Always warn about irreversible data loss on membership change and need for explicit parent remapping. Prefer filters or a regular list for "smaller list for analysis/input" unless a valid subset use case applies. See [modeling_subsets.md](./modeling_subsets.md).
