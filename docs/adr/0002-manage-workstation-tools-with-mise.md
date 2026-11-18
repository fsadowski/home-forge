# ADR-0002: Manage workstation tools with mise

- **Status:** Partially superseded by [ADR-0004](./0004-lock-ansible-python-dependencies-with-uv.md)
- **Why:** Pinning only the direct Python dependencies left transitive ones unlocked and unchecked, so uv now creates `ansible/.venv` from a hashed `uv.lock`. In "Python and Ansible", only the rule that mise pins Python and activates the virtual environment still applies; the `requirements.txt` cost and the out-of-scope item on locking transitive dependencies no longer do.
- **Formerly:** ADR-0003
- **Date:** 2026-11-15

## Context

`AGENTS.md` requires pinned versions, and [ADR-ANSIBLE-0001](../../ansible/docs/adr/0001-organise-ansible-by-host-type.md) pins `ansible-core` and `ansible-lint` in `requirements.txt`. The tools that run those pins are not pinned yet. The Python interpreter behind the Ansible virtual environment is whatever the operating system ships, and it changes with an OS upgrade. Each new area of the repository adds CLIs of its own: OpenTofu for `opentofu/`, and `kubectl` and Helm for `kubernetes/`. Installed by hand, their versions drift between workstations and away from what the repository was tested with.

The homelab is run by one person from one Linux x86-64 workstation. A rebuilt workstation should get the same tools from the repository alone.

## Decision

Use [mise](https://mise.jdx.dev/) to install and pin the CLIs the repository runs on a workstation.

**Configuration**

- There is one `mise.toml`, at the repository root.
- It lists only tools for directories that exist. Adding an area to the repository adds that area's tools to `mise.toml` in the same change.
- Every version is exact, such as `1.2.3`. Prefixes such as `1.2` and `latest` are not used.
- `min_version` is a hard minimum set to the mise release the configuration was tested with.

**Backends**

- Each tool uses the backend that the mise registry maps it to, preferring aqua.
- asdf plugins are not used.

**Lockfile**

- `mise.lock` is committed. It records the resolved version and the checksum of each download.
- The lockfile covers only `linux-x64`, the platform in use. Other platforms are added when they are used.

**Python and Ansible**

- mise pins Python and creates the Ansible virtual environment in `ansible/.venv`.
- The virtual environment is created with Python's own `venv` module, never with uv, even when uv is installed.
- `ansible/requirements.txt` stays the only pin for `ansible-core` and `ansible-lint`, as ADR-ANSIBLE-0001 decides. Installing it into the virtual environment is a documented setup step.

This ADR does not list tool versions. They live only in `mise.toml` and `requirements.txt`, so that a version bump does not change this ADR.

## Alternatives considered

| Option                                           | Why it was not chosen                                                                                                                                                 |
| ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Installing tools by hand                         | Versions drift between workstations and from what was tested, and nothing in the repository records what is needed.                                                   |
| Distribution packages                            | The versions follow the distribution's release cycle, not the repository's, and several tools are not packaged or lag well behind.                                    |
| asdf                                             | Each tool needs a third-party plugin, which is a shell script that runs on the workstation. mise reads the same version files and verifies downloads without plugins. |
| Nix or devbox                                    | Fully reproducible, but the learning cost and maintenance burden are large for a handful of CLIs.                                                                     |
| A virtual environment on the system Python       | Pins the Ansible packages but not the interpreter, which changes with an OS upgrade. It does nothing for OpenTofu, `kubectl`, or Helm.                                |
| Installing Ansible with the mise `pipx:` backend | Each pipx tool gets its own environment, so `ansible-lint` would bring its own copy of `ansible-core`, which can differ from the pinned one that runs the playbooks.  |
| Locking every platform mise supports             | Checksums for platforms nobody uses are never verified.                                                                                                               |

## Consequences

**Benefits**

- A rebuilt workstation gets the tested tool versions with `mise install`, and the lockfile checks every download against a recorded checksum.
- The Python interpreter for Ansible is pinned, so an OS upgrade does not change it.
- The tool versions are in one file that a reviewer can read, and a bump shows up as a diff.

**Costs and risks**

- mise is a new tool to keep up to date. It is the one tool that the repository cannot install, so the README documents a pinned install command.
- `mise.toml` sets environment variables, so each workstation has to run `mise trust` once before mise applies it.
- mise has to be activated in the shell, or the commands run through `mise exec`. Without it, the system versions run instead.
- Packages are not installed into the virtual environment automatically. After a change to `requirements.txt`, the install step has to be run again.
- The lockfile covers only the tools mise installs. `requirements.txt` pins only `ansible-core` and `ansible-lint`, so their transitive dependencies can resolve to different versions on a fresh install, and pip does not check the packages against recorded checksums. Revisit this with a Python lockfile, such as `uv.lock` or `pip-compile --generate-hashes`, when an unpinned dependency breaks a run or when CI needs installs that are identical byte for byte.
- Tools come from the backends the mise registry chooses. A registry change can move a tool to another backend on a mise upgrade.

## Out of scope

- mise tasks, such as a task that runs the validation checks
- Locking transitive Python dependencies
- Installing the tools in CI

## References

- [Installing mise](https://mise.jdx.dev/installing-mise.html)
- [mise configuration](https://mise.jdx.dev/configuration.html)
- [mise lockfile](https://mise.jdx.dev/dev-tools/mise-lock.html)
- [mise Python and virtual environments](https://mise.jdx.dev/lang/python.html)
- [mise trust](https://mise.jdx.dev/cli/trust.html)
