# ADR-ANSIBLE-0003: Check every Host before any change

- **Status:** Proposed
- **Date:** 2026-11-22

## Context

Several decisions say that a wrong Host must be stopped before Ansible changes it:

- [ADR-ANSIBLE-0001](./0001-organise-ansible-by-host-type.md) adds `check_host_type.yml`, which asserts that every Host has exactly one Host Type.
- [ADR-0005](../../../docs/adr/0005-run-ubuntu-lts-on-every-host.md) requires every run to fail on a Host that does not report Ubuntu 26.04, "in the same checks that run before the roles".
- [ADR-0006](../../../docs/adr/0006-name-hosts-by-placement-environment-and-host-type.md) fixes the format of Host names. A misnamed Host is expensive to fix once DNS, certificates, and k3s depend on its name.
- [ADR-ANSIBLE-0002](./0002-keep-one-inventory-per-environment.md) sets `host_environment` in each Environment's inventory.

The `hardening` role, the first role to be written, stops root from logging in over SSH. A run that connects as root would lock itself out of the Host.

## Decision

Replace `check_host_type.yml` with `check_hosts.yml`, which runs every check below on `all`. Every Host Type playbook imports it before any role, as ADR-ANSIBLE-0001 decides for its predecessor.

The checks that use only inventory data run in a first play with fact gathering off, so they need no connection to any Host:

1. The Host is in exactly one of the Host Type groups listed in `host_types`.
2. The Host's inventory name matches the format in the [Host Naming Convention](../../../docs/host-naming.md).
3. The name's Host Type segment matches the Host's group: `dkr` for `docker_hosts`, and `k3s-cp` or `k3s-wrk` for `cluster_nodes`.
4. The name's Environment segment equals `host_environment`.
5. Ansible does not connect as `root`. It is meant to connect as the Automation User, but the check rules out only `root`, because that is the account Hardening locks out.

The checks that depend on what a Host reports about itself run in a second play, after facts are gathered:

6. The Host reports Ubuntu 26.04.
7. The name's prefix matches the `ansible_virtualization_role` fact: `vm` requires `guest`, and `srv` requires any other value.

- Both plays set `any_errors_fatal`, so one failed check stops the run for every Host, before any role runs.
- Each failed check names the Host and the rule it broke.
- A new rule that must hold before any change is added to these checks, not to a role.

This replaces these parts of ADR-ANSIBLE-0001:

- the `check_host_type.yml` entry in its layout
- the "One Host Type per Host" bullet on what `check_host_type.yml` asserts
- the Playbooks bullet that imports `check_host_type.yml` first, which now imports `check_hosts.yml`

Its rule that every Host has exactly one Host Type still applies, as check 1.

For check 7, the pinned `ansible-core` 2.22.0 reports `guest` on a VM Host whose SMBIOS system vendor is `QEMU`. That is the Proxmox default for a VM Host. On a Physical Host it reports `host` or `NA`. Its Docker detection looks only for signs of running inside a container, so running Docker on the Host does not make it report `guest`. This was read from the module's source (`ansible/module_utils/facts/virtual/linux.py`), not yet confirmed on a live Host. The runbook that comes with the `hardening` role is to confirm it.

## Alternatives considered

| Option                                        | Why it was not chosen                                                                                                                                                  |
| --------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| One check playbook per rule                   | Smaller files, but each Host Type playbook would import them all, and a missed import would skip a rule without any warning.                                           |
| All checks in one play, after gathering facts | Simpler, but an inventory mistake could then be found only by connecting to a Host. With the inventory checks first, they can be tested with no Host at all.           |
| Let a failed Host drop out, and continue      | Ansible's default. The other Hosts would be configured, but a Host that fails these checks usually means the inventory is wrong, so the run as a whole is not trusted. |
| Check names only in a separate linting step   | Catches mistakes earlier, but only if it is run. These checks run on every playbook run.                                                                               |
| Leave out the prefix check                    | One fewer fact to rely on, but a Physical Host named as a VM Host would go unnoticed until Provisioning tried to manage it.                                            |

## Consequences

**Benefits**

- A Host that is in the wrong group or inventory, misnamed, on another operating system, or reached as root is stopped before anything changes on any Host.
- Checks 1 to 5 can be tested on the Workstation against inventories with mistakes made on purpose, without a Host.
- The rules from earlier ADRs, and the two new ones in checks 5 and 7, are enforced in one place, which every Host Type playbook imports.

**Costs and risks**

- One misconfigured Host stops a run for every Host in the Environment until it is fixed.
- Check 7 depends on Ansible's fact detection and on the Proxmox SMBIOS defaults. Changing a VM's SMBIOS vendor, or an `ansible-core` upgrade that changes detection, can make a correct Host fail. The error names the rule, so the cause is visible.
- The name format is written both in the Host Naming Convention and in the check, so a change to the convention must update the check in the same change.

## References

- [`assert` module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/assert_module.html)
- [Error handling in playbooks: aborting a play on all hosts](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_error_handling.html#aborting-a-play-on-all-hosts)
- [Proxmox VE: VM SMBIOS settings](https://pve.proxmox.com/wiki/Manual:_qm.conf)
