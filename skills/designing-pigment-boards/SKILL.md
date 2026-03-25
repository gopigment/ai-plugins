---
name: designing-pigment-boards
description: Always use when creating or editing a Board. This skill includes supporting files in this directory - explore as needed.
metadata:
  skill_path: /designing-pigment-boards/SKILL.md
  base_directory: /designing-pigment-boards
  includes:
    - "*.md"
---

** CRITICAL:** You must read and apply the widgets sizing guidelines: [board_widget_sizes.md](./board_widget_sizes.md)

# How to Use This Skill

**Progressive Disclosure Pattern**: This `SKILL.md` provides an overview. Most details live in supporting files.

**This file alone is often not sufficient**

**Required workflow**:

1. **Read this file first** - Understand available resources and when to use them
2. **Identify relevant topics** - Match your task to any of the supporting documents
3. **Read supporting files** - Use `read_file` or `grep` to access detailed documentation
4. **Explore as needed** - Use `ls`, `grep`, or `glob` to discover additional resources in this directory (some might not be explicitly mentioned in this file)

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
  - ** CRITICAL:** You must read and apply the widgets sizing guidelines: [board_widget_sizes.md](./board_widget_sizes.md)
  - Must read: [board_pages.md](./board_pages.md)

- When you decide you need a View:
  - Must read: [relevant_views.md](./relevant_views.md)
  - **CRITICAL**: You must read and apply the View Widget guidelines: [view_widgets.md](./view_widgets.md)
  - MUST check first if an existing View can be used before creating one using `get_block_views` tool.

---

## Core Principles of Board Design in Pigment

- Boards tell a **story**, not just display data
- Structure and hierarchy matter more than visual density
- Consistency across Boards improves usability and trust
- Boards describe _what should be displayed_, not _how it is calculated_
- Simplicity beats completeness for executive and operational dashboards
- Do not answer with too much detail to avoid overloading the user's chat.

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
- ❌ **Do not use other widget types** (ActionButton, Image) unless explicitly asked

### Text Widget Usage

Use **text widgets** for:

- Section titles and subtitles
- Explanatory text and commentary

**Do NOT use text widgets** for describing intended data visualizations. Use actual View widgets instead.

### View Widget Usage

Use **View widgets** for data visualizations (Grids, Charts, KPIs).

**⚠️ CRITICAL:** Every View widget requires a View ID. There is NO such thing as:

- ❌ "View ID: Not applicable"
- ❌ "Using the Metric directly for KPI/Chart display"
- ❌ Referencing only a Block ID without a View

Even for a simple KPI showing a single Metric value, you must:

1. Create or find a View that references the Metric
2. Configure the View appropriately (e.g., no row pivots for KPIs)
3. Reference that View ID in the View widget

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

3. **Plan Board Pages** (filters to be applied in Step 4):
   - Time filter (Month, Quarter, Year) - only if Metrics have time Dimensions
   - Version filter (Actuals, Budget, Forecast, or combinations) - only if Metrics have Version
   - Scenario filter (Default or multiple scenarios) - only if Metrics have Scenario
   - Other dimensional filters as needed
   - See **[board_pages.md](./board_pages.md)** for detailed guidance

### Step 2: Create Board Structure

1. Create a board with (in board settings):
   - Board name and description
   - Icon and color

2. Add sections and widgets:
   - Section titles and subtitles (text widgets)
   - View widgets for data visualizations
   - Spacer widgets between sections

### Step 3: Find and Add Views

1. **Identify Blocks** you want to visualize based on board purpose

2. **Use `get_block_views` tool** for each Block:
   - Find existing Views on the Blocks
   - Review View descriptions and display modes (Grid, Chart, KPI, etc.)
   - Select Views that match your analytical intent

3. **Add View widgets** to the board referencing the selected Views

### Step 4: Update Board Pages

1. Use `update_board` tool to set Board Pages (filters)
2. Apply the Time, Version, Scenario, and other filters you defined in Step 1
3. All View widgets will inherit these board-level filters

**Key Points:**

- Plan Board Pages in Step 1, but apply them in Step 4 (after Views are added)
- Only define filters for dimensions that exist in your metrics
- All View widgets inherit Board Pages by default

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

- Apply Standard widget sizing consistently for each widget.
