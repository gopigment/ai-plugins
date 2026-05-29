# View Components

In the **UI** we say **Values** (what appears in the cells) and **Pages / Rows / Columns**.

## Step 1: Data & Layout

### Values (cells)


That is what shows in the **cells**. It depends on the Block:

- **Metric Block**: The Metric itself
- **Table Block**: The Metrics within the Table
- **List Block**: The List properties

**Configuration options:**

- **`displayed`**: Controls whether the value is visible in the View (set to `true` to show)
- **Formatting**: Can specify number format, decimal places, currency, etc.
- **Order**: Multiple values can be displayed in a specific sequence

**Example:** For a Table containing "Revenue", "Cost", and "Profit" Metrics, you could display only "Revenue" and "Profit" by setting those value fields to `displayed: true`.

### Pivots (Rows, Columns & Pages)

Dimensions used to organize and break down data:

- **Rows**: Primary breakdown Dimension (e.g., Products, Departments, Accounts). Users read top-to-bottom for comparisons.
- **Columns**: Secondary breakdown, often time (e.g., Months, Quarters). Users read left-to-right for trends.
- **Pages**: Dimensions the user views one value at a time via a selector (e.g., Country, Year, Scenario, Version).

**CRITICAL — Same dimension on Pages and on Rows or Columns**

In Pigment, the **same dimension** may appear on **Pages** and on **Rows** or **Columns** at the same time. Example: **Month** on **Columns** and **Month > Year** (grouping) on **Pages**—year(s) chosen in the page selector determine which months are shown as columns. **Page selectors** (page filters) **filter which modalities** appear on the row/column axes according to the user’s selection (single- or multi-select depends on the page configuration). **Do not** ask the user to “resolve a conflict” when they request this; it is supported behavior, not a mistake.

When the goal is to **restrict** what appears on rows/columns, prefer **Pages** (with **Default items** on the relevant page selector) rather than substituting with View **Filter** objects that duplicate the same narrowing—unless you truly need a [view_filtering.md](./view_filtering.md) filter type.

**Examples: OK patterns vs. anti-patterns**

1. **Goal:** show **Country** in **rows** and focus the view on **France**.
   - **Anti-pattern:** **Country** only in **rows**, plus a View **Filter** in the spirit of _Keep — Country — is in — France_.
   - **OK pattern:** **Country** on **Pages** and **rows**; set **France** as the **Default item** on the Country page selector.

2. **Goal:** show **Country** in **rows** and only countries in region **EMEA**.
   - **Anti-pattern:** **Country** only in **rows**, plus a View **Filter** like _Keep — Country > Region — is in — EMEA_.
   - **OK pattern:** **Country** in **rows** and **Country > Region** (grouping) on **Pages**; set **EMEA** as the **Default item** on that page selector.

**Rows/Columns vs. Pages**: use this to decide what must appear together on the grid for comparison (**Rows** / **Columns**) versus what is driven by **Page** selectors. That guidance does **not** forbid putting the same dimension on **Pages** and on an axis when page selections should narrow the visible rows/columns—see the **CRITICAL** block above.

### Pivot Field Types

Pivot fields can have different types (kinds) depending on their configuration:

1. **Dimension Pivot**: A simple pivot on a Dimension
   - Has `dimensionId` only
   - Displays modalities directly from that Dimension

2. **Grouping Pivot**: Groups data by following List Properties to a target Dimension
   - Has both `dimensionId` AND `listPropertyPath`
   - Allows hierarchical grouping (e.g., Month → Quarter → Year)

3. **Scenario Pivot**: Special pivot for scenarios
   - Has no `dimensionId` (null)
   - Used for scenario selection

4. **Joined Pivot** aka Mapped Dimensions. Uses a mapping Metric to join data.
   - Has `dimensionId` and `mappingMetricId`

5. **Slice Pivot**, aka Data Slice: Uses a slice configuration
   - Has `dimensionId` and `sliceConfigurationId`

### How List Properties Work in Pivots (Grouping)

When you add a **List Property path** to a pivot field, it transforms from a simple Dimension pivot into a **Grouping pivot**. This allows you to aggregate data along Dimension hierarchies.

**Example Hierarchy:**

- You have a Metric defined on Dimension **Month**
- **Quarter** is a Dimension Property of List Month
- **Year** is a Dimension Property of List Quarter

**To group by Year:**

```json
{
  "dimensionId": "<Month's GUID>",
  "listPropertyPath": ["quarter", "_year"]
}
```

**What happens:**

1. Starts at the **source Dimension** (Month)
2. Follows the List Property path: Month → Quarter → Year
3. Groups data by the **target Dimension** (Year)
4. Displays aggregated values at the Year level

**Important notes:**

- **ListPropertyPath contains friendly names** of Dimension properties (the display names shown to users)
- Each step navigates one level in the Dimension hierarchy
- The source Dimension must exist and be valid for the View's underlying Block
- The List Property path must be valid (each Property must exist on the respective Dimensions)

**Configuration:**

```json
{
  "dimensionId": "8f301e67-dda4-4276-bc1b-4db418b8b3ff",
  "listPropertyPath": ["quarter", "_year"]
}
```

This creates a row/column that shows data grouped by Year, even though the underlying Metric is defined on Month.
