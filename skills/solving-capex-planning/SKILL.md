---
name: solving-capex-planning
description: "Planning skill. Use this to plan and build a CapEx planning use case: asset register staging, new capex requests, depreciation engine (multi-method SWITCH), net book value and P&L/cash export. Defines the reference app architecture, folders, naming prefixes and boards."
metadata:
  base_directory: /skills/solving-capex-planning
  includes:
    - "SKILL.md"
    - "*.md"
---

# Capex Planning

## Use Case Definition

CapEx planning allows for our customers to plan for depreciation hitting the P&L by planning for new assets (and also the cash out) added on top of the current asset register & depreciation schedule.

### Scope of this skill

*This skill is giving you instructions on how to build a solid app architecture. The basics have to be followed but depending on the requests or data set provided by the user, you can add/remove to it*

*You can choose to modify the architecture ONLY if you have a valid reason for it*


It describes how to build a stand alone app which should be connected to your data hub and main FP&A app (with the P&L). It could also be built directly into your main FP&A app.

The logic of this app follow a single accounting framework. It is extendable to support multiple ones.

### User & Roles

End user: Department leaders planning to buy new assets

Power User/Admin: Finance team who is managing the data collection process, new asset validation and in fine using the planned depreciation into the wider P&L

### Data Sets & sources

- Asset Register: loaded from the ERP, this is the list of all previously bought assets and their characteristics (see details in the data model section)

- Existing depreciation schedule: Optional, this can be added into the logic if the customer already has it calculated in their ERP. Otherwise the app logic will calculate depreciation for existing assets as well


### Use case pitfalls


- asset register data will be re-loaded frequently and reload must not impact past plans -> do a staging logic
- Asset data and properties change by version and potentially over time, properties of the dimension are invariant only, so very few and technical only (besides the manual input properties required for new requests creation)
- put existing and new assets onto the same dimension so the users easily have a single calculation chain & reporting
- New Capex Request have to be Asset ID assign to Entity and Department to drive access rights, these needs to be properties (of Asset) but they are not the final properties used for calculations and they are not dimensions of the input metrics
- Assumptions (useful life, default method, capitalization threshold) change by Version and Year: they are ASP_ metrics, NEVER properties of the Asset Category dimension.
- Depreciation must be fully written off over the useful life, and net book value only exists from the in-service month. Every method has a closed form: never use PREVIOUSOF or an iterative cycle for depreciation (slow, and it hides end-of-life residuals).
- The plan also needs the cash out of new requests, not only the depreciation (payment month + OUT_CapEx_Cash).

### Questions to ask the user

**ALWAYS ask these questions with `tool:ask_user` before building anything.** It takes 5 questions max per call: send the two ASK questions in a first call, then the five confirmations in a second one.

ASK:
- do you want to use existing depreciation schedule calculated by your ERP ? Whatever the answer, build the switch (SET_Use_ERP_Schedule + PULL_ERP_Depreciation_Schedule, see 3.2) so a schedule can be plugged in later: the answer only sets the default of the switch.
- do you want to include an asset disposal logic?

And confirm with them:
- the depreciation methods you will create, and the declining balance multiplier (default 2 = double declining)
- the assumptions per asset category: useful life, default method, capitalization threshold (applied to the line total, Quantity × Unit Cost)
- the Versions to create and the last actuals month of each (this is the switchover that freezes the register load each Version sees) and the Calendar ranges to create
- the request validation: by default only Approved requests count in the plan
- the access rights: by default Department leaders (role Capex Input) only see and edit the assets of their departments; Entity can be added as a second access dimension


## Pigment Architecture

### Data Model

#### Asset 

This is the main dimension of the app, containing both existing and new assets.
A key design choice is that the description, entity and department properties are inputs, dedicated for new assets request creation.

This dimension is getting access rights applied based on the department (and optionally the entity), through a mapping metric: see 5. Security

| Field | Type | Data / Formula | Description |
|---|---|---|---|
| Asset ID | Text | Data (unique) | Identifier of the line. ERP asset number for existing assets (FA-2023-001); typed or generated for requests. Join key to the register load. |
| Description | Text | Data | Free-text label, typed by the requester for new lines. |
| Entity | Dimension → Entity | Data | Legal entity typed by the requester at creation, so the line is identifiably theirs. Used by the engine for new requests only; existing assets take the staged register entity. |
| Department | Dimension → Department | Data | Cost centre typed by the requester at creation. Same role as Entity. |
| Auto-generated | Integer | Data (auto ID) | Sequential number assigned by Pigment when a line is created. Feeds the display name of requests. |
| Display Name | Text | Formula: `IF(Asset.'Asset Source'.isExisting, Asset.'Asset ID', "REQ-" & Asset.'Auto-generated')` | Label shown to users: the ERP asset number for existing assets, REQ-n for requests. ERP IDs are not under our control: check the register's ID format first, and if it could collide with REQ-n, pick another request prefix (e.g. NEW-). |
| Asset Source | Dimension → Asset Source | Formula: `IF(Data_Asset_Register.'Asset ID'[BY COUNT: Data_Asset_Register.'Asset ID'] > 0, 'Asset Source'."Existing Asset", 'Asset Source'."New Request")` | Derived, never typed: a line found in the register load is an existing asset, anything else is a new request. Create it after the transaction list exists (see Build order). |
| Is Existing | Boolean | Formula: `Asset.'Asset Source'.isExisting` | Boolean form of Asset Source, used by logic in the app to drive formulas behavior |
| Created By | Dimension → User | Data (history property) | User who created the line. Audit trail for requests, auto generated by Pigment |
| Created Time | Text | Data (history property) | Timestamp of creation. Audit trail for requests, auto generated by Pigment |

Once you have created this dimension, set Display Name as the default display name of the dimension with the 'Display Name' property

Create a View called "New Request" and show only the 3 input properties: Description, Entity, Department

#### Asset Source

Simple technical dimension with 2 items, Existing Asset and New Request with a boolean property isExisting for easy referencing in formulas

#### Asset Category

Dimension for the category/type of asset, which drives assumptions such as useful life. You need to ask the user for their own categories or derive it from their Asset Register.
The assumptions themselves are ASP_ metrics (3.0), not properties of this dimension. Its only extra property is the invariant mapping `Depreciation PnL Account` (Dimension → P&L Account).

#### Depreciation Method

The calculation method by asset category/asset, used in the asset register and new capex requests. Drives the calculation engine.
Property `ID` (Integer) drives the SWITCH: 1 Straight-line, 2 Declining Balance, 3 Expensed. Expensed is not chosen by users: the engine forces it when the line total is below the capitalization threshold.

#### Request Status

Draft, Submitted, Approved, Rejected, with a Boolean property `Counts in Plan` (TRUE for Approved only). This is how Finance validates new requests: only requests that count in plan reach the depreciation, cash and export.

#### P&L Account

The accounts the depreciation is exported to: one depreciation account per asset category plus one account for expensed items. Usually shared from the FP&A app; create a local one only if none exists.

#### Version dimension

This version is a must in any Pigment app, will be in the structure of almost all metrics.
Property `Last Actuals Month` (Dimension → Month) on each item: the switchover. It decides which register load the Version sees.
Use in combination with two SET_ metrics (Version × Month, Boolean), created right after the Version items and BEFORE any STG_ metric:
- `SET_Switchover_IsActuals` = `Month <= Version.'Last Actuals Month'`
- `SET_CalculatePlan` = `Month > Version.'Last Actuals Month'`

Do not intersect SET_Switchover_IsActuals with a version window start: a register load older than the window would silently disappear from staging. If skill:building-versions-and-planning-cycles already created Is_Actual / Is_Plan, reuse them only if they follow this rule.
This dimension is usually shared with/from others apps, check the workspace status before creating it.

#### Calendar dimensions

Month and Year are required: Month for the calculations, Year for the assumptions.

This dimension is usually shared with/from others apps, check the workspace status before creating it.

#### Data = Transaction List = Asset Register

This is the transactional data loaded from the ERP, in a transaction list called Data_Asset_Register, one row per asset and per load.
If the load tool does not create Asset items on the fly, add them first from the distinct asset IDs of the file.

| Field | Type | Data / Formula | Description |
|---|---|---|---|
| Asset ID | Dimension | Data (display property) | ERP asset number, e.g. FA-2023-001. The join key to the Asset dimension. This must be unique as Pigment will auto create the Asset dimension list by loading into this property. |
| Description | Text | Data | Free-text label of the asset from the ERP. |
| Entity | Dimension → Entity | Data | Legal entity owning the asset. Staged into the entity used by every P&L and cash mapping for existing assets. |
| Department | Dimension → Department | Data | Cost centre owning the asset. Same role as Entity. |
| Asset Category | Dimension → Asset Category | Data | Class of fixed asset. Drives the P&L depreciation account and the lookup of assumptions (life, threshold, default method). |
| Quantity | Integer | Data | Number of units in the line. Blank is treated as 1. |
| Unit Cost | Decimal | Data | Cost per unit. Total cost = Quantity × Unit Cost. A blank unit cost means the asset is absent from that load. |
| In-Service Date | Date | Data | Capitalization date from the ERP, e.g. 2023-02-01. Raw date, never used directly by the engine. |
| Useful Life Months | Integer | Data | Life carried by the register, when the ERP has one. Blank falls back to the category assumption. |
| Depreciation Method | Dimension → Depreciation Method | Data | Method carried by the register, when the ERP has one. Blank falls back to the category default. |
| Accumulated Depreciation | Decimal | Data | Accumulated depreciation at the load date, as booked in the ERP. Compared with the recomputed schedule by the reconciliation check. |
| Load Month | Dimension → Month | Data | Month of the ERP extract this row belongs to. Critical for the app logic, it can also be generated by Pigment at load time |



### Architecture and details

Follow the instructions below unless the user has expressed a new requirement that requires you to change something.

The flow of the app is as follows. Ensure you create the FOLDERS in this way (or similar if some things are pre-existing) and put all the BLOCKS in the correct folder.

The formulas below are the reference implementation: use them as written, adapting only block names.

#### Build order

Build in this order. Each step needs the blocks of the previous ones, and the order avoids the rework seen in previous builds:

1. Calendar (Month, Year). Range: ask the user, usually will be past year, current and next
2. Folders.
3. Lookup dimensions and their items: Entity, Department, Asset Category, Depreciation Method (with ID), Asset Source, Request Status, P&L Account.
4. Version and its items (with Last Actuals Month), then SET_Switchover_IsActuals and SET_CalculatePlan. Every STG_ metric needs them.
5. Asset dimension with its identity and input properties only: Asset ID, Description, Entity, Department, Auto-generated, Created By / Created Time.
6. Data_Asset_Register and the data load. THEN the Asset formula properties (Asset Source, Is Existing, Display Name): they read the transaction list, so they cannot exist before it.
7. STG_ metrics (1.2).
8. ASP_ and SET_ parameters (3.0), with their values entered: the engine returns blanks until they are.
9. NCI_ inputs (2).
10. ATR_ attributes (3.1).
11. CALC_ engine and CHK_ reconciliation (3.2).
12. Verify the numbers (checks 1 to 5 of the Definition of done) BEFORE building any table or board: an engine bug found after the boards means re-checking every board.
13. OUT_ and PUSH_ metrics (4).
14. Security (5): INP_ access inputs, ARM_, MAP_, then the rules. MAP_ reads ATR_Department, so it cannot come earlier.
15. Tables and their default views, then the boards.
16. One sample request end to end (check 6), then a final `list_issues`.

#### Dimensions
Folder name: 0. Dimensions

This section and its folder contain all the dimensions of the app

#### Asset Register
Folder name: 1. Data
This folder contains all the data sets loaded externally, at a minimum the asset register, potentially the existing depreciation schedule.

##### Sub Folder: 1.1 Raw data 
In here build the Asset Register transaction list as described above.
Also `PULL_ERP_Depreciation_Schedule` (Asset × Month, Decimal, loaded from the ERP when available, empty otherwise).

##### Sub Folder 1.2 Staging Data

Here put all the Staging Data metrics, by Asset and Version, prefixed by STG_

The STAGING metrics formula/logic:
Data_Asset_Register.propertyX[BY SUM: Data_Asset_Register.'Load Month',Data_Asset_Register.'Asset ID'][ADD: Version][FILTER:SET_Switchover_IsActuals][REMOVE LASTNONBLANK:Month]
This is staging the data onto MONTH, removing data for AFTER the switchover of the version THEN taking the last value.
BY SUM Only works for the numbers, use LASTNONBLANK for dates, text and dimensions:
Data_Asset_Register.'Asset Category'[BY LASTNONBLANK: Data_Asset_Register.'Load Month',Data_Asset_Register.'Asset ID'][ADD: Version][FILTER:SET_Switchover_IsActuals][REMOVE LASTNONBLANK:Month]

Stage every register field: STG_Description, STG_Entity, STG_Department, STG_Asset_Category, STG_Quantity, STG_Unit_Cost, STG_In_Service_Date, STG_Useful_Life_Months, STG_Depreciation_Method, STG_Accumulated_Depreciation, STG_Load_Month (the 'Load Month' itself, staged the same way: it tells which load the Version is frozen to). Plus:
- `STG_In_Service_Month` (Dimension → Month) = `TIMEDIM(STG_In_Service_Date, Month)`. The engine works on months, never on the raw date.

#### 2. New Capex Inputs

Folder 2. New Capex Input

Put here all the input metrics (prefixed by NCI_), by Asset and Version, necessary to plan for a new asset and their table.
- NCI_Asset_Category (Dimension → Asset Category), NCI_Quantity (Integer), NCI_Unit_Cost (Decimal), NCI_In_Service_Month (Dimension → Month, not a Date)
- NCI_Payment_Month (Dimension → Month): month the cash goes out, blank = in-service month
- NCI_Useful_Life_Months, NCI_Depreciation_Method: optional overrides, blank = category assumption
- NCI_Request_Status (Dimension → Request Status), NCI_Justification (Text)

Description, Entity and Department are NOT NCI_ metrics: they are the Asset properties typed when the request is created (access rights rely on them).

The Default view of the table should be usable for humans: Asset in rows, Filter on new asset (Is Existing = false), metrics in columns, Versions, Entity and Department in pages. Give a friendly name to all metrics and ensure header columns are the correct height and width. Both are part of the build, not polish: set the names with the `friendlyName` of each value in `update_view_values`, and the widths with `update_view_grid_layout` (`perMetricColumnWidthOverrides`, `rowHeaderColumnWidths`).

#### 3. Calculations

Folder 3. Calculations

This will contain the business assumptions, the asset attribute metrics and the actual calculation engine

##### Sub Folder 3.0 Assumptions
Metrics by Asset Category, Version and year: Useful life, Depreciation Method, Capitalization Threshold
Prefixed by ASP_: `ASP_Useful_Life_Months` (Integer), `ASP_Default_Depreciation_Method` (Dimension → Depreciation Method), `ASP_Capitalization_Threshold` (Decimal). Manual inputs, filled for every Year and Version in the calendar range.

Global parameters, prefixed by SET_ (no dimension unless stated):
- `SET_DB_Multiplier` (Decimal, default 2)
- `SET_Use_ERP_Schedule` (Boolean): use PULL_ERP_Depreciation_Schedule for existing assets instead of the computed schedule
- `SET_Expensed_PnL_Account` (Dimension → P&L Account): account of the lines below the capitalization threshold
- SET_Switchover_IsActuals and SET_CalculatePlan (see Version dimension)

##### Sub Folder 3.1. Asset Attributes 

Metrics by Asset x Version, prefixed by ATR_
Joining existing and new assets. Use these for page selectors and BY mappings and for the calculation engine.

Every attribute is resolved once, here, and the engine only reads ATR_:
- `ATR_x = IF(Asset.'Is Existing', STG_x, NCI_x)` for Asset Category, Quantity, Unit Cost, In-Service Month
- `ATR_Entity = IF(Asset.'Is Existing', STG_Entity, Asset.Entity)`, same for Department and Description
- `ATR_Useful_Life = IFBLANK(IF(Asset.'Is Existing', STG_Useful_Life_Months, NCI_Useful_Life_Months), ASP_Useful_Life_Months[BY: ATR_Asset_Category, ATR_In_Service_Month.Year])`, same pattern for ATR_Depreciation_Method with ASP_Default_Depreciation_Method. The register or the request wins, the category assumption of the in-service year fills the blanks.
- `ATR_Total_Cost = IFBLANK(ATR_Quantity, 1) * ATR_Unit_Cost`
- `ATR_Method_ID = IF(ATR_Total_Cost < ASP_Capitalization_Threshold[BY: ATR_Asset_Category, ATR_In_Service_Month.Year], 3, ATR_Depreciation_Method.ID)`
- `ATR_PnL_Account = IF(ATR_Method_ID = 3, SET_Expensed_PnL_Account, ATR_Asset_Category.'Depreciation PnL Account')`
- `ATR_Payment_Month = IFBLANK(NCI_Payment_Month, NCI_In_Service_Month)` (requests only: existing assets are already paid)
- `ATR_Included = IF(ISDEFINED(ATR_Unit_Cost), IF(Asset.'Is Existing', TRUE, IF(NCI_Request_Status.'Counts in Plan', TRUE)))`: existing assets present in the Version's load, plus approved requests. Use nested IF, not OR: OR with a blank input returns BLANK.

##### Sub Folder 3.2.  Depreciation Calc 

Metrics by Asset x Version x Month, prefix CALC_
This is where the actual depreciation calculation happens.

This is the example of the core engine calculation allowing for multiple depreciation method and easy addition

Metric CALC_Schedule_Computed_All_Methods = SWITCH(ATR_Method_ID, 1, CALC_Dep_StraightLine, 2, CALC_Dep_DecliningBalance, 3, CALC_Dep_Expensed)

1,2,3 being an ID property of the Depreciation Method dimension.

The full engine:
- `CALC_Months_In_Service = MONTHDIF(ATR_In_Service_Month.'Start Date'[ADD: Month], Month.'Start Date') + 1` (1 = in-service month; MONTHDIF needs the [ADD: Month])
- `CALC_Depreciating = IF(CALC_Months_In_Service >= 1 AND CALC_Months_In_Service <= ATR_Useful_Life, TRUE)`
- `CALC_DB_Rate = SET_DB_Multiplier / ATR_Useful_Life` (Asset × Version)
- `CALC_Dep_StraightLine = IF(ATR_Method_ID = 1 AND CALC_Depreciating, ATR_Total_Cost / ATR_Useful_Life)`
- `CALC_Dep_DecliningBalance = IF(ATR_Method_ID = 2 AND CALC_Depreciating, IF(CALC_Months_In_Service = ATR_Useful_Life, ATR_Total_Cost * POWER(1 - CALC_DB_Rate, ATR_Useful_Life - 1), ATR_Total_Cost * CALC_DB_Rate * POWER(1 - CALC_DB_Rate, CALC_Months_In_Service - 1)))`. Closed form: the last month of the life writes off the remaining book value, so the asset ends at zero. No PREVIOUSOF, no cycle.
- `CALC_Dep_Expensed = IF(ATR_Method_ID = 3 AND CALC_Months_In_Service = 1, ATR_Total_Cost)`
- `CALC_Depreciation = IF(SET_Use_ERP_Schedule AND Asset.'Is Existing', PULL_ERP_Depreciation_Schedule[ADD: Version], CALC_Schedule_Computed_All_Methods)[FILTER: ATR_Included]`: the ERP swap point and the inclusion gate, used by everything downstream
- `CALC_Accumulated_Depreciation = CUMULATE(CALC_Depreciation, Month)`
- `CALC_Gross_Asset_Value = IF(CALC_Months_In_Service >= 1 AND ATR_Method_ID <> 3, ATR_Total_Cost)[FILTER: ATR_Included]`
- `CALC_Net_Book_Value = IF(ATR_Method_ID <> 3, CALC_Gross_Asset_Value - CALC_Accumulated_Depreciation)`: blank before the in-service month, zero after the life, never negative (expensed lines have no book value)
- `CHK_Register_AccDep_Variance` (Asset × Version) = `IF(Asset.'Is Existing', STG_Accumulated_Depreciation - CALC_Schedule_Computed_All_Methods[FILTER: Month <= STG_Load_Month][REMOVE: Month])`: the ERP's booked accumulated depreciation against the recomputed schedule up to the load month. Do not write it with `[SELECT: Month = <a metric>]`: it compiles but returns wrong values when the month differs by Version.

This folder should Contain a CALC_Engine table with all these metrics with the goal of displaying how the calculation works. It should be human readable (Assets in page, Metrics in columns, Month in Rows), display the column headers properly.

Don't put friendly names but ensure metric name allow for rules understanding.

#### 4. Reporting metrics 
Folder 4. Reporting metrics

Metrics by Asset x Version x Month for the app reporting and control
It's not necessary to create metrics specifically for this, you can re-use the CALC_ ones by aggregating views. Two outputs are needed as metrics:
- `OUT_Depreciation_Plan = CALC_Depreciation[FILTER: SET_CalculatePlan]`
- `OUT_CapEx_Cash = IF(Month = ATR_Payment_Month, ATR_Total_Cost)[FILTER: ATR_Included][FILTER: SET_CalculatePlan]`

##### Sub folder 4.1 Export

Add here PUSH_ metrics mapping Assets to Department and Entity and the P&L Account for exporting to the FP&A / P&L. They are aggregated at the P&L grain, not per asset:
- `PUSH_Depreciation_Plan = OUT_Depreciation_Plan[BY: Asset -> ATR_Entity, ATR_Department, ATR_PnL_Account]` (Entity × Department × P&L Account × Version × Month)
- `PUSH_CapEx_Cash_Plan = OUT_CapEx_Cash[BY: Asset -> ATR_Entity, ATR_Department]`

Use the arrow BY (`Asset -> mapping`): a plain BY with a mapping metric that carries Version drops the Version.

#### 5. Security (Access Rights)

No folder to create: every block below goes into the application's Security folder. It is not returned by `search_folders`; get its id from the folderId of `Users roles` in `list_security_metrics`, and pass it as the folder of `create_metric`.

End users (department leaders, role `Capex Input`) only read and write the assets of their departments. Finance and admin roles keep full access. Department is the access-bearing dimension, Asset the main dimension: the rule is written on Department and remapped onto Asset.

Same pattern as every AR setup: boolean inputs by the access-bearing dimension, one ARM and one rule per access dimension, a mapping metric from the access dimension to the main dimension.

| Block | Type | Dimensions | Data / Formula |
|---|---|---|---|
| INP_User_Department_Read | Boolean | User × Department | Manual input: TRUE where the user can read that department |
| INP_User_Department_Write | Boolean | User × Department | Manual input: TRUE where the user can write that department |
| ARM_Capex_Department_Access | Access Rights | User × Department | see below |
| MAP_Asset_to_Department | Dimension → Department | Asset | `ATR_Department[REMOVE LASTNONBLANK: Version]` |

```pigment
// Access rights driven by Department, restricted only for the "Capex Input" role
IFDEFINED('Users roles',
  IF('Users roles' = Role."Capex Input",
     ACCESSRIGHTS(
       IF(INP_User_Department_Read, TRUE, BLANK),
       IF(INP_User_Department_Write, TRUE, BLANK)
     ),
     ACCESSRIGHTS(TRUE, TRUE)
  )
)
```

The inputs are by User, not Role: each department leader owns their own departments, which a role cannot express.

MAP_Asset_to_Department reads ATR_Department, not the Asset property `Department`: that property is typed for new requests only and is blank for existing assets. `list_possible_access_right_rule_remappings` also offers a parent property remapping on Asset.Department: do not use it, for the same reason. The mapping metric must be Asset only (a remapping cannot carry Version), hence the REMOVE LASTNONBLANK.

Rules, created with `create_access_right_rule`, all Read and Write and active:
- **Asset AR**: backing metric ARM_Capex_Department_Access, target `dynamicMetrics` on the Asset dimension, `folderIds` = every top-level folder of the app EXCEPT the Security folder (otherwise the rule applies to MAP_Asset_to_Department, the metric it is remapped through). `dimensionRemappings`: Department → `metricMapping` MAP_Asset_to_Department, remapped to Asset (take the ids from `list_possible_access_right_rule_remappings`).
- **Register AR**: same backing metric, target `specificList` = Data_Asset_Register, no remapping: the list carries its own Department property.
- Keep the role rule backed by `Users roles` (all blocks) as it is.

Entity, when the user asks for it: same blocks and same Asset rule for Entity (INP_User_Entity_Read / Write, ARM_Capex_Entity_Access, `MAP_Asset_to_Entity = ATR_Entity[REMOVE LASTNONBLANK: Version]`, rule on Asset with Entity → MAP_Asset_to_Entity). Two rules on the same metric combine, the most restrictive wins.

## Potential modifications

### Connectivity to other apps

This app will mostly be built in a workspace with other applications. Dimensions such as versions and Calendars should come from the existing HUB. 
P&L_Account, Entity and Department as well will exist in other applications and should be shared

### Adding a new Depreciation method

To add a new method, you will need to add a new item in the dimension, create the corresponding CALC metric for that method and then include it in the SWITCH of the CALC_Schedule_Computed_All_Methods metric.
Write the new method as a closed form of CALC_Months_In_Service (like CALC_Dep_DecliningBalance), and check that it sums to the total cost over the life.

### Multiple accounting standards

For a multi-national company, the customer might want to have the depreciation calc to be usable for the group AND for the subsidiaries.
The best way for that is to create a new dimension Accounting Standards containing Group and Local.
Add it to the structure of: the Assumptions metrics, the CALC_ metrics and potentially the reporting ones.

## Definition of done

Run these checks with `query_data` / `get_list_items` before handing over. Checks 1 to 5 come before any table or board is built.

1. `list_issues` is empty.
2. Data_Asset_Register row count = rows in the file; Asset items = distinct asset IDs; every register asset has Is Existing = TRUE.
3. Staging: take an asset whose value changes between two loads (or two Versions with different Last Actuals Months). Each Version must show the value of the load at its own Last Actuals Month.
4. Depreciation integrity, for at least one asset per method: the sum of CALC_Depreciation over all months = ATR_Total_Cost; CALC_Net_Book_Value is blank before the in-service month, zero after the last month of life, and never negative. Check a declining balance asset at its last month of life specifically.
5. Fallback: an asset with a blank register life or method takes the ASP_ value of its category at its in-service year.
6. One sample request end to end: create the Asset item, fill the NCI_ inputs, set it to Approved. Check the total cost, the depreciation starting at the in-service month, the cash in the payment month, and the PUSH_ values at Entity × Department × P&L Account. Set it back to Draft: it must leave the plan.
7. The NCI_ table has friendly names and column widths set; the CALC_Engine table reads Month in rows with readable headers.
8. `list_access_right_rules`: Asset AR and Register AR are active, Asset AR carries the Department → MAP_Asset_to_Department remapping and its folders exclude the Security folder. MAP_Asset_to_Department has a department for every Asset, existing and requests.

## Outputs

## Board 0 - '0. Setup & Assumptions'

This board is for the admin or finance role. It should display all the parameter metrics (all the input metrics not for the end user) and explain them in text widgets: 
- the version dimension and its parameter / properties (switchover date)
- parameters: declining balances, P&L account mapping, Boolean switch for existing depreciation schedule
- assumptions: the "business" assumptions: Useful Life per Asset Category, Capitalization threshold etc

Each parameter or assumption gets a text widget next to it saying what it does and what it changes in the engine (e.g. "Capitalization threshold: lines whose total cost is below it are expensed in their in-service month instead of depreciated"). A title per section is not enough.

## Board 1 - '1. Asset register & data'

This board is for the admin/ finance role, a summary of the Existing asset data, the asset register.
It should contain:
- a widget summarizing the existing asset depreciation + net book value
- the existing asset data and check that this data was loaded correctly (row counts per Load Month, and CHK_Register_AccDep_Variance per asset)


## Board 2 — `2. CapEx Request Input`

This is a board that focuses on the End User who is inputting new assets or any other action by this profile.

At a minimum, this board is to allow this user to request a new Asset and input its planning information. The board must have:
- the NCI_* metrics into a nicely formatted table for this user to input new capex (including NCI_Request_Status, which Finance sets to Approved)
- a widget of the dimension Asset with the form activated so they can request a new item (an AddListItem action button on the Asset list using its "New Request View")
- It must have at least the following page selectors: Version, defaulted to one item, department and entity
- a summary of the requests' planned cash out and depreciation (OUT_CapEx_Cash, OUT_Depreciation_Plan) for the selected Version

