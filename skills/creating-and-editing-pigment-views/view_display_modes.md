# Display Modes

The View Widget on the Board defines which display mode is used via its `display_type`.

- **KPI**: No row pivots allowed. `metricsLocation` MUST NOT be `Rows` — use `Columns` (default) or `Pages`. Multiple column pivots result in multiple KPI cards (one per column value). Favor a single-column KPI unless you have a specific intent for multi-card. **Not available for List blocks.**
- **Grid** (Table / List / Spreadsheet): Tabular format. This is the only display mode available for List blocks.
- **Chart**: The `display_type` is set on the View Widget, but the **chart type** (exact kind of chart visualization) is defined in the View object itself. **Not available for List blocks.**
