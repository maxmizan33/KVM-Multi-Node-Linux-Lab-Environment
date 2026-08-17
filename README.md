<div align="center">

  <h1>🖥️ Multi-Node KVM Linux Lab Environment</h1>

  <p>
    <img src="https://img.shields.io/badge/OS-Ubuntu%2022.04%20LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu 22.04 LTS" />
    <img src="https://img.shields.io/badge/Hypervisor-KVM%2Flibvirt-CC0000?style=for-the-badge&logo=redhat&logoColor=white" alt="KVM Hypervisor" />
    <img src="https://img.shields.io/badge/Architecture-3--Node%20Cluster-0078D4?style=for-the-badge&logo=linux&logoColor=white" alt="Architecture" />
    <img src="https://img.shields.io/badge/License-MIT-28A745?style=for-the-badge" alt="License" />
  </p>

  <p><b>A step-by-step, beginner-friendly guide to rapidly building a lightweight 3-node Ubuntu 22.04 LTS virtual machine lab on KVM/QEMU with pre-configured SSH access, static IPs, and mutual hostname resolution.</b></p>

</div>

<hr />

<h2>🎯 What is this Lab Environment Suitable For?</h2>

<p>This multi-node environment provides a sandbox designed for hands-on learning and testing across key IT domains:</p>

<table width="100%">
  <tr>
    <td width="50%" valign="top">
      <h3>🛠️ DevOps Practice</h3>
      <ul>
        <li>Deploying CI/CD runners (Jenkins, GitLab)</li>
        <li>Docker Swarm cluster management</li>
        <li>Microservices integration testing</li>
      </ul>
    </td>
    <td width="50%" valign="top">
      <h3>📜 Ansible Automation</h3>
      <ul>
        <li>Mastering inventory management</li>
        <li>Writing and testing playbooks &amp; roles</li>
        <li>Automated configuration across nodes</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <h3>⚙️ System Administration</h3>
      <ul>
        <li>User and access management</li>
        <li>LVM disk expansion &amp; partitioning</li>
        <li>Systemd services &amp; UFW firewall setup</li>
      </ul>
    </td>
    <td width="50%" valign="top">
      <h3>🌐 Infrastructure &amp; Clustering</h3>
      <ul>
        <li>HAProxy load balancing &amp; Keepalived</li>
        <li>Nginx reverse proxies</li>
        <li>Master-replica database setups (MySQL/PostgreSQL)</li>
      </ul>
    </td>
  </tr>
</table>

<hr />

<h2>📐 Lab Network &amp; Node Architecture</h2>

<table width="100%">
  <thead>
    <tr style="background-color: #f6f8fa;">
      <th align="left">Node Name</th>
      <th align="center">IP Address</th>
      <th align="center">vCPUs</th>
      <th align="center">RAM</th>
      <th align="center">OS Version</th>
      <th align="center">Default User</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>🖥️ <b>node1</b></td>
      <td align="center"><code>192.168.122.10</code></td>
      <td align="center">2</td>
      <td align="center">2 GB</td>
      <td align="center">Ubuntu 22.04 LTS</td>
      <td align="center"><code>ubuntu</code></td>
    </tr>
    <tr>
      <td>🖥️ <b>node2</b></td>
      <td align="center"><code>192.168.122.11</code></td>
      <td align="center">2</td>
      <td align="center">2 GB</td>
      <td align="center">Ubuntu 22.04 LTS</td>
      <td align="center"><code>ubuntu</code></td>
    </tr>
    <tr>
      <td>🖥️ <b>node3</b></td>
      <td align="center"><code>192.168.122.12</code></td>
      <td align="center">2</td>
      <td align="center">2 GB</td>
      <td align="center">Ubuntu 22.04 LTS</td>
      <td align="center"><code>ubuntu</code></td>
    </tr>
  </tbody>
</table>

<hr />

<h2>🛠️ Prerequisites (Run on Host Machine)</h2>

<p>Ensure <b>KVM</b>, <b>libvirt</b>, and <b>guestfs-tools</b> are installed on your Linux host:</p>

<pre><code class="language-bash"># Update repository index and install KVM hypervisor tools
sudo apt update &amp;&amp; sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager libguestfs-tools</code></pre>

<hr />

<h2>⚡ Phase 1: Base Image &amp; Primary VM Setup (node1)</h2>

<h3>1. Download Official Ubuntu Cloud Image</h3>

<pre><code class="language-bash"># Navigate to standard libvirt storage directory
cd /var/lib/libvirt/images/

# Download Ubuntu 22.04 (Jammy) cloud image
# 💡 CHANGE URL BELOW: Replace if using a different release or mirror
sudo wget https://cloud-images.ubuntu.com/jammy/20260725/jammy-server-cloudimg-amd64-disk-kvm.img -O base-ubuntu-22.04.img

# Create primary disk image for node1
sudo cp base-ubuntu-22.04.img node1-disk.qcow2

# Expand virtual disk space (+18GB addition makes it ~20GB total)
sudo qemu-img resize node1-disk.qcow2 +18G</code></pre>

<h3>2. Pre-Configure node1 Image</h3>

<div style="background-color: #fff8c5; border-left: 4px solid #d97706; padding: 12px; margin: 12px 0;">
  <strong>🚨 IMPORTANT:</strong> Replace <code>YOUR_SSH_PUBLIC_KEY_HERE</code> with your host machine's public SSH key string (e.g., contents of <code>cat ~/.ssh/id_ed25519.pub</code>).
</div>

<pre><code class="language-bash"># Inject user accounts, passwordless sudo, SSH keys, and config into disk image
sudo virt-customize -a node1-disk.qcow2 \
  --hostname node1 \
  --root-password password:ubuntu \
  --run-command 'useradd -m -s /bin/bash -G sudo ubuntu' \
  --password ubuntu:password:ubuntu \
  --run-command 'echo "ubuntu ALL=(ALL) NOPASSWD:ALL" &gt; /etc/sudoers.d/ubuntu &amp;&amp; chmod 0440 /etc/sudoers.d/ubuntu' \
  --run-command 'mkdir -p /home/ubuntu/.ssh &amp;&amp; chmod 700 /home/ubuntu/.ssh' \
  --run-command 'echo "YOUR_SSH_PUBLIC_KEY_HERE" &gt; /home/ubuntu/.ssh/authorized_keys' \
  --run-command 'chmod 600 /home/ubuntu/.ssh/authorized_keys &amp;&amp; chown -R ubuntu:ubuntu /home/ubuntu' \
  --run-command 'mkdir -p /run/sshd &amp;&amp; chmod 0755 /run/sshd' \
  --run-command 'ssh-keygen -A' \
  --run-command 'sed -i "s/PasswordAuthentication no/PasswordAuthentication yes/g" /etc/ssh/sshd_config' \
  --run-command 'systemctl enable ssh'</code></pre>

<h3>3. Launch node1 VM</h3>

<pre><code class="language-bash"># Set proper owner for libvirt/QEMU access
sudo chown libvirt-qemu:kvm node1-disk.qcow2

# Launch the virtual machine using virt-install
# 💡 CHANGE PARAMETERS: Adjust --memory 2048 (MB) or --vcpus 2 as needed
sudo virt-install \
  --name node1 \
  --memory 2048 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/node1-disk.qcow2,device=disk,bus=virtio \
  --os-variant ubuntu22.04 \
  --boot uefi \
  --network network=default,model=virtio \
  --graphics spice \
  --import \
  --noautoconsole</code></pre>

<h3>4. Set Static IP via Netplan (Run Directly Inside node1)</h3>

<p>To configure a permanent static IP directly inside the VM, use <b>Netplan</b> (Ubuntu's default network management tool). Access the VM console or dynamic IP first via SSH, then execute:</p>

<p><b>Step 1: Write Netplan Configuration</b></p>
<pre><code class="language-bash">sudo bash -c 'cat &lt;&lt; EOF &gt; /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: no
      addresses:
        - 192.168.122.10/24
      routes:
        - to: default
          via: 192.168.122.1
      nameservers:
        addresses:
          - 192.168.122.1
          - 8.8.8.8
EOF'</code></pre>

<p><b>Step 2: Bring Interface Up, Apply &amp; Test Configuration</b></p>
<pre><code class="language-bash"># Secure permissions on configuration file
sudo chmod 600 /etc/netplan/01-netcfg.yaml

# Ensure interface state is UP before applying configuration
sudo ip link set enp1s0 up

# Apply network settings
sudo netplan apply

# Verify assigned static IP and interface state
ip -br addr show enp1s0</code></pre>

<hr />

<h2>🔄 Phase 2: Deploy Cloned Nodes (node2, node3, etc.)</h2>

<p>Since <code>node1</code> is fully configured, clone its disk image to spin up additional lab nodes instantly without repeating setup.</p>

<h3>Step 1: Clone Disk &amp; Launch VM</h3>

<details>
  <summary><b>🔹 Click to expand setup commands for node2</b></summary>
  <br />
  <pre><code class="language-bash"># Clone node1 disk to node2
sudo cp /var/lib/libvirt/images/node1-disk.qcow2 /var/lib/libvirt/images/node2-disk.qcow2

# Update hostname inside node2 disk
sudo virt-customize -a /var/lib/libvirt/images/node2-disk.qcow2 --hostname node2

# Set ownership and spin up VM
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/node2-disk.qcow2

sudo virt-install \
  --name node2 \
  --memory 2048 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/node2-disk.qcow2,device=disk,bus=virtio \
  --os-variant ubuntu22.04 \
  --boot uefi \
  --network network=default,model=virtio \
  --graphics spice \
  --import \
  --noautoconsole</code></pre>
</details>

<details>
  <summary><b>🔹 Click to expand setup commands for node3</b></summary>
  <br />
  <pre><code class="language-bash"># Clone node1 disk to node3
sudo cp /var/lib/libvirt/images/node1-disk.qcow2 /var/lib/libvirt/images/node3-disk.qcow2

# Update hostname inside node3 disk
sudo virt-customize -a /var/lib/libvirt/images/node3-disk.qcow2 --hostname node3

# Set ownership and spin up VM
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/node3-disk.qcow2

sudo virt-install \
  --name node3 \
  --memory 2048 \
  --vcpus 2 \
  --disk path=/var/lib/libvirt/images/node3-disk.qcow2,device=disk,bus=virtio \
  --os-variant ubuntu22.04 \
  --boot uefi \
  --network network=default,model=virtio \
  --graphics spice \
  --import \
  --noautoconsole</code></pre>
</details>

<h3>Step 2: Configure Static IPs for Cloned Nodes (Inside node2 &amp; node3)</h3>

<p>Log into each cloned VM and update <code>/etc/netplan/01-netcfg.yaml</code> with their target static IP addresses:</p>

<p><b>For node2 (192.168.122.11):</b></p>
<pre><code class="language-bash">sudo bash -c 'cat &lt;&lt; EOF &gt; /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: no
      addresses:
        - 192.168.122.11/24
      routes:
        - to: default
          via: 192.168.122.1
      nameservers:
        addresses:
          - 192.168.122.1
          - 8.8.8.8
EOF'

sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo ip link set enp1s0 up
sudo netplan apply</code></pre>

<p><b>For node3 (192.168.122.12):</b></p>
<pre><code class="language-bash">sudo bash -c 'cat &lt;&lt; EOF &gt; /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: no
      addresses:
        - 192.168.122.12/24
      routes:
        - to: default
          via: 192.168.122.1
      nameservers:
        addresses:
          - 192.168.122.1
          - 8.8.8.8
EOF'

sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo ip link set enp1s0 up
sudo netplan apply</code></pre>

<hr />

<h2>🌐 Phase 3: Hostname Resolution (/etc/hosts)</h2>

<p>Inject lab IP mappings across all nodes simultaneously directly from your host machine:</p>

<pre><code class="language-bash"># Run this loop on your host terminal to update /etc/hosts across all 3 nodes
# 💡 CHANGE IPS: Update the IP list below if your lab uses a different subnet
for ip in 192.168.122.10 192.168.122.11 192.168.122.12; do
  ssh ubuntu@$ip "sudo bash -c 'cat &lt;&lt; EOF &gt;&gt; /etc/hosts

# --- Lab Host Resolution ---
192.168.122.10 node1
192.168.122.11 node2
192.168.122.12 node3
EOF'"
done</code></pre>

<hr />

<h2>📑 Cheat Sheet: Essential Management Commands</h2>

<div style="background-color: #ddf4ff; border-left: 4px solid #0969da; padding: 12px; margin: 12px 0;">
  <strong>ℹ️ NOTE:</strong> Useful <code>virsh</code> commands for day-to-day lab management.
</div>

<table width="100%">
  <thead>
    <tr style="background-color: #f6f8fa;">
      <th align="left" width="30%">Action</th>
      <th align="left" width="70%">Command</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>List Running VMs</b></td>
      <td><code>sudo virsh list</code></td>
    </tr>
    <tr>
      <td><b>List All VMs</b></td>
      <td><code>sudo virsh list --all</code></td>
    </tr>
    <tr>
      <td><b>Check Assigned IPs</b></td>
      <td><code>sudo virsh net-dhcp-leases default</code></td>
    </tr>
    <tr>
      <td><b>Open Console</b></td>
      <td><code>sudo virsh console node1</code> <i>(Exit: <code>Ctrl + ]</code>)</i></td>
    </tr>
    <tr>
      <td><b>Start VM</b></td>
      <td><code>sudo virsh start node1</code></td>
    </tr>
    <tr>
      <td><b>Graceful Shutdown</b></td>
      <td><code>sudo virsh shutdown node1</code></td>
    </tr>
    <tr>
      <td><b>Force Stop</b></td>
      <td><code>sudo virsh destroy node1</code></td>
    </tr>
    <tr>
      <td><b>Delete VM &amp; Storage</b></td>
      <td><code>sudo virsh destroy node1 &amp;&amp; sudo virsh undefine node1 --remove-all-storage</code></td>
    </tr>
  </tbody>
</table>
