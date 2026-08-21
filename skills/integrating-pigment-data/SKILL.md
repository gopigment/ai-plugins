---
name: integrating-pigment-data
description: Use this skill when integrating external data into Pigment - importing an attached CSV file, deciding whether to import into dimensions vs transaction lists, mapping source columns to properties, configuring cross-application (P2P) imports, or troubleshooting data imports. For the step-by-step CSV file import, read data_import_csv.md. Do NOT use this skill for formula updates or list creation unrelated to a data import. This skill includes supporting files in this directory - explore as needed.
metadata:
  skill_path: /skills/integrating-pigment-data/SKILL.md
  base_directory: /skills/integrating-pigment-data
  includes:
    - "*.md"
---


# Integrating Pigment Data

This skill provides guidance for importing external data into Pigment applications efficiently.

## When to Use This Skill

- **Create lists for CSV import** - Creating new dimensions or transaction lists that will receive CSV data
- **Import CSV files** - Loading data from CSV into dimensions or transaction lists
- **Map CSV columns** - Matching columns to properties using semantic matching
- **Decide import targets** - Choosing between dimensions and transaction lists
- **Configure P2P imports** - Moving data between Pigment applications
- **Optimize import performance** - Scoping and filtering strategies
- **Troubleshoot imports** - Resolving connector issues and data quality problems

## Import Workflow

### Step 1: Identify Data Type

- [ ] Determine if master data (entities like customers, products) or transactional data (events like orders, sales)
- [ ] Read the relevant documentation file (see Task-Based Routing below)

### Step 2: Decide Import Target

**Use Decision Framework:**

| Data Characteristic                           | Import To            | Reason                                  |
| --------------------------------------------- | -------------------- | --------------------------------------- |
| Master data (customers, products, employees)  | **Dimension**        | Relatively static, used as dimension    |
| Transactional data (orders, sales, movements) | **Transaction List** | High volume, time-stamped events        |
| Static entities with properties               | **Dimension**        | Need to maintain properties/hierarchies |
| Granular event-based data                     | **Transaction List** | Aggregate to metrics using formulas     |

### Step 3: Map Columns & Import

- [ ] Apply property type heuristics from [Property Type Selection](../modeling-pigment-applications/modeling_fundamentals.md#21-dimensions-and-properties)
- [ ] Map CSV columns to properties (semantic matching handles translations/abbreviations/synonyms)
- [ ] Create missing properties if needed
- [ ] Configure and execute import
- [ ] Validate results

---

## Prerequisites

**From modeling-pigment-applications skill:**

- Core Pigment concepts (dimensions, metrics, transaction lists, sparsity)
- When to use dimensions vs transaction lists

**If unfamiliar** → Use modeling-pigment-applications skill first

---

## Task-Based Routing

### Importing CSV Data

**🚨 CRITICAL: Before importing a CSV, read the CRITICAL RULES section in data_import_csv.md.**
It covers column analysis, dimension vs transaction list, column mapping (semantic matching), import scope
and post-import verification.

**Read**: [./data_import_csv.md](./data_import_csv.md)

### Understanding Integration Types

For the available integration types (CSV vs API vs native connectors vs P2P imports between applications):

**Read**: [./integration_overview.md](./integration_overview.md)

---

## Documentation Files

- **[./integration_overview.md](./integration_overview.md)** - Integration patterns and best practices
- **[./data_import_csv.md](./data_import_csv.md)** - CSV import to dimensions and decision framework

---

## Cross-References

**Before Integration**:

- **modeling-pigment-applications** - Dimensions, metrics, transaction lists

**After Integration**:

- **writing-pigment-formulas** - Aggregating transaction lists (BY modifier)
- **optimizing-pigment-performance** - Import performance optimization

---

## Critical Notes

- **Always determine data type first** - Master vs transactional drives all decisions
- **Use semantic matching** - Column names don't need to match exactly
- **Import to dimensions for master data** - Customers, products, employees
- **Import to transaction lists for events** - Orders, sales, movements
- **Validate after import** - Check data quality and completeness
- **Performance matters** - Large transaction lists need aggregation formulas
- **Document your decision** - Explain dimension vs transaction list choice
