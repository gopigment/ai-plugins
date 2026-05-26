---
name: optimizing-pigment-performance
description: Always use this skill when troubleshooting slow calculations or timeouts, analyzing profiler output to identify bottlenecks, understanding scope propagation, managing sparsity, optimizing formula performance, improving iterative calculations, optimizing access rights performance, or conducting systematic performance audits. Provides the optimization mental model (profile -> classify bottleneck -> apply pattern -> re-profile), the five core principles, an anti-pattern table, profiler chip and scope notation references, and the pre-delivery performance checklist. Always profile first; never optimize based on assumptions.
metadata:
  skill_path: /optimizing-pigment-performance/SKILL.md
  base_directory: /optimizing-pigment-performance
  includes:
    - "*.md"
---

# Optimizing Pigment Performance

Performance optimization for Pigment applications: profiler-driven, sparsity-aware, scope-conscious. Read first to get the mental model and pick the right deep dive.

## When to Use This Skill

- Slow calculations or timeouts
- Profiler analysis (chips, scope notation, computation chains)
- Densification or sparsity loss
- Iterative calculations (PREVIOUS, PREVIOUSOF, CUMULATE) over long horizons
- Access Rights-heavy formulas
- Systematic performance audit on an application

---

## Mental Model

Every optimization follows the same loop. Skipping the profile step is the most common failure mode.

1. **Profile** (mandatory). No optimization without a profiler reading.
2. **Classify the bottleneck.** One of: scope loss, sparsity densification, iterative horizon, AR overhead, calendar iteration, or formula shape (`IF` / `FILTER` / `BY`).
3. **Apply the right pattern.** Scope-first, BY over ADD, IFDEFINED over ISBLANK, etc.
4. **Re-profile, compare, document** the gain.

---

## Core Principles

1. **Scope First.** Start formulas with scoping clauses (FILTER, EXCLUDE, IFDEFINED).
2. **Preserve Sparsity.** Use ISDEFINED instead of ISBLANK. Use BLANK instead of 0 or FALSE.
3. **Reduce Early.** Aggregate or filter data before downstream operations.
4. **Profile Systematically.** Use the profiler, not assumptions. Measure before and after every change; document the delta.
5. **Understand Scope Propagation.** Know when and why scope is lost (REMOVE, CUMULATE, AR).

---

## Optimization Workflow

1. **Profile.** Identify chips (black / blue / gray), scope notation (3/3, 0/3), and the dominant computation chain.
2. **Classify the bottleneck.** Scope loss, densification, iterative horizon, AR overhead, calendar iteration, or formula shape.
3. **Pick the deep dive** from the routing table below.
4. **Apply the pattern.** Validate the formula syntax with `tool:validate_formula` when relevant.
5. **Re-profile.** Compare before / after. Document the gain.

---

## Bottleneck Routing

| Bottleneck signal | Read |
|---|---|
| Reading profiler output, chip colors, scope notation | [./performance_profiler_usage.md](./performance_profiler_usage.md) |
| Analyzing a profile response from the MCP tool (`GetChangeProfileResponse`) | [./performance_profiler_analysis.md](./performance_profiler_analysis.md) |
| Scope loss after REMOVE, CUMULATE, AR; scope propagation rules | [./performance_scoping_patterns.md](./performance_scoping_patterns.md) |
| Metric is much bigger than expected, ISBLANK / ISNOTBLANK in formulas | [./performance_sparsity_deep_dive.md](./performance_sparsity_deep_dive.md) |
| Formula shape (IF vs FILTER, REMOVE vs SELECT, BY vs ADD) | [./performance_formula_optimization.md](./performance_formula_optimization.md) |
| PREVIOUS, PREVIOUSOF, CUMULATE over long horizons | [./performance_iterative_calculations.md](./performance_iterative_calculations.md) |
| AR-heavy formulas, `ISDEFINED(User)` pattern | [./performance_access_rights.md](./performance_access_rights.md) |
| Calendar-driven iteration, time dimension granularity | [./performance_calendar_considerations.md](./performance_calendar_considerations.md) |
| Systematic audit, where to start | [./performance_troubleshooting_workflow.md](./performance_troubleshooting_workflow.md) |

---

## Anti-Patterns (Quick Reference)

| Anti-Pattern | Why it hurts | Fix |
|---|---|---|
| `ISBLANK` instead of `ISDEFINED` | Densifies the metric | Use `ISDEFINED` |
| `IF(ISBLANK(A), B, A)` | Verbose, densifies | Use `IFBLANK(A, B)` |
| `ISBLANK` / `ISNOTBLANK` for sparsity gates | Densifies over a large space | Use `BY` on a dimension-typed metric, or `ISDEFINED` / `IFDEFINED` / `IFBLANK` / `EXCLUDE` |
| `IF(ISBLANK(metric), BLANK, ...)` guarding a `BY` | Redundant, densifies | Use `BY` alone; the dimension-typed metric drives sparsity |
| No scoping at the start of the formula | Computes unnecessarily | Add `FILTER` or `EXCLUDE` first |
| Unnecessary `REMOVE` | Loses scope | Remove only when needed |
| Long dense horizons in `PREVIOUS` | Exponential time | Subset the time dimension |
| AR formula without `ISDEFINED(User)` guard | Computes for all users | Wrap AR in `ISDEFINED(User)` |

---

## Profiler Reference

### Scope Chips

| Chip | Meaning |
|---|---|
| Black | Scope preserved and passed downstream |
| Blue | New scope introduced (dimension added) |
| Gray | Computation triggered but no output change |

### Scope Notation

| Notation | Meaning |
|---|---|
| `3/3` | Full scope (all dimensions scoped) |
| `2/3` | Partial scope (some dimensions scoped) |
| `0/3` | No scope (full recomputation required) |

---

## Pre-Delivery Checklist

Every optimized formula must pass:

- [ ] Scoping clauses appear first (FILTER, EXCLUDE, IFDEFINED)
- [ ] Use ISDEFINED instead of ISBLANK
- [ ] Use IFBLANK instead of IF(ISBLANK())
- [ ] Use BY with dimension-typed metrics for sparsity; avoid ISBLANK / ISNOTBLANK for sparsity
- [ ] Aggregate early with BY
- [ ] Avoid unnecessary REMOVE
- [ ] Subset time dimensions for iterative calculations
- [ ] AR-heavy formulas wrapped in ISDEFINED(User)
- [ ] Profile before and after the change; document the delta

---

## Glossary

- **Scope**: the dimensional context in which a formula evaluates.
- **Scope chip**: profiler color marker (black / blue / gray) indicating scope behavior at a node.
- **Scope notation**: `X/Y` ratio of scoped vs total dimensions at a profiler node.
- **Densification**: turning a sparse metric into a fuller one (often by FALSE, 0, or ISBLANK / ISNOTBLANK).
- **Sparsity-first**: pattern that preserves blanks and never materializes 0 / FALSE cells.
- **Reduce early**: aggregate or filter as close to the source as possible.
- **AR overhead**: cost of evaluating Access Rights formulas; mitigated with `ISDEFINED(User)` and structural choices.

---

## Prerequisites

- **modeling-pigment-applications**: core concepts, dimensional design
- **writing-pigment-formulas**: formula syntax, modifiers, functions

If unfamiliar with these, read them first.

---

## Critical Rules

- **Always profile first.** Identify the actual bottleneck before changing anything.
- **Measure before and after.** Document baseline and improvement.
- **ISDEFINED over ISBLANK.** Preserves sparsity.
- **Scope early.** Start formulas with scoping clauses.
- **Subset time dimensions** for iterative calculations over long horizons.
- **ISDEFINED(User)** for AR-heavy formulas.
- **Document profiler findings.** Explain what the profiler showed and why the change was made.

---

## Cross-References

- **modeling-pigment-applications**: dimensional design, MS rules
- **writing-pigment-formulas**: formula syntax, function details, conditionals style
- **securing-pigment-applications**: AR formula patterns
