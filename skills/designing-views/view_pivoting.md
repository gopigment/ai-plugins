This guide defines how Dimensions (“pivots”) are ordered and split between rows, columns and pages for boards.

It covers:

1. Ordering pivots Dimensions
2. Allocating them to Rows vs Columns

---

# **1. Ordering Rules**

## **Global Ordering Principles**

1. Parent Dimensions before child Dimensions
2. Dimensions Order: Time → Business → Metric → Comparison or Scenario

### **Example**

Input:

- Month > Year
- Scenario
- Segment
- Country
- Country > Region
- Month

Reordered:

- Month > Year
- Month
- Segment
- Country > Region
- Country
- Scenario

## **How Order Maps To Display**

Within an axis, the order of pivots determines how the data nests:

- **Rows**: the first pivot is the outermost (leftmost) grouping; each subsequent pivot nests inside it, the last being the most granular.
- **Columns**: the first pivot is the top-most header band; each subsequent pivot nests beneath it.
- **Pages**: order changes the order in which the page selectors appear in the UI but has no impact on the data grouping.

Reordering pivots on an axis changes the grouping hierarchy of the rendered data, not just their listing.

---

# **2. Discovering Valid Pivots**

Before editing a View's pivots, discover which ones are valid instead of guessing.

- **Discover before editing**: call `tool:get_available_pivots` for the View before `tool:update_view_pivots` or `update_list_view_pivots` . It returns the pivots that are actually compatible with the View, so you do not have to guess which dimensions, groupings, mapping metrics or slice configurations are valid. 
- **Using the payload**: each candidate carries `kind` + `dimensionId`, plus `listPropertyPath` for a `Grouping` candidate, `mappingMetricId` for a `Joined` candidate and `sliceConfigurationId` for a `Slice` candidate. To add a candidate as a new pivot, copy those fields straight into the `tool:update_view_pivots` axis pivot and omit `id` (the server generates one).
- **Choosing with `pivotSummary`**: each candidate also carries `pivotSummary`, a short human-readable name (e.g. `Product > Category > Department`, or `Region (via slice "EMEA actuals")`) to help you pick the right pivot. It is a hint only — read it to choose, but never copy it into `tool:update_view_pivots`; always copy the id fields above.

## Pivot kinds

- **Dimension**: a plain block dimension. Copy `dimensionId`.
- **Grouping**: a traversal of a Dimension-typed list property (a parent-child level). Copy `dimensionId` and `listPropertyPath`.
- **Joined**: a dimension reached through a mapping metric. Copy `dimensionId` and `mappingMetricId`. Use the `mappingMetricId` from the payload — do not reconstruct it or invent a new mapping metric. To display data the metric is **not** structured on, prefer a mapped-dimension (Joined) pivot over changing the metric's dimensions, and call `tool:get_available_pivots` first to confirm a valid mapping exists.
- **Slice**: a dimension reached through a slice configuration. Copy `dimensionId` and `sliceConfigurationId`. Use the `sliceConfigurationId` from the payload — do not reconstruct it or invent a new slice configuration.

---

# **3. Special Behavioral Rules**

## **Filtering (“by metric value”)**

Filtering overrides all display rules:

- They must be placed last (most granular position)
- If multiple filtering pivots exist:
  - Only the first is guaranteed to work
  - Others may lose filters (known limitation)

## Grouping Dimensions

Related dimensions (parent-child or same hierarchy) must always be allocated together. They cannot be split between rows and columns.

### Building a hierarchy in Rows

To expose a **multi-level grouping** on the same entity, add **each level as its own pivot in Rows**, ordered from the **shallowest** path to the **deepest** (parent chain before children). Do not skip intermediate levels if you want the full drill-down in the grid.

Example on an `Entity` list: **`Entity > Grouping L1`**, then **`Entity > Grouping L2`**, then **`Entity > Grouping L3`**, and so on — one pivot per level, all in **Rows**, respecting the global ordering (parents before children on that chain).

### Tree layout vs tabular layout (Grid)

For a **Grid** widget, the product can render the **same** row pivots either as **tabular** row headers (one column per pivot level) or as a **treeview** (single hierarchy column with indentation / expand–collapse). `tool:create_view` does not take this display mode; set it after creation with `tool:update_view_grid_layout`.

---

# **4. Display-Type Driven Allocation**

Pivot allocation depends primarily on the **display type**.

---

## **4.1 KPI**

- All pivot Dimensions → **columns**
- `metricsLocation` MUST be `Columns` (or `Pages`) — **never `Rows`**. KPI views have no row pivots, so Rows produces a broken layout. Default to `Columns`.

---

## **4.2 Pie Chart**

- Rows define slices (series)
- Dimensions in columns are aggregated

### **Rules**

- All pivots Dimensions → **rows**

---

## **4.3 Line Chart & Bar Chart & Combined Chart**

- Columns: horizontal axis
- Rows: series
- If you need to create a comparison, Dimension should be placed in Rows

### **With time dimension**

- Time dimensions → columns
- All others → rows

### **Without time dimension**

- First **non-comparison** Dimension → columns
- Others → rows

---

## **4.4 Grid**

If you need to create a comparison, Dimension should be placed in Columns

### **With calendar dimension**

- Calendar Dimensions → columns
- Others → rows

### **Without calendar dimension**

- First pivot Dimension (with its parent Dimension) or Comparison Dimension → columns
- All others → rows
- Keep related Dimensions together

### **Example 1**

revenue by segment, country, region

Ordered:

- Segment
- Country > Region
- Country

Allocation:

- Columns: Segment
- Rows: Country > Region, Country

### **Example 2**

Ordered:

- Country > Region
- Country
- Segment

Allocation:

- Columns: Country > Region, Country
- Rows: Segment

## **4.5 Waterfall Variation**

- Similar to grid behavior

## **4.6 Waterfall Contribution**

- All pivot Dimensions → **rows**
- Dimensions in columns are aggregated

---

# **5. Summary Heuristics**

1. Always group pivots first
2. Order: Time → Business → Comparison
3. Apply display-type rules
4. Handle filters last
5. Ensure groups Dimensions remain together (in Rows or in Columns)

---
