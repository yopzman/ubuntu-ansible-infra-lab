# yopzman.ubuntu_platform

A collection of Ansible roles for preparing Ubuntu Server 22.04/24.04 LTS nodes to run containerized and Kubernetes workloads.

## Roles

| Role | Purpose |
| :--- | :--- |
| `yopzman.ubuntu_platform.node_hardening` | Disables swap, loads kernel modules, applies sysctl network parameters, hardens UFW and SSH. |
| `yopzman.ubuntu_platform.container_engine` | Installs official Docker Engine + Compose plugin, configures `daemon.json`, manages Docker group membership. |

## Install

```bash
ansible-galaxy collection build collections/ansible_collections/yopzman/ubuntu_platform
ansible-galaxy collection install yopzman-ubuntu_platform-1.0.0.tar.gz
```

When working inside this repository the collection is already on the `collections_paths`, so no install step is needed.
