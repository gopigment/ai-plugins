---
name: sharing-data-between-applications
description: "Execution skill. Use when sharing metrics or lists between Pigment Applications via Libraries and cross-Application formula references."
metadata:
  base_directory: "/skills/sharing-data-between-applications"
  includes:
    - "SKILL.md"
---

# Decide Whether Cross-Application Sharing Is Needed

Pigment Applications are self-contained. Data does not flow between them unless explicitly shared.

## Use Cross-Application Sharing When

- **Same data needed by 2+ Applications**: single source of truth, no duplication
- **Security boundary required**: teams access only their Application; shared metrics expose only sanitized outputs
- **Different team ownership**: source team owns calculations; consuming teams reference results
- **Hub pattern**: one central Application (FX Hub, Dimension Hub) feeds many consumers

Do **not** share when data is used in only one Application, or when a simple import snapshot is sufficient and live linkage is unnecessary.

## Share a Metric to Another Application

1. **Share the block** using `tool:batch_share_blocks` to make it available to other Applications. If `tool:get_block_sharing_plan_for_batch_share` is available, call it first with the same metrics and lists, review the extra blocks and Libraries it lists, then pass the returned plan back unchanged as `reviewedBlockSharingPlan`; `tool:batch_share_blocks` rejects the call without it.
2. In the consuming Application, **activate the source Library** using `tool:enable_application_library`.
3. Reference the shared block in formulas using cross-Application syntax (see below).

Only share PUSH_ output metrics that other Applications need. Avoid sharing internal calculation intermediaries. Dimension lists shared across apps do not use PUSH_/PULL_; share them directly under their normal names.

## Write Cross-Application Formula References

```pigment
'Source Application Name'::'Block Name'
```

Always use the full form with Application prefix. Sharing a block also shares its underlying dimensions.

Use `tool:list_application_libraries` to check which Libraries are activated. A shared block is not usable until its Library is activated via `tool:enable_application_library`.

Changes to shared blocks in the source Application propagate automatically to all consumers.

## Sharing Dimension Lists

To share dimension lists, two ways:

- When batching, same tool as metrics: `tool:batch_share_blocks`
- Through the list's sharing status:
  - When creating a new dimension: `tool:create_list` with its sharing status set to Shared
  - When sharing an existing dimension: `tool:bulk_update_lists` with the sharing status set to Shared

This applies to all dimension lists including calendar time dimensions (Month, Quarter, Year). Changing a list's sharing status also changes its subsets (sublists).

`tool:batch_share_blocks` shares metrics and lists only, not tables. It also shares the dimensions of the metrics you pass, so you do not need to share those lists first.

## Limitations

- Shared metrics propagate structure changes -- update shared metrics when source dimensions change.
- Deactivating sharing is blocked while other Applications reference the block.
- Sharing a block shares its dimensions -- dimension renames in the source affect all consumers.
- Prefer fewer, well-dimensioned shared metrics over sharing every intermediate calculation.
