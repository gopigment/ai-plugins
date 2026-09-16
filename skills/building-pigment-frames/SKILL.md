---
name: building-pigment-frames
description: Use this skill whenever you're building, updating, or troubleshooting a Pigment Frame — a full-page, custom-coded visualisation (JavaScript, canvas, or inline SVG) that reads Pigment data through the PigmentSDK. Reach for it any time a native View (Grid, Chart, KPI) or Board can't produce what's being asked for - a bespoke chart, a custom canvas drawing, an interactive widget, or a standalone page in the left navigation that behaves like a small app. Also use it if a Frame you've already built shows up blank or broken and needs debugging.
metadata:
  skill_path: /skills/building-pigment-frames/SKILL.md
  base_directory: /skills/building-pigment-frames
---

# Building Pigment Frames

A **Frame** is a standalone, full-page custom visualisation rendered from author-supplied JavaScript that reads Pigment data through the PigmentSDK. Use a Frame only when native Views, Boards, and chart types cannot express the required visual.

## When to Use

- Bespoke visual (canvas, inline SVG, custom DOM) that no native View mode (`Grid` / `Chart` / `KPI`) can produce. No CDN; inline all code.
- Full page in left navigation (like Boards). No Frame widget; a Frame *is* the page.

Frames do not support scenarios. Do not attempt to build them, edit them, compare them, or provide workarounds regarding them, it won't work. If the user requests to work with scenarios, tell them early that it is not supported.

## Tools and Workflow

The Frame JS lives in a file on your filesystem. `tool:create_frame` and `tool:update_frame` take the `path` of that file, never the source itself.

| Tool | Use |
| --- | --- |
| `tool:write_file` | Author the JS file. The only way to get JS to `tool:create_frame` / `tool:update_frame`. |
| `tool:edit_file` | Change the JS file in place. Prefer it over rewriting the whole file. |
| `tool:create_frame` | Create once (name, `path`, bindings, data sources). Errors on name collision. |
| `tool:update_frame` | Push the current file contents to an existing Frame; full replacement of `name`, `path`, `bindings`, `data_sources`. |
| `tool:search_frames` | Look frames up. Pass `id` or `name` for one frame, which returns its full definition (`bindings` included, ready for update). Pass neither to list ids + names; add `show_details: true` to get every body too. Empty result means no match. |

1. Identify the raw data (Metrics & Lists) that you would like to display on the Frame (ask the user to confirm if needed). No backing View or Table is needed: the Frame reads Metrics and List Properties directly through **data sources** that shape the rows (`labels`), columns (`values`) and pre-aggregation filters (`selectors`) it needs. Ask the user to confirm that shape before writing any JS.
2. Declare the **bindings** (Metrics, Lists, List Properties, Variables) and one **data source** per dataset the Frame reads (see [Data sources](#data-sources)). Warnings: a) rows are capped at 1,000 per window ; b) concurrent subscriptions per Frame are limited to 10 data-source subscriptions and 20 List subscriptions.
3. `tool:write_file` the JS body to a file (e.g. `/frames/revenue-heatmap.js`), then `tool:create_frame` once with that `path`. Never re-create (name collision). Use `tool:search_frames` with a `name` if unsure it exists.
4. For every later change, `tool:edit_file` the same file, then `tool:update_frame` with the same `path`. A Frame write reads the file at call time, so the file is the source of truth - keep editing it rather than rewriting it from scratch.
5. Always resend the complete `bindings` and `data_sources` arrays when calling `tool:create_frame` / `tool:update_frame`; an omitted `data_sources` drops every data source. `tool:search_frames` by `id` or `name` gives you the `bindings` array but not the data sources — keep track of the ones you declared; a bare listing gives neither.
6. **Grow a large body in the file, not in tool arguments.** A Frame's JS body can be long enough that emitting it whole in one response risks hitting the model's max output size. For a substantial new Frame, `tool:write_file` a working skeleton (IIFE, `#app` setup, `__cleanup`, and clearly unique placeholder markers for the sections still to come — e.g. `// SECTION: subscribe`, `// SECTION: render`), then fill each placeholder with `tool:edit_file`, one section at a time. Call `tool:create_frame` / `tool:update_frame` once the file is complete. Make one of those sections `// SECTION: styles`.
7. If the user's request asks for a Pigment-native, on-brand, or polished look, and skill:designing-pigment-frames is among your available skills (it is feature-flagged, so it may not be): code the Frame first, leaving `// SECTION: styles` as a placeholder, then load that skill and fill the placeholder with its design pass before you finish. Without that ask, or when the skill is unavailable, a plain placeholder style is fine; do not load the skill on your own.
8. If a Frame write reports the file was not found, the file was never written or the path is wrong: check with `tool:ls`, then `tool:write_file` before retrying.
9. Whenever the user reports a bug (blank page or anything that does not work expectedly), you can ask the user to record the logs using the dedicated buttons in the UI. You will be able to read the recorded logs.

## Sandboxed Runtime

Iframe: `sandbox="allow-scripts"`, opaque origin. Host shell loads `/pigment-sdk.js`, then your file contents as a separate script into `#app`.

**Allowed:** ES2015+ JS, DOM, Canvas, inline SVG/CSS. **Blocked:** network (`fetch`, CDN, `@import`), storage, `alert`/`confirm`/`prompt`, Workers, nested iframes, `parent`/`top`.

**Rules:**
1. The file is pure JS (not HTML). Populate `#app`; no `<script>` wrapper.
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

JSON array on `create_frame` / `update_frame`. Tool input uses **snake_case** (`metric_id`, `list_id`, `list_property_technical_name`, `can_read`, `can_write`).

| `type` | id field | SDK use |
| --- | --- | --- |
| `Metric` | `metric_id` | `subscribeToDataSource` via data source `values`; `editValue` |
| `List` | `list_id` | `subscribeToDataSource` via data source `labels` and `selectors`; `dynamicFilters`; `subscribeToItems`; `addItem`; `editItem` |
| `ListProperty` | `list_id` + `list_property_technical_name` | `subscribeToDataSource` via data source `values` and `labels`; `editItem` |
| `Variable` | `variable_id` | `dynamicFilters` on `subscribeToDataSource` |
| `View` | `view_id` | legacy `subscribeToVizualization` only, migrate away from it for `subscribeToDataSource` |

- `can_read: true` on every binding referenced in `values` of a data source
- `can_read: true` also required for `subscribeToVizualization` / `subscribeToItems`.
- `can_write: true` for List bindings used with `addItem` / `editItem`, for ListProperty bindings used with `editItem`, and for Metric bindings used with `editValue`.

A binding only grants access to a raw entity and names it for later use.

## Data sources

A data source describes how a Frame can access data. They are declared as a JSON array called `data_sources` and passed to `create_frame` / `update_frame`, next to `bindings`. `data_sources` can only reference existing entities by referencing `bindings` declared above by their names.

Where a binding just grants access to one entity, a data source shapes several of them into a dataset returned via `subscribeToDataSource`. The same binding can back several data sources.

| Field | Role |
| --- | --- |
| `name` | The name used to reference the data source from the code |
| `values` | Where we want to read the data from? How to aggregate data for missing structuring Dimensions? Used as columns of the resulting data |
| `labels` | Which Dimensions (or their properties that are themselves Dimensions) do we want to see and keep? Used as rows of the resulting data |
| `selectors` | What dynamic filtering capability we want to have? Selectors filter the data before `values` aggregate it for missing Dimensions |

We support two kinds of data sources: data source responsible to extract data linked to one or several Metrics and data source on List to extract values linked to some specific properties of the considered List.

Here are the extra constraints for a data source on Metrics:

| Field | Requirements |
| --- | --- |
| `values[i].binding` | Binding of type `Metric`. The Metric we want to extract the data from |
| `values[i].aggregator` | How the value is combined over the Dimensions it depends on that are not set in `labels` (`Sum`, `Avg`, `Min`, `Max`, `Count`, `First`, `Last`, ...). Unspecified means to keep the Metric's own aggregation |
| `labels[i].binding` | Binding of type `List`, or of type `ListProperty` whose property targets a Dimension. Must abide by the Dimension rules below |
| `selectors[i].binding` | Must abide by the same rules as `labels`. Filter applied before aggregation |

_Dimension rules._ Both `labels` and `selectors` must satisfy, against every Metric specified in `values`:

- Binding of type `List`: that List must be a structural Dimension of the Metric.
- Binding of type `ListProperty`: the List the property is extracted from must be a structural Dimension of the Metric. The property only groups the rows, so the Dimension it targets is never what gets checked.

And for one on a List:

| Field | Requirements |
| --- | --- |
| `values[i].binding` | Binding of type `ListProperty`. The properties we want to extract the data from. Must all belong to the same List |
| `values[i].aggregator` | Must stay unspecified |
| `labels[i].binding` | Binding of type `List` or `ListProperty`. Must all belong to the same List as the `values`. Last `labels` must be a binding to the List itself, all others must be properties targeting a Dimension |
| `selectors[i].binding` | Binding of type `ListProperty`, the property targeting a Dimension. Must all belong to the same List as the `values` |

The example below declares a data source on a single Metric declared as `revMetric` in the bindings. It asks for the `monthList` to be used as `labels` and `countryList` to be used as `selectors`. It implicitely implies that `revMetric` must be at least dimensioned by `monthList` and `countryList`. For any Dimension not declared in `labels`, at least `countryList`, it will apply a sum of the values for the countries being asked (or all of them if not specified).

```json
{
  "bindings": [
    { "name": "revMetric", "type": "Metric", "metric_id": "<uuid>", "can_read": true, "can_write": false },
    { "name": "monthList", "type": "List", "list_id": "<uuid>", "can_read": false, "can_write": false },
    { "name": "countryList", "type": "List", "list_id": "<uuid>", "can_read": false, "can_write": false }
  ],
  "data_sources": [
    {
      "name": "revenueByTime",
      "values": [{ "binding": "revMetric", "aggregator": "Sum" }],
      "labels": [{ "binding": "monthList" }],
      "selectors": [{ "binding": "countryList" }]
    }
  ]
}
```

## PigmentSDK

Methods: `subscribeToDataSource`, `subscribeToItems`, `addItem`, `editItem`, `editValue`. `subscribeToVizualization` still exists but is legacy and should never be used in new code and be removed from existing code.

### Data source subscription

Use it to pull data from Pigment for your Frame. You can only pull data from data sources you previously defined into the `data_sources` field. Data source get pulled via `subscribeToDataSource` by passing their name.

```js
const dsSub = window.PigmentSDK.subscribeToDataSource('revenueByTime', {
  onData: function (data) { if (!isReady(data)) return; render(data); },
  onError: function (err) { /* show err.message */ },
  dynamicFilters: [], // optional; selections on the data source `selectors`, no selection or empty means All
  scroll: { offset: 0, numberOfRows: 200 } // optional; only usable on List case, omit to fetch the default window
});
// dsSub.updateDynamicFilters([{ binding: 'countryList', selection: ['France', 'Spain'] }]);
// dsSub.updateScroll({ offset: 200, numberOfRows: 200 });
// dsSub.unsubscribe();
```

#### Data shape

The shape of the `data` received on `onData` is the following:

- `data.rows` is an array made of all the rows pulled from the data set. You should expect one row per combination of `labels` coming with data linked to it on one of the requested `values`. When defining for no `labels`, you should expect to receive one row not more
- `data.rows[i]` contains two fields:
  - `labels` is an array containing the labels you requested in the order defined on the definition of the data source. Entries in this array are either strings, `{ kind: 'blank' }` for a label being explicitly set to unassigned, or `{ kind: 'loading' }` if we are still pulling the label.
  - `values` is an array containing the values you requested in the order defined on the definition of the data source. Each value is either a number, boolean, ISO-8601 date string, item label, plain string, `null` (no value), or `{ kind: 'loading' | 'unknown' }` for a List item reference still resolving or unaccessible. The type you get matches the type of the value you requested.
- `data.rowOffset` is the 0-based index of the first row in `data.rows` within the full data source, i.e. `data.rows[n]` is row `data.rowOffset + n` of the data source.
- `data.totalRowCount` is the total number of rows in the data source, regardless of the requested scroll window.

Applied to the same `revenueByTime` data source defined above (`values: [revMetric]` summed with `Sum`, `labels: [monthList]`, `selectors: [countryList]`), subscribing with no `dynamicFilters` (all countries aggregated together) delivers one row per month:

```json
{
  "rows": [
    { "labels": ["January"], "values": [145285.91] },
    { "labels": ["February"], "values": [156964.98] },
    { "labels": ["March"], "values": [162172.59] }
  ],
  "rowOffset": 0,
  "totalRowCount": 3
}
```

Since no `dynamicFilters` were applied, January's value is `revMetric` summed across every `countryList` item, restricted to that item's data for January. If we filtered on France and Italy instead, January's value would only be the sum of `revMetric` for those two countries' January data.

#### Dynamic filters

`dynamicFilters` items are `{ binding, selection }`:
- `binding` must be a `List` binding, not a `ListProperty` one. In case the `selectors` declared a `ListProperty`, the `binding` used here must be the List being targeted by the property. In other words, the List that we reach from `list_id.list_property_technical_name`.
- `selection` is an array of Item labels of that List to keep. An empty `selection` keeps all the Items

Dynamic filters apply to `selectors` before aggregation. An unfiltered selector aggregates all its Items together, while a `selection` aggregates only those Items. A filter matching none of the data source's `selectors` is silently ignored.

```js
dsSub.updateDynamicFilters([{ binding: 'countryList', selection: ['France', 'Italy'] }]);
```

Use `updateDynamicFilters` on the existing handle, do not resubscribe when relying on the same data source.

#### Scroll & windowing

`scroll`/`updateScroll` and `data.rowOffset`/`data.totalRowCount` are two sides of the same mechanism:
- Restricted to data sources on Lists. Passing `scroll` on a data source on Metrics has no effect for now but this may change in the future.
- `data.rowOffset`/`data.totalRowCount` are always returned, regardless of whether `scroll` applies.
- `numberOfRows` is limited to 1,000

```js
dsSub.updateScroll({ offset: 200, numberOfRows: 200 });
```

Use `updateScroll` on the existing handle, do not resubscribe when relying on the same data source.

### List subscription

```js
const listSub = window.PigmentSDK.subscribeToItems('countryList', {
  onData: function (d) { d.items; d.partialResult; },
  onError: function (err) { /* show error */ }
});
```

`partialResult === true`: truncated list; no pagination API. Show a warning.

Warning: subscribeToItems only returns the Item name, not the Properties. To fetch List Items with Properties, use `subscribeToDataSource`.

### Writes

```js
// Add a new Item to a List (can_write: true required)
await window.PigmentSDK.addItem('countryList', { 'Country Name': 'France' });

// Edit an existing Item in a List (can_write: true required on each ListProperty binding used)
await window.PigmentSDK.editItem('countryList', 'France', { countryCodeProperty: 'FR' });

// Edit a cell value in a Metric (can_write: true required on the Metric binding)
await window.PigmentSDK.editValue('revMetric', { 'countryList': 'France', 'timeList': '2024' }, 42000);
```

`editItem(listAlias, item, values)`: `item` is the current name of the Item; `values` is a partial map of `ListProperty` binding aliases (declared in `bindings`) to new values.

`editValue(metricAlias, coordinates, value)`: `coordinates` maps each List alias (dimension) to the selected item label; `value` is `boolean | number | string | null`.

## Implementation Patterns

### Loading guard

`onData` fires multiple times (loading → final, then again on every refresh). Guard with `isReady()`; zero rows after loading is valid empty.

```js
function hasLoadingKind(arr) {
  for (let i = 0; i < arr.length; i++) {
    const item = arr[i];
    if (typeof item === 'object' && item.kind === 'loading') return true;
  }
  return false;
}
function isReady(data) {
  for (let r = 0; r < data.rows.length; r++) {
    if (hasLoadingKind(data.rows[r].labels) || hasLoadingKind(data.rows[r].values)) return false;
  }
  return true;
}
```

### Lifecycle

- One subscription per data source; filter changes via `updateDynamicFilters`; never resubscribe to refresh.
- Always implement `onError`; show user-visible loading / error / empty states.
- `partialResult`: show warning banner.

### Render and performance

- Prefer canvas/SVG for charts; create once, clear and redraw. Use canvas if `rows × cols > 500`.
- **HiDPI/Retina**: always scale canvas by `devicePixelRatio` — setting canvas dimensions in CSS pixels only (`canvas.width = el.offsetWidth`) causes blurriness on Retina screens because the browser stretches the low-res bitmap to fill the CSS size. Set physical pixels via `canvas.width = el.offsetWidth * dpr; canvas.height = el.offsetHeight * dpr; ctx.scale(dpr, dpr);` and keep CSS size via `canvas.style.width/height`.
- `root.innerHTML` tears down listeners; call `attachEvents()` after each render, or delegate on `root` once.
- Debounce `onData` renders (~16ms) and resize (~120ms). Cache `lastData`.
- Resize: `window.resize` + `ResizeObserver` on `document.documentElement` (host uses `AutoSizer`).
- Tooltips on `document.body`; remove in `__cleanup`.
- For PDF printing, use the browser's native print-to-PDF (`window.print()`) only.

### Cleanup (leak prevention)

In `root.__cleanup`: `unsubscribe()` all subs; `removeEventListener` all named global listeners; `clearTimeout`/`clearInterval`; `cancelAnimationFrame`; `disconnect()` observers; remove `document.body` nodes; set `lastData = null`. Never use anonymous functions for global listeners.

## Legacy: View subscription (`subscribeToVizualization`)

> **Discouraged, scheduled for decommission.** `subscribeToVizualization` reads a pre-built View through a `View` binding. We highly recommend migrating away from it. Never use it in a new Frame and migrate any Frame you touch that still relies on it to `subscribeToDataSource`. The rest of this section only exists to understand such legacy code.

- **Backing Views:** legacy Frames needed a backing View on a List, Metric or Table with the right layout. Data sources replace this step entirely.
- **Bindings:** a `View` binding (`view_id`, `can_read: true`) per View, plus `List` / `Variable` bindings for `pageDefinitions` (no `can_read` needed).
- **Limits:** shares the 10 concurrent subscriptions with `subscribeToDataSource`; rows capped at 1,000 per window.

```js
const vizSub = window.PigmentSDK.subscribeToVizualization('salesView', {
  onData: function (data) { if (!isReadyViz(data)) return; render(data); },
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
| --- | --- |
| 1D bar (one metric) | `cells[0][r]`, row label from `labels.rows[r]` |
| Grouped bars | `cells[c][r]` per column `c` |
| KPI | `cells[0][0]` |

**Cells:** `number | string | boolean | null | { kind: 'loading' | 'unknown' }` (dates as ISO strings). **Labels:** `string | { kind: 'total' | 'blank' | 'loading' }`. Only `'loading'` means not ready.

**Windowed data:** `data.rowOffset` is the 0-based index of the first row in the current window. `data.totalRowCount` is the total number of rows in the View (independent of the window). Use `updateScroll` to page through large datasets; `numberOfRows` is capped at 1,000.

### Page selection

String aliases work for both **List** and **Variable** bindings. Pass `{ kind: 'metric' }` as the alias to select specific Metrics from the View by their ID. No scenario placeholder.

```js
// Select by List/Variable alias
vizSub.updatePageDefinitions([{ alias: 'countryList', selection: ['France'] }]);

// Select by Metric (selection contains Metric IDs from the View)
vizSub.updatePageDefinitions([{ alias: { kind: 'metric' }, selection: ['metric-id-1'] }]);
```

Use `updatePageDefinitions` on the existing handle; do not resubscribe.

### Loading guard

`onData` fires multiple times (empty → loading → final). Guard with `isReady()`; zero rows after loading is valid empty.

```js
function hasLoadingKind(arr) {
  for (let i = 0; i < arr.length; i++) {
    const item = arr[i];
    if (typeof item === 'object' && item.kind === 'loading') return true;
  }
  return false;
}
function isReadyViz(data) {
  if (!data || !data.labels || !data.labels.columns.length) return false;
  return !hasLoadingKind(data.labels.rows) && !hasLoadingKind(data.labels.columns) && !hasLoadingKind(data.cells);
}
```
