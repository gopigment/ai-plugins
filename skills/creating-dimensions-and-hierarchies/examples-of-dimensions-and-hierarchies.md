# Dimension & Hierarchy Examples

Knowledge extension for the `creating-dimensions-and-hierarchies` skill.

- **Part 1 — Tidbits of known  things to reproduce** apply to every dimension you create. Read them once per task.
- **Part 2 — Known dimensions** are the recurring business cases. Read only the ones your task matches.

---

## Part 1 — Tidbits of known  things to reproduce

- **A1 · Account numbers are Text.** Chart-of-accounts codes are identifiers, not quantities — store them in a **Text** property, never Number.
- **A2 · The leaf level carries one property per level.** In a hierarchy (dimensions linked by a Dimension-type property, i.e. a foreign key), the most granular dimension should hold a property for *every* level above it, so Page Filters are easy to use at any level. This applies only to that dimension's **MAIN** hierarchy.
- **A3 · Reuse the native calendar** — Day, Week, Month, Quarter, Year. The exception is a Week calendar whose definition differs from the one the user requires.
- **A4 · Never create a second Year dimension for a different fiscal year.** Add a Fiscal Year property on the native Year dimension instead.

---

## Part 2 — Known Dimensions

### B1 · Account / P&L Hierarchy

**When:** the chart of accounts — a backbone of FP&A use cases, running from ledger accounts up to high-level P&L roll-ups.

**Levels:** typically 2-3 levels of dimensions, sometimes far more. Common shapes:

- `L1 Ledger / L2 P&L Line / L3 P&L Summary / L4 P&L Roll Up`
- `A1 Account / A2 P&L Line`
- `P&L1 / P&L2 / P&L3 / P&L4`

**Leaf level:** the lowest level is the *leaf* — the one transactional data from the accounting software loads onto. It always carries a Dimension-type property for each level above it (see A2).

**P&L vs balance sheet:** both can arrive in the same data source. They can share one leaf dimension, but splitting them is better — and often required — at the parent levels.

**Ragged is fine here:** branches may differ in depth and Pigment handles it.

**Do not confuse** with the Account hierarchy of Sales use cases, which is the customer/prospect list.

Domain patterns: `skill:solving-financial-planning`.

### B2 · Organisational Hierarchy

**When:** the backbone of almost every use case, especially FP&A ones.

**Levels:** for FP&A, Cost Center is usually the most granular level, grouped by Department then Business Unit. Naming varies a lot between customers — take it from the user.

**Naming convention:** This hierarchy being the backbone of every metric, having a strong naming convention is important, with a Prefix O1, O2, O3. This adds ligebility in the formulas as well.

**Keep it balanced:** unlike B1, these should **not** be ragged. Inputs and access rights at aggregated levels are common, so every level needs a member for every branch.

**Critical:** these hierarchies typically bear access rights, challenge the user on that front if you see a risk with an unbalanced hierarchy

### B3 · Employee Dimension

**When:** the backbone of the workforce planning use case; typically created from the Employee Roster.

**One dimension, two populations:** keep existing employees and future hires (TBH, for To Be Hired) in the **same** Employee dimension. Separate them with Boolean flags plus Employee and TBH subsets (`tool:create_sublist` — read `skill:using-list-subsets` first, subset membership changes are data-destructive).

**Properties stay minimal** — the Employee dimension does not store employee data:

| Property | Type | Notes |
| --- | --- | --- |
| `Employee ID` | Text | The unique identifier. Create the dimension with this. |
| `Name` | Text | Pigment defaults `Name` to unique, but employee names are not unique — make it non-unique and identify items by `Employee ID`. |
| `Is Employee` | Boolean | Flags the existing-employee population. |
| `Is TBH` | Boolean | Flags the to-be-hired population. |
| `Display Name` | Text | |

Attributes such as department, salary and hire date belong on the Roster transaction list or in metrics, not here — see `skill:solving-workforce-planning`.

### B4 · Self-Referencing Dimensions

**When:** a dimension references itself through one of its own properties, creating a hierarchy inside a single list.

**Example:** Employee → Manager, where `Manager` is a Dimension-type property on Employee pointing back at Employee.

### B5 Flat self referencing hierarchies

**When** In Pigment, the user can only input on the lowest level of a hierarchy in one metric. In some business cases, the users will want to input at every level of the hierarchy in a simple way. 

**Solution** One solution for this is to create all items, including the L2 and LX categories into the leaf level of the dimension. 

Then, create self referencing properties for L2 and L3 to allow for page selectors filtering.


