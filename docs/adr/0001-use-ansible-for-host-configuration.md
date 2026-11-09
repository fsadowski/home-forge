# ADR-0001: Use Ansible for host configuration

- **Status:** Proposed
- **Date:** 2026-11-09

## Context

Hosts in the homelab need repeatable configuration: packages, services, and system settings. Changes must be made through this repository rather than by hand on a host, and a repeated run with no changes must report no changes.

The repository is public and is also a learning project, so the tool has to be readable to someone reviewing the code, useful to know in industry, and light enough for one person to operate. Provisioning is handled by OpenTofu and is not part of this decision.

## Decision

Use Ansible for host configuration, with all playbooks, roles, and inventory examples kept in `ansible/`.

Responsibilities are divided as follows:

| Tool       | Owns                                                                |
| ---------- | ------------------------------------------------------------------- |
| OpenTofu   | Creating and destroying infrastructure                              |
| Ansible    | Configuring the operating system of a host that already exists      |
| Docker     | Defining containerized services (`docker/`)                         |
| Kubernetes | Defining cluster workloads (`kubernetes/`)                          |

The following rules apply to all Ansible work in this repository:

- Pin `ansible-core` and every collection to explicit versions. Collections are listed in a `requirements.yml` file.
- Use `--check` to preview a change before it is applied.
- Prefer purpose-built modules to `command` and `shell`. When those two are unavoidable, set `creates`, `removes`, or `changed_when` so that the task stays idempotent.
- Never commit secrets, or inventory data that exposes private network details. Use encrypted files or committed `*.example` files instead.
- Define how Ansible code is validated in `ansible/AGENTS.md`, which is added together with the first Ansible code.

## Alternatives considered

| Option                 | Why it was not chosen                                                                                                                                               |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Shell scripts          | No dependencies, but idempotency, error handling, and previews all have to be written by hand.                                                                      |
| cloud-init             | Well suited to first-boot setup, but it does not manage the ongoing state of a running host. It may still complement Ansible for bootstrapping.                     |
| Puppet or Chef         | Mature and built for continuous enforcement, but the usual agent and server deployment is more than a homelab needs. Both also have their own language to maintain. |
| Salt                   | Capable and fast at scale, but its usual master and minion model adds components that bring little benefit with a small number of hosts.                            |
| NixOS                  | The most declarative option, with reproducible builds and rollbacks, but it requires every host to run NixOS and has a steep learning curve.                        |

## Consequences

**Benefits**

- Ansible is agentless. Managed hosts need SSH and Python, so there is no agent or control server to run and patch.
- Playbooks are YAML, which keeps them approachable for reviewers.
- The large collection ecosystem covers most host configuration tasks.
- `--check` fits the repository's preference for previewing a change before applying it.

**Costs and risks**

- Ansible is convergent rather than fully declarative. Removing a task does not undo what it did, so anything being retired needs an explicit `state: absent` step. Without that, hosts can drift away from the code.
- Idempotency depends on how tasks are written, and `ansible-lint` only catches part of the problem.
- `--check` is not a guarantee. Tasks that depend on the result of an earlier change can report differently, or fail, in check mode.
- Nothing enforces state between runs, because there is no agent. Manual changes stay in place until the next run.
- Ansible's push model and SSH execution are slower than agent-based tools at large scale. This is acceptable at homelab size.

## Out of scope

These topics need their own decisions when the work begins:

- The layout of the `ansible/` directory and the inventory
- How secrets are stored and how the vault password is handled
- How playbooks are run, for example manually or from CI

## References

- [Ansible documentation](https://docs.ansible.com/)
- [Validating tasks: check mode and diff mode](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_checkmode.html)
- [Installing roles and collections from a requirements file](https://docs.ansible.com/projects/ansible/latest/galaxy/user_guide.html)
- [Protecting sensitive data with Ansible Vault](https://docs.ansible.com/projects/ansible/latest/vault_guide/index.html)
