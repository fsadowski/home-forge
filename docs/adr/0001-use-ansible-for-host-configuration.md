# ADR-0001: Use Ansible for host configuration

- **Status:** Accepted
- **Date:** 2026-11-09

## Context

The homelab has a small and growing number of [Hosts](../../ansible/GLOSSARY.md) of two kinds: physical Hosts, and VM Hosts that run on a Hypervisor. Today these are a physical Docker Host that runs the Base Services, and VM Hosts that are Cluster Nodes of the k3s cluster. After its Bootstrap, every Host needs repeatable configuration: Hardening, packages, services, and system settings. Changes must be made through this repository rather than by hand on a Host, and a repeated run with no changes must report no changes.

The repository is public and is also a learning project, so the tool has to be readable to someone reviewing the code, useful to know in industry, and light enough for one person to operate. Provisioning is handled by OpenTofu and is not part of this decision.

## Decision

Use Ansible to configure every Host from the end of its Bootstrap onward, with all playbooks, roles, and inventory examples kept in `ansible/`.

How a Host reaches Ansible depends on whether it is physical or a VM, not on its role:

- **VM Hosts:** Provisioning creates the VM, and cloud-init performs the Bootstrap at first boot. Ansible takes over once the Host is reachable.
- **Physical Hosts:** the Bootstrap is done by hand, because Provisioning does not create physical machines. Ansible takes over once the Host is reachable.
- **Hypervisors:** not Hosts. Ansible does not configure them.

A new Host follows the same pattern as the existing Hosts of its kind.

## Alternatives considered

| Option          | Why it was not chosen                                                                                                                                                                                                                                                                                                                                     |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Shell scripts   | No dependencies, but idempotency, error handling, and previews all have to be written by hand.                                                                                                                                                                                                                                                            |
| cloud-init only | Well suited to the Bootstrap, and used for it on VM Hosts, but it runs at first boot and does not manage the ongoing state of a running Host.                                                                                                                                                                                                             |
| Rebuild Hosts   | Replace a VM Host with a fresh VM whenever its configuration changes, or run an immutable OS such as Talos Linux on the Cluster Nodes. Not chosen because physical Hosts cannot be rebuilt cheaply, so a configuration tool would still be needed for them. One tool for both kinds of Host keeps a single workflow. Talos would also mean replacing k3s. |
| Puppet or Chef  | Mature and built for continuous enforcement, but the usual agent and server deployment is more than a homelab needs. Both also have their own language to maintain.                                                                                                                                                                                       |
| Salt            | Capable and fast at scale, but its usual master and minion model adds components that bring little benefit with a small number of Hosts.                                                                                                                                                                                                                  |
| NixOS           | The most declarative option, with reproducible builds and rollbacks, but it requires every Host to run NixOS and has a steep learning curve.                                                                                                                                                                                                              |

## Consequences

**Benefits**

- Ansible is agentless. A Host needs only SSH and Python, which the Bootstrap provides, so there is no agent or control server to run and patch.
- Playbooks are YAML, which keeps them approachable for reviewers.
- The large collection ecosystem covers most host configuration tasks.
- `--check` fits the repository's preference for previewing a change before applying it.

**Costs and risks**

- Ansible is convergent rather than fully declarative. Removing a task does not undo what it did, so Hosts can drift away from the code. Anything being retired needs an explicit `state: absent` step before its task is removed.
- Idempotency depends on how tasks are written, and `ansible-lint` only catches part of the problem. Purpose-built modules are preferred to `command` and `shell`; when those two are unavoidable, `creates`, `removes`, or `changed_when` keeps the task idempotent.
- Module behavior can change between `ansible-core` releases, so `ansible-core` is pinned to an explicit version along with the collections.
- `--check` is not a guarantee. Tasks that depend on the result of an earlier change can report differently, or fail, in check mode.
- Nothing enforces state between runs, because there is no agent. Manual changes stay in place until the next run.
- Ansible's push model and SSH execution are slower than agent-based tools at large scale. This is acceptable at homelab size.

## Out of scope

These topics need their own decisions when the work begins:

- The layout of the `ansible/` directory and the inventory
- How secrets are stored and how the vault password is handled
- How playbooks are run, for example manually or from CI

## References

- [Host Configuration glossary](../../ansible/GLOSSARY.md)
- [Ansible documentation](https://docs.ansible.com/)
- [Validating tasks: check mode and diff mode](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_checkmode.html)
- [Installing collections with a requirements file](https://docs.ansible.com/projects/ansible/latest/collections_guide/collections_installing.html)
- [Protecting sensitive data with Ansible Vault](https://docs.ansible.com/projects/ansible/latest/vault_guide/index.html)
