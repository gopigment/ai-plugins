# Cards & Tiles

The generic content container for dashboards, reports and grouped information. All three surface
variants share `--radius-xs` corners and 16px internal padding.

A card composes from an optional `column-title` eyebrow label plus an overflow icon button in a
header row, an optional square icon tile, a title tightly paired with a `highlight-figure` value, an
optional status chip as its own sibling (never grouped with the title and figure), body copy in
`running-regular` / `--text-secondary`, and an optional footer row of actions.

### C0 · Shared base + surface variants

Every card is a `.card` plus **exactly one** surface modifier — never a visible border and a shadow on the same card, and never a border and a filled background. `is-outlined` is the default choice; reach for `is-elevated` to lift a card above a flat page, and `is-filled` to single it out as a `background-alpha`-colored surface.

```html
<div class="card-grid">
  <div class="card is-outlined">…</div>
  <div class="card is-elevated">…</div>
  <div class="card is-filled">…</div>
</div>
```

```css
.card-grid {
  width: 100%;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: var(--space-4);
}
.card {
  display: flex;
  flex-direction: column;
  gap: var(--space-4);
  box-sizing: border-box;
  padding: var(--space-4);
  background: var(--color-white);
  border-radius: var(--radius-xs);
}
.card.is-outlined {
  border: 1px solid var(--border);
}
.card.is-elevated {
  box-shadow: var(--shadow-card);
}
.card.is-filled {
  background: var(--bg-secondary-surface);
}
```

### C1 · Basic card

Just a title and a description, this is the minimum viable card.

```html
<div class="card is-outlined">
  <div class="card-grouped-text">
    <h3 class="type-running-heavy">Insights</h3>
    <p class="type-running-regular" style="color: var(--text-secondary)">
      Specialized agent for data analysis and report generation
    </p>
  </div>
</div>
```

```css
.card-grouped-text {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
}
```

### C2 · Card with complete hierarchy

Header row (kicker + overflow menu), title tightly paired with a highlighted figure, a status chip as its own sibling, body copy, and a footer of actions.
_Don't confuse with:_ C4 Value card, which also uses `.card-grouped-figure-text` but for a figure + short context line, not a title + figure. Also don't confuse `card-header-row`'s small `column-title` kicker with a [Section header](../patterns/section-headers.md) — any card's full title/description/actions (e.g. a chart's breakdown toggles) always live in a section header sitting outside and above the card, never inside this row.

```html
<div class="card is-outlined">
  <div class="card-header-row">
    <span class="type-kicker">Finance</span>
    <button class="icon-button button-ghost-grey icon-button-medium" aria-label="menu">
      <svg viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
        <circle cx="3" cy="8" r="1.5" />
        <circle cx="8" cy="8" r="1.5" />
        <circle cx="13" cy="8" r="1.5" />
      </svg>
    </button>
  </div>
  <div class="card-grouped-figure-text">
    <h3 class="type-content-title">Travel Expenses</h3>
    <span class="type-highlight-figure">+2,000</span>
  </div>
  <span
    class="chip type-chip"
    style="background: var(--color-positive-20); color: var(--color-positive-90)"
    >+ 42% vs last month</span
  >
  <p class="type-running-regular" style="color: var(--text-secondary)">
    Increase driven by the return of more frequent in-person client meetings, resulting in higher
    travel, accommodation, and related business expenses compared with the previous period.
  </p>
  <div class="card-footer">
    <button class="button button-primary button-medium">
      <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path
          d="M2.667 8L5.967 11.3L13.038 4.229"
          stroke="currentColor"
          stroke-width="1.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
      </svg>
      Approve
    </button>
    <button class="button button-secondary button-medium">Reject</button>
    <button class="icon-button button-secondary icon-button-medium" aria-label="Flag">
      <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path
          d="M2.667 14V10.458M2.667 10.458C6.545 7.425 9.455 13.491 13.333 10.458V2.876C9.455 5.909 6.545 -0.157 2.667 2.875V10.458Z"
          stroke="currentColor"
          stroke-width="1.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
      </svg>
    </button>
  </div>
</div>
```

```css
.card-header-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-2);
}
.card-footer {
  margin-top: auto;
  padding-top: var(--space-3);
  display: flex;
  align-items: center;
  gap: var(--space-2);
}
```

### C3 · Textual card with icon

A decorative icon tile above a kicker + title + body, e.g. for a marketing feature grid.

```html
<div class="card is-outlined">
  <div class="card-icon">
    <svg fill="currentColor" viewBox="0 0 16 16" xmlns="http://www.w3.org/2000/svg">
      <path
        clip-rule="evenodd"
        d="M8.965.645a.75.75 0 0 1 .446.781l-.561 4.49H14a.75.75 0 0 1 .576 1.23l-6.667 8a.75.75 0 0 1-1.32-.572l.561-4.49H2a.75.75 0 0 1-.576-1.23l6.667-8a.75.75 0 0 1 .874-.209ZM3.601 8.583H8a.75.75 0 0 1 .744.843l-.35 2.795L12.4 7.417H8a.75.75 0 0 1-.744-.843l.35-2.795L3.6 8.583Z"
      />
    </svg>
  </div>
  <div class="card-grouped-text">
    <span class="type-kicker" style="color: var(--text-secondary)">For Retail & CPG</span>
    <h3 class="type-content-title" style="color: var(--text-primary)">
      Help teams make better merchandising and planning decisions
    </h3>
  </div>
  <p class="type-running-regular" style="color: var(--text-secondary)">
    Turn market signals into clearer next steps and more confident pricing, promotions, and
    assortment decisions. <a href="#" class="anchor-link">Learn more</a>
  </p>
</div>
```

```css
.card-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: var(--space-12);
  height: var(--space-12);
  background: var(--color-primary-light-transparent);
  color: var(--text-highlight);
  border-radius: var(--radius-xs);
}
.card-icon svg {
  width: var(--space-4);
  height: var(--space-4);
}
.card.is-filled .card-icon {
  background: var(--color-background-alpha);
  color: var(--text-primary);
}
```

### C4 · Value card

A kicker, a large figure, and a short context line. For a KPI tile in a dashboard.
_Don't confuse with:_ C0's `.card-grouped-text` (4px gap, used everywhere else two text blocks stack) — a value card's figure+context pair uses `.card-grouped-figure-text` instead, which has no gap, since a highlight figure needs to sit flush against its context line rather than at the standard text-stack spacing.

```html
<div class="card is-outlined">
  <span class="type-kicker">Active Users</span>
  <div class="card-grouped-figure-text">
    <span class="type-highlight-figure" style="color: var(--text-primary)">1,204</span>
    <span class="type-running-regular" style="color: var(--text-secondary)">Dec 26</span>
  </div>
</div>
```

```css
.card-grouped-figure-text {
  display: flex;
  flex-direction: column;
  gap: 0;
}
```

### C5 · Statistics card

A key figure paired with a longer descriptive paragraph. For marketing/statement pages.

```html
<div class="card is-outlined">
  <div class="card-grouped-text">
    <span class="type-highlight-figure" style="color: var(--text-primary)">7x</span>
    <p class="type-running-regular" style="color: var(--text-secondary)">
      As organizations expand across hybrid, multicloud, SaaS and AI-powered environments,
      privileged access becomes the most valuable target for attackers.
    </p>
  </div>
</div>
```

### C6 · Editorial card (gradient)

A soft-gradient background with expressive colored heading text. Reserved for presentational interfaces or as a means to highlight one specific card above all others in an interface.

```html
<div
  class="card is-filled has-gradient"
  style="
    --card-base-background-color: var(--color-cobalt-soft);
    --card-glow-color-left: var(--color-amethyst-vivid);
    --card-glow-color-right: var(--color-amethyst-soft);
  "
>
  <div class="card-header-row">
    <span class="type-kicker">Financial Planning</span>
    <button class="icon-button button-ghost-grey icon-button-medium" aria-label="menu">
      <svg viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
        <circle cx="3" cy="8" r="1.5" />
        <circle cx="8" cy="8" r="1.5" />
        <circle cx="13" cy="8" r="1.5" />
      </svg>
    </button>
  </div>
  <h3 class="type-content-title" style="color: var(--color-cobalt-bold)">
    Plan how you need, not how you're told.
  </h3>
  <p class="type-running-regular" style="color: var(--text-secondary)">
    Pigment fits how your business operates, no matter how complex.
  </p>
</div>
```

```css
.card.has-gradient {
  background-color: var(--card-base-background-color);
  background-image:
    linear-gradient(6deg, rgba(255, 255, 255, 0.8) 20%, transparent 80%),
    linear-gradient(12deg, var(--card-base-background-color) 0%, transparent 100%),
    radial-gradient(1000% 100% at 0% 100%, var(--card-base-background-color), transparent),
    radial-gradient(356px circle at 120% -20%, var(--card-glow-color-right), transparent),
    radial-gradient(128px circle at 0 -10%, var(--card-glow-color-left), transparent);
}
```

### C7 · Clickable elevated card

The hover-lift (`shadow-card-hover`) is opt-in, not a default of `is-elevated` — add `is-clickable` only when the whole card genuinely navigates or opens something, e.g. a dashboard tile. A plain `is-elevated` card without this class stays static on hover.
_Don't confuse with:_ C0's `is-elevated` alone (a static "lifted" card with no interactive affordance).

```html
<div class="card is-elevated is-clickable" role="button">
  <div class="card-grouped-text">
    <h3 class="type-running-heavy">Insights</h3>
    <p class="type-running-regular" style="color: var(--text-secondary)">
      Specialized agent for data analysis and report generation
    </p>
  </div>
</div>
```

```css
.card.is-clickable {
  cursor: pointer;
}
.card.is-elevated.is-clickable:hover {
  box-shadow: var(--shadow-card-hover);
}
```

### C8 · Empty / placeholder state

No recipe — compose it from the base above. Centre the content and give the card generous vertical
padding: 48px, or 16px in a compact context. The content is an icon, a short message, and optionally
one action.
