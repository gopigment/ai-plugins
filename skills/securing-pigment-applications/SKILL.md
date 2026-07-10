---
name: securing-pigment-applications
description: Always use this skill when designing, applying, or debugging Access Rights and security in Pigment applications. Provides the AR mental model (User, Role, dimension axis, AR Metric, Apply rule), the canonical decision order, mandatory formula patterns (IFDEFINED guard, BLANK over FALSE), multi-app AR, debugging "why can this user see / not see this data?", and security governance. AR is part of model architecture, not an afterthought.
metadata:
  skill_path: /securing-pigment-applications/SKILL.md
  base_directory: /securing-pigment-applications
  includes:
    - "*.md"
---

# Securing Pigment Applications

Access Rights design, application, and debugging in Pigment. Read first to get the mental model and non-negotiable rules. Jump to the deep dive for procedures, formulas, and debugging recipes.

## When to Use This Skill

Read this skill when the user asks to:

- Design Access Rights by Country, Region, Department, Entity, or any dimension
- Decide between Role-based and User-based Access Rights
- Configure Apply vs Ignore rules in Data Access Rights
- Build an AR metric (Boolean by User x Dimension, or Role x Dimension)
- Write or debug AR formulas (`IFDEFINED('Users roles', ...)`, `ACCESSRIGHTS`)
- Share AR across multiple applications
- Debug "why can or cannot this user see this data?"
- Reuse AR patterns from a Hub app
- Working on any Role feature (Read, Create, Update, Delete)

---

## Mental Model

Access Rights in Pigment is a two-step pipeline. **Building the AR metric is not applying it.** You must do both.

1. **Identity axes** (reserved): `User` (app login identities) and `Role` (security roles).
2. **Business axis**: any dimension that drives access (Country, Entity, Department, ...).
3. **AR Metric** (Boolean, sparse): defined as `User x Business Dim` OR `Role x Business Dim`. Use BLANK for no access; never FALSE.
4. **Apply rule** in Data Access Rights: activates the AR Metric on dependent metrics. Without it, the metric is inert.
5. **Effective access**: what a user actually sees on a given metric, after all Apply rules evaluate.

Invariants:

- **User and Role are reserved security dimensions.** Never use them as business dimensions. Use `Employee` for business, `User` for app login identities.
- **AR Metrics are Boolean over a security axis x business axis**. They must be sparse (BLANK, not FALSE).
- **AR is not active until the Apply rule exists** in Data Access Rights.
- **AR is part of architecture**, designed alongside the dimensional model, not after.

---

## Decisions in Order

1. **Identify the axis.** Which dimension drives access (Country, Entity, Department, ...).
2. **Pick the security axis.** Role-based by default. User-based only when access is per individual and does not cluster.
3. **Build the AR Metric.** Boolean over `User x Dim` or `Role x Dim`. Use BLANK for no access; never FALSE.
4. **Guard the formula** with `IFDEFINED('Users roles', ...)` so it does not break when the User or Role context is missing.
5. **Create the Apply rule** in Data Access Rights. Without it, the metric is inert.
6. **Decide Apply vs Ignore** per dependent metric. Apply restricts; Ignore bypasses (rare, intentional).
7. **Multi-app.** Share AR from a Hub app via Push / Pull; do not re-implement per app.
8. **Validate.** Impersonate a user, check effective access, profile for AR-heavy formulas.

---

## Non-Negotiable Rules

1. **Use BLANK, never FALSE** in AR metrics. BLANK preserves sparsity. FALSE densifies and hurts performance.
2. **Always guard AR formulas with `IFDEFINED('Users roles', ...)`** so the formula does not break when the User or Role context is missing.
3. **Building an AR metric is not applying it.** You must create the metric and the Apply rule in Data Access Rights.
4. **User and Role dimensions are reserved for security.** Never use User as a business dimension (use Employee). Never reuse Role for business roles.
5. **AR is part of architecture.** Design AR alongside the dimensional model, not after.

For details, examples, and debugging, read [./securing_access_rights.md](./securing_access_rights.md).

---

## Managing Roles (`list_roles`, `create_role`, `update_role`, `delete_role`)

When these tools are available, the agent can read and manage an application's security **Roles** directly. A Role carries a `permissions` object (what the role can do), an `accessRights` object, and optional `permissionsByListId` / `readWriteAssignmentByScenarioId` assignments.

**To duplicate a role** (e.g. "create `Reader_2` with the same permissions as `Reader`"):

1. Call `list_roles` for the application. Each role returns `modalityId`, `friendlyName`, `permissions`, `accessRights`, `permissionsByListId`, `readWriteAssignmentByScenarioId`.
2. Find the source role (e.g. `Reader`) in that output.
3. Call `create_role` with the new `friendlyName` and the source role's `permissions`, `accessRights`, `permissionsByListId`, and `readWriteAssignmentByScenarioId` **copied verbatim** from step 1.

**Critical — use the exact fields `list_roles` returns.** Copy the `permissions` object as-is from `list_roles`. Do NOT invent permission flags, and do NOT borrow fields from `list_permissions` or `list_access_rights` — those describe Access Rights configuration, which is a different concept from a role's `permissions`.

**Other operations:**
- `update_role` — send the role's full desired state, including its `modalityId`.
- `delete_role` — send the role id. Destructive: users with that role lose its permissions.

**Constraints:** role management works only on **non-admin** applications, and `friendlyName` must be unique within the application.

---

## Glossary

- **User dimension**: reserved system dimension holding application login identities.
- **Role dimension**: reserved system dimension holding security roles. Items are grouped via the `Users roles` mapping.
- **AR Metric**: Boolean metric over a security axis (User or Role) x a business axis. Sparse, BLANK = no access.
- **Apply rule**: configuration in Data Access Rights that activates an AR Metric on dependent metrics.
- **Apply vs Ignore**: per-metric decision. Apply restricts data through AR; Ignore bypasses AR (use sparingly).
- **Effective access**: the resulting set of cells a user can read or write after all Apply rules are evaluated.
- **Hub AR**: AR built once in a Hub app and pushed to domain apps. The only safe pattern for multi-app security.

---

## Critical Rules (quick check)

- **Always read [./securing_access_rights.md](./securing_access_rights.md) before building or debugging Access Rights.** Do not rely on this SKILL.md summary alone.
- **Build != Apply.** Always create the rule in Data Access Rights after building the AR metric.
- **BLANK over FALSE.** Always.
- **IFDEFINED guard.** Always wrap AR formulas referencing `'Users roles'` or `User`.
- **User dimension is for application users, not employees.**

---

## Deeper Dives

| Need | Doc |
|---|---|
| AR design, Apply vs Ignore, User/Role/Dimension patterns, mandatory formulas, multi-app AR, debugging, governance | [./securing_access_rights.md](./securing_access_rights.md) |
| Formula syntax for security (ACCESSRIGHTS, IFDEFINED) | [`../writing-pigment-formulas/functions_security.md`](../writing-pigment-formulas/functions_security.md) |
| AR performance (ISDEFINED(User), AR-heavy formulas) | [`../optimizing-pigment-performance/performance_access_rights.md`](../optimizing-pigment-performance/performance_access_rights.md) |
| Modeling foundations (mental model, core concepts) | `skill:modeling-pigment-applications` |
| Modeling principles, T&D safety | [`../modeling-pigment-applications/modeling_principles.md`](../modeling-pigment-applications/modeling_principles.md) |
| Architecture (Hub pattern, AR in Hub) | [`../modeling-pigment-applications/modeling_architecture_design.md`](../modeling-pigment-applications/modeling_architecture_design.md) |
