---
name: designing-pigment-frames
description: "Apply Pigment's visual design language to a Frame that already renders — use when an existing Frame must look on-brand, native, or less generic. Not for initial Frame creation. Build it with `skill:building-pigment-frames` first, then come back here for the design passes."
metadata:
  skill_path: /skills/designing-pigment-frames/SKILL.md
  base_directory: /skills/designing-pigment-frames
  includes:
    - "*.md"
---

# Styling Pigment Frames

Bring a Frame that already works up to the visual language of the Pigment platform. You tend to
converge toward generic, "on distribution" output, which users find off-putting. The reference files
here exist so that you do not have to invent anything.

## When to Use

**Build first, design after.** Do not load this skill during initial Frame creation. Build with
`skill:building-pigment-frames` until the Frame renders its data, show it, then style it — a user
cannot judge how a Frame looks until they can see it.

The one design step that belongs in the build is the **base stylesheet**: the contents of
[`references/styles/stylesheet.md`](references/styles/stylesheet.md) in the
[styles section](#the-styles-section). That alone keeps v1 from looking generic, because it gives the
Frame Pigment's color, spacing, radius and motion tokens as CSS variables. Type styles and every
component's own CSS come from their own recipe file, in the design passes.

## What You Are Styling

A Frame body is **one pure-JS file**, not an HTML document: the host loads it as a script into `#app`
in a sandboxed iframe (`allow-scripts`, opaque origin). There is no `<body>`, no `<head>` you author,
no second file, and no network — so no Google Fonts, no `<link>`, no `@import`. The JS creates every
style itself.

## The Styles Section

Emit the base stylesheet **once**, into one marked block — this is the token file only, not any
component's CSS:

```js
// SECTION: styles
const style = document.getElementById('pg-styles') || document.createElement('style');
style.id = 'pg-styles';
style.textContent = `
  /* stylesheet.md, verbatim — never re-encoded, never subset by hand */
`;
document.head.appendChild(style);
root.style.cssText =
  'position:fixed;inset:0;width:100%;height:100%;overflow:auto;' +
  'font-family:var(--font-sans);-webkit-font-smoothing:antialiased;background:var(--color-white);';
// SECTION: styles end
```

The id lookup makes the block idempotent, so a hot reload does not stack a second sheet. A template
literal is right here despite the build skill's advice against them: the CSS holds no backtick and
no `${`, and the alternative is tens of thousands of escaped characters.

Every later pass **appends** to this same block with a targeted `tool:edit_file` call — paste the CSS
straight out of whichever recipe file that pass opened, right before the closing `` ` ``. Never
rewrite the whole block: a Frame only ships CSS for the components it actually uses.

**Change styles with targeted `tool:edit_file` calls against this block.** Never `tool:read_file` the
whole Frame file for a style change and never rewrite it from scratch: `read_file`, `write_file` and
`edit_file` output is never evicted, so each whole-file read or rewrite sits in context for the rest
of the session and costs you the room to finish.

## Design Passes

Run these in order, pushing the file with `tool:update_frame` after each one so the user sees the
result. Each pass ends with its own [checks](#audit-checklist). Fixing a fault in pass 1 is cheap;
finding the same fault in a finished artifact is not.

1. **Base** — tokens, typography and spacing present and applied. If the build already wrote the
   styles section, verify it is complete rather than writing it again. If it is missing, write it
   now — once.
2. **Structure** — page canvas, headers, grid, section rhythm.
3. **Components** — one component at a time: each reference file gives you the markup and its CSS
   together — copy both in, appending the CSS to the styles block.
4. **Sweep** — the whole-output checks.

## Routing: Read Only What the Pass Needs

Every reference file is small and single-purpose. **Open only what the pass in front of you needs**
— never the whole `references/` tree. There is no index file to read first; this table is the index.

Three kinds of file live under `references/`:

- `foundations/` — the rules: which color, which type style, which spacing, which radius.
- `design-system/` and `patterns/` — copy-pasteable recipes for one component or composition,
  markup and CSS together.
- [`styles/stylesheet.md`](references/styles/stylesheet.md) — every design token as a CSS variable
  (colors, spacing, radius, shadows, motion, breakpoints), and nothing component-specific. This is
  the single source of truth for those values, and the only file you emit whole.

Prefer a recipe over deriving a look from the foundation tables: re-deriving by hand is where weight,
sizing and motion violations creep in.

**Before pass 1, once:** [`foundations/overview.md`](references/foundations/overview.md) for what the
system is, and [`foundations/one-off-interfaces.md`](references/foundations/one-off-interfaces.md) for
the starting shape of the page type you are building (dashboard, form, report, infographic).

### Pass 1 — Base

| Need | File |
| --- | --- |
| The stylesheet you emit | [`styles/stylesheet.md`](references/styles/stylesheet.md) |
| Which type style goes where, and the weight rules | [`foundations/typography.md`](references/foundations/typography.md) |
| Copy-paste type classes, if you need one on its own | [`design-system/typography.md`](references/design-system/typography.md) |

### Pass 2 — Structure

| Need | File |
| --- | --- |
| 4px grid, page canvas, safe area, row/column composition, widget grid | [`foundations/layout-and-spacing.md`](references/foundations/layout-and-spacing.md) |
| Page title row | [`patterns/page-headers.md`](references/patterns/page-headers.md) |
| Section titles | [`patterns/section-headers.md`](references/patterns/section-headers.md) |
| Page background | [`patterns/gradient-backgrounds.md`](references/patterns/gradient-backgrounds.md) |

### Pass 3 — Components

Open [`foundations/depth-radius-motion.md`](references/foundations/depth-radius-motion.md) once for
this pass — it carries the surface idiom, radius and motion rules that every component needs — then
one component file at a time:

| Styling | File |
| --- | --- |
| Cards, KPI tiles, widget surfaces | [`design-system/cards.md`](references/design-system/cards.md) |
| Buttons | [`design-system/buttons.md`](references/design-system/buttons.md) |
| Icon-only actions | [`design-system/icon-buttons.md`](references/design-system/icon-buttons.md) |
| Pill toggles, filter pills, button groups | [`design-system/toggle-buttons.md`](references/design-system/toggle-buttons.md) |
| Fieldsets, text inputs, search, selects | [`design-system/inputs.md`](references/design-system/inputs.md) |
| Status chips, tags, chart legends | [`design-system/chips.md`](references/design-system/chips.md) |
| List rows | [`design-system/lists.md`](references/design-system/lists.md) |
| Avatars | [`design-system/avatars.md`](references/design-system/avatars.md) |
| Tooltips | [`design-system/tooltips.md`](references/design-system/tooltips.md) |
| Table header, filter row, table surface and rows | [`patterns/table-headers.md`](references/patterns/table-headers.md) |
| Checkbox, radio, switch, segmented toggle, slider, tabs, modal, toast, badge, icons | [`design-system/spec-only-components.md`](references/design-system/spec-only-components.md) |

Add [`foundations/colors.md`](references/foundations/colors.md) when the component carries color you
have to choose: a status chip or banner, a chart series, a per-entity categorical color, or an
editorial gradient.

### Pass 4 — Sweep

| Need | File |
| --- | --- |
| The whole-output rules | [`foundations/dos-and-donts.md`](references/foundations/dos-and-donts.md) |

The [audit checklist](#audit-checklist) below is in this file — you do not need to open anything to
run it.

## Design Language Non-Negotiables

- **Typography**: add no new type styles. Build hierarchy by switching between `-regular` and
  `-heavy` or picking another style — not with `<strong>`.
- **Color**: the documented palettes, in their documented usages, through the CSS variables.
- **Spacing and layout**: symmetrical, simple column/row compositions and grids.
- **Motion**: sparing, CSS-only. One orchestrated load with staggered reveals (`animation-delay`)
  delights more than scattered micro-interactions.
- **Surfaces**: prefer depth over solid colors.

## Composing New Components

The reference files will not have every component you need. When you must build one, do not derive
its look from token tables in isolation. Run it through these questions first, in order:

1. **Is this content, an action, or a status?**
   - Content (text, values, data) → typography scale only. Never invent a new font size/weight; pick
     the closest existing type token even if it feels slightly off.
   - Action (clickable) → must be one of the 6 Button variants or an Icon Button. Never style a
     `<div>` to look clickable outside these two components.
   - Status (state of something) → must use the `10`-bg/`90`-text semantic pairing. Never a bare
     colored dot or unpaired color.

2. **What's its surface idiom: bordered, filled or elevated?**
   Pick one, not multiple. Default to bordered (white + 1px border, no shadow) unless the component
   needs to read as "floating" or "clickable-and-lifts," in which case use elevated (no border,
   `shadows.card` → `shadows.cardHover`). This single decision resolves most "does this need a
   shadow" uncertainty.

3. **What's its density tier — and which role does each number play?**
   Don't treat a tier as "a range to pick a number from." Each spacing token is tied to a specific
   structural _role_ — gap between sibling elements, or padding/gutter inside a container — and you
   use whichever discrete value matches the role, not anything in between. The grid only has fixed
   stops (4/8/12/16/24/32/48px); there's no legal 18px or 20px.
   - **Dense** — gap: `8px` (icon-to-text, tight list-row internals) · padding: `12px` (table-row
     vertical padding, compact card interior). Use for anything that repeats many times on screen at
     once — table rows, dense list items — where legibility-at-scale trades off against breathing room.
   - **Standard** — gap: `16px` (space between two related-but-distinct elements: a label and its
     field, two buttons in a row) · padding: `24px` (a card's interior, or the page gutter — space
     between a container's border and its content, or between content and the page edge). This is the
     default for most non-repeating surfaces: a card, a form section, a panel.
   - **Spacious** — gap: `32px` (separating unrelated blocks) · padding: `48px` (empty-state
     interior). Rare and deliberate — hero moments, empty states, report covers — not everyday UI.
     Decide the tier first using the repetition-count heuristic above, then within that tier ask "is
     this space _between_ two elements, or space _inside_ a container around its content?" — that
     answer picks the specific number.

4. **Is it primary or supporting in its context?**
   Only one element per page/section gets the strongest visual weight (primary button, `primary-50`
   accent, heaviest available type). Everything else defaults to secondary/ghost treatments or
   `text-secondary` color. If two things in your new component both feel like they need emphasis,
   that's a sign the component's information hierarchy isn't resolved yet — fix the hierarchy, don't
   give both of them a strong style.

5. **Before finalizing: name its closest relative.**
   State explicitly which existing `references/design-system` component or `references/patterns`
   pattern this new one is closest to (e.g. "this notification-rail item is closest to a list-item
   row + status chip"), and inherit that relative's spacing/radius/border values by default. Only
   deviate if you can point to a specific reason in one of the `references/foundations` files. Never
   deviate without reasoning.

## Audit Checklist

The audit is mandatory and blocking, not optional feedback for when the user asks. Each pass ends
with its own checks, run against what that pass produced. If any box fails, fix it before moving on
— do not describe the fix, apply it.

### Pass 1 checks

- [ ] **Font stack**: exactly `--font-sans` (`sans-serif`) for text and `--font-mono` (`monospace`)
      for code and formulas. The bare generics are the whole stack: name no family, and add no
      system-font fallback such as `-apple-system` or `'Segoe UI'`. The skill bundles no face on
      purpose, so every viewer gets the same platform generic and the rendering stays predictable.
- [ ] **Font weight**: grep every `font-weight` in your output. Only `600` on an element styled as
      `section-title` is allowed. Everything else must be `400` or `500`. No `700`, no `bold`, no
      `<strong>`.
- [ ] **No font loading**: grep the output for `@font-face` — there must be none, because the skill
      bundles no face to embed. Grep for `fonts.googleapis` / `<link` font references and confirm
      there are none either: the sandbox has no network, so a fetched font can only fail.

### Pass 2 checks

- [ ] **4px grid**: grep every `px` value used for sizing/spacing (widths, heights, padding, gap,
      margin, border-radius). Each must be a multiple of 4 (exceptions only allowed for the
      documented cases).
- [ ] **Header action slot**: if a page/section header has a right-hand slot, it must contain an
      actionable button — not a static/decorative chip or badge. A page header's actions slot is
      major, page-level actions only (export, refresh, create, submit) or nothing — never filters,
      search, or other refinement controls, even on a list/table page; those belong in the table
      header instead.

### Pass 3 checks

Re-run the pass 1 type greps and the pass 2 grid grep over the CSS this pass added, then:

- [ ] **Motion curves**: grep every `transition` and `animation`. Only `150ms ease-in` (snappy) and
      `200ms ease-in-out` (soft) durations/easings are allowed as the curve itself. `animation-delay`
      offsets for stagger are fine; a _third distinct duration+easing pair_ is not.
- [ ] **Semantic color pairing**: any status chip/badge/banner must pair the `-10` shade as
      background with the `-90` shade as text — never `-50` as text-on-light or as a large fill.
- [ ] **Radius**: rectangular controls/cards/tables = `4px` (or `6px` for modals); round
      markers/avatars/pills = fully circular. Nothing in between.
- [ ] **Row height consistency**: for every row mixing multiple interactive controls (input/select
      next to buttons/icon-buttons/toggle-buttons), confirm they are using a consistent size variant
      and they all render with the same height. Don't add padding overrides to force it, instead
      chose a proper combination of size variants.
- [ ] **Table cell spacing**: data-grid cells use a `24px` horizontal gap between cells, not per-cell
      horizontal padding (12px vertical padding is correct).
- [ ] **Table surface and rows**: the table sits in a bordered-outlined surface (never
      elevated/filled); rows have left/right edge padding so content doesn't touch the surface
      border; a sticky header keeps a visible bottom divider above the scrolling rows; the last row
      has no bottom divider (it would double up with the surface's own bottom border).
- [ ] **Design system reuse**: for every component you built, check whether `references/design-system`
      has that exact component. If yes and your CSS differs from it, that difference is a bug unless
      justified.

### Final sweep

- [ ] **No dangling classes (do this one first, it's mechanical)**: every `class="..."` value
      referenced in the generated markup/JS must have a matching CSS rule shipped in the output. A
      class used but never defined fails silently. Extract every class name used in the markup,
      extract every class name defined in the CSS, and diff them: anything used-but-not-defined is a
      bug.
- [ ] **Page canvas / safe area**: the outermost content wrapper has 36px top/bottom and 32px
      left/right padding — not margin on the last child, not a fixed spacer. That padding lives on
      the element that actually scrolls (never a nested child of it), and that scrolling element uses
      `min-height`, never a capped `height` with `overflow: hidden` — confirm content taller than one
      viewport isn't clipped and the trailing 36px is reachable by scrolling to the true bottom.
- [ ] **Header/surface separation**: any title, description, or action row describing a piece of
      content (a card, table, list, chart, widget — anything) is a Page/Section/Table header sitting
      outside and above that content's own surface — never merged into the same
      bordered/elevated/filled surface as the content. A card's own `card-header-row` kicker is not a
      substitute for this. That header sits `12px` above its content (a table header above its table
      is the same rule), and unrelated sections are separated by `48px`.
