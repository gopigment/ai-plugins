---
name: setting-up-calendar
description: "Execution skill. Use when configuring an application calendar — choosing calendar type, fiscal year, date range, and time dimensions."
metadata:
  skill_path: /skills/setting-up-calendar/SKILL.md
  base_directory: /skills/setting-up-calendar
---

# Setting Up Calendar

Every Pigment application has a **Calendar** defining the time structure. Configure it immediately after creating the application and before creating any dimension or metric.

Use `tool:calendar_get` to read the current configuration, `tool:calendar_create` to create a new calendar, `tool:calendar_expand` to extend the date range, `tool:calendar_add_time_dimension` and `tool:calendar_remove_time_dimension` to adjust granularity.

## Creating a Calendar

Use `tool:calendar_create` with these parameters:

- `application_id` (required): the target application UUID
- `start_date` (required): e.g. "2020-01-01"
- `end_date` (required): e.g. "2026-12-31"
- `selected_time_dimensions` (required): list of time dimensions, e.g. ["Month", "Quarter", "Year"]
- `actual_vs_forecast_enabled` (required): **always set to `false`.** See below.
- `gregorian_config.fiscal_year_starting_month` (optional): 1-12, defaults to 1 (January)

> **`actual_vs_forecast_enabled` is not how you split actuals from plan**, despite what the tool description suggests. It is legacy, and it gives one global switchover for the whole application: it cannot vary per planning cycle, so a Budget and a Forecast with different switchover dates cannot coexist. Pass `false` and model the split with a Version Dimension carrying a `Switchover Month` property plus `Is_Actual` / `Is_Plan` metrics — see `skill:building-versions-and-planning-cycles`.

## Sharing Calendar Dimensions

`tool:calendar_create` produces Private time dimension lists (Month, Quarter, Year). For a Hub application where calendars must be shared:

1. After creating the calendar, call `tool:calendar_get` to retrieve the time dimension list IDs.
2. For each time dimension list, call `tool:update_list` with `sharing_status: "Shared"`.

This is separate from `tool:batch_share_blocks` which is for metrics and other blocks.

## Choose Calendar Type

| Type | Periods | Use when |
| --- | --- | --- |
| **Gregorian** | Month-based (Jan–Dec) | Standard financial planning, month/quarter/year reporting. Most common choice. |
| **Weekly** | Week-based | Operations or retail planning requiring week-level cycles. |

Gregorian supports all time dimensions: Day, Week, Month, Quarter, Half, Year. Weekly supports Week and Day only.

## Set Fiscal Year Start (Gregorian Only)

The fiscal year starting month determines how Year, Quarter, and Half periods align.

- **January**: Calendar-year companies (most common)
- **April**: UK, Japan, many government entities
- **July**: Australian companies
- **October**: US Federal fiscal year

Impact:

- `TIMEDIM(..., 'Year')` returns fiscal year items
- `YEARTODATE` resets at fiscal year start
- Quarter boundaries shift (Q1 starts at fiscal year month)

Example with April start: FY 2026 = April 2025 - March 2026; Q1 = Apr-Jun 2025.

**Set the fiscal year start during initial setup.** Changing it later affects existing formulas and YEARTODATE calculations.

## Define the Date Range

- **Start date**: 2-3 years before current year (historical data needs)
- **End date**: 3-5 years after current year (planning horizon)

Common pattern: 3 years historical + 5 years forward = 8-year range.

Use `tool:calendar_expand` to extend the range later. Extending is safe; existing data is preserved. Shortening is not possible if data exists in those periods.

## Select Time Dimensions

Include only the time dimensions you need. Each adds complexity and performance cost.

**Default: enable Year, Quarter, and Month** unless the user explicitly requests fewer. Quarter is essential for standard financial reporting and costs little in performance. Only omit Quarter if the user specifically says they do not need quarterly aggregation.

### Standard Time Dimensions

| Dimension | Granularity | Include when |
| --- | --- | --- |
| **Year** | Annual | Always for any planning application |
| **Half** | Semi-annual | Mid-year reporting, strategic reviews |
| **Quarter** | Quarterly | Enable by default for any planning application |
| **Month** | Monthly | Required for Gregorian calendars; standard for FP&A |
| **Week** | Weekly | Operations, retail, capacity planning |
| **Day** | Daily | Detailed operational analysis, daily transaction tracking |

### Extra Time Dimensions (Non-Hierarchical)

These repeat across years and enable pattern analysis:

| Dimension | Items | Use for |
| --- | --- | --- |
| **DayOfWeek** | Mon–Sun | Weekday vs weekend patterns |
| **MonthOfYear** | Jan–Dec | Seasonal analysis across years |
| **QuarterOfYear** | Q1–Q4 | Quarterly pattern comparison |
| **WeekOfYear** | Week 1–53 | Week-based pattern analysis |
| **HalfOfYear** | H1, H2 | Semi-annual comparison |

## Time Hierarchy

```
Year → Half → Quarter → Month → Week → Day
```

Each level aggregates its children. This hierarchy powers `CUMULATE`, `LAG`, `YEARTODATE`, and `TIMEDIM`. Week may overlap month boundaries; Pigment handles this automatically.

## Common Setup Patterns

- **Standard Financial Planning**: Gregorian, fiscal year January (or match company), Year/Quarter/Month, 3yr historical + 5yr forward
- **Operations Planning**: Gregorian or Weekly, Month/Week (or Week/Day), 1yr historical + 2yr forward, add DayOfWeek if needed
- **Strategic Planning**: Gregorian, Year/Half/Quarter, 5yr historical + 10yr forward

## Rules

- **Never recreate** Month, Year, or any calendar dimension manually. Always use the built-in calendar.
- **Calendar properties are protected**: do not edit or delete them.
- Actuals vs Plan separation is handled by the Version Dimension (`skill:building-versions-and-planning-cycles`), not by the calendar.
- If daily granularity is enabled, consider subsetting the calendar to the relevant date range for performance (`skill:diagnosing-performance-issues`).
