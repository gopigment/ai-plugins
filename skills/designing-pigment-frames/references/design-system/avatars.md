# Avatars

Two sizes only, always fully circular. Label text is `--font-sans` 12px/500. The background is
either a solid `--color-grey-90` with `--text-light-primary` initials, or a categorical slot's `-bg`
fill paired with its `-fg` initials. Either way the color is assigned deterministically per entity
(by index or a stable hash of its id), never hand-picked — the same rule as categorical chips.

### AV1 · Avatar

A circular initials badge, colored from the categorical palette (matching [chips.md](chips.md)'s CH1) so the same person/entity reads consistently wherever their avatar appears. `avatar-large` (32px, 2 initials) is for a prominent row (e.g. a people picker); the bare `avatar` (24px, 1 initial) is for dense contexts like a list-item decoration.

```html
<span
  class="avatar avatar-large"
  style="background: var(--color-categorical-1-bg); color: var(--color-categorical-1-fg)"
  >DK</span
>
<span
  class="avatar"
  style="background: var(--color-categorical-1-bg); color: var(--color-categorical-1-fg)"
  >D</span
>
```

```css
.avatar {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: var(--space-6);
  height: var(--space-6);
  border-radius: var(--radius-circular);
  color: var(--text-light-primary);
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 500;
}
.avatar-large {
  width: var(--space-8);
  height: var(--space-8);
}
```
