# Glossary Map

## Contexts

- [Host Configuration](./ansible/GLOSSARY.md): configures the operating system of existing Hosts with Ansible. Decisions: [ADRs](./ansible/docs/adr/README.md)

Each context keeps its glossary in its own directory.

Planned contexts, added when their directories are created:

- Provisioning (`opentofu/`): creates and destroys infrastructure
- Containerized Services (`docker/`): defines the Base Services
- Cluster Workloads (`kubernetes/`): defines what runs on the Cluster

## Relationships

- **Provisioning → Host Configuration**: Provisioning creates VM Hosts on a Hypervisor and runs their Bootstrap with cloud-init; Host Configuration takes over from there.
- **Bootstrap of a Physical Host**: done by hand, outside any context; Host Configuration takes over once the Host is reachable.
- **Host Configuration → Cluster Workloads**: Host Configuration installs k3s on the Cluster Nodes; Cluster Workloads run on top of it.
- **Host Configuration → Containerized Services**: Host Configuration prepares the Docker Host; Containerized Services define what runs on it.

## Shared language

**Hypervisor**:
A physical machine that runs Proxmox and hosts VM Hosts. It is not a Host: Ansible does not configure it.
_Avoid_: Proxmox node, PVE host

**Cluster**:
The k3s cluster that runs the Cluster Workloads. Its members are the Cluster Nodes.
_Avoid_: k8s cluster, Kubernetes cluster

**Environment**:
The stage a workload belongs to: production or test. It is not a hardware boundary: VM Hosts of both Environments run on the same Hypervisors.
_Avoid_: stage, tier

**Workstation**:
The machine the owner runs this repository's tools from. It is not a Host: Ansible does not configure it.
_Avoid_: laptop, dev machine, control node
