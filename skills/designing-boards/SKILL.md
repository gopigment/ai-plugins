---
name: designing-boards
description: Always use when creating or editing a Board. This skill includes supporting files in this directory - explore as needed.
metadata:
  skill_path: /skills/designing-boards/SKILL.md
  base_directory: /skills/designing-boards
  includes:
    - "*.md"
---

# How to Use This Skill

**Progressive Disclosure Pattern**: This `SKILL.md` provides an overview. Most details live in supporting files.

**This file alone is often not sufficient**

**Required workflow**:

1. **Read this file first** - Understand available resources and when to use them
2. **Identify relevant topics** - Match your task to any of the supporting documents
3. **Read supporting files** - Use `tool:read_file` or `tool:grep` to access detailed documentation
4. **Explore as needed** - Use `tool:ls`, `tool:grep`, or `tool:glob` to discover additional resources in this directory (some might not be explicitly mentioned in this file)

# Pigment Board Design Knowledge Base

This skill provides comprehensive guidance for designing Boards in Pigment. It covers board structure, layout rules, widget sizing standards, content conventions, and proven board design patterns for common business use cases.

Board design in Pigment is the practice of translating business questions into clear, structured, and visually coherent dashboards. A well-designed Board improves readability, adoption, decision-making speed, and executive trust.

This skill focuses on **what a Board should contain and how it should be laid out**, not on modeling or formula logic.

---

## When to Use This Skill

Use this skill whenever you need to:

- **Design a new Board from scratch**
- **Structure a Board logically** with sections and hierarchy
- **Apply consistent layout rules** using Pigment's 12-column grid
- **Choose appropriate widget sizes** for KPIs, charts, and grids
- **Ensure governance and consistency** across Boards
- **Translate business needs into dashboard structure**
- **Follow best practices for executive-ready dashboards**

This skill should be used **after modeling is done** and **before or during Board creation**.

---

## Supporting documents

When doing the following tasks, you MUST read these documents:

- When creating or editing a Board:
  - Must read to the end: [board_design_rules.md](./board_design_rules.md)
  - Must read: [board_pages.md](./board_pages.md)
  - Apply the inline widget sizing rules in the **Widget Sizing** section below.
  - Before finishing, if the `tool:board_view_reviewer` subagent is available, self-check the Board and its Views with it.

- When you need a View:
  - **You MUST also load `skill:designing-views`** — as soon as the Board involves creating, editing, or picking a View, including choosing a chart type or chart config, pivots (rows / columns / pages), filters, sorts, totals or aggregators.
  - Read [relevant_views.md](./relevant_views.md) and [view_widgets.md](./view_widgets.md) (**CRITICAL** for widgets).
  - **All three are required and cover different layers** — [view_widgets.md](./view_widgets.md) for how the **widget** points at a View and which `display_type` is valid for it, [relevant_views.md](./relevant_views.md) for reuse-vs-create, and `skill:designing-views` for configuring the **View** itself. Neither substitutes for the other: do not skip [view_widgets.md](./view_widgets.md) because you loaded `skill:designing-views`, and do not skip `skill:designing-views` because you read the files here.
  - Use `tool:get_block_views` to **see** what exists. **Reusing** is optional: generic names (e.g. _View 1_) often mean you should **create** with `tool:create_view`. Do not block on a long “search for similar” pass.

---

## Core Principles of Board Design in Pigment

- Boards tell a **story**, not just display data
- Structure and hierarchy matter more than visual density
- Consistency across Boards improves usability and trust
- Boards describe _what should be displayed_, not _how it is calculated_
- Simplicity beats completeness for executive and operational dashboards
- Do not answer with too much detail to avoid overloading the user's chat.
- If the user asks to avoid new modeling blocks (e.g. no new Metrics, Lists, or structural changes), creating a Table block can still be appropriate: a Table bundles existing Metrics for visualization and/or input on a Board. It's a layout container, not a new structural object dimension.

---

### Design Principles

**[board_design_rules.md](./board_design_rules.md)** - Board design principles
**Covers**:

- Global Board Structure
- Widget Sizes Guidelines
- Column Layout Strategy

---

## Content Guidelines

### Supported Widget Types

- ✅ **Text widgets** - Titles, descriptions, explanatory content
- ✅ **View widgets** - Data visualizations (Grids, Charts, KPIs)
- ✅ **Spacer widgets** - Visual separation between sections
- ✅ **ActionButtons** - Navigation or import buttons
- ❌ **Do not use Images unless explicitly asked**

### Text Widget Usage

Use **text widgets** for:

- Section titles and subtitles
- Explanatory text and commentary

**Do NOT use text widgets** for describing navigation intentions. Use actual ActionButton widgets instead.

### View Widget Usage

Use **View widgets** for data visualizations (Grids, Charts, KPIs).

**⚠️ CRITICAL:** Every View widget requires a View ID. There is NO such thing as:

- ❌ "View ID: Not applicable"
- ❌ "Using the Metric directly for KPI/Chart display"
- ❌ Referencing only a Block ID without a View

Even for a simple KPI showing a single Metric value, you must:

1. Create or find a View that references the Metric
2. Configure the View appropriately
3. Reference that View ID in the View widget

### KPI Widget Usage

- No row pivots
- `metricsLocation` MUST NOT be `Rows` — use `Columns` (default) or `Pages`. KPIs have no row pivots, so Rows produces a broken layout.
- The KPI widget displays as many columns as the underlying View.
- Do not put a Dimension in columns unless it has very few items.
- To display several Metrics side-by-side, prefer a single KPI Widget on a View with multiple Metrics in `values` (and `metricsLocation: Columns`) over multiple KPI Widgets — even if individual single-Metric Views already exist.
- Not available for List blocks (KPI requires a Metric or Table block).

### Spacer Widget Usage

Use **spacer widgets** for:

- Visual separation between sections
- Standard size: `width=12`, `height=1`

### Prioritization Rules

- Prioritize Blocks that directly support the Board's purpose
- Avoid unnecessary information
- Aim for clarity, hierarchy, and narrative flow
- Design Boards that are: visually clean, easy to scan, logically ordered

---

## Board Creation Workflow

Follow this 4-step workflow when creating a Board:

### Step 1: Define Board Purpose and Plan Board Pages

1. **Define board purpose** (1-2 sentences)
   - Example: "Track Q1 2024 actual performance against budget"

2. **Use Search tool** to check what Dimensions your Metrics have

3. **Plan Board Page Selectors** (not View Filters — defaults applied in Step 4):
   - Time Page Selector (Month, Quarter, Year) - only if Metrics have time Dimensions
   - Version Page Selector (Actuals, Budget, Forecast, or combinations) - only if Metrics have Version
   - Scenario Page Selector (Default or multiple scenarios) - only if Metrics have Scenario
   - Other dimensional Page Selectors as needed
   - See **[board_pages.md](./board_pages.md)** for detailed guidance

4. **Before treating Board Page Selectors as shared context for every widget**, verify that **each View you intend to place on the Board** includes a **compatible** page for every dimension you will set at board level (e.g. if the board should narrow by Year, each View must have Year in Pages—or a grouping page that resolves to Year—see **[board_pages.md](./board_pages.md)**). If a View is missing that dimension in Pages, **edit or create the View first**; the board cannot force a dimension onto a View that does not expose it in Pages.

### Step 2: Create Board Structure

1. Create a board with (in board settings):
   - Board name and description
   - Icon and color

   If not specified otherwise by the user, apply the following default settings:
   - Full width
   - Header formatting: show Board icon and show color

2. Add sections and widgets:
   - Section titles and subtitles (text widgets)
   - View widgets for data visualizations
   - Spacer widgets between sections

### Step 3: Find or create Views, then add widgets

1. **Identify Blocks** for the story.

2. **For each Block**, `tool:get_block_views` — pick a **reusable** View only if name + pivots fit this board; otherwise **`tool:create_view`** (see [relevant_views.md](./relevant_views.md)). Ensure Pages align with [board_pages.md](./board_pages.md).

   - Before adding pivots to a View, call `tool:get_available_pivots` and build pivots from the returned candidates
   - to show data the metric isn't structured on, prefer a mapped-dimension pivot over editing the metric unless user explicitly asked you to change the metric — follow [view_pivoting.md](../designing-views/view_pivoting.md).

3. **After each View**, align Pages with sibling widgets ([board_pages.md](./board_pages.md) — Cross-View Page Alignment).

4. **Add View widgets** that reference those View IDs.

### Step 4: Update Board Pages

1. Use `tool:update_board` to set Board Page Selectors
2. Apply the Time, Version, Scenario, and other Page Selector defaults you defined in Step 1
3. Set **default selected items** for each Board Page at board level (e.g. default Year to FY25 / 2025 when that is the intended analytical context)
4. Confirm each View widget is **linked** to the Board Pages you care about (widgets do not “inherit” a dimension the View never had in Pages—see **[board_pages.md](./board_pages.md)**, Board-to-Widget Page Compatibility Rule)

**Key Points:**

- Plan Board Pages in Step 1, but apply defaults and selections in Step 4 (after Views are added)
- Align compatible Pages across **all** Views before setting board defaults ([board_pages.md](./board_pages.md))

---

## Learning Path: Read in This Order

### 1. START HERE: Board Structure

Based on the list of relevant Blocks to display, focus on:

- Board structure (title, description, sections hierarchy)
- For each section, selecting appropriate widgets (View, Text, Spacer, etc.) to display data and provide context

---

### 2. THEN: Content & Widgets

Focus on:

- Use View widgets to display data from Metrics, Lists, or Tables
- Use Text widgets for titles, descriptions, and context
- Use Spacer widgets for visual separation

---

### 3. FINALLY: Widget Sizing

You MUST follow these height guidelines.

**Text & ActionButton widgets** — height depends on the variant and its content. Variant maps to the widget's `layout` value — Text: **Open** = `Regular`, **Filled (Box)** = `Box`; ActionButton: **Button**, **Card**. **Filled = Open + 1**: every Filled (Box) Text variant is exactly 1 taller than its Open equivalent.

| Widget type  | Variant      | Content                 | Height |
| ------------ | ------------ | ----------------------- | ------ |
| Text         | Open         | Title only (H2)         | 2      |
| Text         | Open         | Title only (H1)         | 3      |
| Text         | Open         | Link / Normal only      | 2      |
| Text         | Open         | Title + body            | 3-4    |
| Text         | Open         | Title + body + takeaway | 4      |
| Text         | Filled (Box) | Title only (H2)         | 3      |
| Text         | Filled (Box) | Title only (H1)         | 4      |
| Text         | Filled (Box) | Link / Normal only      | 3      |
| Text         | Filled (Box) | Title + body            | 4      |
| Text         | Filled (Box) | Title + body + takeaway | 5      |
| ActionButton | Button       | Any label               | 2      |
| ActionButton | Card         | Any label               | 10     |

**View & Spacer widgets** — when a View widget (KPI/Chart/Grid) has a title, add 1 to the minimum height.

| Widget type         | Height |
| ------------------- | ------ |
| Spacer              | 1      |
| KPI without title   | 4-6    |
| KPI with title      | 5-7    |
| Chart without title | 11-18  |
| Chart with title    | 12-18  |
| Grid without title  | 11-24  |
| Grid with title     | 12-24  |

Chart/Grid height depends on data complexity (rows, columns, legends, axis labels).
