# ADR-ANSIBLE-0002: Keep one inventory per Environment

- **Status:** Accepted
- **Date:** 2026-11-22

## Context

[ADR-ANSIBLE-0001](./0001-organise-ansible-by-host-type.md) planned one static inventory for one Environment. [ADR-0006](../../../docs/adr/0006-name-hosts-by-placement-environment-and-host-type.md) has since planned a test Environment: `tst` VM Hosts that run on the same Hypervisors as production. ADR-0006 left open how Ansible keeps them apart.

A run meant for test Hosts must not change production Hosts, even when a command is mistyped or a flag is forgotten. No real inventory exists yet, so the layout can still change for free.

## Decision

Give each Environment its own inventory directory, and make every run name the one it targets.

```text
ansible/
├── ansible.cfg              # sets no inventory
├── group_vars/
│   └── all.yml              # the same in every Environment, such as host_types
└── inventory/
    ├── prd/
    │   ├── hosts.example.yml
    │   ├── hosts.yml        # gitignored
    │   └── group_vars/
    │       └── all.yml      # host_environment: prd
    └── tst/
        ├── hosts.example.yml
        ├── hosts.yml        # gitignored
        └── group_vars/
            └── all.yml      # host_environment: tst
```

- Each Environment has one inventory directory under `inventory/`, named after its Environment code from the [Host Naming Convention](../../../docs/host-naming.md).
- Each one holds a committed `hosts.example.yml` with placeholder Hosts, and a gitignored `hosts.yml` with the real Hosts. The groups are the same in every Environment, as ADR-ANSIBLE-0001 defines them.
- Each one sets `host_environment` to its Environment code. The name avoids `environment`, which Ansible reserves as a play keyword.
- A variable that is the same in every Environment lives in `group_vars/` next to the playbooks. A variable that differs between Environments lives in each inventory's `group_vars/`. No variable is set in both places. Files for a single group, such as `docker_hosts.yml`, follow the same rule and are created when the group has a variable.
- As ADR-ANSIBLE-0001 decides, variables hold only settings that are not private, and there are no `host_vars`.
- `ansible.cfg` sets no inventory. Every run passes `-i` with the inventory of one Environment, for example `-i inventory/tst`.

This replaces these parts of ADR-ANSIBLE-0001:

- the `inventory/` part of its layout
- "There is one static YAML inventory and one environment."
- the bullet that points `ansible.cfg` at `inventory/hosts.yml`
- the bullet that puts every variable in `inventory/group_vars/`
- the rejected alternative "Separate staging and production inventories"

Its other inventory decisions still apply.

## Alternatives considered

| Option                                              | Why it was not chosen                                                                                                                             |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| One inventory with `prd` and `tst` groups           | Each run would rely on `--limit` to pick an Environment. Running `site.yml` without it changes every Host in both Environments.                   |
| One inventory file per Environment in one directory | Simpler, but `group_vars/` belongs to a directory, so the Environments could not set different values for the same variable.                      |
| Default inventory set to `tst`                      | Shorter commands, but a forgotten `-i` still changes Hosts. With no default, a forgotten `-i` matches no Host and changes nothing.                |
| Every variable copied into each inventory           | Each inventory is complete on its own, but settings that must be the same, such as `host_types`, can drift apart between Environments.            |
| Wait until the first test Host exists               | Nothing to maintain for an Environment with no Hosts, but the inventory path, `ansible.cfg`, and `.gitignore` would change once real Hosts exist. |

## Consequences

**Benefits**

- A run reaches only the Hosts of the Environment it names, so test work cannot change production Hosts.
- A forgotten `-i` changes nothing.
- Shared settings are written once, so the Environments cannot disagree about them.
- The Environment of every Host is known to the playbooks, so the host checks can compare it with the Host's name.

**Costs and risks**

- Every command is longer, and `-i` resolves against the working directory, unlike the paths in `ansible.cfg`. From the repository root it is `-i ansible/inventory/tst`.
- Variables set next to the playbooks take precedence over the same group's variables in an inventory. A value set in both places would silently ignore the inventory's value, so the rule is that a variable lives in only one of them.
- A run without `-i` still uses `ANSIBLE_INVENTORY` if it is set, or `/etc/ansible/hosts` if it exists. Neither is set up on the Workstation: Ansible is installed with uv rather than from a system package, and the repository sets no `ANSIBLE_INVENTORY`.
- There are two example inventories to keep in step with the groups.
- A run cannot target both Environments at once. That is intended.

## Out of scope

- How secrets are stored, and whether the real inventories are committed encrypted. That is still the open decision on secrets.
- How runs are started, for example manually or from CI.

## References

- [How to build your inventory: organizing host and group variables](https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html#organizing-host-and-group-variables)
- [Variable precedence](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)
- [Reserved play keywords](https://docs.ansible.com/projects/ansible/latest/reference_appendices/playbooks_keywords.html#play)
