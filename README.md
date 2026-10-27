# Ubuntu Ansible Infrastructure Lab 🚀

Welcome to the **Ubuntu Ansible Infrastructure Lab**, a structured portfolio repository showcasing advanced automation, configuration management, and infrastructure orchestration tailored for **Containerization & Virtualisation Engineers**.

This repository is organized into distinct structural branches, demonstrating different architectural scales of Ansible automation—ranging from standalone modular roles to enterprise-grade collections and end-to-end orchestration playbooks.

---

## 📂 Repository Structure & Branch Guide

You can navigate through the repository branches to explore different implementation patterns:

| Branch Name | Component Type | Description & Focus |
| :--- | :--- | :--- |
| [`ansible-role`](../../tree/ansible-role) | **Reusable Role** | Standalone, idempotent, and modular roles designed for specific tasks (e.g., KVM/libvirt setup, Docker installation, system hardening). |
| [`ansible-collection`](../../tree/ansible-collection) | **Enterprise Collection** | Standardized namespace-based collection (`yopzman.ubuntu_infrastructure`) bundling multiple roles and dependencies. |
| [`ansible-playbook`](../../tree/ansible-playbook) | **Orkestrasi Playbook** | End-to-end multi-node orchestration playbooks, inventory management, and variable structuring for Ubuntu server deployment. |

---

## 🛠️ Core Tech Stack & Target Environment
* **Operating System:** Ubuntu Server 22.04 / 24.04 LTS
* **Automation Tool:** Ansible (Core & Collections)
* **Target Domains:** 
  * Virtualization Host & KVM/QEMU Management
  * Container Engine Bootstrap (Docker & Containerd)
  * Security Baseline & OS Hardening (CIS Benchmark compliance)

---

## 🚀 Quick Navigation & Usage
To inspect a specific project structure, switch to the respective branch using Git:

```bash
# Clone the repository
git clone [https://github.com/yopzman/ubuntu-ansible-lab.git](https://github.com/yopzman/ubuntu-ansible-lab.git)
cd ubuntu-ansible-lab

# Switch to a specific architecture branch (e.g., Ansible Role)
git checkout ansible-role
