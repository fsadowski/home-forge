# Glossary Map

## Contexts

- [Host Configuration](./ansible/GLOSSARY.md): configures the operating system of existing Hosts with Ansible

Planned contexts, added when their directories are created:

- Provisioning (`opentofu/`): creates and destroys infrastructure
- Containerized services (`docker/`): defines the Base Services
- Cluster workloads (`kubernetes/`): defines what runs on the k3s cluster

## Relationships

- **Provisioning → Host Configuration**: Provisioning creates the Cluster Node VMs on the Hypervisor and runs their Bootstrap with cloud-init; Host Configuration takes over from there.
- **Docker Host Bootstrap**: done by hand, outside any context; Host Configuration takes over once the Docker Host is reachable.
- **Host Configuration → Cluster workloads**: Host Configuration installs k3s on the Cluster Nodes; cluster workloads run on top of it.
- **Host Configuration → Containerized services**: Host Configuration prepares the Docker Host; containerized services define what runs on it.

## Shared language

**Hypervisor**:
The physical machine that runs Proxmox and hosts the Cluster Node VMs. It is not a Host: Ansible does not configure it.
_Avoid_: Proxmox node, PVE host
