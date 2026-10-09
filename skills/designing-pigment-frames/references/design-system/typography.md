# Typography (copy-paste type classes)

### T1 · Marketing title

Rare: a hero statement on a report cover or landing-style moment. Not for in-app screens.

```html
<h1 class="type-marketing-title">Plan with confidence.</h1>
```

```css
.type-marketing-title {
  font-family: var(--font-sans);
  font-size: 56px;
  font-weight: 500;
  line-height: 68px;
  letter-spacing: -1px;
  color: var(--text-primary);
  margin: 0;
}
```

### T2 · Screen title

The page's main H1: one per screen.

```html
<h1 class="type-screen-title">Revenue Planning</h1>
```

```css
.type-screen-title {
  font-family: var(--font-sans);
  font-size: 24px;
  font-weight: 500;
  line-height: 32px;
  color: var(--text-primary);
  margin: 0;
}
```

### T3 · Content title

A secondary large heading, e.g. inside a content surface or a modal.

```html
<h2 class="type-content-title">Scenario Overview</h2>
```

```css
.type-content-title {
  font-family: var(--font-sans);
  font-size: 22px;
  font-weight: 500;
  line-height: 32px;
  color: var(--text-primary);
  margin: 0;
}
```

### T4 · Section title

Section headers within a page (card titles, panel headers, page sections).

```html
<h3 class="type-section-title">Recent Activity</h3>
```

```css
.type-section-title {
  font-family: var(--font-sans);
  font-size: 16px;
  font-weight: 600;
  line-height: 24px;
  color: var(--text-primary);
  margin: 0;
}
```

### T5 · Column title

Table/column headers, small eyebrow labels, kickers. `.type-kicker` (used for a card's eyebrow
label, e.g. [cards.md](cards.md)'s C2–C4) shares this exact style.

```html
<span class="type-column-title">Last Updated</span>
```

```css
.type-kicker,
.type-column-title {
  font-family: var(--font-sans);
  font-size: 11px;
  font-weight: 400;
  line-height: 16px;
  letter-spacing: 0.6px;
  text-transform: uppercase;
  color: var(--text-secondary);
}
```

### T6 · Running regular

Default body text and paragraph copy.

```html
<p class="type-running-regular">
  Update your forecast to reflect the latest actuals before closing the period.
</p>
```

```css
.type-running-regular {
  font-family: var(--font-sans);
  font-size: 14px;
  font-weight: 400;
  line-height: 20px;
  color: var(--text-primary);
  margin: 0;
}
```

### T7 · Running heavy

Row titles, list item titles, smaller headings, emphasized inline text at body size.

```html
<p class="type-running-heavy">Q3 Marketing Budget</p>
```

```css
.type-running-heavy {
  font-family: var(--font-sans);
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: var(--text-primary);
  margin: 0;
}
```

### T8 · Running small

Secondary/supporting text, helper text, captions, timestamps.

```html
<p class="type-running-small">Last edited 2 hours ago</p>
```

```css
.type-running-small {
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 400;
  line-height: 16px;
  color: var(--text-secondary);
  margin: 0;
}
```

### T9 · Running small heavy

Emphasized small text (badges, tags, compact labels).

```html
<p class="type-running-small-heavy">New</p>
```

```css
.type-running-small-heavy {
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 500;
  line-height: 16px;
  color: var(--text-secondary);
  margin: 0;
}
```

### T10 · Fieldset regular

Form field labels (default).

```html
<label class="type-fieldset-regular">Scenario name</label>
```

```css
.type-fieldset-regular {
  font-family: var(--font-sans);
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  color: var(--text-secondary);
}
```

### T11 · Fieldset small

Form field labels (for compact/dense forms).

```html
<label class="type-fieldset-small">Currency</label>
```

```css
.type-fieldset-small {
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 500;
  line-height: 16px;
  color: var(--text-secondary);
}
```

### T12 · Highlight figure

Large emphasized figures: KPI values, key stat callouts on cards.

```html
<span class="type-highlight-figure">+2,450</span>
```

```css
.type-highlight-figure {
  font-family: var(--font-sans);
  font-size: 28px;
  font-weight: 500;
  line-height: 36px;
  color: var(--text-primary);
  margin: 0;
}
```

### T13 · Legal notice

Rare: Fine print, footnotes, legal text.

```html
<p class="type-legal-notice">Figures are illustrative and subject to change.</p>
```

```css
.type-legal-notice {
  font-family: var(--font-sans);
  font-size: 10px;
  font-weight: 400;
  line-height: 12px;
  color: var(--text-secondary);
  margin: 0;
}
```

### T14 · Code running regular

Formulas and code, exclusively. Never table figures.

```html
<code class="type-code-running-regular">SUM(Revenue) / SUM(Headcount)</code>
```

```css
.type-code-running-regular {
  font-family: var(--font-mono);
  font-size: 14px;
  font-weight: 400;
  line-height: 20px;
  color: var(--text-primary);
}
```

### T15 · Code running small

Formulas and code, exclusively. Never table figures. Smaller variant for denser displays.

```html
<code class="type-code-running-small">=A1+B2</code>
```

```css
.type-code-running-small {
  font-family: var(--font-mono);
  font-size: 12px;
  font-weight: 400;
  line-height: 16px;
  color: var(--text-primary);
}
```

### T16 · Chart label

Axis ticks and small in-chart annotations (e.g. a date under a bar).
_Don't confuse with:_ T5 Column title — same 11px/16px size, but this is sentence case with no letter-spacing, never uppercase/tracked.

```html
<code class="type-chart-label">Jan 26</code>
```

```css
.type-chart-label {
  font-family: var(--font-sans);
  font-size: 11px;
  font-weight: 400;
  line-height: 16px;
  color: var(--text-secondary);
  margin: 0;
}
```

### T17 · Anchor link

An inline text link, e.g. a "Learn more" at the end of a paragraph (see [cards.md](cards.md)'s C3).

```html
<a href="#" class="anchor-link">Learn more</a>
```

```css
.anchor-link {
  font-family: var(--font-sans);
  font-weight: 400;
  color: var(--text-highlight);
  text-decoration: underline;
  text-underline-offset: var(--space-0-5);
  text-decoration-thickness: 1px;
  width: fit-content;
}
```