# Gradient Backgrounds

### GB1 · Gradient background

An ambient gradient at the top of the page container. Depth and glow colors are overridable via custom properties. Only one gradient background per page, applied to the page container only — never to a section, panel, modal, or a card grid/table (those must always sit on plain white; this is page-chrome decoration, not a content-area treatment).

```html
<div class="gradient-background" style="padding: var(--space-8); min-height: 200px">
  <h1 class="type-screen-title">Annual Report FY26</h1>
  <p class="type-running-regular" style="color: var(--text-secondary); margin-top: var(--space-1)">
    Page opening on an ambient gradient background that resolves to white
  </p>
</div>
```

```css
.gradient-background {
  --gradient-depth: 128px;
  --gradient-glow-1: var(--color-turquoise-soft);
  --gradient-glow-2: var(--color-ochre-soft);
  --gradient-glow-3: var(--color-cobalt-soft);
  background-color: var(--color-white);
  background-image:
    linear-gradient(rgba(255, 255, 255, 0) 0%, var(--color-white) var(--gradient-depth)),
    linear-gradient(
      rgba(255, 255, 255, 0) 0%,
      var(--color-white) calc(var(--gradient-depth) - 28px)
    ),
    linear-gradient(
      rgba(255, 255, 255, 0) 0%,
      var(--color-white) calc(var(--gradient-depth) - 36px)
    ),
    radial-gradient(
      circle calc(var(--gradient-depth) * 2) at 0 0,
      var(--gradient-glow-1),
      transparent
    ),
    radial-gradient(
      circle calc(var(--gradient-depth) * 2) at 40% 0,
      var(--gradient-glow-2),
      transparent
    ),
    radial-gradient(
      circle closest-corner at 20% calc(var(--gradient-depth) / 2),
      var(--gradient-glow-3) 0%,
      transparent 78%
    );
}
```

### GB2 · Spacious variant

Override `--gradient-depth` to 192px (48 spacing units) and the glow colors for a taller, more prominent gradient.

```html
<div
  class="gradient-background"
  style="
    --gradient-depth: 192px;
    --gradient-glow-1: var(--color-amethyst-soft);
    --gradient-glow-2: var(--color-fuchsia-soft);
    --gradient-glow-3: var(--color-cobalt-soft);
    padding: var(--space-8);
    min-height: 240px;
  "
>
  <h1 class="type-screen-title">Scenario Overview</h1>
  <p class="type-running-regular" style="color: var(--text-secondary); margin-top: var(--space-1)">
    Spacious variant — override the depth and any glow via the custom properties
  </p>
</div>
```

### GB3 · Compact variant

Override `--gradient-depth` to 64px (16 spacing units) for a shallow gradient on pages where dense content starts early — e.g. a dashboard's heading, where the card grid begins almost immediately below.

```html
<div
  class="gradient-background"
  style="
    --gradient-depth: 64px;
    --gradient-glow-1: var(--color-emerald-soft);
    --gradient-glow-2: var(--color-turquoise-soft);
    --gradient-glow-3: var(--color-emerald-soft);
    padding: var(--space-8);
    min-height: 160px;
  "
>
  <h1 class="type-screen-title">Headcount Plan</h1>
  <p class="type-running-regular" style="color: var(--text-secondary); margin-top: var(--space-1)">
    Compact variant — a shallow gradient background for pages where dense content starts early
  </p>
</div>
```