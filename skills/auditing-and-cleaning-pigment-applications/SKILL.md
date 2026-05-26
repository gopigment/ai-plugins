---
name: auditing-and-cleaning-pigment-applications
description: Always use this skill when auditing a Pigment application (modeling, formula hygiene, folders, boards, governance) or cleaning it (deletion of unused dimensions, metrics, tables, properties, and boards by usage). Provides the audit vs cleaning mental model, the canonical workflow, severity definitions (HIGH / MEDIUM / LOW), and the strict deletion-only cleaning workflow with mandatory object order (Dimensions, Metrics, Tables, Properties) and machine-readable "unused" definitions.
metadata:
  skill_path: /auditing-and-cleaning-pigment-applications/SKILL.md
  base_directory: /auditing-and-cleaning-pigment-applications
  includes:
    - "*.md"
---

# Auditing and Cleaning Pigment Applications

Two related but distinct workflows on a Pigment application: **audit** surfaces findings with severity, **cleaning** deletes unused objects in a strict order. Read first to pick the right mode. Jump to the deep dive for procedures.

## When to Use This Skill

Read this skill when the user asks to:

- Audit an app for modeling, governance, or board issues
- Review formula hygiene, folder structure, board size, or naming
- Find unused metrics, dimensions, tables, properties, or boards
- Define what "unused" means for each object type
- Apply the correct deletion order (Dimensions, Metrics, Tables, Properties)
- Identify dead boards based on usage
- Produce a structured audit report with HIGH / MEDIUM / LOW severity
- Validate cleanup with the user before deleting anything

---

## Mental Model

Two modes, never mixed in the same pass:

- **AUDIT** (diagnostic, non-destructive)
  - Inputs: app structure, formulas, folders, boards, governance
  - Output: findings with severity (HIGH / MEDIUM / LOW) + proposed fixes
  - Never: deletes, renames, refactors
- **CLEANING** (execution, destructive, deletion-only)
  - Inputs: Pigment system truth (settings, dependency graph, usage analytics)
  - Output: deletions in strict order
  - Order: 1. Dimensions  2. Metrics  3. Tables  4. Properties  5. Boards
  - Loop: recompute usage after each pass; iterate
  - Never: renames, formula edits, folder reorganization

Invariants:

- **Audit is diagnostic.** It proposes; the user decides.
- **Cleaning is deletion only.** No refactor, no rename, no reorganization.
- **System truth over judgment.** "Unused" is read from Pigment settings, dependency graphs, and usage analytics, not from the agent's guess.
- **Order is mandatory.** Deleting Metrics before their dependent Tables (or Dimensions before Metrics that use them) breaks the model.
- **Iterate.** After each deletion pass, recompute usage; new objects become unused once their consumers are gone.

---

## Audit vs Cleaning: Pick the Right One

| Goal | Mode | Doc |
|---|---|---|
| Surface issues with severity, propose actionable improvements | **Audit** | [./auditing_application.md](./auditing_application.md) |
| Delete unused objects in the right order, no formula edits | **Cleaning** | [./cleaning_application.md](./cleaning_application.md) |

---

## Severity Definitions (Audit)

| Severity | Meaning | Examples |
|---|---|---|
| **HIGH** | Breaks a critical rule, blocks T&D, or causes data loss risk. Must be fixed. | Direct item references with T&D, List Subset misuse, missing AR Apply rule, naming with `.` or `:` |
| **MEDIUM** | Hurts performance, maintainability, or governance. Should be fixed. | Densifying formulas (ISBLANK over ISDEFINED), oversized boards, blocks in `No Folder`, inconsistent naming |
| **LOW** | Cosmetic, minor hygiene. Nice to fix. | Missing comments on non-trivial formulas, redundant intermediate metrics |

---

## Cleaning Workflow

1. **Compute usage.** Read Pigment system truth for every object type.
2. **Filter unused.** Apply the machine-readable "unused" definitions from `cleaning_application.md`.
3. **Validate with the user.** Show the deletion plan. Deletion is irreversible.
4. **Delete in order:** Dimensions, Metrics, Tables, Properties, then Boards (boards are independent but usage-driven).
5. **Recompute usage.** Items that became unused as a side effect of the previous pass should now appear.
6. **Iterate** until no new candidates appear.

---

## Glossary

- **Audit**: diagnostic pass that produces severity-tagged findings and proposed fixes; never destructive.
- **Cleaning**: destructive pass that deletes unused objects in a strict order; never edits or refactors.
- **Unused**: definition is per object type, machine-readable, derived from settings, dependency graph, and usage analytics.
- **Deletion order**: Dimensions, Metrics, Tables, Properties, Boards. Mandatory.
- **Dead board**: a board not opened by any user over the lookback window (definition in `cleaning_application.md`).
- **System truth**: Pigment's authoritative source (settings, dependency graph, usage analytics). Always preferred over modeled judgment.

---

## Critical Rules

- **Cleaning = Deletion.** No renaming, no formula rewriting, no folder reorganization in the cleaning phase.
- **System truth over human judgment.** "Unused" is decided from Pigment settings, dependency graphs, and usage analytics.
- **Deletion order is mandatory:** Dimensions, Metrics, Tables, Properties. Cleaning is iterative. After deletions, recompute usage before the next pass.
- **Always validate cleanup with the user** before executing irreversible deletions.
- **Always read the matching deep dive** before auditing or cleaning. Do not rely on this SKILL.md alone.

---

## Deeper Dives

| Need | Doc |
|---|---|
| Audit an app (modeling, formulas, folders, boards, governance, cleanup candidates) and report with severity | [./auditing_application.md](./auditing_application.md) |
| Strict deletion workflow for unused dimensions, metrics, tables, properties, and boards by usage. Mandatory order and machine-readable definitions | [./cleaning_application.md](./cleaning_application.md) |
| Modeling foundations (mental model, core concepts) | `skill:modeling-pigment-applications` |
| Formula hygiene (conditionals, sparsity, MP02) | `skill:writing-pigment-formulas` |
| Performance audit (profiler, scope, sparsity) | `skill:optimizing-pigment-performance` |
| Access Rights audit | `skill:securing-pigment-applications` |
| Board design rules | `skill:designing-pigment-boards` |
| Modeling principles, T&D safety | [`../modeling-pigment-applications/modeling_principles.md`](../modeling-pigment-applications/modeling_principles.md) |
| What the agent cannot do (UI-only operations) | `skill:agent-capabilities-and-behavior` |
