# ADR-0003: Keep ADRs with the context they belong to

- **Status:** Accepted
- **Date:** 2026-11-16

## Context

The repository is split into contexts, which [`GLOSSARY-MAP.md`](../../GLOSSARY-MAP.md) lists. Each context lives in its own directory, such as Host Configuration in `ansible/`, and each has its own glossary. Until now every ADR lived in `docs/adr/` with one number sequence, whether it applied to the whole repository or only to the inside of one context.

That mix grows with every context. The layout of `ansible/` matters only to someone working in `ansible/`, but it sat in the same index as the choice of mise, which applies everywhere. Once OpenTofu, Docker, and Kubernetes add their own decisions, the root index will be mostly decisions that a given reader does not need, and the decisions for a context will not be found next to its code.

When this decision was made, there were three ADRs, and only one of them was internal to a context: the layout of `ansible/`, then ADR-0002.

## Decision

Keep each ADR in the context it belongs to, and give each context its own number sequence.

**Location**

- An ADR that only affects the inside of one context lives in `<context directory>/docs/adr/`, such as `ansible/docs/adr/`.
- An ADR that spans contexts or chooses a tool lives in `docs/adr/`. The test is whether someone working outside the context needs to know about the decision.

**Identifiers**

- Each directory with ADRs has its own sequence, starting at `0001`. File names follow the same `NNNN-short-title.md` pattern everywhere.
- ADRs in `docs/adr/` are identified as `ADR-NNNN`. ADRs in a context are identified as `ADR-<DIRECTORY>-NNNN`, with the directory name in upper case, such as `ADR-ANSIBLE-0001`.
- A number is never reused. A move leaves a gap in the sequence it left.

**Moving an ADR**

- An ADR may be moved to another directory, which gives it a new ID. A move may change the ID, the title prefix, and links, and nothing else. This holds for accepted ADRs too.
- A moved ADR gets a `Formerly:` line under its status with its previous ID, and the root index records the move in its table of renumbered or moved ADRs.

**Indexes**

- `docs/adr/README.md` holds the conventions and the template, lists the ADRs in `docs/adr/`, and links to the index of each context.
- Each context has a `docs/adr/README.md` that lists its own ADRs and points to the root for the conventions.
- `GLOSSARY-MAP.md` links each context to its ADRs.

**Migration**

- ADR-0002, the layout of `ansible/`, moves to `ansible/docs/adr/` as ADR-ANSIBLE-0001.
- ADR-0003, the choice of mise, is renumbered to ADR-0002 to close the gap in the root sequence. This is a one-off, done while only two merged pull requests refer to the old numbers. From now on, the rule against reusing numbers applies.

## Alternatives considered

| Option                                         | Why it was not chosen                                                                                                                                                                       |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Keep every ADR in `docs/adr/`                  | One place to look, but the root index fills up with decisions internal to a context, and a context's decisions are not next to its code.                                                    |
| Context folders with one global sequence       | IDs stay plain numbers, but choosing the next number means checking every index, and a context's ADRs get scattered numbers.                                                                |
| Prefixes named after the domain context        | `ADR-HC-0001` matches the context names in `GLOSSARY-MAP.md`, but readers have to learn the abbreviations. The directory name is where the file is found.                                   |
| A prefix on every ADR, root included           | Consistent, but it renumbers the root ADRs for no gain.                                                                                                                                     |
| Leave the gap that the move leaves in the root | Every old reference would still point to the decision it meant. Closing the gap once, while few references exist, keeps the root sequence contiguous. The `Formerly:` lines cover the rest. |
| Close every gap on every move                  | Each move would renumber every later ADR and its references, so moves get more expensive and old references less reliable over time.                                                        |
| One index that lists every ADR                 | Easier to scan, but the root changes whenever a context adds an ADR, and a context is no longer self-contained.                                                                             |

## Consequences

**Benefits**

- Each context keeps its glossary and its decisions in its own directory.
- The root index shows only the decisions that apply across the repository.
- An ID says where to find the ADR, and IDs stay unambiguous as contexts are added.

**Costs and risks**

- References to ADR-0002 and ADR-0003 in pull requests #7 and #8 and in commit history now point to different decisions. The `Formerly:` lines and the table of renumbered or moved ADRs map the old IDs to the new ones.
- Whether a decision spans contexts is a judgment call. A wrong call is fixed by moving the ADR, which the rules above allow.
- Links between a context ADR and a root ADR are longer relative paths, and break if either file moves. A move has to update them.
- A context ID depends on its directory name, so renaming a context directory means moving its ADRs.
- A new ADR has to go into the right index, and a new context needs an index of its own and a link from the root index and `GLOSSARY-MAP.md`.

## Out of scope

- Where other context-specific docs, such as runbooks, live. `AGENTS.md` covers this as a working guideline.
- Checking links automatically, for example in CI
