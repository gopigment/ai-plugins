---
name: creating-and-editing-pigment-views
description: Always use this skill when creating or editing Views, or needing to pick a View.
metadata:
  skill_path: /creating-and-editing-pigment-views/SKILL.md
  base_directory: /creating-and-editing-pigment-views
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

---

# CRITICAL RULES

- **Never edit a View directly.** Always work with Draft Views (see next section).
- **Never merge a Draft View yourself** — leave it for the user to review and merge.
- When editing a View in the context of a widget on a Board, you must:
  - use the **draft + override workflow** to allow safe, user-specific preview before committing changes that affect all users.
  - read: [view_widgets.md](../designing-pigment-boards/view_widgets.md).

# Definitions

## Views

Views are configurations that specify _how_ to display data from a Block (Metric, List, or Table) — including pivots, filters, sorting, and formatting. A View references a single Block, but the same Block can have multiple Views showing the data differently. Think of a View as a "lens" through which users see Block data.

## Draft Views

A **Draft View** is a private working copy of a View. It is persisted in the database, but it is not visible to other users — they cannot discover it unless they have a direct link to it. A Draft only becomes visible to everyone when it is **merged** back into the original View or **saved as a new View**. This means you can freely modify pivots, filters, sorting, etc. on the Draft without affecting anyone.

- **Creating a new View** → always create it as a Draft View
- **Editing an existing View** → create a Draft from the original, then edit the Draft
- **Never merge** — leave the Draft for the user to review, confirm, and merge themselves

---

# View Design Process

## Step 1: Define your ideal View

Before looking at existing Views, clarify what you need:

1. What data should be shown? (which Metric, List, or Table?)
2. How should it be broken down? (which Dimensions for rows/columns/pages?)
3. What filtering is needed? (default items, filters?)
4. What sorting is needed?

Refer to the "View Components" section below for detailed guidance on value fields, pivots, filtering, and sorting.

### Display Modes

See [display_modes.md](./display_modes.md) for display mode definitions and block type rules.

## Step 2: Search for existing Views

Look for existing Views on the Block that match your intent. Must read: [relevant_views.md](../designing-pigment-boards/relevant_views.md). Use `get_block_views` to retrieve and evaluate candidates.

## Step 3: Decide: reuse, edit, or create

Compare the best existing View against your ideal design from step 1:

- **Matches your design** → use it as-is
- **Close but needs changes** → create a Draft from the existing View, then edit the Draft to match your ideal (pivots to add/remove, filters to adjust, sorting to change)
- **No good match** → create a new Draft View following your ideal design

In all cases where you create or edit, work exclusively with Draft Views. Never merge — leave the Draft for the user to review.

View creation and modification uses an **incremental approach**: create a basic Draft View first, then configure it step-by-step through multiple operations until it matches your design.

## Step 4: Validate the returned View

After creating or editing a Draft View, check that the returned object matches what you sent in the request. If fields are missing or different, the backend performed silent sanitization (e.g., invalid pivots or filters were dropped).

If the tool returns an error or the response doesn't match your request, refer to [troubleshooting.md](./troubleshooting.md).

---

# View Components

## Step 1: Data & Layout

### Value Fields

**CRITICAL REQUIREMENT**: There must be at least one value field in the View configuration.

Value fields determine what data is displayed in the View cells. What counts as a value field depends on the Block type:

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

**Rows/Columns vs. Pages**: ask yourself what data needs to be seen together on the same screen for coherence. Dimensions whose modalities (individual values) the user compares simultaneously belong in Rows or Columns. Dimensions whose modalities the user views one at a time belong in Pages.

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

4. **Joined Pivot**: Uses a mapping Metric to join data
   - Has `dimensionId` and `mappingMetricId`

5. **Slice Pivot**: Uses a slice configuration
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

## Step 2: Filtering

When a View shows more data than needed, you can narrow it down with **Pages** (simple inclusion) or **Filters** (more complex conditions).

### Pages (Default Items)

The simplest way to reduce what's shown: configure **Default Items** on a Page to restrict which data is displayed. For example, setting Default Items to "Q1 2024" on a Quarter Page means only Q1 data is displayed — no filter configuration needed.

Grouping pivots can also be used in Pages. Selecting a value on a Grouping Page filters base items (e.g., Months) whose property chain matches the selected target (e.g., Year = FY 2024 → only Months with `_year` = FY 2024).

### Filters

For more control than Pages, use Filters. There are several types:

**CRITICAL REQUIREMENT**: The `pivotFieldId` in filters MUST reference a pivot from the **rows or columns** arrays, NOT from the pages array.

### PivotField Filters (By Items)

Used to exclude Dimension items based on which modalities are present in a pivot.

**Note**: To filter on specific items (inclusion), prefer using **Pages** instead. Use PivotField Filters only when you need to exclude items or apply complex filtering logic.

**Configuration:**

```json
{
  "type": "PivotField",
  "pivotFieldFilteringOption": {
    "pivotFieldId": "<pivot-field-id>", // MUST be from rows or columns, NOT pages!
    "compareOperator": "IsIn",
    "modalityIds": ["<modality-id1>", "<modality-id2>"],
    "variableIds": []
  }
}
```

### ValueField Filters (By Value)

Filter rows/columns based on Metric values (e.g., "show only products where Revenue > 1000").

**CRITICAL REQUIREMENT**: When there are pivots on the **opposite axis** from the filtered pivot, you **MUST provide projections** for each of those pivots.

**Why projections are needed:**

- When filtering rows by value, but columns exist, you need to specify WHICH column value to use for comparison
- Example: If filtering "Product" rows by "Revenue > 1000" and "Month" is in columns, you must specify which month (e.g., "January 2024") to use for the comparison
- Without valid projections, the filter will be silently removed during View creation

**Configuration:**

```json
{
  "type": "ValueField",
  "valueFieldFilteringOption": {
    "pivotFieldId": "<pivot-field-id-being-filtered>", // Must be innermost pivot on its axis
    "valueFieldId": "<value-field-id>",
    "compareOperator": "Gt",
    "values": ["1000"],
    "variableIds": [],
    "projections": [
      // REQUIRED if opposite axis has pivots
      {
        "pivotFieldId": "<opposite-axis-pivot-id>",
        "modalityId": "<modality-id>" // Which modality to use for comparison
      }
    ]
  }
}
```

**Example scenario:**

- Rows: Product Dimension
- Columns: Month Dimension
- Want to filter: "Show only products where Quantity Sold > 100"
- You MUST specify which month to use for comparison (e.g., the first month modality)

### PivotListProperty Filters

Filter based on List Property values.

**Configuration:**

```json
{
  "type": "PivotListProperty",
  "pivotListPropertyFilteringOption": {
    "pivotFieldId": "<pivot-field-id>", // MUST be from rows or columns, NOT pages!
    "listPropertyPath": ["propertyName"],
    "compareOperator": "Eq",
    "values": ["value"],
    "variableIds": []
  }
}
```

**Example - Filtering by Product Name:**

If you want to filter to show only "Choco Bites" product:

1. Ensure the Product Dimension appears in **rows or columns** (not just pages)
2. Use the pivotFieldId from that row/column pivot (e.g., from the columns array)
3. Use the Property name (typically `"_name_XXXXXX"` where XXXXXX is a suffix)

```json
{
  "type": "PivotListProperty",
  "pivotListPropertyFilteringOption": {
    "pivotFieldId": "da327057-0d83-4ef4-9ec6-ba3934f6ce5f", // From columns array
    "listPropertyPath": ["_name_D81JNJ"],
    "compareOperator": "Eq",
    "values": ["Choco Bites"],
    "variableIds": []
  }
}
```

## Step 3: Sorting

Sorting controls how data is ordered in the View. It applies to both **Grid** and **Chart** display (e.g., bar order in a bar chart). Multiple sorting options can be applied, with the first option having the highest priority.

### Types of Sorting

There are two types of sorting options:

### 1. Sort by Property (ByProperty)

Sorts data based on a Property value of a Dimension (e.g., sort by Name, Code, or any other Dimension Property).

**When to use:**

- Sorting List items alphabetically by name
- Sorting by a Dimension Property like "Code" or "Category"
- Ordering data by manual Dimension order (when `property_friendly_name` is null)

**Configuration:**

```python
PydanticSortingOption(
    order=Order.Asc,  # or Order.Desc
    type=SortingOptionType.ByProperty,
    by_property_sorting_option=PydanticByPropertySortingOption(
        pivot_field_id="<pivot-field-id>",  # The pivot field to sort
        property_friendly_name="Name",       # Friendly name of property to sort by (or None for manual order)
    ),
)
```

**Example use cases:**

- Sort products alphabetically by product name
- Sort employees by employee code
- Sort departments by a custom "Priority" Property

### 2. Sort by Metric Value (ByMetricValue)

Sorts data based on Metric values (e.g., sort products by their revenue).

**When to use:**

- Sorting by calculated values (revenue, cost, profit, etc.)
- Ranking items by performance Metrics
- Ordering data by aggregated values

**Configuration:**

```python
PydanticSortingOption(
    order=Order.Desc,  # or Order.Asc
    type=SortingOptionType.ByMetricValue,
    by_metric_value_sorting_option=PydanticByMetricValueSortingOption(
        pivot_field_id="<pivot-field-id>",      # The pivot whose modalities are sorted
        value_field_id="<value-field-id>",      # The metric to sort by
        projections=[                            # Define which modality to use for sorting
            PydanticSingleModalityProjection(
                pivot_field_id="<other-pivot-field-id>",
                modality_id="<modality-id>",     # Can be None for null modality
            )
        ],
        subtotal_pivot_field_ids=None,          # Optional: for sorting on subtotals
    ),
)
```

**Important notes about projections:**

- **Projections** specify which specific modality to use when sorting by Metric values
- All pivot fields on the **opposite axis** must be either projected or included in subtotals
- For example, if sorting rows by a Metric, all column pivot fields must be projected
- Each projection selects a specific modality (or null modality) for a pivot field

**Example use cases:**

- Sort products by total revenue (descending)
- Sort regions by sales performance
- Rank employees by their productivity Metrics

### Multiple Sorting Options

You can apply multiple sorting options to a View. They are applied in order, with the first option having the highest priority.

**Example:**

```python
sorts=[
    # Primary sort: by category (ascending)
    PydanticSortingOption(
        order=Order.Asc,
        type=SortingOptionType.ByProperty,
        by_property_sorting_option=PydanticByPropertySortingOption(
            pivot_field_id=category_pivot_id,
            property_friendly_name="Name",  # Friendly name
        ),
    ),
    # Secondary sort: by revenue (descending)
    PydanticSortingOption(
        order=Order.Desc,
        type=SortingOptionType.ByMetricValue,
        by_metric_value_sorting_option=PydanticByMetricValueSortingOption(
            pivot_field_id=product_pivot_id,
            value_field_id=revenue_value_field_id,
            projections=[...],
        ),
    ),
]
```

This would first sort by category name (A-Z), then within each category, sort products by revenue (highest to lowest).

### Common Sorting Patterns

1. **Alphabetical sorting of List items:**
   - Use `ByProperty` with `property_friendly_name="Name"` (friendly name)
   - Order: `Asc` for A-Z, `Desc` for Z-A

2. **Top N analysis (e.g., top 10 products by revenue):**
   - Use `ByMetricValue` with `order=Order.Desc`
   - Combine with filters to limit to top N items

3. **Time-based sorting:**
   - Use `ByProperty` with the time Dimension's natural order
   - Set `property_friendly_name=None` to use manual/natural order

4. **Multi-level sorting:**
   - Apply multiple sorting options in priority order
   - First sort establishes primary grouping, subsequent sorts refine within groups
