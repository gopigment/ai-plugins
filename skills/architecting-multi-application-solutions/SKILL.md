---
name: architecting-multi-application-solutions
description: "Planning skill. Use when deciding how to split work across Pigment applications, designing Hub-and-spoke data flows, or planning Test & Deploy across environments."
metadata:
  skill_path: /skills/architecting-multi-application-solutions/SKILL.md
  base_directory: /skills/architecting-multi-application-solutions
---

# Architecting Multi-Application Solutions

## Decide When to Split into Multiple Applications

Split when one or more drivers apply:

- **Team ownership**: One team owns and administers a domain exclusively (Sales Ops, HR, Finance).
- **Security boundaries**: Sensitive data (payroll, individual salaries) must be isolated with minimal user exposure.
- **Release cycles**: Domains need independent Test & Deploy cadences or structural change schedules.
- **Size and complexity**: Dozens of blocks serve one function but only one output metric is shared downstream.
- **Dimension divergence**: A business unit needs different time granularity, country lists, or version cadence.
- **Planning cadence**: Revenue plans daily with ten scenarios; OPEX plans monthly with one scenario.

Multiple applications share data in real time via Libraries — no import/export latency.

## Place Shared Reference Data in a Hub Application

The Hub is a central application holding shared reference data.

**Place in the Hub:**

- Common dimensions: Region, Cost Center, Product, Chart of Accounts, Time, Version
- Shared external data: accounting actuals, CRM opportunities, FX rates, global parameters
- Master access-rights dimensions and mapping metrics

**Rule of thumb:** If two or more applications need a dimension or dataset, put it in the Hub — unless segregation of duty requires keeping it in a domain app (e.g., employee master in HR).

**Keep out of the Hub:**

- Domain-specific calculation chains with many intermediate metrics
- Sensitive employee-level data (share from HR app instead)

Keep the Version dimension in the Hub and share it across all planning apps for synchronized planning cycles.

## User Navigation Constraint

Users cannot natively navigate between applications — manual links on boards are required. Architect each application to provide a complete journey for a specific user type, minimizing cross-application navigation.

## Dimension Placement Exception

When dimension items are created by end users or business logic within a specific application, that dimension stays local to that application and is shared outward (not placed in the Hub).

## Share Blocks between Applications

For sharing mechanics and cross-application formulas, see `skill:sharing-data-between-applications`.
