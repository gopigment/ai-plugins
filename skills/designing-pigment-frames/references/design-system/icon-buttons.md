# Icon Buttons

A square, icon-only counterpart to [buttons.md](buttons.md): same shape language, same 2px inset
border, same sizing rhythm, but width equals height and a single 16px icon sits centered inside with
no visible label. Always pair it with an `aria-label`.

| Size             | Width × Height | Padding |
| ---------------- | -------------- | ------- |
| Medium (default) | 32px           | 8px     |
| Small            | 28px           | 6px     |

Only four of the six Button variants apply. There is no `danger-*` and no dark-surface `transparent`
icon button: a destructive action must be unambiguous, so use a labeled Button for it instead.

`button-ghost-grey` is the default for neutral, non-brand icon-only actions — overflow menus, close
buttons, row-level actions. Reserve `button-primary` / `button-secondary` / `button-ghost` for
brand-colored actions that would otherwise be a labeled Button.

### IB0 · Shared base

Compose icon buttons by combining a size (`icon-button-medium` / `icon-button-small`) and a color variant (`button-primary` / `button-secondary` / `button-ghost` / `button-ghost-grey`).

```css
.icon-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  border: none;
  border-radius: var(--radius-xs);
  box-shadow: inset 0 0 0 2px var(--button-border-color, transparent);
  cursor: pointer;
  transition:
    background-color var(--motion-snappy),
    box-shadow var(--motion-snappy),
    color var(--motion-snappy);
}
.icon-button:focus-visible {
  outline: none;
  box-shadow:
    inset 0 0 0 2px var(--button-border-color, transparent),
    var(--focus-ring);
}
.icon-button svg {
  width: 16px;
  height: 16px;
}
.icon-button-medium {
  padding: var(--space-2);
}
.icon-button-small {
  padding: var(--space-1-5);
}
.icon-button[disabled] {
  color: var(--text-disabled);
  cursor: not-allowed;
  pointer-events: none;
}
```

### IB1 · Primary

The one high-emphasis action, e.g. an "Add" button on a list.

```html
<button class="icon-button button-primary icon-button-medium" aria-label="Add">
  <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
    <path d="M8 3V13M3 8H13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" />
  </svg>
</button>
```

```css
.button-primary {
  background: var(--color-primary-50);
  --button-border-color: var(--color-primary-50);
  color: var(--text-light-primary);
}
.button-primary:hover {
  background: var(--color-primary-90);
  --button-border-color: var(--color-primary-90);
}
.button-primary[disabled] {
  background: var(--color-grey-10);
  --button-border-color: var(--color-grey-10);
}
```

### IB2 · Secondary

A tinted icon action, e.g. "Edit" next to a row's primary content. The default icon button to reach for.

```html
<button class="icon-button button-secondary icon-button-medium" aria-label="Edit">
  <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
    <path d="M8 3V13M3 8H13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" />
  </svg>
</button>
```

```css
.button-secondary {
  background: var(--color-primary-light-transparent);
  --button-border-color: transparent;
  color: var(--color-primary-50);
}
.button-secondary:hover {
  --button-border-color: var(--color-primary-light-transparent);
}
.button-secondary[disabled] {
  background: var(--color-grey-10);
  --button-border-color: var(--color-grey-10);
}
```

### IB3 · Ghost

A brand-colored icon action with no background until hovered, e.g. a "Favorite" toggle. Use only on busy compositions with multiple icon buttons as a way to create a hierarchy from [Primary](#ib1--primary) (high) to [Ghost](#ib3--ghost) (low).

```html
<button class="icon-button button-ghost icon-button-medium" aria-label="Favorite">
  <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
    <path
      d="M2.951 8.208L8 13.257L13.049 8.208C14.317 6.940 14.317 4.885 13.049 3.617C11.782 2.350 9.727 2.350 8.459 3.617L8 4.076L7.541 3.617C6.273 2.350 4.218 2.350 2.951 3.617C1.683 4.885 1.683 6.940 2.951 8.208Z"
      stroke="currentColor"
      stroke-width="1.5"
      stroke-linejoin="round"
    />
  </svg>
</button>
```

```css
.button-ghost {
  background: transparent;
  --button-border-color: transparent;
  color: var(--color-primary-50);
}
.button-ghost:hover {
  background: var(--color-primary-light-transparent);
}
.button-ghost[disabled] {
  background: transparent;
  --button-border-color: transparent;
}
```

### IB4 · Ghost-grey

The default for neutral actions: overflow menus, dismissing controls, "more options".

```html
<button class="icon-button button-ghost-grey icon-button-medium" aria-label="More options">
  <svg viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
    <circle cx="3" cy="8" r="1.3" />
    <circle cx="8" cy="8" r="1.3" />
    <circle cx="13" cy="8" r="1.3" />
  </svg>
</button>
```

```css
.button-ghost-grey {
  background: transparent;
  --button-border-color: transparent;
  color: var(--text-secondary);
}
.button-ghost-grey:hover {
  background: var(--color-background-alpha);
  color: var(--text-primary);
}
.button-ghost-grey[disabled] {
  background: transparent;
  --button-border-color: transparent;
}
```
