# Depth, Radius & Motion

The three rules that decide how any surface or control reads: which surface idiom it uses, how round
its corners are, and how it reacts to the pointer. They sit in one file because a component needs all
three at once.

## Depth & Elevation

Depth is **soft, cool-toned, and layered** — never a single hard black drop-shadow. There are three parallel idioms for "this is a distinct surface," and the system deliberately mixes all three across the product (see [design-system/cards.md](../design-system/cards.md)'s C0 for the copy-pasteable base + variants):

1. **Bordered flat surface** (`is-outlined`, the default): `--color-white` background, `1px solid var(--border)` border, `--radius-xs` (4px) corner radius, no shadow. Used whenever the surface sits statically among other content.
2. **Filled flat surface** (`is-filled`): no border, no shadow — background recedes to `--bg-secondary-surface` (`--color-background-alpha`) instead. Used to visually group or single out a surface against a white page, without adding a border or a shadow.
3. **Elevated shadow surface** (`is-elevated`): no border, a soft multi-layer shadow instead (`--shadow-card` at rest). Used when a card needs to read as "lifted" above the page, e.g. when composing several cards side by side.

Don't combine idioms on the same surface (no border + shadow, no border + filled background) — pick exactly one.

Floating UI (popovers, dropdowns, side panels) use heavier, more diffuse shadows and never a border alone: `--shadow-floating-container` for menus/popovers, `--shadow-floating-side-panel` for panels sliding in from an edge, `--shadow-modal` for modal dialogs, `--shadow-dragged-item` for anything being dragged. All of these shadows are tinted with the same dark-blue ink (`hsl(226.7deg 73% 7.25%)` at low opacity) rather than neutral black — this is what gives Pigment's depth its characteristic "cool" quality instead of a muddy grey.

The modal backdrop is `--color-backdrop` (rgba(2,13,35,0.6)) — again, ink-blue, not pure black.

### Focus Is Depth Too

Every focusable control gets the exact same focus treatment: `--focus-ring` (#95B9FF), sitting just outside the control's own border. It's implemented as a `box-shadow`, not a native `outline`, so it composes cleanly alongside each control's own border/box-shadow. This is applied identically to buttons, inputs, checkboxes, radios, switches, tabs — never customize focus styling per component.

## Shapes & Radius

| Value                                     | Use                                                                                                                                                                     |
| ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--radius-xxs` (2px)                       | Checkboxes only                                                                                                                                                         |
| `--radius-xs` (4px)                        | The default radius for almost everything: buttons, inputs, chips (rectangular badges), cards, tiles, tooltips, snackbars, segmented-toggle container, tabs focus states |
| `--radius-sm` (6px)                        | Modals, horizontal-tab focus ring corners                                                                                                                               |
| `--radius-md` (8px)                        | Occasional larger panels                                                                                                                                                |
| `--radius-lg` / `--radius-xl` (12px / 16px) | Large decorative surfaces only — rare in functional UI                                                                                                                  |
| `--radius-circular` (5000px)               | Fully round controls: radio buttons, switches, avatars, pill-shaped chips (`ChipButton`)                                                                                |

The rule of thumb: **rectangular controls get `--radius-xs`, pill/round controls get fully circular, nothing in between.** A "slightly rounded rectangle" at `--radius-lg`/`--radius-xl` is reserved for large, infrequent surfaces (illustration containers, hero cards) — using it on a button or input would read as off-brand.

### Border Weights

- **1px solid `--border`** is the universal structural border — cards, dividers, table rows, tab dividers. Inputs are the one exception, defaulting to `--color-grey-20` instead.
- ****2px**** borders exist only on buttons and toggle-style controls (the border doubles as part of the click target and color language, e.g., a secondary button's transparent border becomes visible on state changes).
- Error state always swaps the border color to `--color-negative-50`, never changes the border width.

## Motion

Two transition curves cover the entire system:

- **`--motion-snappy`** (150ms ease-in) — hover/press feedback on buttons, toggles, tabs, and inputs: border-color, background-color, color, and box-shadow changes — including the focus ring, which is itself a box-shadow and transitions at this speed.
- **`--motion-soft`** (200ms ease-in-out) — reserved for anything involving opacity/layout change: menus opening, modals appearing.

Don't introduce bouncy easing, longer durations, or spring physics — motion in this system is fast and utilitarian, in service of feedback, never decoration.
