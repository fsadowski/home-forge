# ADR-ANSIBLE-0001: Organise Ansible by Host Type

- **Status:** Partially superseded by [ADR-0004](../../../docs/adr/0004-lock-ansible-python-dependencies-with-uv.md), [ADR-ANSIBLE-0002](./0002-keep-one-inventory-per-environment.md), and [ADR-ANSIBLE-0003](./0003-check-every-host-before-any-change.md)
- **Why (ADR-0004):** uv's lockfile now pins transitive Python dependencies with hashes, which `requirements.txt` could not. The `requirements.txt` entry in the layout, its bullet under "Pinning", the rejected "`uv` with a lockfile" alternative, and the cost about unlocked transitive dependencies no longer apply.
- **Why (ADR-ANSIBLE-0002):** a test Environment is now planned, so each Environment gets its own inventory. The `inventory/` part of the layout, the inventory bullets on a single Environment, on `ansible.cfg`, and on where variables live, and the rejected "Separate staging and production inventories" alternative no longer apply.
- **Why (ADR-ANSIBLE-0003):** the checks that run before any role now cover names, the Environment, the connection user, and the operating system, not only the Host Type. The `check_host_type.yml` entry in the layout, the "One Host Type per Host" bullet on what it asserts, and the Playbooks bullet that imports it no longer apply.
- **Formerly:** ADR-0002
- **Date:** 2026-11-15

## Context

[ADR-0001](../../../docs/adr/0001-use-ansible-for-host-configuration.md) chose Ansible for host configuration and left the layout of `ansible/` and the inventory for a separate decision.

Every [Host](../../GLOSSARY.md) needs the Baseline and Hardening, and then the configuration for its Host Type. Today there are two Host Types: the Docker Host, which is a Physical Host, and the Cluster Nodes, which are VM Hosts. A Host that misses its Hardening is a security gap, so the layout should make that hard to do by accident.

The inventory holds the addresses of the homelab, and `AGENTS.md` does not allow unencrypted inventory data to be committed. ADR-0001 also requires `ansible-core` and the collections to be pinned.

## Decision

Organise `ansible/` around Host Types, in this layout:

```text
ansible/
├── ansible.cfg
├── requirements.txt
├── requirements.yml
├── collections/             # gitignored
├── inventory/
│   ├── hosts.example.yml
│   ├── hosts.yml            # gitignored
│   └── group_vars/
│       ├── all.yml
│       ├── docker_hosts.yml
│       └── cluster_nodes.yml
├── site.yml
├── check_host_type.yml
├── docker_hosts.yml
├── cluster_nodes.yml
└── roles/
    ├── baseline/
    ├── hardening/
    ├── docker/
    └── k3s/
```

**Inventory**

- There is one static YAML inventory and one environment.
- Hosts are grouped by Host Type only, in groups named after the glossary terms: `docker_hosts` and `cluster_nodes`. Physical Hosts and VM Hosts get groups only when a variable differs between them.
- The real `inventory/hosts.yml` is gitignored, and `inventory/hosts.example.yml` is committed with placeholders. `ansible.cfg` points at `inventory/hosts.yml` rather than at the directory, so the example Hosts are never loaded.
- Variables live in `inventory/group_vars/` and are committed. They hold only settings that are not private. There are no `host_vars`.

**One Host Type per Host**

- Every Host has exactly one Host Type.
- `check_host_type.yml` runs on `all` and asserts that each Host is in exactly one of the groups listed in `host_types`, which is set in `group_vars/all.yml`.

**Playbooks**

- Each Host Type has one playbook. It imports `check_host_type.yml` first, then applies `baseline`, `hardening`, and the roles of that Host Type.
- `site.yml` imports every Host Type playbook.

**Roles**

- Roles are written in this repository and named after what they configure, not after a Host Type.
- Each role follows the standard Ansible role structure, contains only the directories it uses, and has a `README.md` that lists its variables.
- Settings that a group may change go in `defaults/`. `vars/` holds only internal constants. Every variable is prefixed with its role name.
- Roles contain no custom modules or plugins.
- A community role is used only after an explicit decision for that role.

**Pinning**

- `requirements.txt` pins `ansible-core` and `ansible-lint` with `==`, and they are installed into the Python virtual environment in `ansible/.venv`, which mise creates as [ADR-0002](../../../docs/adr/0002-manage-workstation-tools-with-mise.md) decides.
- `requirements.yml` pins the collections, which are installed into `ansible/collections/`.

## Alternatives considered

| Option                                       | Why it was not chosen                                                                                                                                                         |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dynamic inventory from Proxmox or OpenTofu   | The Physical Host is in neither, so a static part would still be needed. It adds a plugin to maintain for a few Hosts. Worth revisiting once VM Hosts change often.           |
| Separate staging and production inventories  | The homelab has one real environment. `--check` and a throwaway VM Host are cheaper than keeping a second inventory in sync.                                                  |
| Groups for Physical Hosts and VM Hosts       | The difference matters only for the Bootstrap, which happens before Ansible takes over. Groups with no variables or plays attached are noise.                                 |
| One play on `all` for Baseline and Hardening | Running a single Host Type playbook would then skip Hardening without any warning.                                                                                            |
| Several Host Types per Host                  | It would let a Host share duties, such as the Docker Host joining the cluster, but it makes Baseline and Hardening run twice and blurs what a Host is for.                    |
| Committing the inventory unencrypted         | The addresses reveal little, but `AGENTS.md` does not allow it. Whether the inventory is committed encrypted is left to the decision on secrets.                              |
| Community roles by default                   | A community hardening role hides the part of the work that is worth understanding and reviewing.                                                                              |
| `uv` with a lockfile                         | It would also lock transitive Python dependencies, but it adds a tool. Exact pins in `requirements.txt` meet the pinning rule for now. ADR-0002 adopts mise but keeps uv out. |

## Consequences

**Benefits**

- Running any Host Type playbook on its own applies the Baseline and Hardening, so a Host cannot be left unhardened by running the wrong playbook.
- A Host that is in no Host Type group, or in two, fails the check before anything changes.
- The layout follows the Ansible sample setup, so a reader who knows Ansible finds things where they expect.
- Collections installed into the project mean that a run uses exactly the pinned versions, not whatever is in `~/.ansible`.

**Costs and risks**

- The real inventory is not version-controlled and lives only on the workstation until the decision on secrets covers it. Losing the workstation means rewriting it from the example.
- A Host cannot have two duties. A Docker Host that also needs to be in the cluster has to become two Hosts.
- Adding a Host Type means a new group, a new playbook, an entry in `host_types`, and an import in `site.yml`. The check catches a Host whose group was left out of `host_types`.
- Each Host Type playbook lists `baseline` and `hardening`, which repeats two lines per Host Type.
- `ansible.cfg` is read only when Ansible runs from `ansible/`, and Ansible ignores it in a world-writable directory.
- Transitive Python dependencies are not locked, so a fresh install can differ from an earlier one.

## Out of scope

- How secrets are stored, and whether the real inventory is committed encrypted
- How playbooks are run, for example manually or from CI
- Whether k3s is installed with a community role

## References

- [Host Configuration glossary](../../GLOSSARY.md)
- [Sample Ansible setup](https://docs.ansible.com/projects/ansible/latest/tips_tricks/sample_setup.html)
- [Roles](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
- [How to build your inventory](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html)
- [Variable precedence](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html)
