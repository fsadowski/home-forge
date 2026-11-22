---
status: accepted
date: 2026-11-22
---

# 0001. Record architecture decisions

## Context

Decisions in this repository shape code across several contexts: Provisioning, Host Configuration, Containerized Services, and Cluster Workloads. The code shows what was decided but not why, or which alternatives were turned down. The repository is a learning project and a portfolio, so the reasoning matters as much as the result. Without a record, a deliberate choice gets questioned again or quietly "fixed."

## Decision

Significant architecture decisions are recorded as Architecture Decision Records (ADRs), using the rules in [`README.md`](README.md):

- **Placement follows scope.** An ADR that affects only one context lives in that context's `docs/adr/`, for example `ansible/docs/adr/`. An ADR that affects more than one context lives in the root `docs/adr/`. The ADR then sits next to the code it governs, and system-wide decisions stay in one place.
- **Each directory numbers its own ADRs.** ADRs are always referred to by relative link, never by number alone.
- **The format is Nygard's:** Context, Decision, Consequences, and an optional Considered Options section, with `status` and `date` in frontmatter.
- **Merged ADRs are immutable.** Only the status changes. A changed decision is a new ADR that supersedes the old one.
- **ADRs record decisions; convention docs explain how to follow them.** A convention doc such as [`host-naming.md`](../host-naming.md) links to its ADR instead of repeating its reasoning.
- **One index** in [`README.md`](README.md) lists every ADR in the repository, grouped by location.

## Consequences

- A bare "ADR 0002" is ambiguous once a context has its own ADRs, so references must be links.
- Every PR that adds an ADR or changes its status also updates the index. Nothing enforces this yet.
- The ADR is added in the same PR as the code it governs. This ADR is the exception, because it governs only itself and the ADRs that follow.
- [`host-naming.md`](../host-naming.md) was merged before this decision and keeps its own "Why" section.

## Considered Options

- **One global sequence across all `adr/` directories:** every number would be unique, but each directory would have gaps, and choosing the next number would mean scanning the whole repository.
- **All ADRs in the root `docs/adr/`:** simpler, but context-specific decisions would sit far from the code they govern.
- **A minimal template (title and a short paragraph):** quicker to write, but it does not prompt for the context and the consequences.
- **MADR:** structured pros and cons for each option. More ceremony than a single-owner repository needs.
