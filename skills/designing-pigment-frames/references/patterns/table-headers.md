# Tables

The header that introduces a table, and the table surface and rows themselves.

### TH1 · Table header

Headlines a specific table or list. Built as a [T4 Section title](../design-system/typography.md#t4--section-title) with an optional `text-disabled` count, with main actions (eg: search, add) on the right. It's a [section header](section-headers.md) in every sense — sits outside and above the table's own surface, 12px before it — this file just documents the table-specific content of that header.

```html
<header class="table-header">
  <div class="table-header-text">
    <h3 class="type-section-title">
      Employees <span style="color: var(--text-disabled)">64</span>
    </h3>
    <p class="type-running-small" style="color: var(--text-secondary)">
      Review and manage your team members
    </p>
  </div>
  <div class="table-header-actions">
    <div class="text-input" style="width: 240px">
      <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
        <circle cx="7" cy="7" r="5" stroke="currentColor" stroke-width="1.5" />
        <path d="M11 11L14 14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" />
      </svg>
      <input type="text" placeholder="Search employees" />
    </div>
    <button class="button button-primary button-medium">
      <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M8 3V13M3 8H13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" />
      </svg>
      New employee
    </button>
  </div>
</header>
```

The search input is [inputs.md](../design-system/inputs.md)'s IN2 Search input; the button is [buttons.md](../design-system/buttons.md)'s B1 Primary.

```css
.table-header {
  width: 100%;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: var(--space-8);
}
.table-header-text {
  display: flex;
  flex-direction: column;
  gap: 0;
}
.table-header-actions {
  display: flex;
  align-items: center;
  gap: var(--space-2);
  flex-shrink: 0;
}
```

### TH2 · Table header with filter row

For more elaborate needs: split TH1 into two rows: the title row (no actions), then a `.filter-header` row with primary actions (eg: search) on the left and secondary actions (eg: selects, "Add filter"/"Add sort") on the right. The filtering row can shaped as needed depending on the intended usage (eg: a searchbar on the left and select filters on the right, select filters on the left and a single-select set of toggle buttons on the right).

```html
<div style="display: flex; flex-direction: column; gap: var(--space-1); width: 100%">
  <header class="table-header">
    <div class="table-header-text">
      <h3 class="type-section-title">
        Employees <span style="color: var(--text-disabled)">64</span>
      </h3>
    </div>
  </header>
  <section class="filter-header">
    <div class="filter-header-primary-actions">
      <div class="text-input">
        <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
          <circle cx="7" cy="7" r="5" stroke="currentColor" stroke-width="1.5" />
          <path d="M11 11L14 14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" />
        </svg>
        <input type="text" placeholder="Search employees" />
      </div>
    </div>
    <div class="filter-header-secondary-actions">
      <div class="field">
        <div class="text-input">
          <select>
            <option>All roles</option>
            <option>Sales</option>
            <option>Engineering</option>
          </select>
          <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path
              d="M4 6l4 4 4-4"
              stroke="currentColor"
              stroke-width="1.5"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
          </svg>
        </div>
      </div>
      <button class="button button-secondary button-medium">
        <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path
            d="M8 3V13M3 8H13"
            stroke="currentColor"
            stroke-width="1.5"
            stroke-linecap="round"
          />
        </svg>
        Add filter
      </button>
      <button class="button button-secondary button-medium">
        <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path
            d="M8 3V13M3 8H13"
            stroke="currentColor"
            stroke-width="1.5"
            stroke-linecap="round"
          />
        </svg>
        Add sort
      </button>
    </div>
  </section>
</div>
```

```css
.filter-header {
  width: 100%;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: var(--space-8);
}
.filter-header-primary-actions {
  display: flex;
  flex-direction: column;
  gap: 0;
}
.filter-header-secondary-actions {
  display: flex;
  align-items: center;
  gap: var(--space-2);
  flex-shrink: 0;
}
```

### TR1 · Table surface and rows

No recipe — compose it from the values below. The header above sits 12px on top of this surface,
outside it.

- **Surface** — the bordered-flat idiom by default: `--color-white` background,
  `1px solid var(--border)`, `--radius-xs` corners. Never elevated and never filled.
- **Header row** — white background, a `1px solid var(--color-grey-20)` bottom border, and
  `column-title` labels. If the table scrolls vertically the header is sticky, and it must keep that
  bottom border visible above the scrolling content: give it its own background and a `z-index`
  above the rows, not just `position: sticky`, or the border ends up hidden behind the next row.
- **Rows** — 16px left and right edge padding so content never sits flush against the surface
  border, and 12px vertical padding. Hover background `--color-background-alpha`, selected-row
  background `--color-primary-light-transparent`.
- **Cells** — space the cells within a row with a consistent 24px horizontal gap, not with
  per-cell horizontal padding.
- **Row dividers** — every row gets a `1px solid var(--border)` bottom border except the last. Omit
  it there, or it doubles up with the surface's own bottom border.
