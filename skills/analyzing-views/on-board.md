# On-board queries

The target view is available in the UVC — a widget of the board the user has open shows it — and the UVC carries the board's active page selectors. Those selectors are what the user is looking at, so they are required constraints on the answer — not suggestions.

**User Visible Context (UVC)**: for the purposes of this skill, the relevant UVC is the board UVC — the structured block injected when the user is viewing a board. It contains the board ID, widget display names, widget names, view IDs, and active page selector values. If the UVC contains a board ID, widget display names, and view IDs, use these directly to identify the block and any active filters — no block search call is needed.

## Board selector application

All active UVC page selectors are required constraints on the **final result** — the analysis must honour every selector. That does **not** mean every selector's values should be passed into every intermediate fetch (see Mapped pivots in SKILL.md).

For each active selector, look up its pivot type in the view's pivot configuration and apply accordingly:

- **Dimension** (direct): apply directly on the target dimension.
- **Grouping**: apply as a property filter on the source dimension using the property path from the pivot config.
- **Mapped** (through a mapping metric): follow the mapped dimension procedure in SKILL.md.

**User override**: if the user explicitly requests different filter values or asks to ignore a selector, their instruction takes precedence. Only override when explicit — never infer an override from absence of mention.

**Multi-value selectors**: if the board selector has multiple selected values, fetch once per selected value, union the results, then aggregate. Never drop values to avoid splitting. Mapped selectors are the exception: the mapping procedure handles all their selected values in one pass.

**Unsupported pivot type — Slice**: if an active UVC board selector has `kind: Slice`, a customer data fetch cannot replicate it. Disclose this explicitly to the user: state which selector is active and that the fetched result will not honour that constraint. Offer to proceed without it.

## Display grain

When the user asks about data on the current board/view, default the answer grain to the view's row/column pivots from its configuration. Only use a different grain if the user explicitly asks.

- **Dimension / Grouping** row or column pivots: pass them as `breakdown_dimensions`.
- **Mapped** row or column pivots: do not pass the mapped dimension directly — resolve via the mapping metric (same procedure as Mapped filters), then present at that mapped grain.

## Selector scope check

A board selector is only "in scope" for a metric if its target dimension appears in the resolved view's pivot config.

- **Not in pivot config** → first check whether this selector applies to a sibling metric (see Shared entity universe below). If no sibling relationship exists, silently ignore. Do not ask the user, do not attempt to apply.
- **In pivot config but mapping unclear** → hard stop. Ask clarification before returning any numbers.
- **Correction rule**: when re-applying a filter after a failed attempt, always re-consult the pivot configuration. Do not guess an alternative property path. The pivot config is the single source of truth — even when fixing mistakes.

## Shared entity universe

When a board selector applies to a sibling metric but NOT the target metric (its dimension is absent from the target metric's pivot config), the visible rows on the board are still constrained — all metrics in a widget share the same visible rows. The agent must replicate this behavior when fetching outside the platform.

**Detection**: cross-reference each active UVC selector's dimension against the target metric's native dimensions. If absent, the selector reaches this metric only through a shared dimension with a sibling.

**When detected**: do not apply the selector directly to the target metric.

1. Explain the mismatch — the selector doesn't directly apply to this metric, but on the board the data appears filtered because all metrics in a widget share visible rows.
2. Offer two options:
  - **(A) Full dataset** — fetch without the selector's constraint.
  - **(B) Board-matching view** — derive the entity set from the sibling metric where the selector applies, then filter the target metric to that set on the shared dimension.

**Procedure for option B**: the sibling metric is shown by the same widget, so its pivot config is the view configuration you already have — do not call `tool:get_views` again. Confirm the selector's dimension appears in that config → fetch the sibling with the selector applied → extract the entity set on the shared dimension → fetch the target metric filtered to that set only.

**Common pattern — ranking + value metric**: one metric defines which entities appear (e.g. top 10 by NNACV), another provides values (e.g. meetings count). Apply selectors to the ranking metric → use entity set as the ONLY filter on the value metric → do NOT re-apply selectors directly unless the value metric's own pivot config defines a mapping for that selector.

**Time selector trap**: time selectors (Snapshot Week, Quarter, Fiscal Period) are the most dangerous case. They often scope one metric ("pipeline as of this snapshot") but have completely different semantics on another ("only meetings in this one week"). If a time selector's dimension is not in the target metric's pivot config, never apply it directly. Be especially cautious with time-based selectors and flag the semantic difference when presenting options.
