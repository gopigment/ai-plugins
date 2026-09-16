# Spec-Only Components

These components have a written spec but **no copy-paste recipe** — build each one from the values
below. This is the only documentation that exists for them, so do not derive their look from the
token tables instead.

Everything here obeys the shared rules: 4px-grid spacing, `--radius-xs` on rectangular controls and
`--radius-circular` on round ones, `--focus-ring` on every focusable element, and `--motion-snappy`
for hover and press feedback. See
[foundations/depth-radius-motion.md](../foundations/depth-radius-motion.md).

## Checkbox / Radio

|           | Checkbox                                       | Radio                                                       |
| --------- | ---------------------------------------------- | ----------------------------------------------------------- |
| Size      | 16px (12px small)                              | 16px                                                        |
| Radius    | `--radius-xxs` (2px)                           | `--radius-circular`                                         |
| Unchecked | 1px border `--color-grey-50`, white fill       | same                                                        |
| Checked   | fill + border `--color-primary-50`, white check | fill + border `--color-primary-50`, white 10px inner dot    |
| Disabled  | fill + border `--color-grey-20`                | same                                                        |
| Error     | border and fill switch to `--color-negative-50` | same, plus a `--color-negative-10` background tint          |

Use **checkboxes** for multi-select, at any count including zero or all. Use **radios** for a single
choice among 3 or more mutually-exclusive options laid out vertically. For 2-5 options that read
better inline, use a [toggle button group](toggle-buttons.md) instead.

## Switch / Segmented Toggle

These are not interchangeable — each answers a different question.

- **Switch** — "is this setting on or off, right now, with no submit step." Track 44px × 24px (28px ×
  16px small), `--radius-circular`, `--color-grey-30` track when off, brand-colored when on, white
  thumb, no shadow. It has no error state by design.
- **Segmented Toggle** — "which of 2-4 views or modes is active", e.g. Table vs. Chart. A container
  with a `--color-background-alpha` background and `--radius-xs`, holding a white sliding indicator
  that animates between segments. The indicator is `--radius-xs` minus 1 with a
  `0px 1px 4px rgba(105, 109, 121, 0.16)` shadow. Type is `fieldset-regular`; inactive text
  `--text-secondary`, hover and active text `--text-highlight`. Height 32px (28px small).
- **Toggle Button** — a button that holds a pressed state, usually grouped. See
  [toggle-buttons.md](toggle-buttons.md).

## Range Input / Slider

- **Rail and track** — a 2px rail in `--color-background-alpha`, with the filled track at the same
  height in `--color-primary-50` for the selected range. Both use `--radius-circular` at the ends.
  No tick or step markers on the track.
- **Thumb** — a 16px `--radius-circular` thumb sitting on the track, with no drop shadow and no halo
  on hover or press. On keyboard focus, apply `--focus-ring` to the thumb.
- **Value readout** — on drag or hover of the thumb, show the current value in a dark
  [tooltip](tooltips.md) above it, in `running-small`.
- **Disabled** — mute the thumb and filled track to `--color-grey-20` and the rail to
  `--color-background-alpha`.

## Tabs

- **Horizontal tabs (underline style)** — top-level navigation within a page or content area. A
  `2px solid var(--color-primary-50)` underline indicator, a 24px gap between tabs, and an optional
  `1px solid var(--border)` bottom border across the whole bar. Inactive text `--text-primary`,
  hover `--text-highlight`. The focus ring corners are `--radius-sm`.
- **Vertical tabs (pill style)** — sidebar-style navigation, e.g. a settings page's left-hand
  section list. Each tab is a 36px-tall, `--radius-xs`-rounded row with 6px 16px padding; selected
  and hovered tabs take a `--color-background-alpha` background fill rather than an underline. Use
  this pattern whenever navigation sits in a vertical list next to the content rather than across
  its top.

## Modal

- **Sizing** — `min-width: 320px`, default `max-width: calc(100vw - 32px)` unless you set an
  explicit width. Typical dialog widths run 400-640px.
- **Structure** — header (24px padding, optional `1px solid var(--border)` bottom divider) → body
  (12px 24px padding) → footer with right-aligned actions, an 8px gap between buttons and an
  optional top divider.
- **Surface** — white, `--radius-sm` (6px) corners, `--shadow-modal`.
- **Overlay** — `--color-backdrop`.
- Use a modal for a focused, blocking task: a confirmation, or a single-object edit form. Not for
  browsing and not for multi-step flows, which belong in a full page or a side panel.

## Snackbar / Toast

Anchored bottom-right, `max-width: 352px`, `--radius-xs` corners, `--color-grey-90` background with
white text.

Pigment's snackbars deliberately **do not color-code by severity** — success, error and info all
share this one flat dark treatment. Do not invent a green or red toast. If severity must be visible,
put it in the message copy and an inline icon, never in the background color.

## Badge

A small count or status marker: `--radius-xs`, 16px tall, 19px minimum width.

## Icons

Functional UI icons are SVG at a 16×16 viewBox — 16px is 4×4px, so they sit on the 4px grid — with a
1.5px stroke width.

Larger, often multi-color "illustrative" icons exist at 24px, 40px and 48px for empty states,
onboarding and feature callouts.

Take the icon color from the same token as the adjacent text: a `--text-secondary` icon next to
`--text-secondary` text. Never pick an icon color independently.
