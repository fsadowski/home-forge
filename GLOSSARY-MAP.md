# Glossary Map

## Contexts

- [Host Configuration](./ansible/GLOSSARY.md): configures the operating system of existing Hosts with Ansible

Planned contexts, added when their directories are created:

- Provisioning (`opentofu/`): creates and destroys infrastructure
- Containerized services (`docker/`): defines the Base Services
- Cluster workloads (`kubernetes/`): defines what runs on the k3s cluster

## Relationships

- **Provisioning → Host Configuration**: Provisioning creates VM Hosts on a Hypervisor and runs their Bootstrap with cloud-init; Host Configuration takes over from there.
- **Physical Host Bootstrap**: done by hand, outside any context; Host Configuration takes over once the Host is reachable.
- **Host Configuration → Cluster workloads**: Host Configuration installs k3s on the Cluster Nodes; cluster workloads run on top of it.
- **Host Configuration → Containerized services**: Host Configuration prepares the Docker Host; containerized services define what runs on it.

## Shared language

**Hypervisor**:
A physical machine that runs Proxmox and hosts VM Hosts. It is not a Host: Ansible does not configure it.
_Avoid_: Proxmox node, PVE host
