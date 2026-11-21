# ADR-0005: Run Ubuntu 26.04 LTS on every Host

- **Status:** Accepted
- **Date:** 2026-11-18

## Context

The homelab has a Physical Host, the Docker Host, and VM Hosts on a Proxmox Hypervisor, the Cluster Nodes. Every role in Host Configuration depends on the operating system of the Hosts it configures: package names, the package manager, the firewall tool, and service defaults. Provisioning depends on it too, because it chooses the cloud image that cloud-init boots for each VM Host.

Supporting more than one operating system means branches in every role, and each branch has to be tested. The homelab is run by one person, so every branch is maintenance with no user.

## Decision

Every Host runs Ubuntu 26.04 LTS, whatever its Host Type and whether it is a Physical Host or a VM Host.

- The roles in `ansible/` support only Ubuntu 26.04 LTS. They contain no branches for other distributions or releases.
- Every playbook run fails before any change on a Host that does not report Ubuntu 26.04, in the same checks that run before the roles.
- Provisioning builds VM Hosts from the Ubuntu 26.04 cloud image.
- Supporting another operating system or release needs a new ADR, which loosens the check in the same change.

## Alternatives considered

| Option                                | Why it was not chosen                                                                                                                                                 |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Debian 13                             | Smaller and just as stable, but the owner prefers Ubuntu, and Ubuntu is more common in the environments this portfolio is aimed at.                                   |
| Ubuntu 24.04 LTS                      | Has been tested longer, but its standard support ends about two years earlier, so the homelab would need a release upgrade soon after it is built.                     |
| A different OS per Host Type          | Lets each Host Type use the best-suited OS, but doubles the work in Baseline and Hardening, which every Host gets.                                                     |
| Roles written distribution-neutral    | Makes adding an OS later cheaper, but the branches it adds go untested until that OS exists.                                                                           |
| An immutable OS for the Cluster Nodes | Already rejected in [ADR-0001](./0001-use-ansible-for-host-configuration.md), because a configuration tool would still be needed for the Physical Host.               |

## Consequences

**Benefits**

- Roles are written and reviewed against one operating system, with no conditionals on the distribution.
- The Docker Host and the Cluster Nodes share Baseline and Hardening unchanged.
- A Host that runs anything else is stopped before it is changed, rather than half-configured.

**Costs and risks**

- Moving to the next Ubuntu LTS is a deliberate project: a new ADR, a looser check, and a release upgrade or rebuild of every Host.
- Ubuntu 26.04 targets must stay within the Python versions that the pinned `ansible-core` supports. `ansible-core` 2.22 supports target Python 3.9 to 3.15, which covers what 26.04 ships.
- Software that runs on the Hosts, such as k3s, must support Ubuntu 26.04. This is checked when its version is pinned.

## References

- [Ubuntu 26.04 LTS release notes](https://ubuntu-release-notes-i18n.readthedocs.io/en/latest/26.04/)
- [ansible-core support matrix](https://docs.ansible.com/projects/ansible/latest/reference_appendices/release_and_maintenance.html)
- [Ubuntu cloud images](https://cloud-images.ubuntu.com/)
