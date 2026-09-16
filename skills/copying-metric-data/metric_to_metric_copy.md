# Metric-to-Metric Copy Configuration

Use this guide when the user wants to create a supported saved metric-to-metric copy configuration with `tool:create_metric_copy_config`.

## Overview

A metric-to-metric copy configuration is a saved import that copies populated values from a **source metric** into a **target metric**. The current tool creates a same-application configuration for the application's default scenario. Source and target metrics must have matching dimensions and data type.

Blank source values do not replace target values. If stale target values must be removed before copying, choose an appropriate clear-values mode.

## Choose the Configuration Strategy

| User intent | Clear-values mode | Use when | Risk to confirm |
| ----------- | ----------------- | -------- | --------------- |
| Preserve existing target values and copy populated source values on top | `none` | The copy should fill or overwrite only populated source intersections and leave other target cells untouched. | Old target values remain where the source is blank. |
| Rebuild the target from the source | `all` | The target should contain only values produced by this copy after the import runs. | All existing target values are cleared before the copy. Confirm before using. |
| Refresh only areas represented by selected source dimensions | `scoped` | The copy should clear only target values associated with selected dimensions, then import new values. | Requires correct dimension IDs. For each scope present in the source, the whole target slice across the non-selected dimensions is cleared, so clearing can reach beyond the source's populated cells; selecting fewer dimensions widens the cleared area. |

## Before Creating

1. Identify the **source metric** (data is copied from) and the **target metric** (data is copied into), then resolve their UUIDs with `tool:search_metrics_and_lists` (`kind: ["Metric"]`). Do not invent UUIDs.
2. Confirm the source and target metrics have the same dimensions and data type. Mismatched metrics are not valid Metric-to-Metric import configurations.
3. Use `tool:get_metric_to_metric_copy_configs` to inspect existing configurations on the target metric and avoid duplicates. If a matching configuration already exists, this tool cannot delete or replace it: choose a different name or tell the user the existing configuration must be removed manually in the product.
4. Choose a required **name** and optional description.
5. Decide the **clear-values** strategy. Ask the user to confirm before choosing `all` or `scoped` if their intent is ambiguous.
6. If using `scoped`, resolve the dimension UUIDs that define the clear scope.
7. Note the permission model: creating this configuration does not check permissions on the source or target metric. Running the import later requires the **Import Data** permission and at least partial **Read access** to the source metric for whichever member triggers the run (which may not be the member who created the configuration). Only values accessible to that member are imported; partial read access silently produces a partial import rather than an error. If the user's intent depends on complete source data, confirm the identity that will trigger the run has adequate read access to the source metric.

## Clear-Values Modes

Controls whether existing target-metric values are cleared before the copy:

| Mode     | Effect                                                                                                                                                                 |
|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `none`   | Keep all existing target values; copy on top of them.                                                                                                                  |
| `all`    | Clear every target value before copying.                                                                                                                               |
| `scoped` | Find which values of the selected dimensions appear in the source, then clear all target cells that have data on those dimension values — including cells for other dimensions that the source won't refill. Selecting fewer dimensions clears more. |

**`dimensionIds` is required when `mode` is `scoped`**. It contains the dimension UUIDs defining the clear scope. Omit it for `none` and `all`.

## Creating the Configuration

Call `tool:create_metric_copy_config` with:

- `name` (required) and an optional `description`
- `sourceMetricId` and `targetMetricId` (required UUIDs)
- `clearValues`: `{ "mode": "none" | "all" | "scoped", "dimensionIds": [...] }`

Use camelCase field names in examples and tool calls because they match the OpenAPI/tool contract. The Python schema also accepts snake_case field names, but the tool contract is camelCase.

Example request shape:

```json
{
    "name": "Refresh validated bookings",
    "description": "Copy reviewed staging values into the connected bookings metric.",
    "sourceMetricId": "00000000-0000-0000-0000-000000000000",
    "targetMetricId": "11111111-1111-1111-1111-111111111111",
    "clearValues": {
        "mode": "scoped",
        "dimensionIds": ["22222222-2222-2222-2222-222222222222"]
    }
}
```

The configuration is saved as a user-saved configuration for the application's default scenario. Selective mappings, scenario selection, parameters, snapshots, recovered applications, scheduling, triggering, deletion, and updates are not configurable through this tool.

## After Creating

Verify the result by listing the target metric's configurations with `tool:get_metric_to_metric_copy_configs`.

## Troubleshooting

- If `clearValues.mode` is `scoped` without dimension IDs, the tool fails with: `clearValues.dimensionIds must be provided when clearValues.mode is 'scoped'.`
- If the user asks for scenario-to-scenario copy, snapshot import, recovered-application import, selective mapping,
  parameterized mapping or cloning data between items of a dimension, explain that the current create tool cannot
  configure those product capabilities.
- If the user asks to schedule, trigger, run, delete, or update the import, explain that this skill creates only the saved configuration. Scheduling, action widgets, manual runs, deletion, and updates are product follow-up steps outside the current tool.
- If the user wants to remove target values where the source is blank, use `all` or `scoped` clearing as appropriate. With `none`, blank source cells leave existing target values unchanged.
- Creating a configuration does not validate the triggering member's Import Data permission or Read access to the source metric. At run time, only values that member can read are copied; partial read access silently produces a partial import, not an error.
