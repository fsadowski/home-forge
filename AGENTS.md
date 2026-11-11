# AGENTS.md

## Project purpose

`home-forge` is the infrastructure as code for a personal homelab. It serves two purposes, and both shape how work is done here:

- **Learning:** the owner is building real systems to gain hands-on experience with DevOps and platform tooling, and to get better at working with AI. The owner must understand everything that gets merged.
- **Portfolio:** the repository is public-facing and shows the owner's engineering standards. Code, commit history, and docs are all part of what gets read.

Everything is held to production-ready standards, even though it runs at home.

## Repository layout

| Directory     | Purpose                                        |
| ------------- | ---------------------------------------------- |
| `opentofu/`   | Infrastructure provisioning with OpenTofu      |
| `ansible/`    | Host configuration and automation              |
| `docker/`     | Containerized services and Compose definitions |
| `kubernetes/` | Kubernetes manifests and cluster configuration |
| `docs/`       | Architecture notes, decisions, and runbooks    |

The repository is at an early stage, and some of these directories may not exist yet. Create them as needed and keep each kind of work in its directory. Use OpenTofu, not Terraform.

## Working with the owner

- Explain the reasoning behind non-trivial choices, including the alternatives considered and the trade-offs.
- Prefer small, reviewable changes that can be understood one at a time.
- Ask before introducing a new tool, or anything else that adds a long-term maintenance burden.
- Do not paper over something you are unsure about. Say what is unverified, and link to official documentation when a choice depends on it.
- Record significant architecture decisions in `docs/`, and add a runbook for any operational procedure that has to be repeated.

## Engineering standards

- **Declarative and idempotent:** a repeated run with no changes must report no changes.
- **No manual drift:** changes to the homelab are made through this repository, not by hand on a host.
- **Pinned versions:** pin OpenTofu providers, Ansible collections, container images, and Helm charts to explicit versions. Never use `latest`.
- **Least privilege:** avoid running containers as root, and scope credentials and permissions to what is needed.
- **Readable over clever:** use clear names, and comment only to explain why something is done.

## Security

This repository is meant to be seen by others. Never commit:

- Passwords, tokens, API keys, private keys, or kubeconfigs
- OpenTofu state files or unencrypted variable files containing secrets
- Unencrypted Ansible inventory data or vault passwords
- Personal details about the home network that do not need to be public, such as public IP addresses, personal domains, or MAC addresses

Use placeholders and committed `*.example` files for values that must stay private. If you find a committed secret, stop and tell the owner: it must be rotated, because deleting it from the latest commit does not remove it from history.

## Validation

Run the checks relevant to the files you changed and report the results. If a tool is not installed or a check could not be run, say so.

| Area       | Checks                                                                 |
| ---------- | ---------------------------------------------------------------------- |
| OpenTofu   | `tofu fmt -check -recursive`, `tofu validate`                          |
| Ansible    | `ansible-lint`, `ansible-playbook --syntax-check <playbook>`           |
| Docker     | `docker compose config --quiet`                                        |
| Kubernetes | `kubectl apply --dry-run=client -f <manifest>`, or `kubectl kustomize` |

## Safety

These commands change live systems, so run them only when the owner explicitly asks:

- `tofu apply` or `tofu destroy`
- `ansible-playbook` without `--check`
- `kubectl apply`, `kubectl delete`, or `helm upgrade` against a real cluster
- `docker compose up` or `down` on a homelab host

Read-only inspection, such as `tofu plan` and `ansible-playbook --check`, is the preferred way to preview a change. Show the owner the output before anything is applied.

## Git workflow

- Never commit directly to `main`. Work on a short-lived branch named `<type>/<short-description>`, for example `docs/agents-md`.
- Use [Conventional Commits](https://www.conventionalcommits.org/): `<type>(<scope>): <summary>`, for example `docs(readme): add README.md`.
- Keep each commit focused on one logical change.
- Commit, push, or open a pull request only when the owner asks.

## Documentation

Write all code, comments, commit messages, and docs in English. When a change affects the repository's structure or how something is operated, update `README.md`, this file, or `docs/` in the same change.

Domain terms are defined in `GLOSSARY-MAP.md` and the per-context glossaries it links. Use those terms in code, docs, and conversation, and update the glossary in the same change when a term is added or its meaning shifts.
