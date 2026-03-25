# Board Pages (Board-Level Filters)

## What They Are

Board Pages are **default filters applied at the Board level**. They define the analytical context for the entire Board and are **automatically used by all View widgets**, unless they opt-out.

**Key rule:** Board Pages come from the Pages of the Views you add to the Board. You cannot add to or remove Page Selectors from the Board Pages. You can only configure the default values of each Board Page.

After you set the Board Pages default values, Users will be able to select different values for each Board Page. The Views that did not opt-out will automatically adjust their data display to match the selected values.

### Unlinking a Page at the Widget Level

Widgets can opt-out of each Board Page by **Unlinking** their corresponding Page. For example, if the Board has a Board Page for `Year`, the Widgets can opt-out of it by unlinking their `Year` Page. Unlinking a Page means that the Widget's View will no longer be affected by the Board Page and will keep displaying the same data, regardless of the Board Page selected values. Its corresponding Page will appear as a regular Page and can be edited independently of the Board Page.

### Page Selector Visibility

Page selectors can be shown, minimized, or hidden. **Never hide a page selector unless the user explicitly asks for it.**

---

## Filter Strategy by Board Purpose

| Board Purpose             | Time Filter     | Version Filter          | Scenario Filter                          |
| ------------------------- | --------------- | ----------------------- | ---------------------------------------- |
| Monthly Review            | Month=Current   | Version=Actuals         | Scenario=Default                         |
| Quarterly Business Review | Quarter=Current | Version=Actuals,Budget  | Scenario=Default                         |
| Annual Planning           | Year=Next       | Version=Budget,Forecast | Scenario=Default                         |
| Variance Analysis         | Month=Current   | Version=Actuals,Budget  | Scenario=Default                         |
| Scenario Planning         | Year=Current    | Version=Forecast        | Scenario=Baseline,Optimistic,Pessimistic |
| Executive Overview        | Quarter=Current | Version=Actuals         | Scenario=Default                         |
| YTD Performance           | Year=Current    | Version=Actuals,Budget  | Scenario=Default                         |

Adapt based on which Dimensions your Views actually have. Skip any column that doesn't apply. Values like "Current" and "Next" are conceptual — resolve them to actual modality IDs from the Dimension's items.

---

## Common Filter Dimensions

### 1. Time-Related Dimensions

Choose based on Board purpose. Only if Views have time Dimensions:

- `Month = [specific month]` (e.g., "Jan 24")
- `Quarter = [specific quarter]` (e.g., "Q1 24")
- `Year = [specific year]` (e.g., "FY 24")

### 2. Version Dimension

Only if Views have a Version Dimension:

- **Single version**: `Version=Actuals` or `Version=Budget` or `Version=Forecast`
- **Comparison**: `Version=Actuals,Budget` (for variance analysis)
- **Multi-version**: `Version=Actuals,Budget,Forecast`

### 3. Scenario Dimension

Only if Views have a Scenario Dimension:

- **Single scenario**: `Scenario=Default`
- **Comparison**: `Scenario=Baseline,Optimistic` (for scenario planning)

### 4. Other Dimensions

Any business Dimensions your Views have in Pages: Region, Department, Product Line, etc.

---

## Examples

### Standard: Quarterly Variance Analysis

```
Board Purpose: Compare actual vs budget performance for Q1 24

Board Pages:
- Time: Quarter=Q1 24
- Version: Actuals,Budget
- Scenario: Default
```

### Edge Case: Views With Only Time Dimensions

```
Board Purpose: Track monthly sales trends

Views have: Month Dimension
Views don't have: Version, Scenario

Board Pages:
- Time: Month=Jan 24 to Dec 24
(No Version or Scenario filters needed)
```

### Edge Case: Views With No Standard Dimensions

```
Board Purpose: Product catalog dashboard

Views have: Product, Category, Region
Views don't have: Time, Version, Scenario

Board Pages:
- Region=All
- Category=All
(Only filter the Dimensions that actually exist)
```

### Edge Case: Mixed Dimension Availability

```
Board Purpose: Mixed KPI dashboard

Some Views have: Year, Version, Scenario
Other Views have: Only Region, Product
Some Views have: No Dimensions at all

Board Pages:
- Year=FY 24
- Version=Actuals
- Scenario=Default

Note: Widgets showing Views without these Dimensions will simply
ignore the Board-level filters.
```

---

### Board Pages and Grouping Pivots in a View's Pages

A Board Page on Dimension D can drive a View Page if that Page is:

- A **simple Page** on D (`dimensionId` = D's ID), or
- A **Grouping Page** whose `listPropertyPath` resolves to D (e.g., base Dimension Month with property path `["_year"]` leading to Year).

When a Board Page selects a value on the target Dimension (e.g., Year = FY 2024), the Grouping Page filters its base Dimension items (e.g., Months) to only those whose property chain matches the selected value (Months whose `_year` = FY 2024).

**Example: Board Page on Year driving a Month-based Grouping Page**

```
Metric base Dimension: Month
Month has a Dimension Property _year → Year

View Page:
  dimensionId = <Month GUID>
  listPropertyPath = ["_year"]    (Grouping Page targeting Year)

Board Page:
  pageIdentifier: { pageIdentifierType: "Dimension", dimensionId: <Year GUID> }
  defaultModalityReferences: [{ type: "Fixed", fixedValue: <FY-2024-modality-id> }]

Result: Only Months whose _year = FY 2024 are included.
The user sees a Year filter, even though the View is modeled at Month level.
```

**When to use Grouping Pages:**

Use when Views are modeled at a fine grain (Month, Store, Employee) but you want Board-level filters on higher-level Dimensions (Year, Region, Division). This requires clean Dimension Properties defining the hierarchy.

**Important:** The `listPropertyPath` in the Grouping Page must correctly resolve to the same target Dimension as the Board Page's `dimensionId`. Use technical property names, not display names.
