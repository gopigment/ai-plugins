# Overview

**Pigment Platform Interface System** (alpha-4). The interface language of the Pigment planning
platform — a calm, data-dense SaaS system built on the platform sans-serif, a single electric-blue
brand color (#0355F3), a near-white/near-black neutral scale, and a strict 4px spacing grid. There is
no decorative flourish: depth comes from thin borders and soft diffuse shadows, corners are gently
rounded, and every interactive control shares the same 150-200ms transition curve. This system
describes the platform's own UI (not a slide deck), and is meant for one-off interfaces — dashboards,
forms, reports, infographics — that render inside the Pigment app shell and must feel native to it.

Token values live in [`../styles/stylesheet.md`](../styles/stylesheet.md), which is the single source
of truth. Every `--variable` name these reference files mention is defined there.

Pigment's interface system is a **calm, data-dense enterprise SaaS language**, not a marketing or editorial system. It exists to make dense planning data — grids, forms, dashboards, dependency graphs — legible for hours at a time without fatigue. Everything is quiet by design: one sans-serif typeface (the platform generic, `--font-sans`), one brand hue (an electric blue, `--color-primary-50` / #0355F3), a near-white-to-near-black neutral scale, and a small, disciplined set of semantic colors for status (positive green, cautious amber, negative red).

The system runs on a **strict 4px spacing unit** — nearly every margin, padding, gap, icon size and component height in the product is a whole multiple of 4px, with three deliberate half-step exceptions (2px, 6px, 10px) for hairlines, borders, and compact control padding. Corners are **gently rounded** (`--radius-xs` for almost everything interactive, up to `--radius-xl` for the rare large surface), never sharp, never pill-shaped except for fully circular controls (radio buttons, avatars, pill chips, switches). Depth is communicated through **thin 1px borders and soft, diffuse, blue-tinted shadows** rather than hard drop-shadows — nothing in the system casts a crisp black shadow. Motion is subtle and fast: a `--motion-snappy` for hover/press feedback and focus rings, a `--motion-soft` for anything that changes layout or opacity (menus, modals).

Because one-off interfaces built with this system **render inside the existing Pigment shell** (resizable left platform sidenav for navigation and resizable right chat conversation panel already present, both using `--bg-primary-surface`), these reference files describe the **content-area language only**: the surface, typography, controls, and layout conventions your page should use so it reads as "part of Pigment," not the surrounding chrome itself.

**Key characteristics:**

- One typeface (the sans-serif stack, `--font-sans`) for everything except code/formula text, which uses the monospace stack (`--font-mono`).
- One brand color, `--color-primary-50` (#0355F3), used for primary actions, links, active states, and focus rings — never for large background fills outside of primary buttons and brand accents.
- The page canvas — the root element your code owns — should use a (`--color-white`) background. Only in justified cases where some parts of the content need to be highlighted for the user (for example, you are building an input form and want the fieldsets to stand out) you should consider using `--color-grey-10` as the background color.
- Neutral surfaces are also white (`--color-white`) — the platform is a "white cards on white" system, not a stark white or dark system. Use `--color-grey-10` intentionally, never by default.
- Borders are always 1px. Structural surfaces (cards, dividers, table rows) use `--color-neutral-alpha`; inputs default to `--color-grey-20` instead. Both switch to `--color-negative-50` on error or `--color-primary-50`/`--color-primary-30` on an active/selected/focused state.
- Radius is small and consistent: `--radius-xs` is the default for buttons, inputs, chips, cards and tiles; `--radius-sm` for modals and focus-ring corners; full circle only for round controls.
- Shadows are soft, cool-toned, and layered (never a single hard shadow) — see `--shadow-card` and `--shadow-floating-container`.
- Every focus state is `--focus-ring` (#95B9FF) — this is non-negotiable for accessibility and is identical across every interactive component.
- Status color is used sparingly and always paired with the same three tones: positive (green), cautious (amber), negative (red) — there is no "info" color beyond the brand blue itself.
