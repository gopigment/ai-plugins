---
name: solving-sales-performance
description: "Planning skill. Sales performance use cases: quota planning, commission calculations, territory assignment, attainment tracking, pipeline coverage. Each use case has inline guidance."
metadata:
  skill_path: /skills/solving-sales-performance/SKILL.md
  base_directory: /skills/solving-sales-performance
---

# Sales Performance

> Prerequisites: Version dimension exists (`skill:building-versions-and-planning-cycles`). Calendar configured (`skill:setting-up-calendar`).

---

## Quota Planning & Cascading

**When:** set annual or quarterly sales targets, cascade top-down targets through a hierarchy (company → region → team → rep).

**Dimensions:** Version, Month (or Quarter), Sales Rep, Team, Region. Use a parent hierarchy: Sales Rep → Team → Region. Add Product or Segment if quotas differ by product line.

**Structure:** top-level target metric (input at Region or Company level), allocation weights or ratios per level, cascaded quota metric that distributes targets down the hierarchy using BY and allocation weights. Keep the allocation method explicit: proportional to prior year, equal split, or manual weights in an input metric.

**Version usage:** each planning cycle (FY26 Budget, Q2 Reforecast) is a Version item. Quota metrics are versioned so prior targets remain available for comparison.

**Reporting:** Quota by Rep (rows) × Month or Quarter (columns). Use view grouping by Team and Region via hierarchy properties. Calculated items for YoY quota change.

---

## Commission & Incentive Calculations

**When:** calculate variable compensation based on attainment, with tiered rates, accelerators, caps, or SPIFs.

**Dimensions:** Version, Month (or Quarter), Sales Rep. Add Commission Component if multiple payout types (base commission, accelerator, SPIF, bonus). Add Tier if rates vary by attainment band.

**Structure:** attainment metric (actual revenue / quota). Commission rate metric by Tier (input: e.g. 0-80% = 5%, 80-100% = 8%, 100%+ = 12%). Commission payout calculated by matching attainment to the applicable tier using IF/SWITCH or tiered lookup pattern (see `skill:choosing-formula-patterns`). Apply caps via MIN.

**Tiered commission pattern:** create a Tier dimension with threshold properties (Floor %, Ceiling %, Rate %). For each rep × month, calculate the portion of revenue falling in each tier and multiply by the tier rate. Sum across tiers with REMOVE.

**Reporting:** Rep (rows) × Month (columns), showing quota, actual, attainment %, payout. Use conditional formatting on attainment bands. Table block combining quota, revenue, attainment, and payout metrics.

---

## Territory Assignment & Coverage

**When:** assign sales reps to territories (geographic, named account, or segment-based), track coverage, and plan territory changes.

**Dimensions:** Version, Territory (or Account list), Sales Rep. Territory properties: Region, Segment, Account Count. Sales Rep properties: Team, Manager, Territory (Dimension type → Territory).

**Structure:** assignment is a property on Sales Rep (or a mapping metric if assignments change by Version). Revenue and pipeline metrics use Sales Rep as structural; territory breakdowns via BY-arrow mapping from Sales Rep.Territory property (see `skill:using-mapped-dimensions`). This avoids duplicating Territory as a structural dimension on every metric.

**Reassignment handling:** if territories change mid-year, use a mapping metric by Version × Sales Rep → Territory instead of a static property. Downstream metrics reference the mapping metric via BY.

**Reporting:** Territory view (using mapped dimension in pivot) showing coverage (reps per territory), quota, pipeline, revenue. Highlight gaps (territories with no rep) using ISBLANK on the assignment.

---

## Attainment Tracking

**When:** compare actual sales performance against quota, track pacing, and forecast year-end attainment.

**Dimensions:** Version, Month, Sales Rep. Reuse existing quota and revenue metrics.

**Structure:** attainment = actual revenue / quota (per rep × month or cumulative). Use YEARTODATE on both revenue and quota for YTD attainment. Pacing metric = YTD attainment / (elapsed months / total months) to show whether a rep is on track. Forecast attainment = YTD actual + remaining months × run rate.

**Reporting:** Rep (rows) × Month (columns) with attainment %. Calculated items for YTD. Conditional formatting: green > 100%, yellow 80-100%, red < 80%. KPI widgets for team-level attainment.

---

## Pipeline Analysis & Coverage

**When:** analyze sales pipeline health, coverage ratios (pipeline / quota), stage conversion rates.

**Dimensions:** Version, Month, Sales Rep (or Territory). Pipeline data in a transaction list with properties: Deal, Stage, Amount, Close Date, Probability.

**Structure:** pipeline value metric aggregated from the transaction list by Sales Rep × Month (BY from deal properties). Weighted pipeline = amount × probability. Coverage ratio = pipeline value / remaining quota. Stage conversion rates from historical win/loss data.

**Reporting:** pipeline waterfall by stage, coverage ratio by rep or territory, deal-level drill-down via transaction list views. Use chart widgets for pipeline by stage (bar) and coverage trend (line).

---

## Price-Volume-Mix Variance Analysis

**When:** decompose YoY revenue changes into price, volume, and mix effects.

For the full three-factor decomposition recipe, Pigment formula idioms, and the mandatory reconciliation check, see [pvm_variance_bridge.md](./pvm_variance_bridge.md).
