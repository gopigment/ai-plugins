# Section Headers

### SH1 · Section header

A similar left-text/right-actions layout as [page-headers.md](page-headers.md), scaled down: [T4 Section title](../design-system/typography.md#t4--section-title) + an optional [T8 Running small](../design-system/typography.md#t8--running-small) (secondary) description, no trailing divider. Use to introduce a section within a screen.
_Don't confuse with:_ a card's own `card-header-row` (see [cards.md](../design-system/cards.md)'s C2) — that's a small `column-title` eyebrow label for content already inside a card, not this. A section header always sits on its own, outside and above whatever surface it introduces — a card, a table, a list, a chart — never merged into that surface. E.g. a chart's title/description/actions go in the section header, then the chart itself lives in a separate bordered card below it, 12px below this header.

Space it 12px above the content surface it introduces, and 48px above/below neighboring, unrelated sections.

```html
<header class="section-header">
  <div class="section-header-text">
    <h2 class="type-section-title">Essential contacts</h2>
    <p class="type-running-small" style="color: var(--text-secondary)">
      Ensure the right stakeholders are reachable for time-sensitive notifications
    </p>
  </div>
  <div class="section-header-actions">
    <button class="button button-secondary button-medium">
      Manage
      <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path
          d="M4 6l4 4 4-4"
          stroke="currentColor"
          stroke-width="1.5"
          stroke-linecap="round"
          stroke-linejoin="round"
        />
      </svg>
    </button>
    <button class="button button-primary button-medium">
      <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M8 3V13M3 8H13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" />
      </svg>
      Add
    </button>
  </div>
</header>
```

```css
.section-header {
  width: 100%;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: var(--space-8);
}
.section-header-text {
  display: flex;
  flex-direction: column;
  gap: 0;
}
.section-header-actions {
  display: flex;
  align-items: center;
  gap: var(--space-2);
  flex-shrink: 0;
}
```