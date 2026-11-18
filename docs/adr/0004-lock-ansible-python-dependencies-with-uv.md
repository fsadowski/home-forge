# ADR-0004: Lock Ansible's Python dependencies with uv

- **Status:** Accepted
- **Date:** 2026-11-18

## Context

[ADR-0002](./0002-manage-workstation-tools-with-mise.md) pins the Python interpreter with mise and creates the Ansible virtual environment in `ansible/.venv` with Python's own `venv` module, never with uv. [ADR-ANSIBLE-0001](../../ansible/docs/adr/0001-organise-ansible-by-host-type.md) pins `ansible-core` and `ansible-lint` with `==` in `ansible/requirements.txt`, and rejects uv because it adds a tool.

Both ADRs record the gap this leaves. Only the two direct dependencies are pinned, so their transitive dependencies can resolve to different versions on a fresh install, and pip does not check any package against a recorded checksum. ADR-0002 names `uv.lock` as the way to close the gap. The install step is also manual: after a change to `requirements.txt`, `pip install` has to be run again, and nothing reports a virtual environment that no longer matches the pins.

## Decision

Manage the Python dependencies of the Host Configuration context as a uv project in `ansible/`, with a committed lockfile.

**Project**

- `ansible/pyproject.toml` replaces `ansible/requirements.txt`. It is not a package (`package = false`).
- `ansible-core` and `ansible-lint` stay in one list of dependencies, pinned with `==`, so that `ansible-lint` checks against the same `ansible-core` that runs the playbooks.
- `ansible/uv.lock` is committed. It records the exact version and hashes of every package, including transitive dependencies.
- The lock is resolved only for Linux on x86-64, the platform in use, matching `mise.lock`. Other platforms are added when they are used.
- The virtual environment is installed with `uv sync --locked`, which fails if `pyproject.toml` and `uv.lock` disagree, instead of re-locking.

**Tools**

- mise pins and installs uv, like every other CLI.
- mise still pins and installs Python. uv is not allowed to download Python and uses only an interpreter it finds on the `PATH`, which is mise's, so it fails rather than fall back to a different one.
- uv creates `ansible/.venv`. mise no longer creates it, but still activates it for the whole repository.
- The uv version is pinned only in `mise.toml`, not again in `pyproject.toml`.

This ADR partially supersedes ADR-0002, where it creates the virtual environment with `venv` and keeps uv out, and ADR-ANSIBLE-0001, where it pins the Python packages in `requirements.txt`. The rest of both ADRs still applies.

## Alternatives considered

| Option                                       | Why it was not chosen                                                                                                                                            |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Keep `requirements.txt` and pip              | Leaves transitive dependencies unlocked and downloads unchecked, the gap this ADR closes.                                                                        |
| uv as an installer only (`uv pip install`)   | Faster, but without a lockfile it changes nothing about reproducibility.                                                                                         |
| `uv pip compile --generate-hashes`           | Produces a hashed lock in pip's format, but keeps two files, `requirements.in` and the compiled lock, that have to be kept in step by hand.                      |
| `pip-compile` from pip-tools                 | The same result as above, with another Python tool to pin, and no equivalent of `uv sync --locked` to catch a stale lock.                                        |
| Let uv manage Python                         | One tool fewer in the chain, but the interpreter would leave `mise.lock`, and mise would no longer be the single place tool versions are pinned and checksummed. |
| Activate the environment with `uv_venv_auto` | mise finds the project by walking up to the nearest `uv.lock`, so it activates only inside `ansible/`. Commands run from the repository root would lose Ansible. |
| Lock every platform uv supports              | Hashes for platforms nobody uses are never verified, the reason ADR-0002 locks only `linux-x64`.                                                                 |

## Consequences

**Benefits**

- A fresh install reproduces every Python package exactly, and each download is checked against a hash in `uv.lock`.
- `uv sync --locked` brings the virtual environment in line with the lock in one command, and refuses a lock that is out of date.
- A version bump is a one-line change in `pyproject.toml` followed by `uv lock`, and the full effect shows up as a diff of `uv.lock`.

**Costs and risks**

- uv is one more tool to keep up to date. mise pins and installs it, so it adds no manual install step.
- uv and mise both deal with Python, and their defaults overlap. Two environment variables in `mise.toml` keep uv off its own interpreters. Without mise activated, uv cannot find the pinned Python and fails.
- The install step is still manual. After a change to `pyproject.toml` or `uv.lock`, `uv sync` has to be run again.
- `uv.lock` is generated and long. Reviews read the change to `pyproject.toml` and treat the lock diff as its result.

## Out of scope

- Running `uv sync` automatically, for example from a mise hook or task
- Installing the dependencies in CI

## References

- [uv: projects](https://docs.astral.sh/uv/concepts/projects/)
- [uv: locking and syncing](https://docs.astral.sh/uv/concepts/projects/sync/)
- [uv: resolution environments](https://docs.astral.sh/uv/concepts/resolution/#limited-resolution-environments)
- [uv: environment variables](https://docs.astral.sh/uv/reference/environment/)
- [mise: Python and virtual environments](https://mise.jdx.dev/lang/python.html)
