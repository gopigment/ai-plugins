# Tooltips

Tooltips are **always dark**, whatever the surface behind them: `--color-grey-90` background,
`--radius-xs` corners, `--shadow-floating-container`, and a 420px max width. There is no light
variant.

A long label in a `dt` / `dd` list truncates with an ellipsis. It must not wrap and must not
overflow the container.

### TT0 · Shared base

A dark floating container — `.tooltip` provides the surface, and either `.tooltip-grouped-text` (a simple label+value stack) or `.tooltip-list` (a `<dl>` of label/value pairs) fills it. Text inside always uses `text-light-primary`/`text-light-secondary`, never the default `text-primary`/`text-secondary` (those are tuned for light surfaces).

```css
.tooltip {
  position: relative;
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  background: var(--color-grey-90);
  padding: var(--space-2-5);
  border-radius: var(--radius-xs);
  box-shadow: var(--shadow-floating-container);
  max-width: 420px;
}
.tooltip-grouped-text {
  display: flex;
  flex-direction: column;
  gap: 0;
}
.tooltip-list {
  display: grid;
  grid-template-columns: auto 1fr;
  grid-column-gap: var(--space-4);
  grid-row-gap: var(--space-1);
  margin: 0;
}
.tooltip-list dt {
  color: var(--text-light-secondary);
  overflow: hidden;
  text-overflow: ellipsis;
  text-wrap: nowrap;
}
.tooltip-list dd {
  color: var(--text-light-primary);
}
.tooltip .divider {
  background-color: rgba(255, 255, 255, 0.24);
  margin: 0;
}
```

### TT1 · Figure value

A single emphasized figure with a small label above it — the simplest tooltip, for hovering a single data point (e.g. one bar in a chart).
_Don't confuse with:_ TT2 Breakdown of value (multiple figures, one per series) or [cards.md](cards.md)'s C4 Value card (a persistent on-page tile, not a hover overlay).

```html
<div class="tooltip">
  <div class="tooltip-grouped-text">
    <h6 class="type-running-small" style="color: var(--text-light-secondary)">May 26</h6>
    <p class="type-section-title" style="color: var(--text-light-primary)">$100,000</p>
  </div>
</div>
```

### TT2 · Breakdown of value

A `<dl>` of label/value pairs, each label prefixed with a small colored `.series-dot` — for hovering a stacked or multi-series chart point, breaking the total down by series.
_Don't confuse with:_ [chips.md](chips.md)'s CH2 Chart legend pill — same dot-and-color idea, but CH2 is a persistent on-page legend chip, while `.series-dot` here is scoped to a single hover tooltip's `<dt>` and always pairs with a value in the adjacent `<dd>`.

```html
<div class="tooltip">
  <dl class="tooltip-list">
    <dt class="type-running-small">
      <span class="series-dot" style="background: var(--color-chart-1)"></span>France
    </dt>
    <dd class="type-running-small-heavy">$100,000</dd>
    <dt class="type-running-small">
      <span class="series-dot" style="background: var(--color-chart-2)"></span>Germany
    </dt>
    <dd class="type-running-small-heavy">$100,000</dd>
    <dt class="type-running-small">
      <span class="series-dot" style="background: var(--color-chart-3)"></span>Italy
    </dt>
    <dd class="type-running-small-heavy">$100,000</dd>
  </dl>
</div>
```

```css
.tooltip-list dt .series-dot {
  display: inline-block;
  width: var(--space-1-5);
  height: var(--space-1-5);
  border-radius: var(--radius-circular);
  background: var(--color-background-alpha);
  border: 1px solid var(--border);
  margin-right: var(--space-1);
}
```

### TT3 · Object attributes

A title, a divider, then a `<dl>` of plain (no dot) label/value pairs — for hovering an entity (e.g. a job posting, a user) to reveal a handful of its attributes.
_Don't confuse with:_ TT2 (its `<dt>`s carry a colored series dot; this variant's don't).

```html
<div class="tooltip">
  <h6 class="type-running-heavy" style="color: var(--text-light-primary)">
    Senior Frontend Engineer
  </h6>
  <hr class="divider" />
  <dl class="tooltip-list">
    <dt class="type-running-small">Location</dt>
    <dd class="type-running-small-heavy">San Francisco, CA</dd>
    <dt class="type-running-small">Salary</dt>
    <dd class="type-running-small-heavy">$120,000 - $140,000</dd>
    <dt class="type-running-small">Posted</dt>
    <dd class="type-running-small-heavy">2 days ago</dd>
  </dl>
</div>
```
