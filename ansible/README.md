# Ansible

Host Configuration: configures the operating system of existing Hosts. Terms are defined in [`GLOSSARY-MAP.md`](../GLOSSARY-MAP.md).

Ansible takes over once a Host is reachable over SSH. VM Hosts are created by Provisioning (`opentofu/`), and Physical Hosts are set up by hand until then. The Hypervisors and the Workstation are not Hosts, so Ansible does not configure them.

## Layout

| Path                                | Purpose                                                        |
| ----------------------------------- | -------------------------------------------------------------- |
| `ansible.cfg`                       | Project configuration, loaded only when Ansible runs from here |
| `inventory/home-forge.example.yaml` | Example inventory                                              |
| `inventory/group_vars/all.yaml`     | Variables for every Host                                       |
| `playbooks/`                        | Playbooks, one per purpose                                     |
| `roles/`                            | Roles used by the playbooks                                    |

## Inventory

Every Host is in two groups: one for its Host Type and one for its Environment.

| Group          | Hosts            |
| -------------- | ---------------- |
| `docker_hosts` | Docker Hosts     |
| `prd`          | Production Hosts |

A new group is added with the first Host that needs it. Host names follow the [Host naming convention](../docs/host-naming.md).

Only `*.example.yaml` inventories are committed, with placeholder values only. When a Host or group is added, update the example in the same change. Addresses come from the [RFC 5737](https://www.rfc-editor.org/rfc/rfc5737) documentation range, such as `192.0.2.10`.

`group_vars/all.yaml` sets `ansible_python_interpreter` to `/usr/bin/python3`, so Ansible uses the system Python on every Host instead of discovering one.

## Playbooks

| Playbook      | Hosts | What it does                         |
| ------------- | ----- | ------------------------------------ |
| `common.yaml` | `all` | Applies the [`common`](#common) role |

## Roles

### common

The baseline every Host gets. It installs the packages listed in `roles/common/vars/main.yaml`:

- `common`: installed on every Host
- One list per OS family, keyed by the `ansible_facts['os_family']` fact, for example `Debian`. A Host whose OS family has no list gets only the common packages.

To add a package, add it to the matching list.

## Usage

Run every command from this directory, so that `ansible.cfg` is loaded. Ansible reads `ansible.cfg` from the current directory and does not search parent directories.

Preview a change with check mode first, and add `--diff` to see what would change:

```sh
ansible-playbook -i inventory/home-forge.example.yaml playbooks/common.yaml --check --diff
```

Apply it once the preview looks right:

```sh
ansible-playbook -i inventory/home-forge.example.yaml playbooks/common.yaml
```

To limit a run to one Host or group, add `--limit`, for example `--limit srv-prd-dkr-01`.

## Validation

```sh
ansible-lint
ansible-playbook -i inventory/home-forge.example.yaml --syntax-check playbooks/common.yaml
```
