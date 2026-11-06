# Off-board queries

Either no board is open, or the view is not on the board the user has open, so nothing has chosen a scope for you. The danger is not failing to find the data — it is fetching it under a scope nobody asked for and reporting the number as if it were the right one. A metric fetched unfiltered can be an order of magnitude away from what the same metric shows on its board.

Board selectors do **not** carry over from a board the user visited earlier, nor from the board the user has open when the view is not on it. Treat the query as having no board scope from them.

## Picking the view

**If the question names a board**, start from it: call `tool:search_boards`, then call `tool:get_board` on **every** board it returns, in parallel. Once you have read all of them, pick the single widget whose title best answers the question and use its `viewId`. Do not settle on the first board that looks plausible: a better-matching widget often sits on a board ranked lower. The `get_board` result of the board you picked holds its page selectors, which are the scope for step 2 of *Choosing the scope*. Only take the block route that follows when no board is named, or none matches.

1. Find the block, then call `tool:search_views` with its id in `block_ids`.
2. Pass `min_referring_boards_count: 1` so only views that a board actually shows come back — a view no board displays is rarely the one the user means. If no view comes back, search again without it.
3. Prefer a view whose `referringVisibleBoardNames` is non-empty, and note those board names: they tell you which board the user most likely has in mind.
4. Call `tool:get_views` on the chosen view for the full pivot configuration. `tool:search_views` returns dimension names only — Mapped and Dimension pivots look the same — so use it only to pick the view.

If the user names something that reads like a view or widget title rather than a block ("the benchmark analyst view"), search it directly with `friendly_name_regex_search`, escaping any regex characters in their wording.

## Choosing the scope

Apply the first of these that holds, and never move on to the next one silently:

1. **The user stated a scope** — use it. Their wording always wins.
2. **A board shows this view and you can read it** — unless you already read it while picking the view, find its id with `tool:search_boards` on the board name and call `tool:get_board` on it. Use its page selectors' `defaultModalityReferences` as the scope. A selection carries the item's `friendlyName` when it resolves; use that name when you report the figures.
3. **Neither** — fetch with no page filters and say plainly that no filter was applied.

Never guess a period, a version or a scenario because it seems reasonable. A missing filter that you disclose is recoverable; an invented one is not.

## Saying which scope you used

Every off-board answer states the scope it was fetched under, even when that scope is "none". This is not the optional transparency line from SKILL.md — off-board it is required, because the user has no screen to check it against.

- Name the values in business language: *"FY 24, EMEA"*, not item ids.
- A selection with no label — a variable selection, or an item that no longer resolves — cannot be named, so it was not applied. Say the scope is set but unreadable and was not applied, rather than inventing a label for it.
- When you fetched without any scope, say so in one line: *"Fetched with no filters — this is the full dataset, not necessarily what a board shows."*
- If the user's question implies a board they did not name, offer the board you used and let them correct it.

## After a board fallthrough

If the view was found by searching beyond the board the user has open — the block was not found on that board and the search expanded to the application — and that board had active page selectors, always say so, whether or not any filter was applied: e.g. *"Note: this board's active filters (Region = EMEA) were not applied."* Then state the scope you used, as above.
