---
name: building-pigment-frames
description: How to build Pigment Frames, custom full-page JavaScript visualisations that read Pigment data through the PigmentSDK inside a sandboxed iframe. Use when creating or editing a Frame.
metadata:
  skill_path: /skills/building-pigment-frames/SKILL.md
  base_directory: /skills/building-pigment-frames
---

# Building Pigment Frames

A **Frame** is a standalone, full-page custom visualisation rendered from author-supplied JavaScript that reads Pigment data through the PigmentSDK. Use a Frame only when native Views, Boards, and chart types cannot express the required visual.

## When to Use

- Bespoke visual (canvas, inline SVG, custom DOM) that no native View mode (`Grid` / `Chart` / `KPI`) can produce. No CDN; inline all code.
- Full page in left navigation (like Boards). No Frame widget; a Frame *is* the page.

## Tools and Workflow

| Tool | Use |
|---|---|
| `tool:create_frame` | Create once (name, body, bindings). Errors on name collision. |
| `tool:update_frame` | All later edits; full replacement of `name`, `body`, `bindings`. |
| `tool:search_frames` | Look frames up. Pass `id` or `name` for one frame, which returns its full definition (`bindings` included, ready for update). Pass neither to list ids + names; add `show_details: true` to get every body too. Empty result means no match. |

1. Backing **View(s)** on **Metric or Table** with correct pivot layout.
2. `tool:create_frame` once; then `tool:update_frame` for every change. Never re-create (name collision). Use `tool:search_frames` with a `name` if unsure it exists.
3. Always resend the complete `bindings` array on update. `tool:search_frames` by `id` or `name` gives you that array; a bare listing does not.

## Sandboxed Runtime

Iframe: `sandbox="allow-scripts"`, opaque origin. Host shell loads `/pigment-sdk.js`, then your `body` as a separate script into `#app`.

**Allowed:** ES2015+ JS, DOM, Canvas, inline SVG/CSS. **Blocked:** network (`fetch`, CDN, `@import`), storage, `alert`/`confirm`/`prompt`, Workers, nested iframes, `parent`/`top`.

**Rules:**
1. `body` is pure JS (not HTML). Populate `#app`; no `<script>` wrapper.
2. Wrap in an IIFE; call `root.__cleanup()` before re-init on hot-reload.
3. Layout from `window.innerWidth` / `window.innerHeight` (`#app` has near-zero height).
4. Single-quote HTML attributes in JS strings; avoid multiline template literals in tool args.

```js
(function () {
  'use strict';
  const root = document.getElementById('app');
  if (root.__cleanup) root.__cleanup();
  root.style.cssText = 'position:fixed;inset:0;width:100%;height:100%;overflow:hidden;';
  // subscriptions, render, events ...
  root.__cleanup = function () { /* unsubscribe, remove listeners, clear timers */ };
})();
```

Do not copy the Frame editor placeholder; it omits IIFE cleanup and uses template literals.

## Bindings

JSON array on `create_frame` / `update_frame`. Tool input uses **snake_case** (`view_id`, `list_id`, `can_read`, `can_write`).

| `type` | id field | SDK use |
|---|---|---|
| `View` | `view_id` | `subscribeToVizualization` |
| `List` | `list_id` | `subscribeToItems`, `pageDefinitions`, `addItem`, `editItem` |
| `Metric` | `metric_id` | `editValue` |
| `Variable` | `variable_id` | `pageDefinitions`; no other SDK consumer |

- `can_read: true` required for `subscribeToVizualization` / `subscribeToItems`.
- List used only for `pageDefinitions` does not need `can_read`.
- `can_write: true` for List bindings used with `addItem` / `editItem`, and for Metric bindings used with `editValue`.

## PigmentSDK

`window.PigmentSDK` is frozen. Methods: `subscribeToVizualization`, `subscribeToItems`, `addItem`, `editItem`, `editValue`.

### View subscription

```js
const vizSub = window.PigmentSDK.subscribeToVizualization('salesView', {
  onData: function (data) { if (!isReady(data)) return; render(data); },
  onError: function (err) { /* show err.message */ },
  pageDefinitions: [],
  scroll: { offset: 0, numberOfRows: 200 }  // optional; omit to fetch the default window
});
// vizSub.unsubscribe();
// vizSub.updatePageDefinitions([...]);
// vizSub.updateScroll({ offset: 200, numberOfRows: 200 });
```

### Data shape

Column-major: `cells[c][r]`. `cells.length === labels.columns.length`; `cells[c].length === labels.rows.length` (for the current window).

Label paths: `labels.rows[r]` and `labels.columns[c]` are arrays (one entry per pivot level). Use last string entry as display name.

| Visual | Lookup |
|---|---|
| 1D bar (one metric) | `cells[0][r]`, row label from `labels.rows[r]` |
| Grouped bars | `cells[c][r]` per column `c` |
| KPI | `cells[0][0]` |

**Cells:** `number | string | boolean | null | { kind: 'loading' | 'unknown' }` (dates as ISO strings). **Labels:** `string | { kind: 'total' | 'blank' | 'loading' }`. Only `'loading'` means not ready.

**Windowed data:** `data.rowOffset` is the 0-based index of the first row in the current window. `data.totalRowCount` is the total number of rows in the View (independent of the window). Use `updateScroll` to page through large datasets; `numberOfRows` is capped at 1,000.

### List subscription

```js
const listSub = window.PigmentSDK.subscribeToItems('countryList', {
  onData: function (d) { d.items; d.partialResult; },
  onError: function (err) { /* show error */ }
});
```

`partialResult === true`: truncated list; no pagination API. Show a warning.

### Writes

```js
// Add a new Item to a List (can_write: true required)
await window.PigmentSDK.addItem('countryList', { 'Country Name': 'France' });

// Edit an existing Item in a List (can_write: true required)
await window.PigmentSDK.editItem('countryList', 'France', { 'Country Code': 'FR' });

// Edit a cell value in a Metric (can_write: true required on the Metric binding)
await window.PigmentSDK.editValue('revMetric', { 'countryList': 'France', 'timeList': '2024' }, 42000);
```

`editItem(listAlias, item, values)`: `item` is the current name of the Item; `values` is a partial map of property friendly-names to new values.

`editValue(metricAlias, coordinates, value)`: `coordinates` maps each List alias (dimension) to the selected item label; `value` is `boolean | number | string | null`.

### Page selection

String aliases work for both **List** and **Variable** bindings. Pass `{ kind: 'metric' }` as the alias to select specific Metrics from the View by their ID. No scenario placeholder.

```js
// Select by List/Variable alias
vizSub.updatePageDefinitions([{ alias: 'countryList', selection: ['France'] }]);

// Select by Metric (selection contains Metric IDs from the View)
vizSub.updatePageDefinitions([{ alias: { kind: 'metric' }, selection: ['metric-id-1'] }]);
```

Use `updatePageDefinitions` on the existing handle; do not resubscribe.

## Implementation Patterns

### Loading guard

`onData` fires multiple times (empty → loading → final). Guard with `isReady()`; zero rows after loading is valid empty.

```js
function hasLoadingKind(arr) {
  for (let i = 0; i < arr.length; i++) {
    const item = arr[i];
    if (Array.isArray(item)) { if (hasLoadingKind(item)) return true; }
    else if (item && typeof item === 'object' && item.kind === 'loading') return true;
  }
  return false;
}
function isReady(data) {
  if (!data || !data.labels || !data.labels.columns.length) return false;
  return !hasLoadingKind(data.labels.rows) && !hasLoadingKind(data.labels.columns) && !hasLoadingKind(data.cells);
}
```

### Lifecycle

- One subscription per alias; page changes via `updatePageDefinitions`; never resubscribe to refresh.
- Always implement `onError`; show user-visible loading / error / empty states.
- `partialResult`: show warning banner.

### Render and performance

- Prefer canvas/SVG for charts; create once, clear and redraw. Use canvas if `rows × cols > 500`.
- **HiDPI/Retina**: always scale canvas by `devicePixelRatio` — setting canvas dimensions in CSS pixels only (`canvas.width = el.offsetWidth`) causes blurriness on Retina screens because the browser stretches the low-res bitmap to fill the CSS size. Set physical pixels via `canvas.width = el.offsetWidth * dpr; canvas.height = el.offsetHeight * dpr; ctx.scale(dpr, dpr);` and keep CSS size via `canvas.style.width/height`.
- `root.innerHTML` tears down listeners; call `attachEvents()` after each render, or delegate on `root` once.
- Debounce `onData` renders (~16ms) and resize (~120ms). Cache `lastData`.
- Resize: `window.resize` + `ResizeObserver` on `document.documentElement` (host uses `AutoSizer`).
- Tooltips on `document.body`; remove in `__cleanup`.

### Cleanup (leak prevention)

In `root.__cleanup`: `unsubscribe()` all subs; `removeEventListener` all named global listeners; `clearTimeout`/`clearInterval`; `cancelAnimationFrame`; `disconnect()` observers; remove `document.body` nodes; set `lastData = null`. Never use anonymous functions for global listeners.
