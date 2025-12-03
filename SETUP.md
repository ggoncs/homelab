# 🚀 Initial Setup Guide

Complete walkthrough for setting up the homelab from scratch.

---

## 📋 Prerequisites

- [ ] All hardware assembled and connected
- [ ] Network cables labeled and organized
- [ ] ISP router configured in AP mode
- [ ] USB drives prepared for OS installations
- [ ] Static IP plan documented (see NETWORK.md)

---

## 🔧 Phase 1: Physical Setup

### 1.1 Rack Assembly
```bash
# Document rack layout
# Photo/diagram of cable routing
# Power distribution setup
```

**Steps:**
1. Install rack shelves at appropriate U positions
2. Mount patch panel (U10)
3. Install MikroTik switch with rack ears (U11)
4. Mount venting panel (U12)
5. Cable management - use velcro straps

### 1.2 Power Setup
1. Connect UPS to wall outlet
2. Plug PDU into UPS
3. Connect all devices to PDU
4. Test UPS cutover (simulate power loss)

**UPS Configuration:**
```bash
# Document UPS IP address and default credentials
# Screenshot of UPS web interface
```

---

## 🌐 Phase 2: Network Bootstrap

### 2.1 MikroTik Switch Initial Config
```bash
# Connect to switch via WinBox or WebFig
# Default IP: 192.168.88.1
# Username: admin
# Password: (blank)

# Commands to document:
/interface bridge add name=bridge1
/interface bridge port add bridge=bridge1 interface=ether1
# ... (add actual config here)
```

### 2.2 Create VLANs
```bash
# VLAN 10 - Management
# VLAN 20 - LAN
# VLAN 30 - DMZ
# VLAN 40 - Storage

# Document VLAN configuration commands
```

### 2.3 Firewall Installation
**Hardware:** N150 Mini PC (on top of rack)

1. Download pfSense/OPNsense ISO
2. Flash to USB drive with Rufus/Etcher
3. Boot N150 from USB
4. Follow installation wizard

**Initial Config:**
```bash
# WAN Interface: igb0
# LAN Interface: igb1
# DMZ Interface: igb2
# Management: igb3

# Document WAN IP (DHCP from ISP)
# LAN IP: 192.168.1.1/24
# DMZ IP: 10.0.10.1/24
# Management: 10.0.99.1/24
```

---

## 💻 Phase 3: Proxmox Cluster Setup

### 3.1 Install Proxmox VE (Node 1)
**Hardware:** ThinkCentre M710Q (i5-7400T) - U05

1. Download Proxmox VE ISO
2. Flash to USB drive
3. Boot ThinkCentre from USB
4. Follow installation wizard

**Installation Settings:**
```bash
Hostname: pve-node1.homelab.local
IP Address: 10.0.99.11/24
Gateway: 10.0.99.1
DNS: 10.0.99.1
```

**Post-Install:**
```bash
# SSH into node
ssh root@10.0.99.11

# Update repositories
nano /etc/apt/sources.list.d/pve-enterprise.list
# Comment out enterprise repo

# Add community repo
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list

# Update system
apt update && apt upgrade -y

# Remove subscription nag (optional)
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
systemctl restart pveproxy
```

### 3.2 Install Proxmox VE (Nodes 2-4)
Repeat above process for remaining nodes:

| Node | Hostname | IP | Hardware |
|------|----------|----|----- |
| Node 2 | pve-node2 | 10.0.99.12 | M710Q (i5-7500T) - U06 |
| Node 3 | pve-node3 | 10.0.99.13 | M715Q (AMD A10) - U07 |
| Node 4 | pve-node4 | 10.0.99.14 | M710Q (i5-7500T) - U08 |

### 3.3 Create Proxmox Cluster
**On Node 1:**
```bash
pvecm create homelab-cluster
pvecm status
```

**On Nodes 2-4:**
```bash
pvecm add 10.0.99.11
# Enter root password when prompted
pvecm status
```

**Verify cluster:**
```bash
pvecm nodes
# Should show all 4 nodes
```

---

## 💾 Phase 4: TrueNAS Setup

### 4.1 Install TrueNAS SCALE
**Hardware:** Jonsbo N2 (N5105) - U01-U05

1. Download TrueNAS SCALE ISO
2. Flash to USB drive
3. Boot from USB
4. Install to 120GB Kingston SSD

**Initial Config:**
```bash
Hostname: truenas.homelab.local
IP Address: 10.0.40.10/24  # Storage VLAN
Gateway: 10.0.99.1
DNS: 10.0.99.1
```

### 4.2 Create ZFS Pool
```bash
# Via TrueNAS Web UI: http://10.0.40.10

# Pool Name: tank
# Layout: RAID-Z1 (RAID 5 equivalent)
# Disks: 5x 1TB HDDs
# Usable Capacity: ~4TB

# L2ARC Cache: 1TB NVMe
```

### 4.3 Create Datasets
```bash
# Create datasets for different purposes
tank/media         # Jellyfin media files
tank/backups       # VM/container backups
tank/documents     # General file storage
tank/isos          # OS installation ISOs
```

### 4.4 Setup Shares
```bash
# SMB Share for general access
# NFS Share for Proxmox ISO/backup storage
# iSCSI (optional) for VM storage
```

---

## 🥧 Phase 5: Raspberry Pi Setup

### 5.1 Install Raspberry Pi OS Lite
**Hardware:** Raspberry Pi 5 (8GB) with PoE+ HAT - U09

1. Flash Raspberry Pi OS Lite to NVMe
2. Enable SSH before first boot
3. Configure static IP

**Initial Config:**
```bash
Hostname: pi-monitor.homelab.local
IP Address: 10.0.99.20/24
Gateway: 10.0.99.1
DNS: 10.0.99.1
```

### 5.2 Install Monitoring Stack
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Create docker-compose.yml for monitoring stack
# See MONITORING.md for detailed config
```

### 5.3 Configure 7" Display
```bash
# Enable display in /boot/config.txt
# Configure Grafana to show on boot
```

---

## 🔐 Phase 6: Security Hardening

### 6.1 SSH Key Setup
```bash
# Generate SSH keys on workstation
ssh-keygen -t ed25519 -C "homelab-admin"

# Copy to all hosts
ssh-copy-id root@10.0.99.11  # Node 1
ssh-copy-id root@10.0.99.12  # Node 2
# ... etc
```

### 6.2 Disable Password Auth
```bash
# On each host
nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
systemctl restart sshd
```

### 6.3 Configure Firewall Rules
See NETWORK.md for detailed firewall configuration

---

## ✅ Phase 7: Validation

### 7.1 Network Connectivity Tests
```bash
# From workstation, test connectivity to all hosts
ping 10.0.99.11  # Proxmox Node 1
ping 10.0.99.20  # Raspberry Pi
ping 10.0.40.10  # TrueNAS
ping 192.168.1.1 # Firewall

# Test DNS resolution
nslookup pve-node1.homelab.local
```

### 7.2 Cluster Health Checks
```bash
# Proxmox cluster status
pvecm status
pvecm nodes

# Check corosync
systemctl status corosync
systemctl status pve-cluster
```

### 7.3 Storage Tests
```bash
# Test TrueNAS shares
# Mount SMB share from workstation
# Test read/write speeds with dd
dd if=/dev/zero of=/mnt/truenas/test.img bs=1G count=1 oflag=direct
```

---

## 📝 Post-Setup Tasks

- [ ] Document all IP addresses and credentials in password manager
- [ ] Create initial VM templates (Ubuntu, Debian, Windows)
- [ ] Configure automated backups
- [ ] Setup monitoring alerts
- [ ] Create restore procedures documentation
- [ ] Test UPS failover and auto-shutdown

---

## 🔗 Next Steps

- [Network Configuration](./NETWORK.md) - VLAN and firewall setup
- [Proxmox Configuration](./PROXMOX.md) - Advanced cluster setup
- [TrueNAS Configuration](./TRUENAS.md) - Storage optimization
- [Monitoring Setup](./MONITORING.md) - Grafana dashboards

---

## 🆘 Troubleshooting

### Proxmox Cluster Issues
```bash
# If cluster is in "red" state
systemctl restart pve-cluster
systemctl restart corosync

# Check cluster configuration
cat /etc/pve/corosync.conf
```

### Network Issues
```bash
# Check interface status
ip addr show
ip route show

# Test VLAN connectivity
ping -I vlan10 10.0.99.1
```

### TrueNAS Pool Issues
```bash
# Check pool status
zpool status tank

# Scrub pool
zpool scrub tank
```

---

**Setup completed:** [DATE]  
**Time spent:** [HOURS]  
**Issues encountered:** [NOTES]
