<div align="center">

# 🖥️ Multi-Node KVM Linux Lab Environment

![Ubuntu 22.04 LTS](https://img.shields.io/badge/OS-Ubuntu%2022.04%20LTS-orange?style=for-the-badge&logo=ubuntu)
![KVM Hypervisor](https://img.shields.io/badge/Hypervisor-KVM%2Flibvirt-red?style=for-the-badge&logo=redhat)
![Architecture](https://img.shields.io/badge/Architecture-3--Node%20Cluster-blue?style=for-the-badge&logo=linux)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

<p align="center">
  <b>A step-by-step, beginner-friendly guide to rapidly building a lightweight 3-node Ubuntu 22.04 LTS virtual machine lab on KVM/QEMU with pre-configured SSH access, static IPs, and mutual hostname resolution.</b>
</p>

---

</div>

## 🎯 What is this Lab Environment Suitable For?

This multi-node environment provides a sandbox designed for hands-on learning and testing across key IT domains:

<table>
  <tr>
    <td width="50%">
      <h3>🛠️ DevOps Practice</h3>
      <ul>
        <li>Deploying CI/CD runners (Jenkins, GitLab)</li>
        <li>Docker Swarm cluster management</li>
        <li>Microservices integration testing</li>
      </ul>
    </td>
    <td width="50%">
      <h3>📜 Ansible Automation</h3>
      <ul>
        <li>Mastering inventory management</li>
        <li>Writing and testing playbooks & roles</li>
        <li>Automated configuration across nodes</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td width="50%">
      <h3>⚙️ System Administration</h3>
      <ul>
        <li>User and access management</li>
        <li>LVM disk expansion & partitioning</li>
        <li>Systemd services & UFW firewall setup</li>
      </ul>
    </td>
    <td width="50%">
      <h3>🌐 Infrastructure & Clustering</h3>
      <ul>
        <li>HAProxy load balancing & Keepalived</li>
        <li>Nginx reverse proxies</li>
        <li>Master-replica database setups (MySQL/PostgreSQL)</li>
      </ul>
    </td>
  </tr>
</table>

---

## 📐 Lab Network & Node Architecture

| Node Name | IP Address | vCPUs | RAM | OS Version | Default User |
| :--- | :---: | :---: | :---: | :---: | :---: |
| 🖥️ **node1** | `192.168.122.10` | 2 | 2 GB | Ubuntu 22.04 LTS | `ubuntu` |
| 🖥️ **node2** | `192.168.122.11` | 2 | 2 GB | Ubuntu 22.04 LTS | `ubuntu` |
| 🖥️ **node3** | `192.168.122.12` | 2 | 2 GB | Ubuntu 22.04 LTS | `ubuntu` |

---

## 🛠️ Prerequisites (Run on Host Machine)

Ensure **KVM**, **libvirt**, and **guestfs-tools** are installed on your Linux host:

```bash
# Update repository index and install KVM hypervisor tools
sudo apt update && sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager libguestfs-tools
