# AGENTS.md

## Project purpose

`home-forge` is the infrastructure as code for a personal homelab. It serves two purposes, and both shape how work is done here:

- **Learning:** the owner is building real systems to gain hands-on experience with DevOps and platform tooling, and to get better at working with AI. The owner must understand everything that gets merged.
- **Portfolio:** the repository is public-facing and shows the owner's engineering standards. Code, commit history, and docs are all part of what gets read.

Everything is held to production-ready standards, even though it runs at home.

## Repository layout

The directories are listed in the [Structure section of `README.md`](README.md#structure). The repository is at an early stage, and some of them may not exist yet. Create them as needed and keep each kind of work in its directory. Use OpenTofu, not Terraform.

`opentofu/`, `ansible/`, `docker/`, and `kubernetes/` are each a planned context, listed in `GLOSSARY-MAP.md`.

## Working with the owner

- Explain the reasoning behind non-trivial choices, including the alternatives considered and the trade-offs.
- Prefer small, reviewable changes that can be understood one at a time.
- Ask before introducing a new tool, or anything else that adds a long-term maintenance burden.
- Do not paper over something you are unsure about. Say what is unverified, and link to official documentation when a choice depends on it.
- Code comes first. An ADR, a glossary term, or a convention is added in the same pull request as the code that uses it, never ahead of it. Exploratory thinking belongs in the local issue tracker, not in committed docs.
- Record significant architecture decisions as ADRs. The ADR conventions and template are added with the first ADR. Add a runbook for any operational procedure that has to be repeated.

## Engineering standards

- **Declarative and idempotent:** a repeated run with no changes must report no changes.
- **No manual drift:** changes to the homelab are made through this repository, not by hand on a machine. Exceptions, until a decision covers them: the Bootstrap of a Physical Host and the configuration of the Hypervisor.
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

| Tool       | Checks                                                                 |
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
- `docker compose up` or `down` on the Docker Host

Read-only inspection, such as `tofu plan` and `ansible-playbook --check`, is the preferred way to preview a change. Show the owner the output before anything is applied.

## Issue tracker

Specs and tasks are tracked locally, not on GitHub. Each one is a Markdown file in `issues/` at the repository root, named `<number>-<short-description>.md` with a zero-padded number, for example `0001-ansible-skeleton.md`. The directory is gitignored: specs are working notes, and the lasting record is the ADRs, runbooks, and READMEs that the work produces.

Each file starts with YAML frontmatter:

```yaml
---
title: Ansible skeleton
status: ready-for-agent
created: 2026-11-22
---
```

`status` is the triage label, one of:

| Status            | Meaning                                                  |
| ----------------- | -------------------------------------------------------- |
| `needs-triage`    | Not reviewed yet                                         |
| `needs-info`      | Blocked on a question for the owner                      |
| `ready-for-agent` | Fully specified; an agent can implement it unaided       |
| `ready-for-human` | Specified, but the owner implements it                   |
| `wontfix`         | Decided against                                          |
| `done`            | Implemented and merged                                   |

## Git workflow

- Never commit directly to `main`. Work on a short-lived branch named `<type>/<short-description>`, for example `docs/agents-md`.
- Use [Conventional Commits](https://www.conventionalcommits.org/): `<type>(<scope>): <summary>`, for example `docs(readme): add README.md`.
- Keep each commit focused on one logical change.
- Commit, push, or open a pull request only when the owner asks.

## Documentation

Write all code, comments, commit messages, and docs in English, with American spelling. When a change affects the repository's structure or how something is operated, update `README.md`, this file, or `docs/` in the same change.

Domain terms are defined in `GLOSSARY-MAP.md`. Use those terms in code, docs, and conversation, and update the glossary in the same change when a term is added or its meaning shifts.
