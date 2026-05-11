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
- **Bulk-save protocol** if `tool:save_draft_views` is available — after creating or editing one or more Draft Views:
  1. List the draft view names and ask the user for explicit confirmation before saving.
  2. Once confirmed, call `tool:save_draft_views` once with all draft view IDs.
  3. Report each result: view name, resulting ID, and whether it was merged or promoted to a new view.
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
- **Save via bulk-save protocol** — list the draft names, wait for user confirmation, then call `tool:save_draft_views` once with all draft view IDs

---

# View Design Process

Must always read: [view_design_process.md](./view_design_process.md).

---

# View Components

Step 1 (Data & layout): Must always read: [view_components.md](./view_components.md)

Step 2 (Filtering): Must always read: [view_filtering.md](./view_filtering.md)

Step 3 (Sorting): Must always read: [view_sorting.md](./view_sorting.md)

Step 4 (Aggregators & Totals): Must always read: [view_aggregators.md](./view_aggregators.md)
