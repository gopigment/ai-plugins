---
name: securing-with-access-rights
description: "Execution skill. Use when restricting which users can read or write specific data cells based on dimension values, building AR metrics, or debugging access issues. Do NOT use for managing Roles or Permissions."
metadata:
  skill_path: /skills/securing-with-access-rights/SKILL.md
  base_directory: /skills/securing-with-access-rights
---

# Securing with Access Rights

## Roles vs Permissions vs Access Rights

Keep these three concepts separate — they are managed by different tools and control different things:

| Concept | Controls | Managed by |
| --- | --- | --- |
| **Permissions** | Feature access — which *actions* a user can perform (create metrics, edit boards, import data, …) | `tool:list_permissions` (read-only) |
| **Roles** | A named bundle of granted permissions (+ default read/write access) assigned to users | `tool:list_roles`, `tool:create_role`, `tool:update_role`, `tool:delete_role`, `tool:assign_role_to_user` |
| **Access Rights (AR)** | Cell-level *data* read/write (e.g. user can write US data, read FR data) | AR metrics and rules — this skill |

**Golden rule:** Roles and Permissions protect *actions*; Access Rights protect *data*. Fine-grained data security is enforced by AR rules, **not** by roles alone. Managing Roles or Permissions themselves is out of scope for this skill.

---

## Access Rights Mental Model

Access rights control **which data cells** a member can read or write, based on dimension values.

Two layers:

1. **Default access rights**: assigned per Role (Read/Write defaults on the Roles page)
2. **Specific access rights**: custom AR metrics with Apply and Ignore rules in Data Access Rights settings

When multiple rules apply to the same Role x dimension combination, the **most restrictive** wins. BLANK in an AR metric is restrictive (No Read / No Write).

Access rights **inherit** along the metric dependency graph: if Metric B references Metric A, A's access rules flow to B unless cleared.

## AR Components

- **AR Metric**: Access Rights-typed metric dimensioned by Role (or User) x secured dimension. Formula returns read/write settings per combination.
- **Apply Rule**: Links an AR metric to target metrics, lists, or all metrics using specific dimensions.
- **Ignore Rule**: Exempts specific metrics or blocks from an otherwise broad Apply rule.

Building an AR metric and applying it are **two separate steps**. Never skip the Apply rule after creating the metric.

## Build an AR Metric Step by Step

### Step 1 -- Choose Role-based or User-based dimensioning

**Role-based (recommended):** Dimension by `Role` x secured dimension. Assign members to roles via Users roles. Scales cleanly.

**User-based:** Dimension by `User` x secured dimension. Use only when individual exceptions cannot be expressed through roles.

### Step 2 -- Create the AR metric

Use `tool:create_metric` with data type **Access Rights**, dimensioned by Role x the dimension to secure. Example: `[Role] x [Country]`.

### Step 3 -- Populate allowed combinations

Use `tool:update_metric`. Set **TRUE** (via ACCESSRIGHTS) only for allowed pairs; leave all others **BLANK** (never FALSE).

```pigment
ACCESSRIGHTS(
  IF('Role Country Mapping', TRUE, BLANK),
  IF('Role Country Mapping', TRUE, BLANK)
)
```

Read-only access:

```pigment
ACCESSRIGHTS(
  IF('Role Country Mapping', TRUE, BLANK),
  BLANK
)
```

**Critical:** BLANK means no access and is not stored (sparse). FALSE also denies access but is stored and degrades performance. Always prefer BLANK.

### Step 4 -- Wrap in IFDEFINED for performance

```pigment
IFDEFINED('Users roles', 'AR_Metric_Formula')
```

Users with no role produce BLANK, skipping expensive evaluation. Ensure administrators retain access:

```pigment
IF('Users roles' = SET_Admin_Role, ACCESSRIGHTS(TRUE, TRUE), 'Country_AR_Logic')
```

### Step 5 -- Create Apply rules

No agent tool exists for creating Apply/Ignore rules; ask the user to configure them in the Pigment UI (Application Settings -> Roles, permissions & access -> Data access rights):

1. Add a rule, set scope to **Apply**
2. Select the AR metric
3. Target: specific metrics, specific lists, or **all metrics using specific dimension(s)**
4. Enable the rule

Use **Ignore** rules to exempt metrics from the broad Apply rule.

## Formula Patterns for Access Control

- **`ACCESSRIGHTS(ReadBoolean, WriteBoolean)`**: Convert boolean flags into Access Rights values inside the AR metric.
- **`IFDEFINED('Users roles', ...)`**: Skip AR computation for users without role assignments.
- **`RESETACCESSRIGHTS(expression)`**: Clear inherited AR from a referenced block; essential in PULL metrics from source apps.

Apply RESETACCESSRIGHTS to the specific expression referencing the shared block:

```pigment
RESETACCESSRIGHTS('Hub App'::'Shared Revenue')
```

No agent tool exists for the RESETACCESSRIGHTS setting. Ask the user to enable "Use RESETACCESSRIGHTS() to remove inherited access rights for Blocks shared from other Applications" in Data access -> Manage inheritance.

## Multi-Application AR from a Hub

1. Define master AR metrics in the **Hub** (Role x Country, Role x Cost Center)
2. Share AR-related dimensions and mapping metrics via Libraries
3. In consuming apps, reference Hub AR metrics or PULL secured data
4. Apply RESETACCESSRIGHTS on PULL expressions to prevent unintended inheritance
5. Ask the user to create local Apply rules in each consuming app (no agent tool available)

Members need at least Read access in every application whose shared blocks they consume.

## Debug "Why Can/Cannot This User See Data?"

1. **Role assignment**: Confirm the user has a Role in Users roles for this application
2. **AR metric value**: Check the AR metric at Role x dimension item; must be non-BLANK with Read enabled
3. **Apply rule coverage**: Verify an enabled Apply rule targets the metric or dimension
4. **Ignore rules**: Check whether an Ignore rule exempts the target block
5. **Inheritance chain**: Dependency diagram with Show Access Rights mode is UI-only; ask the user to inspect. Trace referenced metrics for inherited restrictions
6. **Default vs specific**: Compare Role default access with specific rules; most restrictive wins
7. **Public blocks**: Blocks with Data visibility management set to Public override read restrictions (UI-only setting)

## Critical AR Rules

- **BLANK not FALSE** for denied access -- same security, better performance
- **IFDEFINED guard** on every AR metric formula referencing Users roles
- **Build then Apply**: creating the metric alone does nothing until an Apply rule exists (no agent tool; ask user)
- **Admin fallback**: always preserve administrator access when restricting defaults
- **List Item Values rules**: protect list property values, not metric data; apply metric-level rules for data security
- **Toggle rules**: disable rules temporarily during maintenance rather than deleting them

## Access Rights Anti-Patterns

Flag and fix these when auditing or building AR:

1. **AR applied at every step of a dependency chain**: AR should be applied **once at the end** of a calculation chain (on the output metric), not on every intermediate step. Applying AR at each step multiplies scheduled computation time.
2. **AR applied before aggregation or filtering**: Place AR **after** aggregation and filtering, not before. If AR restricts cells before aggregation, downstream totals may silently exclude data the user should see in aggregate form.
3. **Version/period-lock rules built on User instead of Role**: Time-lock and version-lock rules (which periods are editable, which versions are closed) should be dimensioned by **Role**, mapped via `'Users roles'`. Building them on User creates per-user maintenance overhead and breaks when users change roles.
4. **Overly complex multi-condition AR formulas**: When a single AR formula has 4+ nested IF/AND/OR conditions, split into **named sub-rule metrics** (e.g. `ARM_Country_Access`, `ARM_Period_Lock`, `ARM_Role_Write`) and combine them in one final AR metric. Easier to debug and maintain.
5. **One giant AR metric covering all dimensions**: Instead of a single AR metric dimensioned by Role x Country x Department x Version x Month, create **composable sub-rules** per concern (geographic access, period lock, role write). Apply each independently or combine at the end. A single mega-metric is hard to debug and expensive to compute.
6. **Business logic embedded directly inside `ACCESSRIGHTS(...)`**: Complex business rules (revenue thresholds, approval status, etc.) should be precomputed into **boolean metrics** outside the AR formula. The AR formula should reference those booleans, not re-derive them. Keeps AR formulas auditable and separates security from business logic.
7. **User x Time dimensioned AR when Role x Time suffices**: If every user in a Role has identical time-based access, dimension the AR by **Role x Time** (not User x Time). User-level time dimensions multiply computation by user count for no benefit.
8. **Security enforced only via Board visibility**: Hiding data on boards does not prevent access via Block Explorer, API, or other boards. Real data security requires **AR metrics with Apply rules**, not just board layout choices.

---

