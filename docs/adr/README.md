# Architecture Decision Records

An Architecture Decision Record (ADR) records one significant decision: the context it was made in, what was decided, and why. ADRs explain the reasons behind the repository, so a reader does not have to guess why something is done the way it is.

## Index

| ADR                                                  | Title                              | Status   |
| ---------------------------------------------------- | ---------------------------------- | -------- |
| [0001](./0001-use-ansible-for-host-configuration.md) | Use Ansible for host configuration | Accepted |
| [0002](./0002-organise-ansible-by-host-type.md)      | Organise Ansible by Host Type      | Accepted |
| [0003](./0003-manage-workstation-tools-with-mise.md) | Manage workstation tools with mise | Accepted |

## When to write an ADR

Write an ADR when a decision meets all three of these conditions:

- **Hard to reverse:** changing it later would take significant work.
- **Surprising without context:** a future reader would wonder why it was done this way.
- **A real trade-off:** there were genuine alternatives, and one was chosen for specific reasons.

Smaller decisions belong in commit messages or pull request descriptions.

## Conventions

- **File name:** `NNNN-short-title.md`, numbered in sequence, for example `0002-tool-responsibilities.md`.
- **One decision per ADR,** so that each can be superseded on its own.
- **Status:** an ADR is `Proposed` while its pull request is open and `Accepted` when it is merged. An accepted ADR is not edited. To change the decision, write a new ADR, and mark the old one `Superseded by ADR-NNNN`.
- **Date:** the date the decision was made.
- **Language:** use the terms defined in the glossaries listed in [`GLOSSARY-MAP.md`](../../GLOSSARY-MAP.md).
- Add each new ADR to the index above in the same change.

## Template

Leave out any optional section that adds nothing.

```markdown
# ADR-NNNN: <Decision, stated as an action>

- **Status:** Proposed
- **Date:** YYYY-MM-DD

## Context

<The situation and constraints that call for a decision.>

## Decision

<What was decided, stated so that it can be checked against the repository.>

## Alternatives considered

| Option   | Why it was not chosen |
| -------- | --------------------- |
| <Option> | <Reason>              |

## Consequences

**Benefits**

- <What becomes easier.>

**Costs and risks**

- <What becomes harder, and how it is mitigated.>

## Out of scope (optional)

- <Related questions this ADR leaves for later decisions.>

## References (optional)

- [<Official documentation>](URL)
```
