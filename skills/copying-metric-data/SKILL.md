---
name: copying-metric-data
description: "Use this skill when a user wants to create a Pigment metric-to-metric copy import/config to freeze formula values, reduce recalculation, seed/refresh a metric, stage validated data, prepare export metrics, or copy data between metrics. It cannot run, schedule, delete, update existing configs, use snapshots/recovered apps, or configure selective mappings. Do NOT use for CSV/file imports or formulas."
metadata:
  skill_path: /skills/copying-metric-data/SKILL.md
  base_directory: /skills/copying-metric-data
  includes:
    - "*.md"
---


# Copying Metric Data

This skill helps with **metric-to-metric copy configurations**: saved Pigment imports that copy populated values from a source metric into a target metric with matching dimensions and data type.

Use it to recognize common Metric-to-Metric import intents, then create a supported saved configuration when the current tool surface is sufficient.

## When to Use This Skill

- **Reduce recalculations** - Copy values into a metric that updates only when the saved import is run, so downstream calculations do not need to stay connected in real time.
- **Freeze formula values as inputs** - Copy formula-calculated values into a target metric so they become stable imported values.
- **Seed or refresh a metric** - Populate a target metric from another metric with the same dimensions and data type.
- **Validate staged data before connecting it** - Import external data into a staging metric, let users review it, then create a copy config from the staging metric to the connected target metric.
- **Prepare export or intermediate metrics** - Copy values into an intermediate metric before export-oriented formulas or external exports run.
- **Reduce access-rights metric recalculation** - Replace a frequently recalculated access-rights formula with a lower-frequency import. Access rights may be stale until the import runs, so use this only when the user understands that risk and has validated the approach with the appropriate solution owner.

Do NOT use this skill for:

- Importing an attached CSV or Excel file - use Pigment's data import UI (not covered by this skill)
- Writing a formula that reads another metric - use the `writing-pigment-formulas` skill

## What This Skill Can Configure Today

- Inspect existing user-saved copy configurations for a target metric with `tool:get_metric_to_metric_copy_configs`.
- Create a new same-application, default-scenario saved copy configuration with `tool:create_metric_copy_config`.
- Set the source metric, target metric, name, optional description, and clear-values mode (`none`, `all`, or `scoped`).

## Current Tool Limits

The current tool creates only the saved configuration. It cannot:

- Run, trigger, schedule, delete, or update a copy configuration.
- Configure scenario selection or scenario-to-scenario copies.
- Import from snapshots or recovered applications.
- Select a cross-application source metric (source and target are always in the same application).
- Configure selective mappings, parameterized mappings, action widgets, or missing-item handling.

Scenario-to-scenario copies, snapshot imports, and recovered-application imports are valid Metric-to-Metric product use cases, but they are not configurable through this create tool. If the user's intent needs one of these capabilities, explain the limit and do not claim the agent can configure it through this skill.

## Task-Based Routing

### Creating a metric-to-metric copy configuration

**Read**: [./metric_to_metric_copy.md](./metric_to_metric_copy.md)

Read this when the user wants the agent to create a supported saved configuration after the source and target metric intent is clear. It covers choosing the clear-values mode, preflight checks, exact tool input, and validation.

## Related Tools

- Inspect the existing copy configurations for a target metric with `tool:get_metric_to_metric_copy_configs`.
- Create a saved metric-to-metric copy configuration with `tool:create_metric_copy_config`.
