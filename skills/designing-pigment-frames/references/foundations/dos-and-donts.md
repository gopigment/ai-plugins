# Do's and Don'ts

## Do

- Build every layout from Row/Column compositions with gaps from the 4px spacing scale — this alone makes generated UI feel native.
- Use `--color-primary-50` for exactly the things Pigment uses it for: primary actions, links, active/selected states, and focus rings. Keep it out of large background fills.
- Keep corner radius at `--radius-xs` for rectangular controls and fully circular for round ones. Don't introduce an in-between radius like 10-14px on a button or input.
- Use soft, blue-tinted, multi-layer shadows (or none at all, with a 1px border instead) for elevation — never a flat black `box-shadow`.
- Reuse the exact focus ring (`--focus-ring`) on every custom interactive element you build.
- Reach for `running-regular` (14px/400) as your default body text and `screen-title` (24px/500) as your default page heading before considering anything else in the scale.
- Pick semantic colors only in their documented pairing: `10`-shade background with `90`-shade text.
- Strictly use additional palettes (editorial/illustrative, categorical, chart) only for their documented usages.
- When building grid-based layout (like a dashboard), use the 12-column / 8px-row grid from [layout-and-spacing.md](layout-and-spacing.md) and the card-tile pattern from [design-system/cards.md](../design-system/cards.md) rather than inventing a bespoke grid.

## Don't

- Don't introduce a second brand color or a saturated accent for functional UI (buttons, links, active states) — the illustrative palette is for charts/tags/decoration only.
- Don't use pure black shadows, pure black text, or pure black overlays — every "dark" token in this system is `--color-grey-90` (#020D23) or an alpha-blend of it, never `#000000`.
- Don't bold text for emphasis — the system tops out at weight 600 (and only for `section-title`); use heavy style variants (`running-heavy` or `running-small-heavy`) or pick a different variant from the typography hierarhcy instead.
- Don't color-code snackbars/toasts by severity — they're deliberately monochrome in this system.
- Don't mix a visible border and a shadow on the same card — pick the bordered-flat idiom or the elevated-shadow idiom, not both.
- Don't uppercase body text or headings — uppercase+tracking is reserved exclusively for column headers and small eyebrow labels.
- Don't invent arbitrary pixel spacing — everything should trace back to the 4px-unit scale, including its three documented half-steps (2px, 6px, 10px).
- Don't design your own app chrome (top bar, side nav) inside a one-off interface — it renders inside the existing shell.
