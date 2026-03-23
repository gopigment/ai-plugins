---
name: integrating-pigment-data
description: Always use this skill when creating new lists to import CSV data into, importing CSV files into Pigment, mapping CSV columns to properties, deciding whether to import into dimensions vs transaction lists, configuring cross-application imports, troubleshooting data import issues. This skill includes supporting files in this directory - explore as needed.
metadata:
  skill_path: /integrating-pigment-data/SKILL.md
  base_directory: /integrating-pigment-data
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
- [ ] Search this SKILL.md for relevant section
- [ ] Read documentation files listed

### Step 2: Decide Import Target

**Use Decision Framework:**

| Data Characteristic                           | Import To            | Reason                                  |
| --------------------------------------------- | -------------------- | --------------------------------------- |
| Master data (customers, products, employees)  | **Dimension**        | Relatively static, used as dimension    |
| Transactional data (orders, sales, movements) | **Transaction List** | High volume, time-stamped events        |
| Static entities with properties               | **Dimension**        | Need to maintain properties/hierarchies |
| Granular event-based data                     | **Transaction List** | Aggregate to metrics using formulas     |

### Step 3: Map Columns & Import

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

**🚨 CRITICAL: Before importing CSV, read the CRITICAL RULES section in data_import_csv.md**

**Questions**:

- "Should I import this CSV into a dimension or transaction list?"
- "How do I map CSV columns to dimension properties?"
- "How do I handle semantic matching for column names?"
- "Should I create separate dimensions for columns with repeated values?"

**Read**: [./data_import_csv.md](./data_import_csv.md)

**Quick Decision**:

- Master data (customers, products, employees) → **Dimension**
- Transactional data (orders, sales, movements) → **Transaction List**

### Understanding Integration Types

**Questions**:

- "What integration types are available?"
- "When should I use CSV vs API vs connectors?"
- "What are P2P imports?"

**Read**: [./integration_overview.md](./integration_overview.md)

### Semantic Column Matching

**Scenario**: CSV columns don't exactly match property names

**Examples**:

- Translations: "Pays" → "Country"
- Abbreviations: "Emp" → "Employee"
- Synonyms: "Client" → "Customer"

**Read**: [./data_import_csv.md](./data_import_csv.md) - "Semantic Column Matching" section

**Key principle**: Match by meaning, not exact name

---

## Common Import Patterns

### Pattern 1: Master Data Import

**Scenario**: Importing customer, product, or employee master data

**Steps**:

- [ ] Import into **Dimension** (not transaction list)
- [ ] Map CSV columns to dimension properties
- [ ] Use semantic matching for column names
- [ ] Create properties if they don't exist

### Pattern 2: Transactional Data Import

**Scenario**: Importing orders, sales, or inventory movements

**Steps**:

- [ ] Import into **Transaction List** (not dimension)
- [ ] Each CSV row = one transaction
- [ ] Aggregate transaction list to metrics using formulas (BY modifier)

### Pattern 3: Semantic Column Matching

**Scenario**: CSV columns don't exactly match property names

**Approach**:

- Use semantic matching (meaning, not exact name)
- System handles translations, abbreviations, synonyms automatically
- Case-insensitive matching ("email" → "Email")

---

## Quick Reference Tables

### Decision Framework: Dimension vs Transaction List

| Use Dimension When                           | Use Transaction List When                     |
| -------------------------------------------- | --------------------------------------------- |
| Master data (customers, products, employees) | Transactional data (orders, sales, movements) |
| Relatively static data                       | High volume, time-stamped events              |
| Used as dimension in metrics                 | Need to aggregate to metrics                  |
| Need to maintain properties/hierarchies      | Granular event-based data                     |

### Semantic Matching Examples

| CSV Column   | Matches Property | Type                 |
| ------------ | ---------------- | -------------------- |
| "Pays"       | "Country"        | Translation (French) |
| "Emp"        | "Employee"       | Abbreviation         |
| "Client"     | "Customer"       | Synonym              |
| "email"      | "Email"          | Case variation       |
| "PRODUCT_ID" | "Product ID"     | Case + spacing       |

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
