# home-forge

Infrastructure as code for my homelab, doubling as a portfolio of my DevOps/platform engineering work.

Built with the help of AI and held to production-ready standards, following industry best practices.

## Goals

- Learn continuously by building and running real systems
- Grow and improve the homelab
- Develop my skills in working with AI
- Gain hands-on experience with a wide range of tools

## Structure

| Directory     | Purpose                                                              |
| ------------- | -------------------------------------------------------------------- |
| `opentofu/`   | Infrastructure provisioning with OpenTofu                            |
| `ansible/`    | Host configuration and automation                                    |
| `docker/`     | Containerized services and Compose definitions                       |
| `kubernetes/` | Kubernetes manifests and cluster configuration                       |
| `docs/`       | Docs that span contexts: architecture notes, decisions, and runbooks |

## Workstation setup

The tools this repository runs are pinned in [`mise.toml`](mise.toml) and installed with [mise](https://mise.jdx.dev/), as decided in [ADR-0002](docs/adr/0002-manage-workstation-tools-with-mise.md).

1. Install mise at the pinned version. The install script verifies the download's checksum before installing:

   ```sh
   curl -fsSL https://mise.run | MISE_VERSION=v2026.11.5 sh
   ```

2. [Activate mise in your shell](https://mise.jdx.dev/getting-started.html#activate-mise), so the pinned tools and the Ansible virtual environment are used inside the repository.

3. From the repository root, trust the configuration and install the tools:

   ```sh
   mise trust
   mise install --locked
   ```

   `--locked` installs only what `mise.lock` records. It fails if a global mise configuration adds tools that are not in this lockfile, so remove those tools from the global configuration, or leave it out of this run with `MISE_IGNORED_CONFIG_PATHS=~/.config/mise mise install --locked`.

4. Create the virtual environment in `ansible/.venv` and install the Python packages that `ansible/uv.lock` records, as decided in [ADR-0004](docs/adr/0004-lock-ansible-python-dependencies-with-uv.md). Run this again whenever `ansible/pyproject.toml` or `ansible/uv.lock` changes:

   ```sh
   uv sync --locked --project ansible
   ```

   `--locked` fails if the lockfile is out of date instead of updating it. After changing a version in `pyproject.toml`, run `uv lock --project ansible` and commit both files.

## Documentation

- [Architecture Decision Records](docs/adr/README.md): significant decisions and the reasons behind them. Decisions internal to one context are kept in that context's `docs/adr/`, linked from this index.
- [Open questions](docs/open-questions.md): decisions that have not been made yet
- [Glossary map](GLOSSARY-MAP.md): the domain terms used across the repository, grouped by context
- [AGENTS.md](AGENTS.md): working agreements and engineering standards for contributors, human or AI
