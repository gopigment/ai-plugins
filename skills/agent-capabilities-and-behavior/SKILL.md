---
name: agent-capabilities-and-behavior
description: >-
  Agent capability boundaries and behavioral protocols.
  Consult when verifying whether the agent can perform an action,
  when handing off work to a human modeler, or when deciding how to
  approach creation tasks (e.g. reuse vs create new).
  Also use when generating instructions for a human
  to complete what the agent cannot do.
  Do NOT use for read-only exploration or modeling how-to questions.
metadata:
  skill_path: /agent-capabilities-and-behavior/SKILL.md
  base_directory: /agent-capabilities-and-behavior
  includes:
    - "*.md"
---

# Agent Capabilities and Behavior

What the agent can and cannot do, and how it should behave.

---

## What the Agent Cannot Do Yet (UI only)

Check this before proposing any action. If listed here, hand off to the user.

| Not available | Notes |
| --- | --- |
| Duplicate | |
| View formatting | Colors, conditional formatting, number formats |
| Org Chart, Geo Map | |
| Scenarios | |
| Access Rights | See [modeling_access_rights.md](../modeling-pigment-applications/modeling_access_rights.md) |
| Permissions | |
| Sequences | |
| Automations | |
| Variables | |
| Subsets | See [modeling_subsets.md](../modeling-pigment-applications/modeling_subsets.md) |
| History | |
| Performance Insights | |
| Snapshots | |
| Test & Deploy | |

---

## Behaviors

### Reuse Existing Objects

Before creating any block, search the application for existing objects that already serve the same purpose. Never create a block without first checking whether a suitable one already exists.


---

## Cross-References

- **modeling-pigment-applications** — Modeling knowledge for structural decisions
- **writing-pigment-formulas** — Formula work
- **optimizing-pigment-performance** — Performance awareness
