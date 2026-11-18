# Host Configuration

Configures the operating system of Hosts that already exist, so that they reach and keep the state defined in this repository.

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
A Host that is a member of the Cluster.
_Avoid_: node, k8s VM, Proxmox node

**Admin User**:
The one account on a Host that both the owner and Ansible log in as. Bootstrap creates it, and Baseline owns it from then on.
_Avoid_: Ansible user, automation user, deploy user

### Services

**Base Services**:
The services the rest of the homelab depends on: DNS, the reverse proxy, and monitoring.
_Avoid_: core services, infra services

### Lifecycle

**Bootstrap**:
The first-boot setup that makes a Host reachable by Ansible: the Admin User, SSH access, and Python.
_Avoid_: provisioning, initial setup

**Baseline**:
The configuration every Host gets apart from Hardening, such as time synchronization, base packages, and the Admin User.
_Avoid_: common, base config

**Hardening**:
The security configuration every Host gets once it is reachable, such as SSH and firewall settings.
_Avoid_: securing, lockdown
