# Chips

A 20px fully circular pill. Use one to show an attribute of an entity — a user's location on a
profile card, say — or as a compact tag in a list item. It can carry a 12px icon before or after the
label.

The default chip is neutral: `--color-background-alpha` background, `--text-secondary` label. Three
colored treatments override that inline, and nothing else is allowed:

| Treatment    | Background                            | Text / icon                     |
| ------------ | ------------------------------------- | ------------------------------- |
| Brand accent | `--color-primary-10`                  | `--color-primary-50`            |
| Status       | the family's `-10` shade              | the same family's `-90` shade   |
| Categorical  | a slot's `--color-categorical-N-bg`   | its `--color-categorical-N-fg`  |

Status chips must use the `-10` background with `-90` text pairing. Never the saturated `-50` shade
as text on a light background, and never as the fill.

### CH0 · Shared base

Every chip is a `.chip` (fully circular pill) with a `.type-chip` label. Wrap a set of chips in `.chip-row` to let them wrap onto multiple lines with a consistent gap.

```css
.chip {
  display: inline-flex;
  align-items: center;
  flex-shrink: 0;
  gap: var(--space-1);
  height: 20px;
  padding: var(--space-0-5) var(--space-2);
  border-radius: var(--radius-circular);
  background: var(--color-background-alpha);
  box-sizing: border-box;
  width: fit-content;
}
.chip svg {
  width: 12px;
  height: 12px;
  flex-shrink: 0;
}
.type-chip {
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 400;
  line-height: 12px;
  color: var(--text-secondary);
  margin: 0;
}
.chip-row {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
}
```

### CH1 · Categorical value chip

A rounded pill for tagging an item with a category, using one of the 23 categorical color slots (`-bg`/`-fg` pairs) so distinct categories stay visually distinguishable across a page.

```html
<div class="chip-row">
  <span
    class="chip type-chip"
    style="background: var(--color-categorical-1-bg); color: var(--color-categorical-1-fg)"
    >Category 1</span
  >
  <span
    class="chip type-chip"
    style="background: var(--color-categorical-2-bg); color: var(--color-categorical-2-fg)"
    >Category 2</span
  >
  <span
    class="chip type-chip"
    style="background: var(--color-categorical-3-bg); color: var(--color-categorical-3-fg)"
    >Category 3</span
  >
</div>
```

### CH2 · Chart legend pill

Pairs a small `.chip-dot` filled with a chart-palette color to a neutral label — use to build the legend for a chart, matching each series' dot to its color in order. The chip itself stays neutral; only the dot is colored. Draw the dot colors from the chart palette only: the categorical or illustrative palettes would break the color-to-series mapping.

```html
<div class="chip-row">
  <span class="chip type-chip">
    <span class="chip-dot" style="background: var(--color-chart-1)"></span>
    Label 1
  </span>
  <span class="chip type-chip">
    <span class="chip-dot" style="background: var(--color-chart-2)"></span>
    Label 2
  </span>
  <span class="chip type-chip">
    <span class="chip-dot" style="background: var(--color-chart-3)"></span>
    Label 3
  </span>
</div>
```

```css
.chip-dot {
  height: var(--space-1-5);
  width: var(--space-1-5);
  border-radius: var(--radius-circular);
  background-color: var(--color-background-alpha);
  border: 1px solid var(--border);
}
```

### CH3 · Clickable chip

Add `is-clickable` (and `role="button"`) when the chip itself is an action — a removable filter pill,
say. It then shares the interactive base of [buttons.md](buttons.md), [icon-buttons.md](icon-buttons.md)
and [toggle-buttons.md](toggle-buttons.md): the same `--motion-snappy` transition and the same
`--button-border-color`-driven inset box-shadow border and focus ring.

```html
<span class="chip type-chip is-clickable" role="button">Region: EMEA</span>
```

```css
.chip.is-clickable {
  /* Inherits the shared interactive base. That base also sets --radius-xs, which
     outranks .chip's own circular radius, so restore the pill shape here. */
  border-radius: var(--radius-circular);
}
```
