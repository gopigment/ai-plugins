---
name: formatting-and-highlighting
description: "Best practices for metric default formatting. Covers: decimals, prefix, suffix, currency ($/€), percent (%), K/M/bp/thousand/million scaling, thousand separator, sign / zero / negative handling, text mode (Text / Rich Text / URL / Image / LocaleDateTime), boolean display (checkbox / button). Load when creating or updating a metric's default format. Use cases: format as, display as, show as percent, in millions, two decimals, no decimals, prefix with $, add currency, as K / M / bp, rich text, checkbox, ratio, url, multiplier."
metadata:
  skill_path: /formatting-and-highlighting/SKILL.md
  base_directory: /formatting-and-highlighting
  includes:
    - "*.md"
---

# How to Use This Skill

**Progressive Disclosure Pattern**: This `SKILL.md` is the entry point. As more formatting topics land (conditional formatting, highlighting), they will live as sibling files in this directory.

**Required workflow**:

1. **Read this file first** - Understand the rules, schema, and inference tables.
2. **Apply the inference** - Choose default format from the metric's name and type before calling `tool:create_metric` / `tool:update_metric`.
3. **Honor modeler context** - If the user stated a formatting preference, that overrides inference for the rest of the session.

---

# When to Use

Load this skill when:

- **Creating a Metric** (`tool:create_metric`) — Creating a Metric that you intend to use to display data in a board.
- **Updating a Metric's formatting** (`tool:update_metric` with default format) — any request that touches decimals, prefix, suffix, currency, percent, K/M/bp scaling, thousand separator, sign / zero / negative handling, text mode, or boolean display.

Use cases in which this skill is useful: "show as percent", "in millions", "no decimals", "two decimals", "prefix with $", "currency", "thousand separator", "format as", "display as", "change decimals", "show as K", "basis points", "rich text", "checkbox".

Skip this skill for `tool:update_metric` calls that **don't** touch default format (renaming a metric, changing its dimensions, editing its description).

Out of scope for this skill:

- View display modes, aggregators, sort, filter.
- Conditional formatting — UI-only today.
- Static cell formatting (background/text color, bold, italic, alignment) — set on the view via `skill:designing-views` (`tool:update_view_formatting`), not here.

---

# CRITICAL RULES

- **Set formatting on the metric, not on the view.** default format lives on `tool:create_metric` / `tool:update_metric`. Views have no number-formatting tools. A metric's default format applies to every View, Board, KPI, Grid, and Chart that displays it. Set it once on the metric.
- **`numberFormatOptions` is required inside default format.** Pass `{}` if you have nothing to set for number formatting (e.g. you only want to set `textFormatOptions`). Do not omit the key.
- **Omitting a field leaves it unset.** There is no way to "clear" an individual field through this input.
- **Modeler context overrides inference.** If the modeler states a preference in the conversation ("use 0 decimals by default", "prefix all financial metrics with €", "no thousand separator"), apply it consistently for all metrics created in that session and skip the name-based rules below.
- **It is almost always a good idea to consider `multiplier` and `multiplierSuffix`** when working with `Number` format or percentages (see Inference from metric name and Examples)
- For textual metrics, **see list of use cases below for which you should always use `RichText`, otherwise use `Text`**

---

**Important nuances:**

- `multiplier` is **numeric**, not a string alias. Use `100` for percent, `0.001` for thousands, `0.000001` for millions, `10000` for basis points.
- `multiplierSuffix` is the visual marker (`"%"`, `"bp"`) shown after the value. Pair it with the matching `multiplier`. It should be used only for percentage and basis points, for other use cases prefer `suffix`
- If the metric is about ratio, assume that a user will mostly like prefer to have it formatted as a percentage and **not as a pure ratio**, and set `multiplier` and `multiplierSuffix` as if dealing with percentages
- `prefix` / `suffix` are independent of `multiplier` / `multiplierSuffix` and stack on either side of the value.
- if your metric is a textual metric, try to set `textDisplayMode` - see rules below when to apply `RichText` and when `Text`

---

# Inference from metric name

| Name signal | Formatting to apply |
|---|---|
| Contains `%`, `Rate`, `Ratio`, `Margin`, `Growth`, `Share`, `Yield`, `Efficiency` | `multiplier: 100`, `multiplierSuffix: "%"`, `numFractionDigits: 1` (use `2` if name implies precision, e.g. `Margin %`) |
| Contains `Revenue`, `Cost`, `Spend`, `Budget`, `Price`, `ARR`, `MRR`, `LTV`, `CAC`, `Salary`, `Fee`, `Expense`, `Income` | `prefix: "$"` (or the currency the modeler specified). Consider `multiplier: 0.000001` + `multiplierSuffix: "M"` or `multiplier: 0.001` + `multiplierSuffix: "K"` when context implies scale (e.g. "in millions") |
| Contains `Headcount`, `Count`, `Number of`, `#`, `Units`, `Quantity`, `FTE` | `numFractionDigits: 0`, no multiplier |
| Contains `bp`, `Basis Point` | `multiplier: 10000`, `multiplierSuffix: "bp"` |
| Friendly name ends with `($)` | `prefix: "$"` |
| Friendly name ends with `(%)` | `multiplier: 100`, `multiplierSuffix: "%"` |

---

# Type-based defaults

| Metric type | Default |
|---|---|
| `Integer` | use `numFractionDigits: 0` |
| `Number` | use `numFractionDigits: 0` (raise to `1` or `2` when name/formula implies fractional precision: rates, ratios, averages, unit prices). |
| `Boolean` | use `booleanDisplayMode: "Checkbox"` |
| `Text` | if  metrics about `comment`, `note`, `description`, `report`, `URL`, `link` **always use `textDisplayMode: "RichText"`** else use `textDisplayMode: "Text"` |

---

# Examples

**Percentage or ratio metric** — numerical metrics, useful for ratios, rates, percentages:

```json
{
  "numberFormatOptions": {
    "multiplier": 100,
    "multiplierSuffix": "%",
    "numFractionDigits": 1
  }
}
```

**Currency metric in millions** — numerical metrics, useful for display of large numbers, revenues, large units:

```json
{
  "numberFormatOptions": {
    "prefix": "$",
    "multiplier": 0.000001,
    "multiplierSuffix": "M",
    "numFractionDigits": 1
  }
}
```

**Headcount** — numerical metrics, useful for scales in hundreds and thousands, where decimals make no sense:

```json
{
  "numberFormatOptions": {
    "numFractionDigits": 0
  }
}
```

**Boolean gate** — boolean metrics, default formatting:

```json
{
  "numberFormatOptions": {},
  "booleanFormatOptions": {
    "booleanDisplayMode": "Checkbox"
  }
}
```

**Rich text note** — textual metrics, for example `comment`, `note`, `description`, `report`, `URL`, `link`: 

```json
{
  "numberFormatOptions": {},
  "textFormatOptions": {
    "textDisplayMode": "RichText"
  }
}
```

---

# Conditional formatting & highlighting

Conditional formatting rules are **UI-only today** — the agent cannot apply them. Static cell highlighting (background/text color, bold, italic, alignment) is available via `tool:update_view_formatting` (`skill:designing-views`).