# Host Configuration ADRs

Architecture Decision Records that apply only to the Host Configuration context in `ansible/`. Decisions that span contexts, such as the choice of Ansible itself, are in the [repository ADRs](../../../docs/adr/README.md), which also hold the conventions and template.

## Index

| ADR                                                          | Title                              | Status                                                                                                                                                                                                                                 |
| ------------------------------------------------------------ | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [ANSIBLE-0001](./0001-organise-ansible-by-host-type.md)      | Organise Ansible by Host Type      | Partially superseded by [ADR-0004](../../../docs/adr/0004-lock-ansible-python-dependencies-with-uv.md), [ANSIBLE-0002](./0002-keep-one-inventory-per-environment.md), and [ANSIBLE-0003](./0003-check-every-host-before-any-change.md) |
| [ANSIBLE-0002](./0002-keep-one-inventory-per-environment.md) | Keep one inventory per Environment | Accepted                                                                                                                                                                                                                               |
| [ANSIBLE-0003](./0003-check-every-host-before-any-change.md) | Check every Host before any change | Accepted                                                                                                                                                                                                                               |
