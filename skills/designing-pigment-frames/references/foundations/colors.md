# Colors

## Neutral Scale

The neutral scale runs from white to near-black, with a warm-cool neutral bias (blue-grey, not true grey):

| Token                       | Value              | Use                                                                                                                                                                                                                                       |
| --------------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--color-white`            | #FFFFFF            | Default surface for cards, inputs, modals, popovers                                                                                                                                                                                       |
| `--color-grey-10`          | #F7F7F8            | Disabled input/button background; dense `list-surface` fill                                                                                                                                                                               |
| `--color-grey-20`          | #E8EAED            | Default border color for inputs — the one exception to the `--color-neutral-alpha` structural border                                                                                                                                     |
| `--color-grey-30`          | #949FB2            | Disabled text, disabled icons, placeholder-on-disabled                                                                                                                                                                                    |
| `--color-grey-50`          | #5A657A            | Secondary text, field labels, unchecked control borders, secondary icons                                                                                                                                                                  |
| `--color-grey-90`          | #020D23            | Primary text color; also the base for every alpha-blended "ink" tone in the system (borders, backdrops, shadows are all `grey-90` at low opacity, not pure black)                                                                         |
| `--color-neutral-alpha`    | rgba(2,13,35,0.08) | The structural border tone (aliased `border`) — cards, dividers, table rows, tab dividers                                                                                                                                                 |
| `--color-background-alpha` | rgba(2,13,35,0.04) | A lighter neutral wash usable on any surface for subtle hover/active backgrounds, secondary-surface fills (aliased `bg-secondary-surface`), and the default base chip background — the go-to "just slightly darker than the surface" tone |

## Brand

A single blue carries the whole brand identity:

| Token                                | Value               | Use                                                                                                            |
| ------------------------------------ | ------------------- | -------------------------------------------------------------------------------------------------------------- |
| `--color-primary-10`                | #F0F5FF             | Light tinted backgrounds (selected rows, secondary button fill, info banners)                                  |
| `--color-primary-20`                | #95B9FF             | Focus rings (universal, on every component), hover borders on light surfaces                                   |
| `--color-primary-30`                | #2684FF             | Input focus border                                                                                             |
| `--color-primary-50`                | #0355F3             | The brand color: primary button fill, links, active tab/nav indicator, checked control fill, selection accents |
| `--color-primary-90`                | #0038A4             | Primary button hover/active fill, deepest brand tone for text-on-light needing extra contrast                  |
| `--color-primary-light-transparent` | rgba(3,85,243,0.05) | 5%-opacity brand wash — secondary button background, active-toggle-button background                           |

## Semantic / Status

Three status families, each with the same 4-step ramp (10 / 20 / 50 / 90 — light background, light-medium, base, dark):

| Family   | 10 (bg) | 20      | 50 (base) | 90 (dark/text) |
| -------- | ------- | ------- | --------- | -------------- |
| Positive | #EEFDEE | #D1FAD1 | #1A9F1A   | #094F09        |
| Cautious | #FFF8E5 | #FFF5CC | #F3B921   | #7A5D11        |
| Negative | #FFF0F2 | #FFD8DD | #D02B41   | #740C19        |

Pattern: a status chip/banner/message uses the `10` shade as background and the `90` shade as text (never the saturated `50` as a text-on-light color and never as a large fill — `50` is reserved for icons, borders and small accents).

## Defaults

- **Default background**: `--color-white`.
- **Default card/panel/input surface**: `--color-white`.
- **Default border**: 1px `--border` (`--color-neutral-alpha`) for structural surfaces (cards, dividers, table rows); inputs default to 1px `--color-grey-20` instead — both switch on error, selected, hover, or focus states.
- **Default body/label text**: `--color-grey-90` (aliased `text-primary`).
- **Default secondary/muted text**: `--color-grey-50` (aliased `text-secondary`) — field labels, helper text, captions, timestamps.
- **Default disabled text/icon**: `--color-grey-30`.
- **Default link/active/highlight color**: `--color-primary-50`.
- **Default focus ring**: `--focus-ring` on every focusable control (buttons, inputs, checkboxes, radios, tabs, switches) — always this exact treatment, never a custom focus style per component.
- **Default overlay/backdrop** (behind modals): `--color-backdrop` = rgba(2,13,35,0.6).

There is no accent color beyond brand blue for interactive UI.
Palettes for editorial/illustrative, categorical items, and charts exist (see below) but should not leak into functional controls — a button, tab, or input is never rendered in green, purple, or orange.

## Editorial / Illustrative Palette (for reports, infographics, decorative accents)

When the interface is more editorial in nature (an infographic, a report cover, a big KPI statement) rather than a working tool, or you need to create a gradient for aesthetic purposes, Pigment's brand system provides a small family of secondary hues, each with three tones (soft background / vivid / bold text-on-light):

| Family    | Soft (bg)         | Vivid             | Bold (text)       |
| --------- | ----------------- | ----------------- | ----------------- |
| Cobalt    | hsl(212,100%,85%) | hsl(219,98%,48%)  | hsl(227,72%,29%)  |
| Emerald   | hsl(80,66%,88%)   | hsl(147,99%,33%)  | hsl(147,56%,26%)  |
| Amethyst  | hsl(274,100%,90%) | hsl(259,100%,77%) | hsl(270,50%,30%)  |
| Ochre     | hsl(47,90%,84%)   | hsl(43,100%,70%)  | hsl(43,100%,16%)  |
| Sienna    | hsl(32,90%,84%)   | hsl(36,100%,70%)  | hsl(19,83%,28%)   |
| Turquoise | hsl(191,100%,85%) | hsl(185,100%,40%) | hsl(190,100%,19%) |
| Fuchsia   | hsl(328,100%,90%) | hsl(345,87%,63%)  | hsl(328,89%,24%)  |

Use this palette for one-off editorial moments — never for a functional control (buttons, links, status), for data visualisations or for items of lists.
Never use `vivid` and `bold` variants as the background of surfaces containing text. We avoid dark-themed UI elements in Pigment. Instead, use the `soft` variants as background color of a surface.

## Gradient Background (page decoration only)

A **gradient background** is a soft multi-color glow at the top edge of the page container that fades to plain white and can go behind content.
It is page-chrome decoration, not a content-area treatment: it belongs on the page container only. Child content regions (tables, grids, card grids, forms) must never be decorated with this gradient.
It is built entirely from `background-image` layers: three white linear-gradient veils that guarantee the fade-to-white, over three radial-gradient glows. See [patterns/gradient-backgrounds.md](../patterns/gradient-backgrounds.md) for copy-pasteable markup.

- **Depth** — one 4px-grid value: `64px` (compact), `128px` (default), `192px` (spacious). The three veils end at depth, depth − 28px and depth − 36px (the staggered stops are what feather the fade); glow radius is 2× depth; the third glow anchors at depth ÷ 2. `Default` is the first choice, pick `dense` for secondary pages or `spacious` for sparse layouts like forms or slide deck title pages.
- **Glow colors** — any three **Soft** tones from the illustrative palette above (repeating a family is fine). Pick with aesthetics in mind, never randomly. Never vivid/bold tones, never semantic colors (a green or red glow reads as status), never the categorical palette, never `--color-primary-50` — if a brand-blue mood is wanted, use cobalt-soft.

## Card Gradient (expressive editorial card only)

The strongest card treatment in the system: a `card-filled` variant where a Soft illustrative base carries two top-corner glows — a small accent anchored top-left and a large sweep entering from the top-right — tamed by white and base-colored veils so the card's lower half stays quiet and fully legible. Use it for the few cards that deserve hero status on an editorial page (top performers, report highlights, cover KPIs) — never for functional tiles, and rarely more than one row of them per page.

Three custom properties drive it: `--card-base-background-color`, `--card-glow-color-left`, `--card-glow-color-right` — see [design-system/cards.md](../design-system/cards.md) (C6) for copy-pasteable markup.

**Choosing glow colors.** Left and right may be the same color or different. Validity is tiered:

1. The base family's own **vivid** is always a valid glow.
2. Otherwise, prefer the base's companions from this table:

| Base (soft)    | Preferred glow companions     |
| -------------- | ----------------------------- |
| cobalt-soft    | amethyst-soft, turquoise-soft |
| emerald-soft   | ochre-soft, turquoise-soft    |
| amethyst-soft  | fuchsia-soft, fuchsia-vivid   |
| ochre-soft     | fuchsia-soft, amethyst-soft   |
| sienna-soft    | ochre-soft, emerald-soft      |
| turquoise-soft | ochre-soft, emerald-soft      |
| fuchsia-soft   | ochre-soft, sienna-soft       |

3. Any other **Soft** tone is acceptable only with deliberate aesthetic intent — never bold shades, semantic colors, the categorical palette, or `--color-primary-50` as glow or base, and never a vivid as the base itself.

When mixing intensities, place the vivid on the small left accent and the soft on the large right sweep. Emphasized text on the card uses a `-bold` shade: the base family's bold by default, or the glow family's bold when both glows share one family.

## Categorical / Random-Assignment Palette

For lists of generic objects — applications, boards, scenarios, users, tags, labels, chips — where each item needs a distinct but arbitrary identity (typically an icon or avatar background), Pigment draws from a fixed, ordered 23-color list.
Colors are assigned deterministically per item (e.g. by index or a stable hash of the entity id), never hand-picked. Each slot has two values: a `-fg` (foreground) value for icons, text, and small solid fills, and a `-bg` (background) value — the same color at 24% opacity — for larger fills that need to sit behind that foreground color (e.g. an icon tile background):

| Slot            | `--color-categorical-N-fg` | `--color-categorical-N-bg` (same color at 24% opacity) |
| --------------- | -------------------------- | ------------------------------------------------------ |
| `categorical-1`  | #013496            | rgba(3, 85, 243, 0.24)          |
| `categorical-2`  | #0a5442            | rgba(20, 184, 146, 0.24)        |
| `categorical-3`  | #32562f            | rgba(116, 199, 109, 0.24)       |
| `categorical-4`  | #414a75            | rgba(143, 161, 255, 0.24)       |
| `categorical-5`  | #4a00a2            | rgba(87, 0, 191, 0.24)          |
| `categorical-6`  | #134289            | rgba(33, 115, 239, 0.24)        |
| `categorical-7`  | #001eb9            | rgba(0, 41, 255, 0.24)          |
| `categorical-8`  | #354156            | rgba(69, 84, 111, 0.24)         |
| `categorical-9`  | #634f32            | rgba(255, 204, 128, 0.24)       |
| `categorical-10` | #004871            | rgba(0, 128, 200, 0.24)         |
| `categorical-11` | #2c5267            | rgba(103, 191, 239, 0.24)       |
| `categorical-12` | #354d72            | rgba(118, 173, 255, 0.24)       |
| `categorical-13` | #003293            | rgba(0, 52, 154, 0.24)          |
| `categorical-14` | #0c5513            | rgba(25, 183, 41, 0.24)         |
| `categorical-15` | #3a2e84            | rgba(60, 48, 137, 0.24)         |
| `categorical-16` | #7b0344            | rgba(215, 5, 118, 0.24)         |
| `categorical-17` | #772d48            | rgba(240, 91, 145, 0.24)        |
| `categorical-18` | #761f54            | rgba(206, 55, 146, 0.24)        |
| `categorical-19` | #6e4223            | rgba(252, 151, 79, 0.24)        |
| `categorical-20` | #004631            | rgba(0, 76, 53, 0.24)           |
| `categorical-21` | #0d4a34            | rgba(18, 104, 73, 0.24)         |
| `categorical-22` | #615101            | rgba(255, 215, 3, 0.24)         |
| `categorical-23` | #001391            | rgba(0, 19, 145, 0.24)          |

Use this palette only for randomly/deterministically assigning a color per entity in a list — never for status, functional controls, or chart series (use the Chart Palette below for data viz, and the Extended / Illustrative Palette above for editorial soft/vivid/bold accents).

## Chart Palette

For data visualization in a dashboard or report, use the default 12-color series in order: `--color-chart-1` through `--color-chart-12`. Reach for this ordered list before inventing new chart colors — consistent chart color order is part of feeling native to the platform. When a chart needs a legend, pair each series to a [design-system/chips.md](../design-system/chips.md) CH2 Chart legend pill in the same order. Axis ticks and other small in-chart annotations use `chart-label`, never `column-title` (which is reserved for uppercase/tracked table headers) or plain `running-small`.

| Token              | Value   | Token               | Value   |
| ------------------ | ------- | ------------------- | ------- |
| `--color-chart-1` | #0355F3 | `--color-chart-7`  | #C2DFFA |
| `--color-chart-2` | #4BC766 | `--color-chart-8`  | #D0FACA |
| `--color-chart-3` | #FFBE5C | `--color-chart-9`  | #FFC8AC |
| `--color-chart-4` | #F95A77 | `--color-chart-10` | #CAEB9D |
| `--color-chart-5` | #6B1CB0 | `--color-chart-11` | #C096E3 |
| `--color-chart-6` | #1B2970 | `--color-chart-12` | #FDAEBC |
