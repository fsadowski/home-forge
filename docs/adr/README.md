# Architecture Decision Records

How decisions are recorded in this repository, and an index of every ADR. The reasoning behind these rules is in [0001. Record architecture decisions](0001-record-architecture-decisions.md).

## When to write an ADR

Write one when a decision is hard to reverse, would surprise a future reader, and came from a real trade-off between alternatives. Add the ADR in the same pull request as the code it governs.

An ADR records that a decision was made and why. A convention doc explains how to follow a decision and links to its ADR instead of repeating its reasoning.

## Placement

- An ADR that affects only one context lives in that context's `docs/adr/`, for example `ansible/docs/adr/`. The contexts are listed in [`GLOSSARY-MAP.md`](../../GLOSSARY-MAP.md).
- An ADR that affects more than one context lives here in `docs/adr/`. If in doubt, put it here.
- Create a context's `docs/adr/` with its first ADR. It does not get its own README: this file covers every ADR directory.

## Naming and numbering

- Name each file `NNNN-kebab-case-slug.md`. `NNNN` is a four-digit number with leading zeros, and the slug names the decision, for example `0002-use-k3s-for-the-cluster.md`, not `0002-kubernetes-distribution.md`.
- Each directory numbers its own ADRs. To choose a number, take the highest one in that directory and add one.
- Numbers repeat across directories, so always refer to an ADR by relative link, never by number alone.

## Writing an ADR

Copy [`template.md`](template.md). The frontmatter holds:

- `status`: `accepted`, `deprecated`, or `superseded by [NNNN](link)`. There is no `proposed` status: the pull request is the proposal.
- `date`: the day the ADR is accepted, which is the day it is merged.

The Considered Options section is optional. Include it only when the rejected alternatives are worth remembering.

## Changing a decision

A merged ADR is never rewritten. Only its `status` changes. To change a decision, write a new ADR that starts with `Supersedes [NNNN](link).`, and set the old ADR's status to `superseded by` a link to the new one.

## Index

Update this index in the same change as any new ADR or status change.

### System-wide

| ADR                                           | Title                         | Status   |
| --------------------------------------------- | ----------------------------- | -------- |
| [0001](0001-record-architecture-decisions.md) | Record architecture decisions | accepted |
