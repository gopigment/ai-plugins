---
name: formatting-and-highlighting
description: "Execution skill. Use when applying metric default formatting — decimals, prefix, suffix, currency ($/€), percent (%), K/M/bp/thousand/million scaling, thousand separator, sign / zero / negative handling, text mode (Text / Rich Text / URL / Image / LocaleDateTime), boolean display (checkbox / button). Load it when creating a metric whose values are not plain counts -- a ratio, percentage, growth, currency, boolean or text metric (comments, notes, links, URLs) -- even if the user said nothing about formatting."
metadata:
  skill_path: /skills/formatting-and-highlighting/SKILL.md
  base_directory: /skills/formatting-and-highlighting
---

# Formatting and Highlighting

Set formatting on `tool:create_metric` / `tool:update_metric` via the `defaultFormat` field. A metric's default format applies to every View, Board, KPI, Grid, and Chart that displays it.

Skip this skill for `tool:update_metric` calls that don't touch default format (renaming, changing dimensions, editing description).

Out of scope: view display modes, aggregators, sort, filter; conditional formatting (UI-only); static cell formatting (background/text color, bold, italic, alignment) -- use `skill:designing-boards-and-views` (`tool:update_view_formatting`).

## Critical Rules

- **Set formatting on the metric, not on the view.** Views have no number-formatting tools.
- **`numberFormatOptions` is required inside default format.** Pass `{}` if you only set `textFormatOptions` or `booleanFormatOptions`.
- **Omitting a field leaves it unset.** There is no way to clear an individual field.
- **Modeler context overrides inference.** If the modeler states a preference ("use 0 decimals", "prefix all financial metrics with €"), apply it consistently for the session and skip name-based rules below.
- **`multiplier` is numeric**, not a string alias. Use `100` for percent, `0.001` for thousands, `0.000001` for millions, `10000` for basis points.
- **`multiplierSuffix`** is only ever `"%"` or `"bp"`. Any other scale marker -- a thousand / million / billion letter, in any case -- goes in `suffix`.
- **If the metric is about a ratio, assume the user will most likely prefer it formatted as a percentage and _not as a pure ratio_**, and set `multiplier` and `multiplierSuffix` as if dealing with percentages. Our terminology deviates from the statistical one: a metric named like a ratio (e.g. `Revenue / COGS Ratio`) is a percentage here, even though the literal reading would be a plain quotient.
- **`prefix` / `suffix`** are independent of `multiplier` / `multiplierSuffix` and stack on either side of the value.
- **For textual metrics**, set `textDisplayMode` -- **always use `RichText`** for comment, note, description, report, URL, link; otherwise use `Text`.

## Inference from Metric Name

**Contains `%`, `Rate`, `Ratio`, `Margin`, `Growth`, `Share`, `Yield`, `Efficiency`**: `multiplier: 100`, `multiplierSuffix: "%"`, `numFractionDigits: 1` (use `2` if name implies precision, e.g. `Margin %`).

**Contains `Revenue`, `Cost`, `Spend`, `Budget`, `Price`, `ARR`, `MRR`, `LTV`, `CAC`, `Salary`, `Fee`, `Expense`, `Income`**: `prefix: "$"` (or the currency the modeler specified). Consider `multiplier: 0.000001` + `suffix: "M"` or `multiplier: 0.001` + `suffix: "K"` when context implies scale.

**Contains `Headcount`, `Count`, `Number of`, `#`, `Units`, `Quantity`, `FTE`**: `numFractionDigits: 0`, no multiplier.

**Contains `bp`, `Basis Point`**: `multiplier: 10000`, `multiplierSuffix: "bp"`.

**Friendly name ends with `($)`**: `prefix: "$"`.

**Friendly name ends with `(%)`**: `multiplier: 100`, `multiplierSuffix: "%"`.

## Type-Based Defaults

- **`Integer`**: `numFractionDigits: 0`
- **`Number`**: `numFractionDigits: 0` (raise to `1` or `2` when name/formula implies fractional precision: rates, ratios, averages, unit prices)
- **`Boolean`**: `booleanDisplayMode: "Checkbox"`
- **`Text`**: **always use `textDisplayMode: "RichText"`** for comment, note, description, report, URL, link; otherwise `textDisplayMode: "Text"`

## Examples

**Percentage or ratio metric**:

```json
{
  "numberFormatOptions": {
    "multiplier": 100,
    "multiplierSuffix": "%",
    "numFractionDigits": 1
  }
}
```

**Currency metric in millions**:

```json
{
  "numberFormatOptions": {
    "prefix": "$",
    "multiplier": 0.000001,
    "suffix": "M",
    "numFractionDigits": 1
  }
}
```

**Headcount**:

```json
{
  "numberFormatOptions": {
    "numFractionDigits": 0
  }
}
```

**Boolean gate**:

```json
{
  "numberFormatOptions": {},
  "booleanFormatOptions": {
    "booleanDisplayMode": "Checkbox"
  }
}
```

**Rich text note**:

```json
{
  "numberFormatOptions": {},
  "textFormatOptions": {
    "textDisplayMode": "RichText"
  }
}
```

## Conditional Formatting & Highlighting

Conditional formatting rules are **UI-only**: the agent cannot apply them. Static cell highlighting is available via `tool:update_view_formatting` (`skill:designing-boards-and-views`).
