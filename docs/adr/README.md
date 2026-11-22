# Architecture Decision Records

An Architecture Decision Record (ADR) records one significant decision: the context it was made in, what was decided, and why. ADRs explain the reasons behind the repository, so a reader does not have to guess why something is done the way it is.

## Index

This index lists the ADRs that apply across the repository. Each context lists its own ADRs:

- [Host Configuration](../../ansible/docs/adr/README.md) (`ansible/`)

| ADR                                                                 | Title                                               | Status                                                                                 |
| ------------------------------------------------------------------- | --------------------------------------------------- | -------------------------------------------------------------------------------------- |
| [0001](./0001-use-ansible-for-host-configuration.md)                | Use Ansible for host configuration                  | Accepted                                                                               |
| [0002](./0002-manage-workstation-tools-with-mise.md)                | Manage workstation tools with mise                  | Partially superseded by [ADR-0004](./0004-lock-ansible-python-dependencies-with-uv.md) |
| [0003](./0003-keep-adrs-with-their-context.md)                      | Keep ADRs with the context they belong to           | Accepted                                                                               |
| [0004](./0004-lock-ansible-python-dependencies-with-uv.md)          | Lock Ansible's Python dependencies with uv          | Accepted                                                                               |
| [0005](./0005-run-ubuntu-lts-on-every-host.md)                      | Run Ubuntu 26.04 LTS on every Host                  | Accepted                                                                               |
| [0006](./0006-name-hosts-by-placement-environment-and-host-type.md) | Name Hosts by placement, Environment, and Host Type | Accepted                                                                               |

ADRs that were renumbered or moved:

| Former ID | Current ID                                                                       |
| --------- | -------------------------------------------------------------------------------- |
| ADR-0002  | [ADR-ANSIBLE-0001](../../ansible/docs/adr/0001-organise-ansible-by-host-type.md) |
| ADR-0003  | [ADR-0002](./0002-manage-workstation-tools-with-mise.md)                         |

## When to write an ADR

Write an ADR when a decision meets all three of these conditions:

- **Hard to reverse:** changing it later would take significant work.
- **Surprising without context:** a future reader would wonder why it was done this way.
- **A real trade-off:** there were genuine alternatives, and one was chosen for specific reasons.

Smaller decisions belong in commit messages or pull request descriptions.

## Conventions

- **Location:** an ADR that only affects the inside of one context lives in that context's `docs/adr/`, such as `ansible/docs/adr/`. An ADR that spans contexts or chooses a tool lives here, in `docs/adr/`. The test is whether someone working outside the context needs to know about the decision. See [ADR-0003](./0003-keep-adrs-with-their-context.md).
- **File name:** `NNNN-short-title.md`, numbered in sequence within its directory, for example `0002-tool-responsibilities.md`. Each directory with ADRs starts at `0001`.
- **ID:** `ADR-NNNN` for ADRs here, and `ADR-<DIRECTORY>-NNNN` for ADRs in a context, with the directory name in upper case, such as `ADR-ANSIBLE-0001`. The title of the ADR starts with its ID.
- **Numbers are never reused.** Moving an ADR leaves a gap in the sequence it left.
- **One decision per ADR,** so that each can be superseded on its own.
- **Status:** an ADR is `Proposed` while its pull request is open and `Accepted` when it is merged. An accepted ADR is not edited, except when it is moved or superseded. To change the decision, write a new ADR, and mark the old one `Superseded by ADR-NNNN`. When the new ADR replaces only part of the old one, mark the old one `Partially superseded by ADR-NNNN`, and say in the new ADR which part it replaces. A superseded ADR also gets a `Why:` line under its status, of one or two sentences, saying why the decision changed and, for a partial supersede, which sections no longer apply. An ADR partially superseded by more than one ADR lists them all in its status and gets one line per superseding ADR, labeled with its ID, such as `Why (ADR-0004):`. Its body is never changed.
- **Date:** the date the decision was made.
- **Language:** use the terms defined in the glossaries listed in [`GLOSSARY-MAP.md`](../../GLOSSARY-MAP.md).
- **Moving an ADR:** a move may change the ID, the title prefix, and links, and nothing else. Add a `Formerly:` line under the status with the previous ID, and a row to the table of renumbered or moved ADRs above.
- Add each new ADR to the index of its directory in the same change. A new context gets its own `docs/adr/README.md` index, linked from the list above and from [`GLOSSARY-MAP.md`](../../GLOSSARY-MAP.md).

## Template

Leave out any optional section that adds nothing.

In a context, start the title with the context's ID, such as `ADR-ANSIBLE-NNNN`.

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
