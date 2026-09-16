# Page Headers

### PH1 · Page header

The top of a screen: [T2 Screen title](../design-system/typography.md#t2--screen-title) + an optional [T6 Running regular](../design-system/typography.md#t6--running-regular) description on the left, page actions on the right, followed by a divider with 24px margin above it. One per screen.
_Don't confuse with:_ [table-headers.md](table-headers.md) or [section-headers.md](section-headers.md) — the page header's actions slot is for major, page-level actions only (eg: export, refresh, create, submit...) or nothing at all. Filters, search, and other refinement controls never belong here, even if the page happens to be a list/table view — those go in the table header's own actions/filter row instead.

```html
<header class="page-header">
  <div class="page-header-text">
    <h1 class="type-screen-title">Scenarios</h1>
    <p class="type-running-regular" style="color: var(--text-secondary)">
      Set up versions to perform what if analysis & compare your data according to various
      assumptions
    </p>
  </div>
  <div class="page-header-actions">
    <button class="button button-secondary button-medium">
      Settings
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
      New
    </button>
  </div>
</header>
<hr class="divider" />
```

A series of [B2 Secondary](../design-system/buttons.md#b2--secondary) + a [B1 Primary](../design-system/buttons.md#b1--primary) is the default pairing for page action buttons.

```css
.page-header {
  width: 100%;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: var(--space-8);
}
.page-header-text {
  display: flex;
  flex-direction: column;
  gap: 0;
}
.page-header-actions {
  display: flex;
  align-items: center;
  gap: var(--space-2);
  flex-shrink: 0;
}
.divider {
  width: 100%;
  height: 1px;
  border: none;
  margin: var(--space-6) 0 0;
  background: var(--border);
}
```