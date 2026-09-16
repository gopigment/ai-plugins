---
name: sharing-data-between-applications
description: "Execution skill. Use when sharing metrics or lists between Pigment Applications via Libraries and cross-Application formula references."
metadata:
  skill_path: /skills/sharing-data-between-applications/SKILL.md
  base_directory: /skills/sharing-data-between-applications
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

1. **Share the block** using `tool:batch_share_blocks` to make it available to other Applications.
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

Dimension lists are shared using the `sharing_status` parameter, not `tool:batch_share_blocks`:

- When creating a new dimension: `tool:create_list` with `sharing_status: "Shared"`
- When sharing an existing dimension: `tool:update_list` with `sharing_status: "Shared"`

This applies to all dimension lists including calendar time dimensions (Month, Quarter, Year).

`tool:batch_share_blocks` is for sharing metrics, tables, and other non-dimension blocks.

## Limitations

- Shared metrics propagate structure changes -- update shared metrics when source dimensions change.
- Deactivating sharing is blocked while other Applications reference the block.
- Sharing a block shares its dimensions -- dimension renames in the source affect all consumers.
- Prefer fewer, well-dimensioned shared metrics over sharing every intermediate calculation.
