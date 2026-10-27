# Ubuntu Ansible Infrastructure Lab 🚀

[![CI](https://github.com/yopzman/ubuntu-ansible-infra-lab/actions/workflows/ci.yml/badge.svg)](https://github.com/yopzman/ubuntu-ansible-infra-lab/actions/workflows/ci.yml)

A **single-branch Ansible monorepo** that provisions KVM virtual machines on Ubuntu, hardens them for container workloads, and installs the Docker Engine — demonstrating the three core Ansible building blocks (standalone Role, custom Collection, and orchestration Playbooks) working together in one repository.

---

## 🏗️ Architecture

```
                         ┌─────────────────────────────┐
                         │  hypervisor (KVM host)      │
                         │  localhost / Ubuntu LTS     │
                         └──────────────┬──────────────┘
                                        │
              ┌─────────────────────────┴──────────────────────────┐
              │  01-provision-kvm.yml                              │
              │  roles/kvm_vm_provisioner                          │
              │  ── cloud image + cloud-init (user-data, meta-data,│
              │     network-config v2 with per-VM static IPs)      │
              │  ── qemu-img overlay disks + libvirt define/start  │
              └─────────────────────────┬──────────────────────────┘
                                        │  VMs boot with static IPs
                    ┌───────────────────┴───────────────────┐
                    ▼                                       ▼
           ┌─────────────────┐                     ┌─────────────────┐
           │ node-01         │                     │ node-02         │
           │ 192.168.122.11  │                     │ 192.168.122.12  │
           └────────┬────────┘                     └────────┬────────┘
                    │                                       │
        ┌───────────┴───────────────────────────────────────┴──────────┐
        │  02-harden-nodes.yml                                         │
        │  yopzman.ubuntu_platform.node_hardening                      │
        │  ── swap off ── overlay/br_netfilter ── sysctl ── UFW ── SSH │
        ├──────────────────────────────────────────────────────────────┤
        │  03-deploy-containers.yml                                    │
        │  yopzman.ubuntu_platform.container_engine                    │
        │  ── Docker Engine + Compose plugin ── daemon.json            │
        └──────────────────────────────────────────────────────────────┘
```

## 📂 Repository Layout

```
.
├── ansible.cfg                  # roles_path + collections_paths prewired
├── requirements.yml             # external collections (libvirt, posix, docker, general)
├── .github/workflows/ci.yml     # CI: yamllint + ansible-lint + syntax-check
├── inventory/
│   ├── hosts.yml                # hypervisor + ubuntu_nodes groups
│   └── group_vars/all.yml       # cloud image, VM specs, node list, SSH keys
├── roles/
│   └── kvm_vm_provisioner/      # Component 1: standalone role
│       ├── defaults/main.yml
│       ├── meta/main.yml
│       ├── tasks/               # main.yml + provision_vm.yml (per-VM loop)
│       └── templates/           # user-data.j2, meta-data.j2, network-config.j2, vm.xml.j2
├── collections/
│   └── ansible_collections/yopzman/ubuntu_platform/   # Component 2: collection
│       ├── galaxy.yml
│       ├── meta/runtime.yml
│       ├── changelogs/          # antsibull-changelog config + release data
│       └── roles/
│           ├── node_hardening/  # swap, modules, sysctl, UFW, sshd
│           └── container_engine/ # Docker CE, Compose, daemon.json
└── playbooks/                   # Component 3: orchestration
    ├── 01-provision-kvm.yml     # provision + wait_for_connection
    ├── 02-harden-nodes.yml
    ├── 03-deploy-containers.yml
    └── site.yml                 # master playbook (all three, in order)
```

## 🛠️ Prerequisites

* Control node: Ubuntu 22.04/24.04 with Ansible ≥ 2.15
* KVM/QEMU enabled on the hypervisor (`kvm-ok` should pass)
* An SSH keypair at `~/.ssh/id_ed25519` (or export `SSH_PUBKEY`)
* Libvirt `default` network with DHCP covering `192.168.122.0/24`

```bash
# Install Ansible core + this repo's collection dependencies
pip install ansible-core
ansible-galaxy collection install -r requirements.yml
```

## ▶️ Usage

Run everything from the repository root — `ansible.cfg` points Ansible at the local `roles/` and `collections/` automatically.

```bash
# Step 1: provision VMs on the KVM host
ansible-playbook playbooks/01-provision-kvm.yml

# Step 2: harden the newly provisioned nodes
ansible-playbook playbooks/02-harden-nodes.yml

# Step 3: install Docker on the nodes
ansible-playbook playbooks/03-deploy-containers.yml

# ...or run the entire pipeline end-to-end
ansible-playbook playbooks/site.yml
```

Useful variants:

```bash
# Dry run
ansible-playbook playbooks/site.yml --check --diff

# Limit to a single node
ansible-playbook playbooks/02-harden-nodes.yml --limit node-01

# Pass VM list / image overrides
ansible-playbook playbooks/01-provision-kvm.yml \
  -e '{"nodes": [{"name": "node-03", "ip": "192.168.122.13", "mac": "52:54:00:11:22:13"}]}'

# Lint the repo
ansible-lint playbooks/ roles/ collections/ansible_collections/yopzman/
```

## 🔒 What the Hardening Role Does

| Area | Effect |
| :--- | :--- |
| Swap | `swapoff` on all active devices, removes swap entries from `/etc/fstab` |
| Kernel modules | Loads `overlay` and `br_netfilter` at boot and immediately |
| Sysctl | Bridge netfilter calls, IP forwarding, anti-spoofing rp_filter, TCP SYN cookies, kernel pointer/dmesg restrictions |
| UFW | Default-deny inbound, allow-list model (SSH by default) |
| SSHD | Drop-in config: no root login, no passwords, key-only, bounded auth tries |

## 🐳 What the Container Engine Role Does

* Removes distro Docker packages, adds Docker's official GPG key + apt repo
* Installs `docker-ce`, `docker-ce-cli`, `containerd.io`, buildx and **compose plugins**
* Writes `/etc/docker/daemon.json` with the **systemd cgroup driver**, `overlay2` storage, and log rotation (`json-file`, 10 MB × 3 files) — restarts Docker via handler only when the config changes
* Manages `docker` group membership for the provisioned user

## ✅ Design Principles

* **FQCN everywhere** — every module call uses its fully qualified name (`ansible.builtin.apt`, `community.libvirt.virt`, `ansible.posix.sysctl`, …)
* **Idempotency** — `creates:` guards on shell steps, seed ISOs rebuild only when their cloud-init files change, handlers only fire on real changes
* **Lint-clean** — CI enforces `yamllint` + `ansible-lint` + playbook syntax checks on every push; role variables follow the `var-naming[no-role-prefix]` convention
* **Single branch** — role, collection, and playbooks coexist and resolve locally through `ansible.cfg`

## 📄 License

MIT — see [LICENSE](LICENSE).
