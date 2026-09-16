# Lists

### L0 · Shared base

A `<ul class="list">` of `.list-item`s. Each item is composed from an optional leading `.list-item-decoration` (bare icon or `.list-item-icon-container`), a `.list-item-content` (a `.list-item-primary` title+description block, plus a `.list-item-secondary` for trailing chips/buttons), and one of four separation styles below.

```css
.list {
  width: 100%;
  display: flex;
  flex-direction: column;
  list-style: none;
  margin: 0;
  padding: 0;
}
.list-item {
  display: flex;
  align-items: center;
  gap: var(--space-2-5);
  padding: 0 var(--space-2-5);
  box-sizing: border-box;
  transition: background-color var(--motion-snappy);
}
.list-item .list-item-content {
  flex: 1;
  display: flex;
  flex-direction: row;
  gap: var(--space-2);
  padding: var(--space-2) 0;
}
.list-item .list-item-decoration {
  min-width: var(--space-8);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--text-secondary);
}
.list-item .list-item-decoration svg {
  width: 16px;
  height: 16px;
}
.list-item .list-item-icon-container {
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--color-background-alpha);
  width: var(--space-8);
  height: var(--space-8);
  border-radius: var(--radius-sm);
}
.list-item .list-item-primary {
  flex: 1;
}
.list-item .list-item-secondary {
  display: flex;
  align-items: center;
  gap: var(--space-1);
  flex-shrink: 0;
}
.list-item.is-clickable {
  cursor: pointer;
}
.list-item.is-clickable:hover {
  background: var(--color-background-alpha);
}
```

### L1 · Row with divider

The default for a dense list on a plain page. An optional hairline between sections of rows can be added.

```html
<ul class="list">
  <li>
    <p class="type-column-title">Section</p>
  </li>
  <li class="list-item has-divider">
    <div class="list-item-decoration">
      <div class="list-item-icon-container">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" fill="currentColor">
          <path
            fill-rule="evenodd"
            d="M14.25 8.309V12a2.75 2.75 0 0 1-2.75 2.75h-7A2.75 2.75 0 0 1 1.75 12V8.309a.75.75 0 0 1-1.03-1.09l5.164-5.164a2.75 2.75 0 0 1 1.944-.805h.344c.729 0 1.428.29 1.944.805L11.78 3.72l3.5 3.5a.75.75 0 0 1-1.03 1.09ZM6.945 3.116a1.25 1.25 0 0 1 .883-.366h.344c.331 0 .649.132.883.366L10.72 4.78l1.665 1.665c.234.234.366.552.366.883V12c0 .69-.56 1.25-1.25 1.25h-.75v-3a2 2 0 0 0-2-2h-1.5a2 2 0 0 0-2 2v3H4.5c-.69 0-1.25-.56-1.25-1.25V7.328c0-.331.132-.649.366-.883l3.329-3.329ZM6.75 13.25h2.5v-3a.5.5 0 0 0-.5-.5h-1.5a.5.5 0 0 0-.5.5v3Z"
            clip-rule="evenodd"
          />
        </svg>
      </div>
    </div>
    <div class="list-item-content">
      <div class="list-item-primary">
        <p class="type-running-heavy">Label</p>
        <p class="type-running-small">Description</p>
      </div>
      <div class="list-item-secondary">
        <span
          class="chip type-chip"
          style="background: var(--color-primary-10); color: var(--color-primary-50)"
          >Urgent</span
        >
        <button class="icon-button button-secondary icon-button-medium" aria-label="Add">
          <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path
              d="M8 3V13M3 8H13"
              stroke="currentColor"
              stroke-width="1.5"
              stroke-linecap="round"
            />
          </svg>
        </button>
        <button class="icon-button button-ghost-grey icon-button-medium" aria-label="More options">
          <svg viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
            <circle cx="3" cy="8" r="1.5" />
            <circle cx="8" cy="8" r="1.5" />
            <circle cx="13" cy="8" r="1.5" />
          </svg>
        </button>
      </div>
    </div>
  </li>
</ul>
```

```css
.list-item.has-divider {
  border-bottom: 1px solid var(--border);
}
```

### L2 · Clickable row

Add `.is-clickable` (and `role="button"`) when the whole row navigates or opens something. Pair with a trailing chevron, an arrow, or a descriptive icon to anticipate the action.

```html
<li class="list-item is-clickable has-divider" role="button">
  <div class="list-item-content">
    <div class="list-item-primary">
      <p class="type-running-heavy">Label</p>
      <p class="type-running-small">Description</p>
    </div>
    <div class="list-item-secondary">
      <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M8 3V13M3 8H13" stroke="currentColor" stroke-width="2" stroke-linecap="square" />
      </svg>
    </div>
  </div>
</li>
```

### L3 · Outlined row

Each row individually boxed. Use in a page where rows need to read as discrete cards rather than a continuous sheet.

```html
<li class="list-item has-outline">
  <div class="list-item-content">
    <div class="list-item-primary">
      <p class="type-running-heavy">Label</p>
      <p class="type-running-small">Description</p>
    </div>
    <div class="list-item-secondary">
      <span class="chip type-chip">Tag</span>
    </div>
  </div>
</li>
```

```css
.list-item.has-outline {
  border: 1px solid var(--border);
  margin-bottom: var(--space-2);
  border-radius: var(--radius-sm);
}
```

### L4 · Elevated row

Each row floats as its own white card with a shadow. This is the highest-emphasis row style.

```html
<li class="list-item is-elevated">
  <div class="list-item-content">
    <div class="list-item-primary">
      <p class="type-running-heavy">Label</p>
      <p class="type-running-small">Description</p>
    </div>
    <div class="list-item-secondary">
      <span class="chip type-chip">Tag</span>
    </div>
  </div>
</li>
```

```css
.list-item.is-elevated {
  box-shadow: var(--shadow-card);
  background: var(--color-white);
  margin-bottom: var(--space-2);
  border-radius: var(--radius-sm);
}
```

### L5 · Elevated list on a surface

Wrap [L4](#l4--elevated-row) rows in `.list-surface` (a `grey-10` panel) to make the elevation read clearly against a white page.

```html
<div class="list-surface">
  <ul class="list">
    <li class="list-item is-elevated">
      <div class="list-item-content">
        <div class="list-item-primary">
          <p class="type-running-heavy">Label</p>
          <p class="type-running-small">Description</p>
        </div>
      </div>
    </li>
  </ul>
</div>
```

```css
.list-surface {
  box-sizing: border-box;
  width: 100%;
  padding: var(--space-1) var(--space-2);
  background: var(--color-grey-10);
  border-radius: var(--radius-xs);
}
```