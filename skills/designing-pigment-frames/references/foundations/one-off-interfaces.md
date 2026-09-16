# Building One-Off Interfaces

A few starting points depending on what you're building, all using the same tokens and components as the rest of these references:

- **Dashboard** — [Gradient background](../patterns/gradient-backgrounds.md) + Page heading (`screen-title`) + optional horizontal tabs for view switching. Body is the 12-column / 8px-row grid of [Card](../design-system/cards.md)-style widgets (KPI numbers in `highlight-figure`, charts using the 12-color chart palette, tables using the [table row pattern](../patterns/table-headers.md)). Keep widget padding at 16px and gutters consistent.
- **Form / Input screen** — A single Column of [Fieldset](../design-system/inputs.md) rows (`4px` internal gap, 16-24px between fieldsets), grouped into Card sections with `section-title` headers when the form is long. Primary action button bottom-right or in a sticky footer; keep to one `primary` button per screen.
- **Report** — Favor `content-title`/`section-title` hierarchy over dense controls; generous 24px+ padding; tables and charts as in dashboards but with less interactivity chrome (fewer buttons, more static Typography). Fine print in `legal-notice`.
- **Infographic / statement page** — The one place `marketing-title` and the extended illustrative palette are appropriate; still keep the 4px grid, the `--font-sans` typeface, and soft-shadow depth language so it doesn't look like a foreign template pasted into the product.

A CSS Grid or Flexbox approximation of "12 columns, 8px row rhythm" is sufficient; you don't need drag-and-drop behavior to feel native.
In every case: start from white cards on a `--color-white` canvas, `--font-sans` type, 4px-multiple spacing, 4px/6px radii, and the single brand blue — then layer in only as much color and size contrast as the content needs.
