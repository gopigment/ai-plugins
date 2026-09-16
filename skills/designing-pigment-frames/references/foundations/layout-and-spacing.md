# Layout & Spacing

## The 4px Grid

Every spacing value in the system comes from the **4px-unit scale** below — padding, margin, gap, icon size, control height. Most are whole multiples of 4px; three half-steps (2px, 6px, 10px) exist for fine detail. When building a new interface, do not invent arbitrary pixel values (5px, 15px, 20px…); pick from the scale below:

| Multiplier | Value | Typical use                                                                |
| ---------- | ----- | -------------------------------------------------------------------------- |
| 0.5×       | 2px   | Hairline offsets, icon-to-badge overlap                                    |
| 1×         | 4px   | Tightest gap (icon-to-text, label-to-helper, fieldset label-to-field gap ) |
| 1.5×       | 6px   | Input vertical padding                                                     |
| 2×         | 8px   | Standard small gap; icon size;                                             |
| 2.5×       | 10px  | Button small horizontal padding; list-item gap and horizontal padding      |
| 3×         | 12px  | Card padding (compact); vertical rhythm inside dense panels; gap between a header (section/table) and its content |
| 4×         | 16px  | Standard gap between related elements; compact page gutter                 |
| 6×         | 24px  | Card/section padding; standard page gutter                                 |
| 7×         | 28px  | Small-size control height (button, icon button, input)                     |
| 8×         | 32px  | Large section spacing (rare — prefer 12× for the default gap between sections) |
| 12×        | 48px  | Empty-state vertical padding; default gap between sections                 |

## Content-Area Context

There is no single enforced "max content width" in the platform — dense data screens (grids, boards) legitimately run full-width, while reports read better capped for line length at a medium breakpoint and form pages should be constrained to a smaller width.
When a cap is appropriate, choose from the platform's own breakpoint scale as your reference (`--breakpoint-md` 960px or `--breakpoint-lg` 1280px) rather than an arbitrary number.

## Page Canvas / Safe Area

Every generated page keeps a safe margin between its content and the page edges: **36px top and bottom, 32px left and right**. This applies to the single outermost content wrapper of the page — not to individual sections, cards, or the header divider, which have their own spacing already.

The page can scroll vertically, and this margin must never get in the way of that:

- Apply the 36px/32px as **padding**, never as a margin on the last child or a fixed empty spacer — padding on the content wrapper always scrolls into view with the content; a margin on the last child can collapse and disappear.
- Apply that padding to whichever element actually **establishes the scroll context** — the scrolling `body`, or your outermost `overflow-y: auto` wrapper if the page renders inside a fixed-height host — never to a nested child of it. Padding set on a descendant of a flex/grid scroll container can end up excluded from the scrollable range, so scrolling to "the bottom" silently loses the last 36px.
- That scrolling element must use `min-height`, never a capped `height` with `overflow: hidden` — it needs to grow taller than the viewport so content can extend past the bottom edge naturally, with the 36px appearing once, after the real last piece of content, not as a rigid gap that clips anything taller than one screen.

## Composition Primitives

Pigment's own UI is built almost entirely from two layout primitives — think in these terms rather than raw CSS Grid/Flexbox trivia:

- **Row** — a horizontal flex container with a controllable gap (from the spacing scale) and vertical alignment (top / center / bottom / baseline / stretch). Used for toolbars, fieldset label+field pairs, button groups, card headers.
- **Column** — the vertical equivalent; used for stacking fieldsets, list items, card content.

Compose interfaces as nested rows and columns with consistent gaps from the spacing scale, rather than one-off absolute positioning. This is the single biggest thing that makes a generated interface "feel like Pigment."

## Dashboard / Widget Grid

For dashboard-style interfaces (KPI tiles, chart widgets, mixed report blocks), follow the platform's own board pattern: a **12-column grid** with an **8px row unit** (widget heights and positions snap to multiples of 8px). Widgets are cards (see [design-system/cards.md](../design-system/cards.md)) placed on this grid with consistent gutters (16px or 24px). This is the same grid Pigment's own dashboards ("Boards") use internally.

A widget with a title, description, or actions (e.g. a chart's breakdown toggles) follows the general header-placement rule: a [Section header](../patterns/section-headers.md) outside and above the card, and a separate card below it containing only the content.

Pigment is primarily a desktop, data-dense product. One-off interfaces should optimize for `--breakpoint-md`–`--breakpoint-xl` (laptop to large monitor); graceful mobile collapse is a secondary concern unless the interface is explicitly a report/read-only view meant for phones.
