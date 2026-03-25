---
name: optimizing-pigment-performance
description: Always use this skill when troubleshooting slow calculations or timeouts, analyzing profiler output to identify bottlenecks, understanding scope propagation, managing sparsity, optimizing formula performance, improving iterative calculations, optimizing access rights performance, or conducting systematic performance audits. This skill includes supporting files in this directory - explore as needed.
metadata:
  skill_path: /optimizing-pigment-performance/SKILL.md
  base_directory: /optimizing-pigment-performance
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

# Optimizing Pigment Performance

This skill provides guidance for optimizing performance in Pigment applications.

## When to Use This Skill

- **Troubleshoot slow calculations** - Identifying and resolving timeouts
- **Analyze profiler output** - Understanding scope chips, computation chains
- **Optimize formulas** - Applying scope-first and reduce-first principles
- **Manage sparsity** - Avoiding densification with ISDEFINED vs ISBLANK
- **Optimize iterative calculations** - Improving PREVIOUS, CUMULATE performance
- **Improve access rights** - Optimizing AR-heavy formulas
- **Conduct audits** - Systematic bottleneck identification

---

## Performance Optimization Workflow

### Step 1: Use the Profiler First

- [ ] Open profiler and analyze actual bottleneck
- [ ] Identify scope chips (⚫ black / 🔵 blue / 🔘 gray)
- [ ] Note scope notation (2/2, 0/3, etc.)
- [ ] Trace computation chains
- [ ] Document findings before optimizing

### Step 2: Search & Read Documentation

- [ ] Identify issue type from profiler (scope loss, densification, iteration, etc.)
- [ ] Search this SKILL.md for relevant section
- [ ] Read documentation files listed

### Step 3: Optimize and Re-Profile

- [ ] Apply optimization patterns
- [ ] Re-profile to verify improvement
- [ ] Document before/after measurements

---

## Prerequisites

**From other skills:**

- **modeling-pigment-applications** - Core concepts, Pigment Modeling Best Practices standards, dimensional design
- **writing-pigment-formulas** - Formula syntax, modifiers, functions

**If unfamiliar** → Use those skills first

---

## Core Performance Principles

1. **Scope First** - Start formulas with scoping clauses
2. **Preserve Sparsity** - Use ISDEFINED instead of ISBLANK
3. **Reduce Early** - Aggregate or filter data early
4. **Profile Systematically** - Use profiler, not assumptions
5. **Understand Scope Propagation** - Know when and why scope is lost

---

## Essential Files

**[./performance_profiler_usage.md](./performance_profiler_usage.md)** - Complete profiler guide

**Covers**: Scope chips, computation chains, profiler interpretation, scope notation

**Always start with profiler** - Don't optimize based on assumptions

**[./performance_scoping_patterns.md](./performance_scoping_patterns.md)** - Scope mechanics

**Covers**: Scope propagation, early scoping strategies, preservation techniques

---

## Task-Based Routing

### Understanding Profiler Output

**Questions**:

- "What does the blue chip mean?"
- "Why is my scope showing 0/3?"
- "How do I trace slow computation?"

**Read**: [./performance_profiler_usage.md](./performance_profiler_usage.md)

**Quick Reference**:

- **⚫ Black Chip**: Scope preserved and passed downstream
- **🔵 Blue Chip**: New scope introduced (dimension added)
- **🔘 Gray Chip**: Computation triggered but no output change

### Optimizing Slow Formulas

**Questions**:

- "My formula takes 10 seconds, how to speed it up?"
- "Should I use IF or FILTER?"
- "How do I scope early?"

**Read in order**:

1. [./performance_formula_optimization.md](./performance_formula_optimization.md)
2. [./performance_scoping_patterns.md](./performance_scoping_patterns.md)
3. [./performance_sparsity_deep_dive.md](./performance_sparsity_deep_dive.md)

### Managing Sparsity

**Questions**:

- "Should I use ISBLANK or ISDEFINED?"
- "Why is my metric suddenly much larger?"
- "How do I avoid densifying metrics?"

**Read**: [./performance_sparsity_deep_dive.md](./performance_sparsity_deep_dive.md)

**Key principles**: Use **ISDEFINED** instead of **ISBLANK** to preserve sparsity. Use **dimension-typed metrics in BY** to drive sparsity. **Avoid ISBLANK/ISNOTBLANK for sparsity** — they densify over large spaces.

### Understanding Scope Loss

**Questions**:

- "Why did I lose scope after using REMOVE?"
- "How does CUMULATE affect scope?"
- "When is scope loss unavoidable?"

**Read**: [./performance_scoping_patterns.md](./performance_scoping_patterns.md)

### Optimizing Iterative Calculations

**Questions**:

- "My PREVIOUS calculation over 3 years of daily data is timing out"
- "How do I optimize CUMULATE?"
- "Should I use FILLFORWARD or PREVIOUS?"

**Read**: [./performance_iterative_calculations.md](./performance_iterative_calculations.md)

### Optimizing Access Rights Performance

**Questions**:

- "Why are my AR metrics so slow?"
- "What's the ISDEFINED(User) pattern?"

**Read**: [./performance_access_rights.md](./performance_access_rights.md)

### Conducting a Performance Audit

**Questions**:

- "Where do I start?"
- "How do I identify biggest bottlenecks?"
- "What's the systematic approach?"

**Read**: [./performance_troubleshooting_workflow.md](./performance_troubleshooting_workflow.md)

---

## Documentation Files

### Profiler and Scoping

- [./performance_profiler_usage.md](./performance_profiler_usage.md) - Profiler guide
- [./performance_scoping_patterns.md](./performance_scoping_patterns.md) - Scope propagation

### Sparsity and Formula Optimization

- [./performance_sparsity_deep_dive.md](./performance_sparsity_deep_dive.md) - ISDEFINED vs ISBLANK
- [./performance_formula_optimization.md](./performance_formula_optimization.md) - Reduce-first principle

### Specialized Optimization

- [./performance_iterative_calculations.md](./performance_iterative_calculations.md) - PREVIOUS, CUMULATE
- [./performance_access_rights.md](./performance_access_rights.md) - Access rights
- [./performance_calendar_considerations.md](./performance_calendar_considerations.md) - Time calculations

### Troubleshooting

- [./performance_troubleshooting_workflow.md](./performance_troubleshooting_workflow.md) - Systematic audit

---

## Quick Reference Tables

### Common Anti-Patterns

| Anti-Pattern                    | Why Bad                 | Fix                        |
| ------------------------------- | ----------------------- | -------------------------- |
| ISBLANK instead of ISDEFINED    | Densifies metrics       | Use ISDEFINED              |
| IF(ISBLANK(A), B, A)            | More complex            | Use IFBLANK(A, B)          |
| ISBLANK/ISNOTBLANK for sparsity | Densify over large space | Use BY + dimension-typed metrics or ISDEFINED/IFDEFINED/IFBLANK/EXCLUDE |
| Guarding BY with IF(ISBLANK(metric), BLANK, …) | Redundant, densifies | Use BY alone; dimension-typed metric in BY drives sparsity |
| Not scoping early               | Unnecessary computation | Add FILTER/EXCLUDE first   |
| Unnecessary REMOVE              | Loses scope             | Remove only when needed    |
| Long dense horizons in PREVIOUS | Exponential time        | Subset time dimensions     |
| No ISDEFINED(User) in AR        | Computes for all users  | Wrap AR in ISDEFINED(User) |

### Scope Chips

| Chip     | Meaning                          |
| -------- | -------------------------------- |
| ⚫ Black | Scope preserved                  |
| 🔵 Blue  | New scope introduced             |
| 🔘 Gray  | Computation but no output change |

### Scope Notation

| Notation | Meaning                                |
| -------- | -------------------------------------- |
| 2/2      | Full scope (all dimensions scoped)     |
| 2/3      | Partial scope (some dimensions scoped) |
| 0/3      | No scope (full recomputation required) |

### Performance Checklist

- [ ] Start formulas with scoping (FILTER, EXCLUDE, ISDEFINED)
- [ ] Use ISDEFINED instead of ISBLANK
- [ ] Use IFBLANK instead of IF(ISBLANK())
- [ ] Use BY with dimension-typed metrics for sparsity; avoid ISBLANK/ISNOTBLANK for sparsity
- [ ] Aggregate early with BY modifier
- [ ] Avoid unnecessary REMOVE
- [ ] Subset time dimensions for iterative calculations
- [ ] Add ISDEFINED(User) to AR-heavy formulas
- [ ] Profile before and after optimization

---

## Cross-References

**Related Skills**:

- **modeling-pigment-applications** - Pigment Modeling Best Practices MS rules, dimensional design
- **writing-pigment-formulas** - Formula syntax, function details

---

## Critical Notes

- **Always profile first** - Use profiler to identify actual bottlenecks, not assumptions
- **Measure before/after** - Document baseline and improvement
- **Use ISDEFINED, not ISBLANK** - Preserves sparsity
- **Scope early** - Start formulas with scoping clauses
- **Subset time dimensions** - For iterative calculations over long horizons
- **ISDEFINED(User) for AR** - Optimizes access rights formulas
- **Document profiler findings** - Explain what profiler showed and why optimization was needed
