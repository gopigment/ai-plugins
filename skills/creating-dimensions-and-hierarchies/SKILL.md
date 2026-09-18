---
name: creating-dimensions-and-hierarchies
description: "Execution skill. Use when creating or modifying dimension lists (hierarchies) and dimension properties"
metadata:
  skill_path: /skills/creating-dimensions-and-hierarchies/SKILL.md
  base_directory: /skills/creating-dimensions-and-hierarchies
---

# Creating Dimensions and Hierarchies

Follow this workflow when you are asked (or are thinking about) creating a dimension list.
If your task is a complex one, beyond creating a simple dimension list, you should read [examples-of-dimensions-and-hierarchies.md](./examples-of-dimensions-and-hierarchies.md) which gives you examples of specific dimension/hierarchies creation: Employee dimension, multi-level P&L, Organisational hierarchy, self referencing dimension, input at all level hierarchies.

For time dimensions:
Pigment provides calendar dimensions: **Day, Week, Month, Quarter, Year**, with relationships between them.

**Always reuse these.** Do not create custom Month or Year lists.
In the case of a Fiscal Year different from the default calendar, this is managed by a new FY property on the existing Calendar dimensions.

Then follow the workflow below


## Step 0: What already exists

Before creating a dimension or making a plan for creating one, search for the existing ones with `tool:filtered_search`.
Do not create a dimension if one with a similar name exists. If there is doubt, read the dimension items and properties with `tool:get_list_items` to be sure. If there is still a doubt, ask the user.


## Step 1: Recognize What a Dimension List Is

A **dimension list** is an analysis axis also known as metadata, master data. It has **items** (rows) and **properties** (columns).

- Items must be unique so Pigment can reference them across multiple blocks. The Default unique property is called Name
- Dimensions define metric structure and are reusable across the workspace. As such, they are the main structuring block of applications.
- Dimensions used for analytical axes: Product, Employee, Country, Month, Cost Center, Account, P&L, Versions
- Dimensions are also used for more specific use cases: Workflow Status, Yes/No, Gender

When a user presents you with data or request a creation, you must decide if you will create a dimension and properties.
When in doubt, err on the side of creating a dimension versus using it as text.

Disambiguation: Transaction lists are another type of block, used to store transactional data, this skill does not cover them (although the tool to create dimension and transaction list is the same).

## Know before you go

### Hierarchies

A hierarchy links dimensions through a Dimension-type property on the child dimension.

Thus a dimension can be in multiple hierarchies although it usually has one Main hierarchy (Chart of Account, Organisation, SKU > Product).
Some examples:
- SKU > Product > Category
- Country > Region
- Account (legder) > P&L Line > P&L Category

The leaf dimension of a hierarchy MUST have a propery dimension for every level of the hierarchy, allowing for easy page filtering.

### Set Unique Properties for Imports and API

Every dimension list requires **at least one unique property** for item identification.

Default: `Name` is unique. Alternatives:

- Set **Unique Item values** on another property (e.g., `Code`, `Employee ID`) using `tool:update_list_property`.
- Use auto-generated unique IDs (Integer property) when no natural key exists: call `tool:create_list_property` with `is_auto_id: true`, `target_type.type=Integer`, and `is_unique=true`. 

Before import:

1. Confirm which property is the unique identifier.
2. Ensure import file values match exactly (case, spacing).
3. Never disable uniqueness on `Name` until another property is marked unique — use `tool:update_list_property` for both operations, sequentially

Changing a property to unique will be rejected as long as there are duplicates

**Converting a property to auto-generated ID is destructive.**

Calling `tool:update_list_property` with `is_auto_id: true`, `new_type.type=Integer`, and `unique=true` on an existing property **replaces all existing values** with sequential IDs (1, 2, 3…). The property can no longer be edited manually or mapped from imports.

- Warn the user explicitly before applying.
- On a live in-use application, work in a sandbox first.
- Prefer creating a new unique property with the auto-generated ID in a production app and then do a cleaning in a later phase if necessary.

### List Subsets

A **subset** is a selected group of items from a parent dimension list, usable as its own dimension in metric structure. Only dimension lists can have subsets.

**Valid use cases:**

- Use the same dimension twice in one metric (mirror dimension).
- Restrict input dropdowns to a subset while keeping full list on parent.
- Target iterative calculations to a smaller item set for performance.
- Maintain a master list but model/input with approved subsets only.

**Do NOT use subsets when:**

- You only need to hide inactive items in a view → use filters instead.
- Membership changes frequently without review → risk of data loss.

If you need to create/update a subset here, see also `skill:using-list-subsets` for safe paterns and data risks (changing subsets is data destructive)


### Handle Ragged and Unbalanced Hierarchies

Pigment handles uneven-depth hierarchies naturally:

- Each item has **one parent** per hierarchy property (or BLANK if unassigned).
- Do not force dummy intermediate items. Leave BLANK where no parent applies.
- A product may map directly to a division while others go through category — valid.
- BLANK parents roll up separately; verify this matches business intent.

Ragged hierarchies work without issue for reporting, but if inputs and access right will be driven off ragged levels, you will need to balance them.


## Step 2: Decide its properties

Most dimensions will have several properties. 
Dimension properties should not be used to store data, only meta-data, describing the dimension not a value of the item

Every list property has exactly one data type:
- **Number**: This is rarely used on Dimensions as they are not meant to hold data. It can be Dimension order, Sign (-1/1)
- **Date**: Mostly used for logging the events of that dimension, when they are created from other systems: creation date, validity date, last change date
- **Text**: Names, codes, descriptions, mostly unique labels. Most dimensions will have at least two
- **Boolean**: flags (active, eligible, included, obsolete)
- **Dimension**: reference to another dimension list (it creates what we call a hierarchy)

It's important to not create a property as text when it should be a dimension. 
Properties with low cardinality data are typically of type Dimensions as they can then be used to filter, pivot, formula logic etc.


## Step 3: Create the dimension

Always create the dimension in a folder. As per `skill:naming-and-organizing-applications`, there should be a "0. Dimension" folder, if not create it with `tool:create_folder`

CRUD order:
1. **Top-level parent dimensions** — no Dimension-type dependencies (e.g., `Division`, `Region`). Use `tool:create_list`.
2. **Child dimensions** — lists whose properties reference parents (e.g., `Category`, then `Product`). Use `tool:create_list` then `tool:create_list_property` for Dimension-type properties.
3. **Properties** — add non-Dimension properties after structure is clear. Use `tool:create_list_property`.
4. **Subsets** — only after parent dimension is stable and use case is confirmed. Use `tool:create_sublist`.

Create the dimension using `tool:create_list`, which accepts its properties in the same call. 
Use `tool:create_list_property` to add a property to a dimension that already exists, if you didn't get it right first time

## Step 4: Populate the list items

If you were provided a CSV, load it using `tool:import_csv_to_list`, especially if the data set is large (> 1000 rows)
If the data was in the prompt, add list items in the dimension(s) (and each level if there are several) with `tool:add_list_items`


