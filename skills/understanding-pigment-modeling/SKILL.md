---
name: understanding-pigment-modeling
description: "Foundational skill. Read this skill first, for planning or doing any action in Pigment (creating an app, modifying an app, writing a formula or designing a view). It's a full tour of the platform and its features, the mental model you need to understand it and an explanation of the principle of Pigment modeling."
metadata:
  skill_path: /skills/understanding-pigment-modeling/SKILL.md
  base_directory: /skills/understanding-pigment-modeling
---

# The Pigment tour

## What is Pigment

Pigment is a multi dimensional modeling engine used by customers to analyze and plan their business. The flexibility of the platform allows us to cover all business use cases: financial planning, workforce planning, financial consolidation, supply chain, sales performance & any number of ad hoc operational use cases.

## The engine

The Pigment engine is called Graphite. It’s a multi-dimensional sparse real time calculation engine with a smart dependency orchestrator. 

When a user inputs a value or takes an action, the engine re-calculates the relevant blocks to display the results to the end users as fast as possible in the collaborative end user experience.

## The blocks and primitive objects

- **Dimension**: the main structuring block of Pigment, the analytical axis (master data): Employee, Month, Cost Center, Product. Dimensions have items (rows, records) and properties (field/attribute in other systems). Properties of type Dimensions allow us to create hierarchies, they are akin to foreign keys.
- **Transaction List**: a block type used to store transactional data such as general ledger, employee roster, sales records etc. It is similar to Dimensions, often has the same tools but cannot be used for metric structure
- **Metric:** the main logical block where user inputs and most logic (formulas) are done. Metrics are structured by Dimensions and of a specific Data Type and thus store data at the intersection of all those dimensions.
- **Table:** tables are a collection of metrics arranged together for better UX. Most User interactions happen in tables, displayed on Boards

## Data Types:

- Number: monetary value, measure, most common
- Date: to store any dates
- Text: free form case sensitive text
- Boolean: TRUE/FALSE and because Pigment is a sparse engine, BLANK
- Dimension: a dimension of the workspace, used for mapping, input, settings or visualization

**Other Objects:**

- Workspace: this is the “container” for all the customer applications and data. Some features are at workspace level, mainly user creation and external connectors
- Applications (app): a set of blocks and boards that cover a specific use for end users. This is what you will build. The app distinction is mostly for the end users, metrics and dimensions can be shared between apps and for the engine it makes almost no difference
- Views: a specific visual representation of a metric or a table in 2 dimensions, usually with dimensions pivoted differently in rows, columns and pages (page selectors)
- Widget: an item of a Board, the main UX component. Most widgets are displayed Views of blocks, other types are specific to the Board (action button, text)
- Board: the “container” of widgets, where end users use Pigment
- Calendar: the calendar is a feature allowing the users to create Time Dimensions (Month, Quarters, Years, Day, Week). They are normal dimensions with some specificities
- Import configuration: when importing data into dimensions, transaction lists or metrics, a config is created of how to load a file
- Automations: automated actions that are governed by rules defined in metrics, for workflow modeling

## The big features/concepts to be aware of

Besides the core modeling, formulas and UX features that you need to learn from the detailed skills available to you, it’s important to have a broad vision of the platform and its major features:

- Access rights (AR): Pigment has a granular access control system defining which user can see what data. AR are calculated in metrics and defined on dimensions: User x Cost Center. This AR metric then needs to be applied by an admin to be enforced.
- Permissions: The action a user can take: open an app, create a board, input data
- Test & Deploy (T&D): our feature to manage dev → test → prod and safely deploy changes. The key thing to know is some dimensions will be “managed dimensions” meaning they will have independent items in prod. That means no hardcoding of dimension items can be done in formulas, this will block the T&D deployments
- Planning cycle / Versioning: Pigment being a planning platform, most of our apps contain a Version dimension in the structure of almost every metric. For finance use case: actuals, plan, forecast. For other use cases, it can vary a lot.
- Snapshots: they are a full copy of an application, all data, formulas and even the boards. Snapshots can be fully explored by users, as the app was and their data can be pulled into the live app through the Data Slices features (or Scenario). This is how we archive historical Versions.
- Scenario is the name of a Pigment feature. Activating it means you have a "Scenario dimension" in every metric and is used for fast, disposable what if input and modeling. It should not be used for the planning cycle modeling, only over the top, non anticipated and disposable what if scenario testing.
- Data Slices: a reporting component applied in views used for “complex” reporting with version variations (P&L). A Slice is a specific combination of dimensions items, regularly used from reporting and applicable on metric that have these dimensions. Slices also pull data from Snapshots
- Page Selectors: almost all end users will interact with page selectors, selecting the Month, version, Country, Product they want to see. Page selectors are the structural dimensions of a metric displayed in Pages (not in row nor columns) and they impact metrics, tables and their views and thus the boards (where the users interact)
- Front end calculations. These features are used for reporting and should be prioritized vs creating new metrics in final reports. For ratios, variations, comparisons, prioritize these versus new metrics. They can’t be re-used for logic in metrics.
    - Show Value As: clone of a metric displayed in specific ways, useful to calculate inline ratios, in a P&L for example
    - Calculated Items: “clone” of a dimension item, allowing comparisons vs another. Example: Prior Month, Year To Date, Year To go, Var % vs Plan

# The modeling fundamentals

Modeling is the core of Pigment and the most complex part. It touches and uses all of the blocks, objects, features & concepts of the platform and controls them via the Formula Language. This is but an overview for you to understand the concepts, there are skills for complete understanding of every element — start with `skill:writing-pigment-formulas`.

## Modeling = Dimension's creation, formula and adapted views

Formulas are in Metrics and in properties of dimensions and transaction list (usually simpler there).

The components:

- source blocks, usually other metrics and dimensions
- functions are predefined operations on the data or blocks, with parameters and an output of a specific data type. Functions are expressed with parenthesis () and are before a block like ITEM(block)
    - some functions like IF(), IFDEFINED(), SWITCH() are structuring a formula
- Modifiers are specific Pigment logic, altering the dimensionality of a block. they are after the block Metric[REMOVE: Dimension]

## How Pigment Executes a formula

The Pigment engine will execute every part of the formula following algebraic rules, including the modifiers in the formula.

It will automatically align the dimensionality of every part of the formula. It can’t operate two parts together if they don’t have the same dimensions

It will do this on every item of the cartesian product of all the dimensions.

Example: Loaded Salary = Base Salary * (1+Tax Rate)

With Base salary by Employee x Month and Tax Rate by Country and Year and Loaded Salary by Employee x Month

If you write this formula, Pigment will ADD the country dimension to Base Salary and the Employee Dimension to (1+Tax Rate) resulting in numbers by Employee x Country x Month.

However the target metric Loaded Salary is structured by Employee x Month so Pigment will have to remove the Country dimension, summing the numbers, making it wrong.

Do not rely on automated dimension adjustment, the correct formula is:

'Base Salary' * (1+'Tax Rate'[BY CONSTANT: Employee.Country, Month.Year])

## Dimensional Modeling

The concept of dimension and how it structures metric is the key to mastering Pigment, Pigment being by definition a multi-dimensional modeling platform.

Deciding what is a dimension is a foundational decision. When you are given data/input, you must decide if a data set is a dimension. In that sense, Pigment is similar to other OLAP tools with the cardinal rule: if you will need to input or make a formula dependent on it, thus being in the structure of any metric, it needs to be a dimension. 

Only dimensions can be in the structure of the metrics and thus have inputs/calculations/results.

The dimensional structure of a metric is a capital decision. It will decide how formulas run, how users input, how it can be displayed. Strive to keep the number of dimensions in the structure to the strict minimum required to allow for input & formula.

Several features will help you in that:

- property aggregation in views: you can use the properties of a dimension in a metric to display & filter it. Example: You have a metric ‘OPEX inputs’ by Cost Center x Country x Month x Version. That metric can be displayed by all the properties of these dimensions. If the user wants it by year, the view can be pivoted by using the Month.Year property. By Region ? Use Country.Region. This can be done for rows, columns and pages. The Month & Country levels can be removed from the view. Pigment will automatically aggregate (or let you choose AVG, MIN etc). All without any formula or structural changes
- Mapped Dimensions: following the principle above, mapped dimensions is how we manage time dependent properties. Example, the Cost Center of an Employee is changing over time. using the Cost Center property of the Employee dimension would be wrong. You will create a mapping metric ‘Employee Cost Center’ by Employee x Month x Version and this metric can be used in the view pivot as the properties we mentioned above. Again without formulas or structure alteration.

These two concepts allow us to maintain a very LEAN dimensional structure while providing ALL the dimensions in the view a user might want, without impacting data volume & calculation load.


# Modeling Principles: know this before you go

This section is summarizing the major modeling principles that are known best practices to follow to build functionalities on Pigment.

## Dimension Usage

Dimensions are an analytical axis serving for metric structures.

Do not store data in dimension properties, only invariant properties of that dimension.

Do not plan for end user inputs (in properties, creation of item is expected) on Dimensions, you will encounter multiple features limitation if you do as it's not made for that

## Transaction List Usage

Transaction List: the only place where external data is loaded, especially transactional data (accounting, HR, sales, POs).

## Metrics Usage

Metrics are where planning manual inputs and business logic (formulas) is done. If an end user is going to use it, it is very likely a metric.

## Table Usage

A table is a collection of metrics, nicely formatted, where users will interact with Pigment (inputs & reporting), displayed on widgets on Boards.
When doing a table, always format it in a way that make it easy for humans to read: pivot the dimensions and metrics in the right positions, add the correct page filters, ensure columns headers are readable and set some friendly names if the table is for end users.

## Structure correctly for access rights:

The access right and permission system of Pigment is flexible. The only imperative: you need the dimension onto which security will be applied in the structure of metrics. 

Impossible to have access rights by Employee if Employee is not a dimension in the metrics.

## Factor and Re-use

In Pigment as in code, it is best to calculate one thing once and then re-use it. Instead of creating a metric with a long formula nesting a lot of Functions and modifiers, create intermediate metrics with clear, re-usable outcomes and re-use them in a final metric.

This also impacts performance of calculations

## Build for sparsity

Pigment is a sparse engine. It means every cell (metric or property) can be BLANK. The engine ignores BLANK values. but it computes 0 or “ “ or FALSE. Avoid formula patterns filling BLANK

## Dimensional transformation

One of the key things being done in Pigment modeling is transforming the dimensionality of data/metrics. This is done by the MODIFIERs — learn their syntax from `skill:writing-pigment-formulas` and the full mechanics from `skill:using-formula-modifiers`.

## Only one metric extracting data from a transaction list

Extracting data from a transaction list is a “costly” operation. There should only be 1 metric doing a BY over one measure in a transaction list, including all its dimensions. Then other metrics can [REMOVE] on that metric as required.

Possible exception: if one of the dimension is very large and only used in one place (exemple: vendor), it’s acceptable to have a specific metric for this

This will also make for a clean dependency diagram of the app/workspace

# Modeling To-dos

Those are a set of rules to follow to avoid issues, follow them strictly

## Property types

When loading (or creating) data in Pigment, if the data is Text,you must analyze if it should be of Type Dimension. Most Text data in Pigment is dimension as this is the only way it can be used for logic. Text is rare and usually dedicated for things like address, comments, description.

## Metric structure

- Transaction list cannot be in the structure of Metrics
- Almost every metric will have a Version dimension in its structure. This is a hard truth for all finance use cases. Do not defer that decision when working, if you do not put a Version dimension it must be a conscious choice for a good reason.
- Dimension subsets increase the risk of loosing data. Use them for a specific purpose, not simply for convenience. See the specific skills for details

## Naming and organization

- Never put a block in the root, always place it in a folder. Follow `skill:naming-and-organizing-applications`.
- Folders do not impact calculation, they are purely for human & agent understanding
- Never use `.`, `:`, `'`, or `"` in block names. Prefer ASCII.

## Build order

In Pigment, you cannot build a block based on another until it has being reate, it is not code. You need a metric to be created before mentioning its name in a formula, a dimension to be created before creating a property of this type.

Suggested build order: Calendar (verify/configure) → Dimensions → Version Dimension → Transaction Lists → Metrics (input, then calculated) → Tables → Boards. Always create prerequisites before dependents.

Never recreate time dimensions. Reuse the app calendar.

## Formula safety

- Never hard-code dimension item references for Time, Version, countries, or planning periods. Use a Dimension-typed input metric. Stable type/class/category items (e.g. `'FX Rate Types'."AVG"`) may stay hard-coded but avoid when possible
- Never use `DATE()` with literal year/month for period bounds. Use a Date-typed input metric.
- When T&D is active: never use a disconnected dimension as the property type on a connected dimension. Ask the user about connectivity if unknown.

## Structural changes on existing blocks

- A structural change (adding/removing a dimension, changing type) does not automatically propagate to the metrics that reference the changed one. Pigment aligns mismatched dimensions silently instead of failing, which can produce wrong numbers with no visible error. Before editing, use `tool:get_data_dependency_tree` (direction `Sources`) to trace back to the originating metric(s) and transaction list, so you know what grain of detail already exists upstream. Before declaring such a change done, use `tool:get_data_dependency_tree` (direction `Usages`) to find dependent formulas, then `tool:validate_formula` (with `target.metric_id`) on each to catch a silent mismatch — see `skill:writing-pigment-formulas` ("Structural Dimension Changes") for the full workflow.
- Removing a dimension from a metric's structure is lossy and irreversible. Once the metric no longer stores that grain the historical detail cannot be recovered from the metric itself. Say so explicitly when proposing the change, even if the detail still exists upstream (e.g. on a source metric or transaction list).
