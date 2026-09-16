# Buttons

Six color variants, two sizes. Every button variant below builds on the shared base. Compose a button
by combining a size (`button-medium` / `button-small`) and a color variant (`button-primary` /
`button-secondary` / `button-ghost` / `button-danger-primary` / `button-danger-secondary` /
`button-transparent`).

| Size             | Height | Padding |
| ---------------- | ------ | ------- |
| Medium (default) | 32px   | 6px 12px |
| Small            | 28px   | 4px 10px |

The height is not set: it falls out of the padding plus the 20px `button-label` line-height. Keep both
sizes matched to the other 32px/28px controls (icon button, input, toggle button) in the same row.

The border is always 2px wide, even when it is transparent, so every variant aligns on the same box.
It is drawn as an inset `box-shadow`, not a `border`, so it composes with the focus ring.

### B0 · Shared base

```css
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-1);
  border: none;
  border-radius: var(--radius-xs);
  box-shadow: inset 0 0 0 2px var(--button-border-color, transparent);
  cursor: pointer;
  transition:
    background-color var(--motion-snappy),
    box-shadow var(--motion-snappy),
    color var(--motion-snappy);
}
.button svg {
  width: 16px;
  height: 16px;
}
.button:focus-visible {
  outline: none;
  box-shadow:
    inset 0 0 0 2px var(--button-border-color, transparent),
    var(--focus-ring);
}
.button-medium {
  padding: var(--space-1-5) var(--space-3);
}
.button-small {
  padding: var(--space-1) var(--space-2-5);
}
.button[disabled] {
  color: var(--text-disabled);
  cursor: not-allowed;
  pointer-events: none;
}
```

### B1 · Primary

The one high-emphasis CTA on a screen. A surface (the page, a section, a card) should have at most one visible primary button at a time. Every other action on that surface is `secondary`, `ghost`, or a plain text link.

```html
<button class="button button-primary button-medium">Save changes</button>
<button class="button button-primary button-small">Save changes</button>
<button class="button button-primary button-medium" disabled>Save changes</button>
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

### B2 · Secondary

A tinted, medium-emphasis action alongside a primary button. The default button to reach for.

```html
<button class="button button-secondary button-medium">Add filter</button>
<button class="button button-secondary button-small">Add filter</button>
<button class="button button-secondary button-medium" disabled>Add filter</button>
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

### B3 · Ghost

The lowest-emphasis brand-colored action, transparent until hovered. Use only on busy compositions with multiple icon buttons as a way to create a hierarchy from [Primary](#b1--primary) (high) to [Ghost](#b3--ghost) (low).

```html
<button class="button button-ghost button-medium">Cancel</button>
<button class="button button-ghost button-small">Cancel</button>
<button class="button button-ghost button-medium" disabled>Cancel</button>
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

### B4 · Danger primary

The one high-emphasis destructive action (e.g. an irreversible delete).

```html
<button class="button button-danger-primary button-medium">Delete scenario</button>
<button class="button button-danger-primary button-small">Delete scenario</button>
<button class="button button-danger-primary button-medium" disabled>Delete scenario</button>
```

```css
.button-danger-primary {
  background: var(--color-negative-50);
  --button-border-color: var(--color-negative-50);
  color: var(--text-light-primary);
}
.button-danger-primary:hover {
  background: var(--color-negative-90);
  --button-border-color: var(--color-negative-90);
}
.button-danger-primary[disabled] {
  background: transparent;
  --button-border-color: transparent;
}
```

### B5 · Danger secondary

A lower-emphasis destructive action, e.g. "Remove" in a list row.

```html
<button class="button button-danger-secondary button-medium">Remove</button>
<button class="button button-danger-secondary button-small">Remove</button>
<button class="button button-danger-secondary button-medium" disabled>Remove</button>
```

```css
.button-danger-secondary {
  background: var(--color-white);
  --button-border-color: var(--color-white);
  color: var(--color-negative-50);
}
.button-danger-secondary:hover {
  background: var(--color-negative-10);
  --button-border-color: var(--color-negative-10);
}
.button-danger-secondary[disabled] {
  background: transparent;
  --button-border-color: transparent;
}
```

### B6 · Transparent

Rare, for use on dark/colored surfaces (e.g. a toast or a dark modal), where the light-alpha variants above wouldn't read.

```html
<div class="dark-surface">
  <button class="button button-transparent button-medium">Dismiss</button>
  <button class="button button-transparent button-medium" disabled>Dismiss</button>
</div>
```

```css
.button-transparent {
  background: rgba(255, 255, 255, 0.16);
  --button-border-color: transparent;
  color: var(--color-white);
}
.button-transparent:hover {
  background: rgba(255, 255, 255, 0.24);
}
.button-transparent[disabled] {
  background: var(--color-grey-10);
  --button-border-color: var(--color-grey-10);
}
```

### B7 · Icon + label

Put a 16×16 icon before or after any button's label. Sizing comes from `.button svg` in the shared base, the 4px icon-to-label gap from the base `gap`, and the icon color from the label through `currentColor` — never set the icon color yourself.

```html
<button class="button button-primary button-medium">
  <svg viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
    <path d="M8 3V13M3 8H13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" />
  </svg>
  Add item
</button>
```
