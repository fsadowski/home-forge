# Open questions

Decisions that have not been made yet. Remove a question in the same change as the ADR or doc that answers it.

## Across contexts

- **Secrets:** how secrets are stored, how the vault password is handled, and whether the real inventory is committed encrypted. From [ADR-0001](adr/0001-use-ansible-for-host-configuration.md) and [ADR-ANSIBLE-0001](../ansible/docs/adr/0001-organise-ansible-by-host-type.md).
- **CI:** whether playbooks run manually or from CI, which CI platform, and how CI installs the pinned tools and Python dependencies. From ADR-0001, [ADR-0002](adr/0002-manage-workstation-tools-with-mise.md), and [ADR-0004](adr/0004-lock-ansible-python-dependencies-with-uv.md).
- **Workstation automation:** whether mise tasks run the validation checks, and whether `uv sync` runs automatically. From ADR-0002 and ADR-0004.
- **Docker and Kubernetes CLIs:** whether mise pins the Docker CLI and `kubectl`, which the validation checks in `AGENTS.md` use.
- **DNS names:** the zone Hosts live in, how their DNS names are built, and how TLS certificates are issued. Left out of [ADR-0006](adr/0006-name-hosts-by-placement-environment-and-host-type.md).
- **Hypervisor:** how the Hypervisors are configured. Today it is done by hand, an exception to the no-manual-drift rule in `AGENTS.md`.

## Host Configuration

- **k3s:** whether k3s is installed with a community role. From ADR-ANSIBLE-0001.
- **Test Environment:** how the inventory separates test Hosts from production. ADR-ANSIBLE-0001 assumes one environment and one inventory, but [ADR-0006](adr/0006-name-hosts-by-placement-environment-and-host-type.md) plans a test Environment.

## Planned contexts

- **Provisioning:** the OpenTofu provider for the Hypervisor, where OpenTofu state is stored, and how cloud-init is templated.
- **Containerized Services:** which DNS server, reverse proxy, and monitoring stack make up the Base Services, and how they are deployed to the Docker Host.
- **Cluster Workloads:** whether manifests are managed with Helm, kustomize, or both. `AGENTS.md` mentions both.
