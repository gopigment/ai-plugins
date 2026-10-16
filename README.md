# Pigment AI Plugins

Connect your AI assistant to Pigment to query, analyze, and build business planning models.

These plugins bundle the [Pigment MCP server](https://kb.pigment.com/docs/mcp-server-1) with domain-knowledge **Skills** that teach AI assistants how Pigment works — its proprietary formula language, modeling patterns, dashboard conventions, and performance best practices.

## Prerequisites

Enable MCP in your Pigment workspace: **Settings > Integrations > MCP**. See the [Pigment documentation](https://kb.pigment.com/docs/mcp-server-1) for details.

## Installation

### Cursor

1. Go to [Cursor plugins](https://cursor.com/dashboard/plugins)
2. Import `https://github.com/gopigment/ai-plugins` as a marketplace
3. Add Pigment plugin to Cursor from the plugins page, or open Cursor and run `/add-plugin pigment` in a new chat

### Claude Code

#### Desktop App

1. Go to **Customize**, select **Add plugin**, then **Add marketplace** and paste `https://github.com/gopigment/ai-plugins`
2. Install Pigment plugin

#### Terminal CLI

```
/plugin marketplace add gopigment/ai-plugins

/plugin install pigment@pigment
```

### Configuration

After installing, set your MCP server URL. The easiest way is to paste your MCP URL into the agent chat and ask it to update the configuration. You can also edit the `.mcp.json` file directly.

Your MCP URL can be found in your Pigment workspace under **Settings > Integrations > MCP**. See the [Pigment documentation](https://kb.pigment.com/docs/mcp-server-1) for detailed instructions.

## What's Included

### MCP Server

The plugin connects your AI assistant to the Pigment MCP server, which has two modes.

#### Default mode

Public mode provides read-only tools for exploring and analyzing your Applications. For instance, public tools let you:

- Find Applications, Blocks, and Boards
- Understand Metric structure and dependencies
- Retrieve Metric and List data

To make a Block's data accessible, you need to enable [AI data access](https://kb.pigment.com/docs/pigment-ai#ai-data-access) on it.

<!-- mcp-tools:default:start -->
<details>
<summary>Available tools (14)</summary>

- fetch_list_data
- fetch_metric_data
- get_applications
- get_data_dependency_tree
- get_metric_context
- get_metric_dependencies
- get_organization_info
- get_views
- search_boards
- search_folders
- search_metrics_and_lists
- search_tables
- search_views
- semantic_search

</details>
<!-- mcp-tools:default:end -->

#### Advanced mode

Advanced mode provides read and write tools for modeling and editing your Applications. For instance, advanced tools let you:

- Create Applications
- Build and edit Blocks (Dimensions, Metrics and Tables)
- Write formulas
- Design Views and Boards
- Input data
- Audit performance

<!-- mcp-tools:advanced:start -->
<details>
<summary>Available tools (83)</summary>

- add_list_items
- batch_share_blocks
- calendar_add_time_dimension
- calendar_create
- calendar_expand
- calendar_get
- calendar_remove_time_dimension
- clear_widget_preview
- copy_formula_as_user_inputs
- create_application
- create_board
- create_cycle
- create_folder
- create_frame
- create_list
- create_list_property
- create_metric
- create_metric_copy_config
- create_sublist
- create_table
- create_view
- delete_application
- delete_block
- delete_cycle
- delete_list_items
- delete_list_property
- delete_sandbox
- delete_views
- duplicate_application
- duplicate_block
- edit_list_item_history_properties
- enable_application_library
- get_all_view_templates
- get_audit_trail
- get_available_pivots
- get_block_sharing_plan_for_batch_share
- get_blocks_performance
- get_board
- get_list_items
- get_metric_to_metric_copy_configs
- get_top_blocks_by_performance
- list_application_libraries
- list_cycles
- list_issues
- list_scenarios
- move_blocks
- performance_changes_statistics
- performance_profile_change
- performance_profile_change_scope_details
- performance_slowest_changes
- save_draft_views
- search
- search_frames
- set_board_permission_metrics
- set_metric_input
- set_widget_preview
- update_application
- update_board
- update_board_page_configurations
- update_board_widgets
- update_cycle
- update_folder_name
- update_frame
- update_frame_body
- update_list
- update_list_property
- update_list_property_input_data
- update_list_view_pivots
- update_metric
- update_table
- update_view
- update_view_aggregations
- update_view_calculated_items
- update_view_chart_config
- update_view_filters
- update_view_formatting
- update_view_grid_layout
- update_view_kpi_config
- update_view_pivots
- update_view_show_value_as
- update_view_sorts
- update_view_values
- validate_formula

</details>
<!-- mcp-tools:advanced:end -->

Each user can enable advanced mode for their account in **Settings > Advanced features > Enable advanced tools on MCP server**. Advanced mode is off by default.

Users connect with their own credentials via OAuth, so their Pigment access rights and permissions apply by design — they can't access or do more than what they can already do in Pigment.

> **Note:** Advanced mode includes a **search** tool that lets the AI assistant scan and understand your entire application logic — this helps AI provide better answers. Search exposes all Block metadata (e.g. names, data types, dimensions) to users, but no actual data is accessible through it. We recommend not putting any sensitive information in Block metadata as it is not subject to access rights (unlike actual data). If you want to prevent a user from accessing Block metadata, that user must not have access to the application in Pigment.

### Skills

Skills are domain-knowledge files loaded automatically by Cursor and Claude Code. They provide the context AI assistants need to work effectively with Pigment.

Skills can be set up in two ways:
- **Plugin**: install the Claude Code and Cursor plugins available in this repository.
- **Manual upload**: download the Skills files from the repository and upload them to your AI tool by following [your provider's documentation](https://kb.pigment.com/docs/supported-tools#skills).

Skills come in four kinds: **foundational** (read first), **planning** (decide what to build), **execution** (do the work), and **analysis** (query existing data).

| Skill | Description |
|-------|-------------|
| **Understanding Pigment Modeling** | Foundational mental model — in-memory engine, blocks, dimensionality, sparsity |
| **Analyzing Views** | On-board and headless filter planning from view pivots and page selectors |
| **Gathering Architecture Requirements** | Structured discovery before building anything |
| **Architecting Multi-Application Solutions** | Splitting work across applications, Hub-and-spoke designs |
| **Building a Full Application** | End-to-end build of a new application |
| **Editing an Existing Application** | Modifying, extending, refactoring or restructuring a live application |
| **Auditing a Pigment Application** | Health checks — formula errors, structural gaps, governance |
| **Naming & Organizing Applications** | Naming conventions, folders, block organization |
| **Creating Dimensions & Hierarchies** | Dimension lists, hierarchies, dimension properties |
| **Creating Metrics & Tables** | Metric and table creation, data types, structure |
| **Creating Transaction Lists** | Transaction lists and row-level aggregation |
| **Using List Subsets** | Mirror dimensions and subset patterns |
| **Using Mapped Dimensions** | Dynamic and time-dependent hierarchies, card metrics |
| **Setting Up Calendar** | Calendar type, fiscal year, time dimensions |
| **Building Versions & Planning Cycles** | Versions, planning cycles, actuals vs forecast |
| **Forecasting in Pigment** | Growth-rate hypotheses and statistical forecasting functions |
| **Securing with Access Rights** | Access rights design, AR metrics, apply rules, debugging visibility |
| **Sharing Data Between Applications** | Libraries and cross-application block sharing |
| **Copying Metric Data** | Metric-to-metric copy configurations |
| **Choosing Formula Patterns** | Picking the right pattern and modifiers before writing a formula |
| **Writing Pigment Formulas** | Pigment's proprietary formula language — quoting, references, syntax |
| **Using Formula Functions** | Built-in function signatures and examples |
| **Using Formula Modifiers** | Dimension mapping when source and target differ |
| **Iterating with Previous & Cycles** | PREVIOUS, PREVIOUSOF, and iterative calculation cycles |
| **Writing Performant Formulas** | Pre-delivery checklist, sparsity rules, anti-patterns |
| **Diagnosing Performance Issues** | Profiling slow calculations and timeouts |
| **Profiling Model Performance** | Performance Insights tools — change profiles, bottlenecks, before/after timings |
| **Designing Boards & Views** | Board sections, widgets, layout, pivots, filters, sorting, aggregators |
| **Aggregating View Data** | Totals, subtotals and roll-up behaviour in Views |
| **Formatting & Highlighting** | Decimals, currency, percent, K/M scaling, text and boolean display |
| **Building Pigment Frames** | Building, updating and troubleshooting Pigment Frames |
| **Designing Pigment Frames** | Applying Pigment's visual design language to a Frame |
| **Using Search Tools** | Finding blocks, folders and dependencies across an application |
| **Solving Financial Planning** | Budget and expense planning, revenue forecasting, P&L, FX |
| **Solving Workforce Planning** | Headcount, hiring pipeline, compensation |
| **Solving Sales Performance** | Quota planning, commissions, territories |
| **Solving Supply Chain** | Demand planning, inventory, procurement |

## Example Prompts

**Data analysis:**

- "List all my Pigment applications"
- "Show me metrics available in the Financial Planning app"
- "Show me Revenue by product for the last 6 months"
- "Compare actual vs. budget for EMEA revenue"

**Advanced mode:**

- "Create a Department dimension with items Engineering, Sales, Marketing, and Finance."
- "Add a Revenue metric with Product and Region dimensions, and write a Gross Margin formula that subtracts COGS from Revenue."
- "Build a headcount planning application with Department, Role, and Location dimensions. Add metrics for FTE Count, Average Salary, and Total Cost with monthly granularity."
- "Create a board for the CFO with a Revenue waterfall chart, an Opex variance table filtered by department, and a headcount trend by quarter."

## Support

If you run into any issues, contact us at [Pigment Support](https://support.pigment.com/).
