# Host Naming Convention

How Hosts, Hypervisors, and the Workstation are named, and how VM IDs follow from those names. Terms are defined in [`GLOSSARY-MAP.md`](../GLOSSARY-MAP.md).

DNS names, zones, and TLS certificates are out of scope.

## Why

Host names end up everywhere: OpenTofu creates VM Hosts by name, the Ansible inventory lists Hosts by name, and k3s identifies Cluster Nodes by hostname. Changing a name later takes significant work, so the format is fixed before the first Host is built.

- **The prefix shows the kind of machine.** Moving a Host between a Physical Host and a VM Host rebuilds it anyway, so a name that changes with the move costs nothing extra.
- **The Environment is always in the name,** even while nearly everything is in production. Adding the segment later would mean rebuilding every Host.
- **There is no cluster ID.** The Environment already tells the planned clusters apart.
- **`cp` and `wrk` rather than k3s's `server` and `agent`,** because `srv` already means Physical Host.
- **Names are never changed in place.** Renaming would drift across DNS, certificates, the inventory, and the k3s node.
- **VM IDs come from the name.** The two Hypervisors are standalone, so Proxmox does not keep VM IDs unique across them, and a shared Proxmox cluster would need an extra quorum device. Unique VM IDs keep backups on shared storage and logs unambiguous.

## Format

```text
{prefix}-{environment}-{host-type}[-{cluster-node-type}]-{number}
```

- Lowercase letters, digits, and hyphens only.
- Segments that do not apply are left out, as the table below shows. No segment is ever empty.

| Machine       | Format                                       | Example            |
| ------------- | -------------------------------------------- | ------------------ |
| Physical Host | `srv-{environment}-{host-type}[-…]-{number}` | `srv-prd-dkr-01`   |
| VM Host       | `vm-{environment}-{host-type}[-…]-{number}`  | `vm-prd-k3s-cp-01` |
| Hypervisor    | `pve-{environment}-{number}`                 | `pve-prd-01`       |
| Workstation   | `wks-{number}`                               | `wks-01`           |

## Segments

### Prefix

Says what kind of machine it is.

| Prefix | Meaning       |
| ------ | ------------- |
| `srv`  | Physical Host |
| `vm`   | VM Host       |
| `pve`  | Hypervisor    |
| `wks`  | Workstation   |

A new prefix is added only when a new kind of device is managed from this repository.

### Environment

The Environment the machine belongs to.

| Code  | Environment |
| ----- | ----------- |
| `prd` | Production  |
| `tst` | Test        |

- Every Host and every Hypervisor has an Environment. The Workstation does not.
- A Hypervisor's Environment says that it is production infrastructure. Both Hypervisors are `prd`, and they run `tst` VM Hosts too, so `tst` is not isolated from `prd` hardware.
- A new code is added only when an Environment is actually used.

### Host Type

| Code  | Host Type    |
| ----- | ------------ |
| `dkr` | Docker Host  |
| `k3s` | Cluster Node |

A Cluster Node also has a Cluster Node type:

| Code  | Cluster Node type  |
| ----- | ------------------ |
| `cp`  | Control Plane Node |
| `wrk` | Worker Node        |

There is no cluster ID: the Environment identifies the Cluster.

### Number

- Two digits, zero-padded, starting at `01`.
- Counted per Environment and Host Type, including the Cluster Node type, and shared by `srv` and `vm`. If `srv-prd-dkr-01` is replaced by a VM Host, the new Host is `vm-prd-dkr-02`.
- Never reused. Gaps are left as they are.
- Hypervisors and Workstations each count on their own.

## Lifecycle

- A name belongs to one Host for its whole life. Names are never changed in place.
- A Host that changes prefix, Environment, or Host Type is a new Host: it is built with a new name, and the old Host is retired.
- A Hypervisor gets its final name when Proxmox is installed, because [renaming a Proxmox node](https://pve.proxmox.com/wiki/Renaming_a_PVE_node) later is error-prone.

## VM IDs

The two Hypervisors are standalone, so Proxmox does not keep VM IDs unique across them. Every VM ID is derived from the VM Host's name instead, which makes it unique across both:

```text
{environment digit}{host type digit}{number}
```

| Digit   | Meaning              | Values                                       |
| ------- | -------------------- | -------------------------------------------- |
| 1st     | Environment          | `1` = `prd`, `2` = `tst`                     |
| 2nd     | Host Type            | `1` = `dkr`, `2` = `k3s-cp`, `3` = `k3s-wrk` |
| 3rd–4th | Number from the name | `01`–`99`                                    |

- `vm-prd-k3s-cp-01` is `1201`, and `vm-tst-k3s-wrk-02` is `2302`.
- VM templates use `9000`–`9099`.
- A VM ID does not depend on which Hypervisor runs the VM Host.

## Examples

| Name                | Machine                                | VM ID  |
| ------------------- | -------------------------------------- | ------ |
| `pve-prd-01`        | First Hypervisor                       |        |
| `pve-prd-02`        | Second Hypervisor                      |        |
| `srv-prd-dkr-01`    | Docker Host, a Physical Host           |        |
| `vm-prd-k3s-cp-01`  | First Control Plane Node in production | `1201` |
| `vm-prd-k3s-wrk-01` | First Worker Node in production        | `1301` |
| `vm-tst-k3s-wrk-01` | First Worker Node in test              | `2301` |
| `wks-01`            | Workstation                            |        |

## Changing this convention

Adding a code to a table above, or changing the format, is a change to this document, with the reason given in the commit message.
