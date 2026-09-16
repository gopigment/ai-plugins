# Typography

## Font Family

Everything renders in the **platform sans-serif** (`--font-sans`, `screen-title` etc.), which reads well for UI density and long reading sessions at small sizes. The only exception is code, formulas, and technical/tabular values, which use the **monospace stack** `--font-mono` (`code-running-regular` / `code-running-small`). Both variables are the bare CSS generics: this system bundles no font file and names no family, so every viewer gets the same rendering. Do not add an `@font-face` rule, a webfont URL, or a named family. There is no serif anywhere in the system, and no third typeface — resist the urge to add a display face for a report or infographic; scale and weight, not a new font, create hierarchy here. See [design-system/typography.md](../design-system/typography.md) for copy-pasteable markup for every style below.

## Type Scale

| Token                                          | Size / Weight / Line-height                  | Use                                                                                        |
| ---------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `marketing-title`                 | 56px / 500 / 68px, -1px tracking             | Rare — a hero statement on a report cover or landing-style moment. Not for in-app screens. |
| `screen-title`                    | 24px / 500 / 32px                            | The page's main H1 — one per screen                                                        |
| `content-title`                   | 22px / 500 / 32px                            | A secondary large heading, e.g. inside a large panel or modal                              |
| `highlight-figure`                | 28px / 500 / 36px                            | Large emphasized figures — KPI values, key stat callouts on cards                          |
| `section-title`                   | 16px / 600 / 24px                            | Section headers within a page (card titles, panel headers)                                 |
| `column-title`                    | 11px / 400 / 16px, uppercase, 0.6px tracking | Table/column headers, small eyebrow labels                                                 |
| `running-regular`                 | 14px / 400 / 20px                            | Default body text and paragraph copy — the workhorse size                                  |
| `running-heavy`                   | 14px / 500 / 20px                            | Row titles, list item titles, smaller headings, emphasized inline text at body size        |
| `running-small`                   | 12px / 400 / 16px                            | Secondary/supporting text, helper text, captions, timestamps                               |
| `running-small-heavy`             | 12px / 500 / 16px                            | Emphasized small text (badges, tags, compact labels)                                       |
| `fieldset-regular`                | 14px / 500 / 20px                            | Form field labels (medium fields)                                                          |
| `fieldset-small`                  | 12px / 500 / 16px                            | Form field labels (compact/dense forms)                                                    |
| `button-label`                    | 14px / 500 / 20px, nowrap                    | All button text, at every button size                                                      |
| `legal-notice`                    | 10px / 400 / 12px                            | Fine print, footnotes, legal text                                                          |
| `chip`                            | 12px / 400                                   | Exclusive for chip labels, line-height equals font size                                    |
| `chart-label`                     | 11px / 400 / 16px                            | Chart axis ticks and small in-chart annotations, never uppercase/tracked                   |
| `code-running-regular` / `-small` | `--font-mono`, 14px or 12px / 400            | Formulas and code, exclusively. Never table figures.                                       |

## Defaults

- **Default page heading**: `screen-title` (24px/500), one per screen, colored `text-primary`.
- **Default body copy**: `running-regular` (14px/400) — this is correct for the vast majority of text in the interface. Reach for `running-small` only for secondary/supporting information, never for primary content.
- **Default form label**: `fieldset-regular` (14px/500) in `text-secondary` color.
- **Default form field values**: `running-heavy` (14px/500) in `text-primary` color.
- **Default table/column header**: `column-title` — uppercase, tracked, `text-secondary`.
- **Default button text**: `button-label` regardless of button size — button text size does not shrink with button size; only padding and height change.
- **Default placeholder text**: `running-heavy` (14px/500) in `text-secondary` — placeholders keep the same weight as a filled value; only color distinguishes them.
- **Weight range is narrow**: 400 (regular body), 500 (labels, emphasis, most headings), 600 (only `section-title` — the single "heavier" weight in the system). There is no 700/bold in normal UI text; if something needs more emphasis than 500/600, increase size or use color (`text-highlight`), not boldness.

## Signature Treatments

- Column/table headers and small eyebrow labels are **always uppercase with positive letter-spacing** (`column-title`: 0.6px on 11px type) — this is the one place the system uses tracked uppercase type; body and headings are always sentence/normal case.
- Headings never exceed weight 600, and only `section-title` uses 600 — every larger heading (`screen-title`, `content-title`) is weight 500, deliberately restrained for a heading that large.
- Numeric/tabular and code content switches font family entirely to `--font-mono` rather than trying to align numerals within the sans-serif face — formulas, cell references, and raw code should always be monospaced.
- Disabled text is never achieved by lowering opacity on `text-primary` — it uses the dedicated `text-disabled` (`--color-grey-30`) token so disabled states stay visually consistent regardless of underlying color.
