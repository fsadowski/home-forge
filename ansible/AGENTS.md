# AGENTS.md

Rules for the Host Configuration context. The repository-wide rules in [`../AGENTS.md`](../AGENTS.md) apply here too. Read [`README.md`](README.md) before running any Ansible command: it covers the layout, the inventory groups, and why every command runs from this directory.

## Writing Ansible code

- Name files `*.yaml`, and start each one with `---`.
- Call modules by their fully qualified name, for example `ansible.builtin.package`.
- Give every task and play a `name`.
- Keep playbooks thin: a play sets `hosts` and `become`, then applies roles. Logic lives in roles.
- In a role, `tasks/main.yaml` only imports task files, one per concern, such as `packages.yaml`.
- Keep data in `vars/` and out of tasks. When a value differs by OS family, key it by `ansible_facts['os_family']` and fall back with `default([])`, as `roles/common` does.
- Read facts through `ansible_facts['…']`, not through top-level fact variables such as `ansible_os_family`.
- Target groups, never single Hosts. Every Host is in one Host Type group and one Environment group.

## Idempotency

A second run with no changes must report `changed=0`. Reach for a module that manages state first. When only `ansible.builtin.command` or `ansible.builtin.shell` will do, set `creates`, `removes`, or `changed_when` so the task reports change truthfully.

## Inventory

Committed inventory holds placeholders only: Host names from the example, `<ssh-user>`-style tokens, and addresses from the RFC 5737 range. Run every check against `inventory/home-forge.example.yaml`. When a change adds a Host or group, update the example in the same change.
