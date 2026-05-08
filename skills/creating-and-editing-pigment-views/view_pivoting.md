This skill defines how Dimensions (“pivots”) should be ordered and allocated between **Rows** and **Columns** when generating Boards.

This skill handles:
1. Ordering pivots Dimensions
2. Allocating them to Rows vs Columns

---

# **1. Ordering Rules**

## **Global Ordering Principles**

1. Related Dimensions (parent-child or same hierarchy) must always stay together and cannot be split between rows and columns (ex: `Employee` , `Employee > Manager` )
2. Parent Dimensions before child Dimensions
3. Dimensions Order: Time → Business → Comparison or Scenario

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

---

# **2. Special Behavioral Rules**

## **Filtering (“by metric value”)**

Filtering overrides all display rules:

- They must be placed last (most granular position)
- If multiple filtering pivots exist:
  - Only the first is guaranteed to work
  - Others may lose filters (known limitation)

## Grouping Dimensions

Related dimensions (parent-child or same hierarchy) must always be allocated together. They cannot be split between rows and columns.

---

# **3. Display-Type Driven Allocation**

Pivot allocation depends primarily on the **display type**.

---

## **3.1 KPI**

- All pivot Dimensions → **columns**
- `metricsLocation` MUST be `Columns` (or `Pages`) — **never `Rows`**. KPI views have no row pivots, so putting the metrics axis on Rows produces a broken layout. Default to `Columns`.

---

## **3.2 Pie Chart**

- Rows define slices (series)
- Dimensions in columns are aggregated

### **Rules**

- All pivots Dimensions → **rows**

---

## **3.3 Line Chart & Bar Chart & Combined Chart**

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

## **3.4 Grid**

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

## **3.5 Waterfall Variation**

- Similar to grid behavior

## **3.6 Waterfall Contribution**

- All pivot Dimensions → **rows**
- Dimensions in columns are aggregated

---

# **4. Summary Heuristics**

1. Always group pivots first
2. Order: Time → Business → Comparison
3. Apply display-type rules
4. Handle filters last
5. Ensure groups Dimensions remain together (in Rows or in Columns)

---
