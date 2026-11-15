# Host Configuration

Configures the operating system of hosts that already exist, so that they reach and keep the state defined in this repository.

## Language

### Hosts

**Host**:
A machine whose operating system is configured by Ansible from this repository.
_Avoid_: server, box, machine

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
The first-boot setup that makes a Host reachable by Ansible: a user, SSH access, and Python.
_Avoid_: provisioning, initial setup

**Hardening**:
The baseline security configuration applied to every Host after it is reachable.
_Avoid_: securing, lockdown
