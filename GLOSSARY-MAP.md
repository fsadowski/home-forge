# Glossary Map

## Contexts

Planned contexts. Each one gets its own glossary, linked from here, when its directory is created:

- Provisioning (`opentofu/`): creates and destroys infrastructure
- Host Configuration (`ansible/`): configures the operating system of existing Hosts
- Containerized Services (`docker/`): defines the services that run on the Docker Host
- Cluster Workloads (`kubernetes/`): defines what runs on the Cluster

## Relationships

- **Provisioning → Host Configuration**: Provisioning creates VM Hosts on a Hypervisor; Host Configuration configures them once they are reachable.
- **Physical Hosts**: set up by hand until they are reachable; Host Configuration takes over from there.
- **Host Configuration → Cluster Workloads**: Host Configuration installs k3s on the Cluster Nodes; Cluster Workloads run on top of it.
- **Host Configuration → Containerized Services**: Host Configuration prepares the Docker Host; Containerized Services define what runs on it.

## Shared language

### Machines

**Host**:
A machine whose operating system is configured by Ansible from this repository.
_Avoid_: server, box, machine

**Physical Host**:
A Host that runs directly on its own hardware.
_Avoid_: bare metal, physical kind, machine type

**VM Host**:
A Host that runs as a virtual machine on a Hypervisor.
_Avoid_: guest, VM kind, machine type

**Hypervisor**:
A physical machine that runs Proxmox and hosts VM Hosts. It is not a Host: Ansible does not configure it.
_Avoid_: Proxmox node, PVE host

**Workstation**:
The machine the owner runs this repository's tools from. It is not a Host: Ansible does not configure it.
_Avoid_: laptop, dev machine, control node

### Host Types

**Host Type**:
What a Host does in the homelab: Docker Host or Cluster Node. Every Host has exactly one Host Type, independent of whether it is a Physical Host or a VM Host.
_Avoid_: role, Host role, purpose

**Docker Host**:
A Host that runs services as containers with Docker.
_Avoid_: Docker server

**Cluster Node**:
A Host that is a member of the Cluster.
_Avoid_: node, k8s VM, Proxmox node

**Control Plane Node**:
A Cluster Node that runs the Cluster's control plane and datastore.
_Avoid_: server, master

**Worker Node**:
A Cluster Node that runs Cluster Workloads but not the control plane.
_Avoid_: agent, worker, minion

### Workloads

**Cluster**:
The k3s cluster that runs the Cluster Workloads. Its members are the Cluster Nodes.
_Avoid_: k8s cluster, Kubernetes cluster

**Environment**:
The stage a workload belongs to: production or test. It is not a hardware boundary: VM Hosts of both Environments run on the same Hypervisors.
_Avoid_: stage, tier
