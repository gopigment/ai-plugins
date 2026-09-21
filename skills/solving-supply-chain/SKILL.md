---
name: solving-supply-chain
description: "Planning skill. Supply chain use cases: demand planning, inventory management, procurement planning, S&OP alignment. Each use case has inline guidance."
metadata:
  skill_path: /skills/solving-supply-chain/SKILL.md
  base_directory: /skills/solving-supply-chain
---

# Supply Chain Planning

> Prerequisites: Version dimension exists (`skill:building-versions-and-planning-cycles`). Calendar configured (`skill:setting-up-calendar`).

---

## Demand Planning

**When:** forecast product demand based on historical sales, seasonality, market signals, or sales team input.

**Dimensions:** Version, Month, Product (or SKU), Region (or Channel). Add Customer segment if demand varies by customer type.

**Structure:** historical demand metric (imported from ERP/WMS). Statistical forecast using FORECAST_ETS (seasonal) or FORECAST_LINEAR (trend-only) (see `skill:forecasting-in-pigment`). Judgmental adjustment metric (input) for sales team overrides on top of the statistical baseline. Final demand = statistical forecast + adjustment, or override-first using IFBLANK(adjustment, statistical).

**Version usage:** each planning cycle produces a demand plan version. Consensus demand (from S&OP) is a separate version or a dedicated metric combining inputs.

**Reporting:** Product (rows) × Month (columns) showing historical, statistical forecast, adjustments, and final demand. Chart widgets for demand trend with actuals overlay. Forecast accuracy = (Actual - Forecast) / Actual, tracked per version.

---

## Inventory Management

**When:** plan inventory levels, safety stock, reorder points, and days of supply by product and location.

**Dimensions:** Version, Month, Product (or SKU), Warehouse (or Location). Product properties: Lead Time (days or months), Unit Cost, Min Order Quantity, Safety Stock Target.

**Structure:** when both opening and closing balances must be visible (reporting, reorder analysis, drill-down), use separate beginning and ending balance metrics dimensioned by Product and Month. Link them with `PREVIOUSOF` over an iterative calculation cycle (see `skill:iterating-with-previous-and-cycles`). Do **not** collapse to a single balance metric with `PREVIOUS(Month)` unless the user only needs one output line.

Beginning balance = prior period ending balance (seed the first period from opening stock). Ending balance = beginning + supply/receipts − consumption/demand. Reorder logic is typically a flag plus quantity metric; supply can feed ending directly or via an intermediate receipts metric.

**Build order:**

1. Create all metrics in the dependency chain (balances, demand, reorder logic, supply/receipts as needed).
2. Create the iterative calculation cycle over Month including the mutually dependent balance metrics.
3. Write PREVIOUSOF on the beginning balance, then the ending roll-forward.

Do not substitute `[SELECT: Month - 1]` for PREVIOUSOF on a separate ending balance; it does not join the cycle.

**Reporting:** Product × Month grid showing inventory levels, days of supply, reorder alerts. Conditional formatting: red when below safety stock. KPI widgets for total inventory value and average days of supply.

---

## Procurement Planning

**When:** plan purchase orders, supplier allocation, and procurement spend based on demand and inventory targets.

**Dimensions:** Version, Month, Product (or SKU), Supplier. Product properties: Lead Time, Unit Cost, Min Order Quantity. Supplier properties: Lead Time Override, Preferred flag.

**Structure:** net requirements = demand plan - projected available inventory. Planned order quantity = net requirements adjusted to Min Order Quantity (round up using CEILING). Order timing = shift planned orders backward by lead time (SELECT: Month - Lead_Time). Supplier allocation: if single source, direct assignment; if multi-source, allocation weights by supplier. Procurement spend = quantity × unit cost.

**Reporting:** Supplier (rows) × Month (columns) showing planned orders and spend. Product-level detail in drill-down views. KPI widgets for total procurement spend by version.

---

## S&OP (Sales & Operations Planning)

**When:** align demand plan, supply plan, and financial plan into a consensus view for executive review.

**Dimensions:** Version, Month, Product (or Product Family). S&OP typically operates at a higher granularity (product family, quarter) than detailed planning.

**Structure:** demand plan (from demand planning, aggregated to product family). Supply plan (from inventory + procurement, aggregated). Revenue plan (demand × price). Gap analysis: demand vs supply capacity, revenue plan vs financial target. Use a Table block to combine demand, supply, revenue, and gap metrics into a single S&OP dashboard.

**Cross-application:** if demand, supply, and finance live in separate applications, use Libraries to share the relevant output metrics into an S&OP reporting app (see `skill:sharing-data-between-applications`). Keep detailed planning in domain apps; S&OP app consumes aggregated outputs only.

**Reporting:** Product Family (rows) × Month or Quarter (columns). One Board page per S&OP topic: demand review, supply review, financial reconciliation, executive summary. Use chart widgets for demand vs supply overlay (combined chart) and revenue gap waterfall.
