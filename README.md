# home-forge

Infrastructure as code for my homelab, doubling as a portfolio of my DevOps/platform engineering work.

Built with the help of AI and held to production-ready standards, following industry best practices.

## Goals

- Learn continuously by building and running real systems
- Grow and improve the homelab
- Develop my skills in working with AI
- Gain hands-on experience with a wide range of tools

## Structure

The planned layout. Each directory is created with the first code that belongs in it.

| Directory     | Purpose                                        |
| ------------- | ---------------------------------------------- |
| `opentofu/`   | Infrastructure provisioning with OpenTofu      |
| `ansible/`    | Host configuration and automation              |
| `docker/`     | Containerized services and Compose definitions |
| `kubernetes/` | Kubernetes manifests and cluster configuration |
| `docs/`       | Docs that span contexts                        |

## Documentation

- [Architecture Decision Records](docs/adr/README.md): how decisions are recorded, and an index of every ADR
- [Host naming convention](docs/host-naming.md): how Hosts, Hypervisors, and the Workstation are named
- [Glossary map](GLOSSARY-MAP.md): the domain terms used across the repository
- [AGENTS.md](AGENTS.md): working agreements and engineering standards for contributors, human or AI
