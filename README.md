# Pigment AI Plugins

Connect your AI assistant to Pigment to query, analyze, and build business planning models.

These plugins bundle the [Pigment MCP server](https://kb.pigment.com/docs/mcp-server-1) with domain-knowledge **Skills** that teach AI assistants how Pigment works — its proprietary formula language, modeling patterns, dashboard conventions, and performance best practices.

## Prerequisites

Enable MCP in your Pigment workspace: **Settings > Integrations > MCP**. See the [Pigment documentation](https://kb.pigment.com/docs/mcp-server-1) for details.

## Installation

### Cursor

Install the Pigment plugin by running the following command in the agent chat:

```
/add-plugin pigment
```

### Claude Code

Add the Pigment marketplace and install the plugin:

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

**Default** — read-only tools for data analysis. Only metrics where [AI data access](https://kb.pigment.com/docs/pigment-ai#ai-data-access) is enabled are accessible.

**Advanced** — write tools for modeling Pigment applications (dimensions, metrics, formulas, calendars, boards, views, and more). Each user can enable it in **Settings > Advanced Features > Advanced MCP Tools**.

Users connect with their own credentials via OAuth, so their Pigment access rights and permissions apply by design — they can't access or do more than what they can already do in Pigment.

> **Note:** Advanced mode includes a **search** tool that lets the AI assistant scan and understand your entire application logic — this helps AI provide better answers. Search exposes all Block metadata (e.g. names, data types, dimensions) to users, but no actual data is accessible through it. We recommend not putting any sensitive information in Block metadata as it is not subject to access rights (unlike actual data). If you want to prevent a user from accessing Block metadata, that user must not have access to the application in Pigment.

### Skills

Skills are domain-knowledge files loaded automatically by Cursor and Claude Code. They provide the context AI assistants need to work effectively with Pigment.

| Skill | Description |
|-------|-------------|
| **Analyzing Data** | Query formulation, data discovery, analysis patterns, result interpretation |
| **Modeling Applications** | Architecture, dimensions, metrics, tables, calendars, subsets, access rights |
| **Writing Formulas** | Pigment's proprietary formula language — syntax, modifiers, functions, performance |
| **Optimizing Performance** | Profiling, scoping, sparsity, iterative calculations, troubleshooting |
| **Designing Boards** | Board structure, widget sizing, layout rules, page organization |
| **Creating Views** | View creation, draft/override workflow, pivots, filters, sorting |
| **Integrating Data** | Data import, CSV mapping, integration overview |
| **Agent Capabilities** | What the AI can and cannot do, behavioral protocols, handoff guidance |

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

If you run into any issues, contact us at [Pigment Support](https://support.pigment.com/en/support).
