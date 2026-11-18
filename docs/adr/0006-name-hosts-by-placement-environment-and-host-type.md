# ADR-0006: Name Hosts by placement, Environment, and Host Type

- **Status:** Accepted
- **Date:** 2026-11-18

## Context

Host names are used in every context: Provisioning creates VM Hosts by name, Host Configuration lists Hosts by name in the inventory, and k3s identifies Cluster Nodes by hostname. Once a name is in all of those places, changing it takes significant work, so the format has to be decided before the first Host is built.

The homelab has two standalone Proxmox Hypervisors, a Docker Host that is a Physical Host, and Cluster Nodes that are VM Hosts. VM Hosts run in two Environments, production and test, both on the same Hypervisors.

## Decision

Names follow `{prefix}-{environment}-{host-type}[-{cluster-node-type}]-{number}`, as specified in [Host Naming Convention](../host-naming.md):

- The prefix says what kind of machine it is: `srv` for a Physical Host, `vm` for a VM Host, `pve` for a Hypervisor, and `wks` for the Workstation.
- Every Host and Hypervisor name includes its Environment (`prd` or `tst`). The Workstation's does not.
- The Host Type is `dkr` or `k3s`. A Cluster Node adds `cp` or `wrk`. There is no cluster ID.
- The number counts per Environment and Host Type, is shared by `srv` and `vm`, and is never reused.
- A name is never changed. A Host that changes prefix, Environment, or Host Type is rebuilt as a new Host.
- VM IDs are derived from the name, so they are unique across both Hypervisors.

## Alternatives considered

| Option                                                   | Why it was not chosen                                                                                                                                                      |
| -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Leave out the prefix, so names say only what a Host does | Names would survive a move between a Physical Host and a VM Host, but the owner wants the kind of machine visible in the name. Such a move rebuilds the Host anyway.       |
| Leave out the Environment while there is one             | Shorter names, but the test Environment is planned, and adding the segment later would rebuild every Host.                                                                 |
| A cluster ID, such as `vm-prd-k3s-01-cp-01`              | The Environment already tells apart the clusters that are planned. Two clusters in the same Environment would need a new ADR.                                              |
| `k8s`, or `server` and `agent`                           | The Cluster runs k3s, and the glossary avoids "k8s". `srv` already means Physical Host, so `cp` and `wrk` are used instead of `server` and `agent`.                        |
| One Proxmox cluster with a shared VM ID space            | Would keep VM IDs unique without a convention, but a two-node cluster needs an extra quorum device. The Hypervisors stay standalone, and the name gives the VM ID instead. |
| Free VM IDs on each Hypervisor                           | Two VMs could share a VM ID, which makes backups on shared storage and logs ambiguous.                                                                                     |
| Rename a Host in place when it changes                   | Manual drift across DNS, certificates, the inventory, and the k3s node, which identifies itself by hostname.                                                               |

## Consequences

**Benefits**

- A name shows the kind of machine, its Environment, and its Host Type without a lookup.
- Names, numbers, and VM IDs each identify one Host in the homelab's history.
- OpenTofu can compute a VM ID from a name instead of storing it.

**Costs and risks**

- Moving a Host between a Physical Host and a VM Host means building a new Host with a new name.
- Every name carries `prd` even while nearly everything is in production.
- A Hypervisor's name says `prd`, but it also runs test VM Hosts. Test workloads can use up resources that production needs.
- The VM ID scheme has ten values per digit, and each new Host Type needs a new digit assigned in the naming convention.

## Out of scope

- DNS names, zones, and TLS certificates.
- How Ansible separates the test Environment from production, which [ADR-ANSIBLE-0001](../../ansible/docs/adr/0001-organise-ansible-by-host-type.md) does not plan for.

## References

- [RFC 1178: Choosing a Name for Your Computer](https://www.rfc-editor.org/rfc/rfc1178)
- [k3s architecture: servers and agents](https://docs.k3s.io/architecture)
- [Proxmox VE: renaming a node](https://pve.proxmox.com/wiki/Renaming_a_PVE_node)
- [Proxmox VE: backup and restore](https://pve.proxmox.com/wiki/Backup_and_Restore)
