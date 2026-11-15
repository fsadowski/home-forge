# Host Configuration

Configures the operating system of hosts that already exist, so that they reach and keep the state defined in this repository.

## Language

### Hosts

**Host**:
A machine whose operating system is configured by Ansible from this repository.
_Avoid_: server, box, machine

**Host Type**:
What a Host does in the homelab: Docker Host or Cluster Node. Every Host has exactly one Host Type, independent of whether it is a Physical Host or a VM Host.
_Avoid_: role, Host role, purpose

**Physical Host**:
A Host that runs directly on its own hardware.
_Avoid_: bare metal, physical kind, machine type

**VM Host**:
A Host that runs as a virtual machine on a Hypervisor.
_Avoid_: guest, VM kind, machine type

**Docker Host**:
A Host that runs the Base Services as containers.
_Avoid_: Docker server

**Cluster Node**:
A Host that is a member of the k3s cluster.
_Avoid_: node, k8s VM, Proxmox node

### Services

**Base Services**:
The services the rest of the homelab depends on: DNS, the reverse proxy, and monitoring.
_Avoid_: core services, infra services

### Lifecycle

**Bootstrap**:
The first-boot setup that makes a Host reachable by Ansible: the user Ansible connects as, SSH access, and Python. From then on, Baseline owns that user.
_Avoid_: provisioning, initial setup

**Baseline**:
The configuration every Host gets that is not Hardening, such as time synchronisation, base packages, and the admin user.
_Avoid_: common, base config

**Hardening**:
The baseline security configuration applied to every Host after it is reachable.
_Avoid_: securing, lockdown
