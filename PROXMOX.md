# ☁️ Proxmox Configuration

Advanced Proxmox VE setup, clustering, high availability, and VM management.

---

## 📋 Cluster Overview

**Cluster Name:** `homelab-cluster`

| Node | Hostname | IP | CPU | RAM | Storage | Role |
|------|----------|----|----|-----|---------|------|
| 1 | pve-node1 | 10.0.99.11 | i5-7400T | 16GB | 500GB SSD | Compute |
| 2 | pve-node2 | 10.0.99.12 | i5-7500T | 16GB | 500GB SSD | Compute |
| 3 | pve-node3 | 10.0.99.13 | AMD A10-9700E | 16GB | 128GB SSD + 1TB NVMe | Compute |
| 4 | pve-node4 | 10.0.99.14 | i5-7500T | 16GB | 500GB SSD | Compute |

**Total Resources:**
- **CPU:** ~14 cores / 20 threads
- **RAM:** 64GB total
- **Storage:** ~2.6TB local storage

---

## 🔧 Post-Installation Configuration

### Remove Enterprise Repository
```bash
# On all nodes
nano /etc/apt/sources.list.d/pve-enterprise.list
# Comment out the line:
# deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise

# Add no-subscription repo
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list

# Update
apt update && apt dist-upgrade -y
```

### Remove Subscription Nag (Optional)
```bash
# Backup original file
cp /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js.bak

# Remove nag
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js

# Restart web service
systemctl restart pveproxy
```

### Configure NTP
```bash
# Ensure all nodes use same time source
timedatectl set-timezone America/Toronto  # Adjust for your timezone
systemctl enable --now systemd-timesyncd
timedatectl status
```

---

## 🌐 Cluster Setup

### Create Cluster (Node 1)
```bash
pvecm create homelab-cluster
pvecm status
```

### Join Cluster (Nodes 2-4)
```bash
# On each additional node
pvecm add 10.0.99.11
# Enter root password when prompted

# Verify membership
pvecm status
pvecm nodes
```

### Configure Corosync
```bash
# Edit corosync config for better performance
nano /etc/pve/corosync.conf

# Add/modify these settings:
totem {
    version: 2
    cluster_name: homelab-cluster
    transport: knet
    crypto_cipher: aes256
    crypto_hash: sha256
    link_mode: passive
}

# Reload corosync on all nodes
systemctl reload corosync
```

### Verify Cluster Health
```bash
# Check cluster status
pvecm status

# Check quorum
pvecm expected 4

# Check corosync status
systemctl status corosync
journalctl -u corosync -f
```

---

## 💾 Storage Configuration

### Local Storage (Each Node)
```bash
# Local disk for VMs/containers
/dev/sda1 → ext4 → /var/lib/vz

# Create directory storage in Proxmox GUI:
Datacenter → Storage → Add → Directory
- ID: local
- Path: /var/lib/vz
- Content: Disk image, Container, ISO image
```

### Shared Storage (TrueNAS)

#### NFS Share for ISOs
```bash
# On TrueNAS, create NFS share: /mnt/tank/proxmox-isos

# On each Proxmox node
mkdir -p /mnt/pve/truenas-isos

# Add to Proxmox GUI:
Datacenter → Storage → Add → NFS
- ID: truenas-isos
- Server: 10.0.40.10
- Export: /mnt/tank/proxmox-isos
- Content: ISO image, Container template
```

#### NFS Share for Backups
```bash
# On TrueNAS, create NFS share: /mnt/tank/proxmox-backups

# Add to Proxmox GUI:
Datacenter → Storage → Add → NFS
- ID: truenas-backups
- Server: 10.0.40.10
- Export: /mnt/tank/proxmox-backups
- Content: VZDump backup file
- Max Backups: 3
```

#### Optional: iSCSI for VM Storage
```bash
# For high-performance VM disks (if needed)
# Configure iSCSI target on TrueNAS
# Add iSCSI storage in Proxmox GUI
```

---

## 🖥️ VM Templates

### Ubuntu 22.04 LTS Cloud-Init Template

```bash
# Download cloud image
cd /var/lib/vz/template/iso
wget https://cloud-images.ubuntu.com/releases/22.04/release/ubuntu-22.04-server-cloudimg-amd64.img

# Create VM
qm create 9000 --name ubuntu-22.04-template --memory 2048 --cores 2 --net0 virtio,bridge=vmbr0

# Import disk
qm importdisk 9000 ubuntu-22.04-server-cloudimg-amd64.img local-lvm

# Attach disk
qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9000-disk-0

# Add cloud-init drive
qm set 9000 --ide2 local-lvm:cloudinit

# Configure boot
qm set 9000 --boot c --bootdisk scsi0

# Add serial console
qm set 9000 --serial0 socket --vga serial0

# Enable QEMU guest agent
qm set 9000 --agent enabled=1

# Convert to template
qm template 9000
```

### Debian 12 Template
```bash
# Similar process as Ubuntu
# Download Debian cloud image
# Create VM ID 9001
# Import and configure
```

### Windows 10/11 Template
```bash
# Create VM with VirtIO drivers
qm create 9010 --name windows-10-template --memory 4096 --cores 4
qm set 9010 --scsihw virtio-scsi-pci
qm set 9010 --scsi0 local-lvm:32,format=qcow2
qm set 9010 --net0 virtio,bridge=vmbr0
qm set 9010 --cdrom local:iso/windows10.iso
qm set 9010 --ide0 local:iso/virtio-win.iso

# Install Windows, then convert to template
```

---

## 🐧 LXC Container Templates

### Download Templates
```bash
# List available templates
pveam update
pveam available

# Download common templates
pveam download local debian-12-standard_12.0-1_amd64.tar.zst
pveam download local ubuntu-22.04-standard_22.04-1_amd64.tar.zst
pveam download local alpine-3.18-default_20230607_amd64.tar.xz
```

### Create Container Template

```bash
# Create base container
pct create 8000 local:vztmpl/debian-12-standard_12.0-1_amd64.tar.zst \
  --hostname debian-template \
  --memory 512 \
  --cores 1 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --storage local-lvm \
  --rootfs local-lvm:8

# Start and configure
pct start 8000
pct enter 8000

# Inside container: update and install tools
apt update && apt upgrade -y
apt install -y curl wget git nano htop

# Exit and convert to template
pct stop 8000
# In Proxmox GUI: right-click → Convert to template
```

---

## 🚀 VM Deployment Examples

### Deploy from Template (CLI)
```bash
# Clone Ubuntu template
qm clone 9000 100 --name web-server --full

# Customize with cloud-init
qm set 100 --ipconfig0 ip=10.0.10.10/24,gw=10.0.10.1
qm set 100 --nameserver 10.0.99.20
qm set 100 --ciuser admin
qm set 100 --sshkeys ~/.ssh/id_rsa.pub

# Start VM
qm start 100
```

### Deploy Container (CLI)
```bash
# Clone Debian container template
pct clone 8000 200 --hostname nginx-proxy --full

# Configure
pct set 200 --memory 1024 --cores 2
pct set 200 --net0 name=eth0,bridge=vmbr0,ip=10.0.10.20/24,gw=10.0.10.1

# Start
pct start 200
```

---

## 🔄 Backup & Restore

### Configure Backup Schedule
```bash
# Via GUI: Datacenter → Backup

# Or via CLI:
# Create backup job
vzdump 100 --storage truenas-backups --mode snapshot --compress zstd

# Schedule with cron
crontab -e
# Add: 0 2 * * * vzdump --all --storage truenas-backups --mode snapshot --compress zstd
```

### Manual Backup
```bash
# Backup specific VM
vzdump 100 --storage truenas-backups --mode snapshot

# Backup all VMs
vzdump --all --storage truenas-backups --mode snapshot
```

### Restore from Backup
```bash
# List backups
ls /mnt/pve/truenas-backups/dump/

# Restore VM
qmrestore /mnt/pve/truenas-backups/dump/vzdump-qemu-100-*.vma.zst 100

# Restore to new VM ID
qmrestore /mnt/pve/truenas-backups/dump/vzdump-qemu-100-*.vma.zst 101
```

---

## 🔐 High Availability (HA)

### Configure HA
```bash
# Create HA group (GUI recommended)
Datacenter → HA → Groups → Create

Group: production
Nodes: pve-node1, pve-node2, pve-node3, pve-node4
```

### Add VMs to HA
```bash
# Via GUI: Datacenter → HA → Add
# Select VM and group

# Or via CLI:
ha-manager add vm:100 --group production --max_restart 3 --max_relocate 3
```

### Test HA Failover
```bash
# Simulate node failure
systemctl stop pve-cluster
# Watch VM migrate to another node
# Restart cluster
systemctl start pve-cluster
```

---

## 📊 Monitoring & Performance

### Install qemu-guest-agent in VMs
```bash
# Ubuntu/Debian
apt install qemu-guest-agent
systemctl enable --now qemu-guest-agent

# CentOS/RHEL
yum install qemu-guest-agent
systemctl enable --now qemu-guest-agent

# Enable in Proxmox
qm set <vmid> --agent enabled=1
```

### Monitor Cluster Performance
```bash
# Real-time cluster status
pvecm status

# Node resource usage
pvesh get /cluster/resources

# VM statistics
qm list
qm status <vmid>
```

### Integration with Grafana
```bash
# Install Proxmox exporter on monitoring node
# Configure Prometheus to scrape Proxmox API
# Create Grafana dashboard for cluster metrics
# See MONITORING.md for details
```

---

## 🔧 Advanced Configuration

### Nested Virtualization
```bash
# Enable on host
echo "options kvm_intel nested=1" > /etc/modprobe.d/kvm-intel.conf
modprobe -r kvm_intel
modprobe kvm_intel

# Verify
cat /sys/module/kvm_intel/parameters/nested  # Should show 'Y'

# Enable for specific VM
qm set <vmid> --cpu host,flags=+vmx
```

### GPU Passthrough
```bash
# Enable IOMMU in GRUB
nano /etc/default/grub
# Add: intel_iommu=on iommu=pt
update-grub
reboot

# Verify IOMMU groups
find /sys/kernel/iommu_groups/ -type l

# Configure PCI passthrough
# See Proxmox documentation for full guide
```

### Custom Network Bridges
```bash
# Edit network config
nano /etc/network/interfaces

# Add VLAN-aware bridge for DMZ
auto vmbr1
iface vmbr1 inet manual
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes

# Apply changes
ifreload -a
```

---

## 🆘 Troubleshooting

### Cluster Issues

#### Red Cluster Status
```bash
# Check corosync
systemctl status corosync
journalctl -u corosync -n 50

# Restart cluster services
systemctl restart pve-cluster
systemctl restart corosync

# Re-sync cluster config
pvecm updatecerts
```

#### Split Brain
```bash
# Check expected votes
pvecm expected 4

# If node can't join cluster
systemctl stop pve-cluster
systemctl stop corosync
pmxcfs -l  # Force local mode
pvecm delnode <failed_node>
systemctl start corosync
systemctl start pve-cluster
```

### VM Issues

#### VM Won't Start
```bash
# Check VM config
qm config <vmid>

# Check locks
qm unlock <vmid>

# Check disk issues
qm rescan

# View detailed error
qm start <vmid> --debug
```

#### High CPU Usage
```bash
# Check VM processes
ps aux | grep kvm

# Check I/O wait
iostat -x 1

# Adjust CPU limits
qm set <vmid> --cpulimit 2  # Limit to 2 cores worth
```

### Storage Issues

#### NFS Mount Failed
```bash
# Test NFS manually
mount -t nfs 10.0.40.10:/mnt/tank/proxmox-backups /mnt/test

# Check NFS exports on TrueNAS
showmount -e 10.0.40.10

# Restart NFS service
systemctl restart nfs-client.target
```

---

## 📚 Useful Commands

```bash
# Cluster management
pvecm status          # Show cluster status
pvecm nodes           # List nodes
pvecm expected <n>    # Set expected votes

# VM management
qm list               # List all VMs
qm start <vmid>       # Start VM
qm stop <vmid>        # Stop VM
qm shutdown <vmid>    # Graceful shutdown
qm reset <vmid>       # Hard reset
qm migrate <vmid> <target>  # Live migration

# Container management
pct list              # List containers
pct start <ctid>      # Start container
pct enter <ctid>      # Enter container
pct exec <ctid> -- <cmd>  # Run command

# Storage
pvesm status          # List storage
pvesm scan <storage>  # Scan storage
```

---

## 🔗 References

- [Proxmox VE Documentation](https://pve.proxmox.com/wiki/Main_Page)
- [Proxmox Forum](https://forum.proxmox.com/)
- [Proxmox YouTube Channel](https://www.youtube.com/c/ProxmoxVE)

---

**Last Updated:** [DATE]  
**Cluster Uptime:** [DAYS]
