# 🌐 Network Configuration (Updated with Dell 10GbE)

Complete network topology, VLAN configuration, and DMZ firewall setup including Dell Tower 10GbE SFP+ integration.

---

## 🎯 Network Design Philosophy

**You are NOT the network administrator** - This homelab operates as a **complete DMZ** within an existing home network you don't control. All routing, VLANs, and security are managed within your 10.0.0.0/8 space.

---

## 📊 Network Overview

```
Internet (50/15 Mbps)
    ↓
ISP Router → Eero Mesh (192.168.4.0/22) ← Home Network (NO ACCESS)
                    ↓ (single cable)
            ┌───────────────────┐
            │   pfSense N150    │ ← Your DMZ Gateway
            │ WAN: 192.168.4.x  │
            │ LAN: 10.0.0.0/8   │
            └───────────────────┘
                    ↓ 2.5GbE
        ┌───────────────────────┐
        │ MikroTik CRS310       │
        │ 8x 2.5GbE + 2x SFP+  │
        └───────────────────────┘
         │   │   │   │   │  │  └─── 10GbE SFP+1
         │   │   │   │   │  │           ↓
    ┌────┴───┴───┴───┴───┴──┴──┐   ┌────────────┐
    │                           │   │ Dell Tower │
Node 1-3   PBS   TrueNAS   Pi  AP  │  (Node 4)  │
(2.5GbE) (2.5G) (2.5GbE) (PoE) (5G)│  10GbE SFP+│
    │                           │   └────────────┘
    └───────────────────────────┘
    Proxmox Cluster (4 nodes)
  + QDevice on Pi (5th voter)
```

**Key Network Features:**
- **Heterogeneous Cluster:** Mixed 2.5GbE and 10GbE nodes
- **Dell Performance Node:** 10GbE SFP+ for heavy workloads (VMs with GUI, CTF labs, Windows)
- **6-VLAN Segregation:** Management, Storage, Services, Trusted LAN, IoT, DMZ
- **QDevice on Pi:** Ensures 5-vote quorum for maximum availability

---

## 🏷️ VLAN Design (All within 10.0.0.0/8)

| VLAN ID | Name | Subnet | Gateway | Purpose | Security Level |
|---------|------|--------|---------|---------|----------------|
| 1 | Native/Unused | - | - | Disabled for security | - |
| 10 | Management | 10.0.10.0/24 | 10.0.10.1 | Proxmox, Switch, IPMI, BMC - **VPN access only** | 🔴 Critical |
| 20 | Storage/SAN | 10.0.20.0/24 | 10.0.20.1 | NFS, SMB, iSCSI - Specialized 2.5GbE/10GbE traffic | 🟡 Isolated |
| 30 | Services | 10.0.30.0/24 | 10.0.30.1 | VMs, Containers, Kubernetes, Web Services | 🟢 Standard |
| 40 | Trusted LAN | 10.0.40.0/24 | 10.0.40.1 | Your laptop, phone, homelab-5G WiFi | 🟢 Standard |
| 50 | IoT/Untrusted | 10.0.50.0/24 | 10.0.50.1 | Guest WiFi, Smart devices, Testing, CTF Labs | 🟠 Restricted |
| 666 | DMZ | 10.0.66.0/24 | 10.0.66.1 | Tor node, Reverse proxy, Public-facing services | 🔴 Exposed |

---

## 🖥️ IP Address Allocation

### Management VLAN (10.0.10.0/24) - **VPN Access Only**

**Purpose:** Critical infrastructure management - Proxmox cluster, switches, IPMI, BMC

**Security:** 🔴 Critical - Only accessible via WireGuard VPN

| Device | IP | Hostname | Notes |
|--------|----|----- |-------|
| pfSense | 10.0.10.1 | fw.homelab.local | Gateway for all VLANs |
| MikroTik Switch | 10.0.10.2 | switch.homelab.local | Switch management (WinBox) |
| Proxmox Node 1 | 10.0.10.11 | pve-node1.homelab.local | i5-7400T (ThinkCentre) - 2.5GbE |
| Proxmox Node 2 | 10.0.10.12 | pve-node2.homelab.local | i5-7500T (ThinkCentre) - 2.5GbE |
| Proxmox Node 3 | 10.0.10.13 | pve-node3.homelab.local | i5-7500T (ThinkCentre) - 2.5GbE |
| **Proxmox Node 4** | **10.0.10.14** | **pve-node4.homelab.local** | **i7-7700 (Dell Tower) - 10GbE SFP+** |
| PBS Node | 10.0.10.20 | pbs.homelab.local | AMD A10 (NOT in cluster) - 2.5GbE |
| TrueNAS | 10.0.10.25 | truenas.homelab.local | NAS management interface - 2.5GbE |
| Raspberry Pi | 10.0.10.30 | pi-jump.homelab.local | QDevice, Jump host, UPS monitor |
| Netgear AP | 10.0.10.40 | ap.homelab.local | homelab-5G wireless AP |
| BMC/IPMI | 10.0.10.50-60 | - | Out-of-band management (if available) |

### Storage/SAN VLAN (10.0.20.0/24) - **High-Speed Backbone**

**Purpose:** Dedicated storage traffic - NFS, iSCSI, SMB, ZFS replication

**Security:** 🟡 Isolated - High-bandwidth, low-latency data plane

| Device | IP | Hostname | Notes |
|--------|----|----- |-------|
| Gateway | 10.0.20.1 | - | pfSense interface |
| TrueNAS Storage | 10.0.20.10 | storage.homelab.local | NFSv4/SMB data interface - 2.5GbE |
| Proxmox Node 1 | 10.0.20.11 | pve-node1-storage | Storage interface - 2.5GbE |
| Proxmox Node 2 | 10.0.20.12 | pve-node2-storage | Storage interface - 2.5GbE |
| Proxmox Node 3 | 10.0.20.13 | pve-node3-storage | Storage interface - 2.5GbE |
| **Proxmox Node 4** | **10.0.20.14** | **pve-node4-storage** | **10GbE SFP+ interface (primary storage path)** |
| PBS Storage | 10.0.20.20 | pbs-storage.homelab.local | Backup server storage - 2.5GbE |

### Services VLAN (10.0.30.0/24)

**Purpose:** Internal services, VMs, containers, Kubernetes cluster

**Security:** Standard internal access, no direct internet exposure

| Device | IP | Hostname | Notes |
|--------|----|----- |-------|
| Gateway | 10.0.30.1 | - | pfSense interface |
| Pi-Hole Primary | 10.0.30.10 | dns1.homelab.local | Primary DNS (VM on Proxmox) |
| Pi-Hole Secondary | 10.0.30.11 | dns2.homelab.local | Backup DNS (VM or on Pi) |
| Unbound DNS | 10.0.30.12 | unbound.homelab.local | Recursive DNS resolver |
| Jellyfin | 10.0.30.20 | jellyfin.homelab.local | Media server (Dell node) |
| Nginx Proxy | 10.0.30.25 | proxy.homelab.local | Internal reverse proxy |
| Authentik | 10.0.30.30 | auth.homelab.local | SSO/Identity management |
| Kubernetes Master | 10.0.30.40-42 | k8s-master-{1,2,3} | K8s control plane |
| **Internal Services** | 10.0.30.50-100 | - | **Non-public services, dev/staging** |

### Trusted LAN VLAN (10.0.40.0/24)

**Purpose:** Your personal devices, homelab WiFi clients

| Device | IP | Hostname | Notes |
|--------|----|----- |-------|
| Gateway | 10.0.40.1 | - | pfSense interface |
| DHCP Pool | 10.0.40.100-200 | - | Your laptop, phone, devices on homelab-5G WiFi |
| Static Devices | 10.0.40.10-50 | - | Reserved for static assignments |

### IoT/Untrusted VLAN (10.0.50.0/24)

**Purpose:** Guest WiFi, IoT devices, experimental/testing VMs

**Security:** Heavily restricted, internet-only access

| Device | IP | Hostname | Notes |
|--------|----|----- |-------|
| Gateway | 10.0.50.1 | - | pfSense interface |
| DHCP Pool | 10.0.50.100-200 | - | IoT devices, Guest WiFi |
| Testing | 10.0.50.10-50 | - | Experimental VMs/containers |
| **CTF Labs** | 10.0.50.60-80 | - | **Isolated Capture The Flag environments** |
| **Kali Linux VMs** | 10.0.50.81-90 | - | **Pentesting VMs (isolated from production)** |

### DMZ VLAN (10.0.66.0/24) - **Public-Facing**

**Purpose:** Internet-exposed services, reverse proxy ingress

**Security:** Maximum isolation, only allowed outbound to internet

| Device | IP | Hostname | Notes |
|--------|----|----- |-------|
| Gateway | 10.0.66.1 | - | pfSense interface |
| HAProxy | 10.0.66.10 | haproxy.homelab.local | **Reverse proxy for daniellaurin.dev** |
| **WordPress (daniellaurin.dev)** | 10.0.66.20 | **www.daniellaurin.dev** | **Your personal website (public)** |
| **WordPress (Other Domain)** | 10.0.66.21 | **www.otherdomain.com** | **Second WordPress site (public)** |
| Tor Relay | 10.0.66.30 | tor.homelab.local | Tor middle relay |
| Public Services | 10.0.66.40-100 | - | Other exposed services |

---

## 🔧 MikroTik Switch Configuration

### Port Allocation (8 Ethernet + 2 SFP+)

| Port | VLAN | Connected To | Speed | Notes |
|------|------|--------------|-------|-------|
| ether1 | Trunk (All VLANs) | → pfSense igb1 | 2.5GbE | Tagged 10,20,30,40,50,666 |
| ether2 | 10,20 (Tagged) | → Proxmox Node 1 | 2.5GbE | Intel i5-7400T (Mgmt + Storage) |
| ether3 | 10,20 (Tagged) | → Proxmox Node 2 | 2.5GbE | Intel i5-7500T (Mgmt + Storage) |
| ether4 | 10,20 (Tagged) | → Proxmox Node 3 | 2.5GbE | Intel i5-7500T (Mgmt + Storage) |
| ether5 | 10,20 (Tagged) | → PBS Node (AMD) | 2.5GbE | Mgmt + Backup traffic |
| ether6 | 10,20 (Tagged) | → TrueNAS | 2.5GbE | Mgmt + Storage |
| ether7 | 10 (Management) | → Raspberry Pi (PoE) | 2.5GbE | Via PoE+ injector, QDevice |
| ether8 | Trunk (All VLANs) | → Netgear AC1750 AP | 2.5GbE | homelab-5G WiFi |
| **sfp-sfpplus1** | **10,20,30,666** | **→ Dell Node 4** | **10GbE** | **Performance node uplink** |
| sfp-sfpplus2 | - | Reserved | 10GbE | Future 10GbE expansion |

**Notes:**
- **Dell Node 4** uses **10GbE SFP+** for high-bandwidth workloads:
  - Windows VMs (VDI, Active Directory)
  - Kali Linux pentesting VMs
  - CTF lab environments
  - Jellyfin with GPU transcoding
  - High IOPS storage access (1TB NVMe)
- **Intel Nodes 1-3** use **2.5GbE** for standard workloads and ZFS replication
- All nodes have **dual interfaces** (Management VLAN 10 + Storage VLAN 20)

### Initial Setup
```routeros
# Access via WinBox or WebFig
# Default IP: 192.168.88.1
# Change to: 10.0.10.2

/ip address
add address=10.0.10.2/24 interface=bridge

# Update admin password
/user set admin password=<strong_password>
```

### VLAN Configuration

```routeros
# Create bridge for all ports
/interface bridge
add name=bridge1 vlan-filtering=yes

# Add all ports to bridge (including SFP+ for Dell)
/interface bridge port
add bridge=bridge1 interface=ether1 pvid=1          # Trunk to pfSense
add bridge=bridge1 interface=ether2 pvid=10         # Node 1 (Mgmt default)
add bridge=bridge1 interface=ether3 pvid=10         # Node 2 (Mgmt default)
add bridge=bridge1 interface=ether4 pvid=10         # Node 3 (Mgmt default)
add bridge=bridge1 interface=ether5 pvid=10         # PBS Node (Mgmt default)
add bridge=bridge1 interface=ether6 pvid=10         # TrueNAS (Mgmt default)
add bridge=bridge1 interface=ether7 pvid=10         # Raspberry Pi
add bridge=bridge1 interface=ether8 pvid=1          # Trunk to AP
add bridge=bridge1 interface=sfp-sfpplus1 pvid=20   # Dell Node 4 (10GbE - Storage default)

# Create VLANs
/interface vlan
add interface=bridge1 name=vlan10-mgmt vlan-id=10
add interface=bridge1 name=vlan20-storage vlan-id=20
add interface=bridge1 name=vlan30-services vlan-id=30
add interface=bridge1 name=vlan40-trusted vlan-id=40
add interface=bridge1 name=vlan50-iot vlan-id=50
add interface=bridge1 name=vlan666-dmz vlan-id=666

# Tag VLANs on bridge ports
# VLAN 10 (Management) - VPN access only
/interface bridge vlan
add bridge=bridge1 tagged=ether1,ether2,ether3,ether4,ether5,ether6,sfp-sfpplus1 untagged=ether7 vlan-ids=10

# VLAN 20 (Storage/SAN) - High-speed backbone
add bridge=bridge1 tagged=ether1,ether2,ether3,ether4,ether5,ether6,sfp-sfpplus1 vlan-ids=20

# VLAN 30 (Services) - VMs and containers
add bridge=bridge1 tagged=ether1,ether8,sfp-sfpplus1 vlan-ids=30

# VLAN 40 (Trusted LAN) - User devices
add bridge=bridge1 tagged=ether1,ether8 vlan-ids=40

# VLAN 50 (IoT/Untrusted) - Guest WiFi, testing
add bridge=bridge1 tagged=ether1,ether8 vlan-ids=50

# VLAN 666 (DMZ) - Public-facing services
add bridge=bridge1 tagged=ether1,ether8,sfp-sfpplus1 vlan-ids=666

# Enable VLAN filtering
/interface bridge
set bridge1 vlan-filtering=yes
```

### Additional MikroTik Optimizations for 10GbE

```routeros
# Set jumbo frames on 10GbE port (if your Dell NIC supports it)
/interface ethernet
set sfp-sfpplus1 mtu=9000

# Enable flow control for better throughput
set sfp-sfpplus1 auto-negotiation=no speed=10Gbps full-duplex=yes

# Priority queue for Storage VLAN (VLAN 20)
/interface bridge port
set [find interface=sfp-sfpplus1] priority=0x7

# IGMP Snooping for multicast (Jellyfin streaming)
/interface bridge
set bridge1 igmp-snooping=yes

# Port mirroring (optional - for Pi network analysis)
/interface ethernet switch
set 0 mirror-source=sfp-sfpplus1 mirror-target=ether7
```

---

## 🚀 Dell Node 4 Network Configuration

### Hardware Overview
- **Model:** Dell Precision 3620 Tower
- **CPU:** Intel i7-7700 @ 3.6GHz (4C/8T)
- **RAM:** 32GB DDR4
- **Storage:** 1TB NVMe + 500GB HDD
- **NIC:** 10GbE SFP+ (likely Intel X520 or X540)
- **Connection:** SFP+ DAC cable or fiber to MikroTik SFP+1

### Proxmox Network Interfaces

```bash
# /etc/network/interfaces on pve-node4

auto lo
iface lo inet loopback

# 10GbE SFP+ Interface (primary)
auto enp1s0
iface enp1s0 inet manual

# Management VLAN (10.0.10.14)
auto vmbr0
iface vmbr0 inet static
    address 10.0.10.14/24
    gateway 10.0.10.1
    bridge-ports enp1s0.10
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes

# Storage VLAN (10.0.20.14) - Primary data path
auto vmbr1
iface vmbr1 inet static
    address 10.0.20.14/24
    bridge-ports enp1s0.20
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    # Jumbo frames for better throughput
    mtu 9000

# Services VLAN (for VMs)
auto vmbr2
iface vmbr2 inet manual
    bridge-ports enp1s0.30
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes

# DMZ VLAN (for public-facing VMs like WordPress)
auto vmbr3
iface vmbr3 inet manual
    bridge-ports enp1s0.666
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
```

### Performance Tuning

```bash
# Enable jumbo frames on 10GbE interface
ip link set enp1s0 mtu 9000
ip link set vmbr1 mtu 9000

# Increase ring buffers (check max with: ethtool -g enp1s0)
ethtool -G enp1s0 rx 4096 tx 4096

# Enable interrupt coalescing
ethtool -C enp1s0 adaptive-rx on adaptive-tx on

# Check link status
ethtool enp1s0 | grep Speed
# Should show: Speed: 10000Mb/s

# Test throughput to TrueNAS
iperf3 -c 10.0.20.10 -P 4 -t 30
# Should see ~9+ Gbps with jumbo frames
```

### VM Configuration for Dell Node

**Windows Server 2022 (Active Directory):**
```bash
# VM Config
vmbr0 (Management VLAN 10)
- IP: 10.0.10.100
- 4 vCPU, 8GB RAM
- VirtIO NIC for best performance
```

**Windows 11 VDI:**
```bash
# VM Config
vmbr0 (Management VLAN 10)
- IP: 10.0.10.101
- 4 vCPU, 8GB RAM
- GPU passthrough (Intel HD 630)
```

**Kali Linux (Pentesting):**
```bash
# VM Config
vmbr2 (Services VLAN 30) - Could also use IoT VLAN 50 for isolation
- IP: 10.0.50.81 (IoT VLAN for safety)
- 2 vCPU, 4GB RAM
- ISOLATED from production data
```

**WordPress (daniellaurin.dev):**
```bash
# VM Config
vmbr3 (DMZ VLAN 666)
- IP: 10.0.66.20
- 2 vCPU, 2GB RAM
- No direct access to internal networks
```

---

## 🌐 Website Hosting Strategy

### Your Domains & Placement

**1. daniellaurin.dev (Personal Website)**
- **VLAN:** 666 (DMZ) - Public-facing
- **IP:** 10.0.66.20
- **Platform:** WordPress in LXC/Docker on Dell Node
- **Access:** HAProxy reverse proxy (SSL termination)
- **Why DMZ?** This is internet-facing, so it should be isolated from your homelab infrastructure

**2. Second WordPress Domain**
- **VLAN:** 666 (DMZ) - Public-facing
- **IP:** 10.0.66.21
- **Platform:** WordPress in LXC/Docker on Dell Node
- **Access:** HAProxy reverse proxy (different domain)

**3. Development/Staging Sites**
- **VLAN:** 30 (Services) - Internal only
- **IP:** 10.0.30.60-80
- **Purpose:** Test changes before pushing to production in DMZ

### Recommended Setup

```
Internet → Port 443
    ↓
pfSense WAN (192.168.4.x)
    ↓
Port Forward: 443 → 10.0.66.10 (HAProxy)
    ↓
HAProxy SSL Termination
    ├─→ daniellaurin.dev → 10.0.66.20 (WordPress 1)
    └─→ otherdomain.com → 10.0.66.21 (WordPress 2)
```

### Using daniellaurin.dev for Homelab Services

You can also use subdomains for internal services:

```
daniellaurin.dev           → WordPress (DMZ, public)
lab.daniellaurin.dev       → Homelab dashboard (Services VLAN, VPN-only)
plex.daniellaurin.dev      → Jellyfin (Services VLAN, VPN-only)
vpn.daniellaurin.dev       → WireGuard endpoint (pfSense WAN)
```

**DNS Strategy:**
- **Public DNS (Cloudflare):** Only daniellaurin.dev and otherdomain.com point to your public IP
- **Internal DNS (Pi-Hole):** lab.*, plex.*, etc. resolve to internal IPs (10.0.x.x)
- **Split-horizon DNS:** Same domain works differently inside vs outside your network

---

## 📈 Network Performance Expectations

### Internet Bandwidth
```bash
Download: 50 Mbps (6.25 MB/s max)
Upload: 15 Mbps (1.875 MB/s max)

# This is shared with home network!
# Be mindful of bandwidth usage
```

### Internal Network
```bash
# 2.5GbE links (Intel nodes)
Node → Node: ~2.5 Gbps (312 MB/s)
Node → Storage: ~2.5 Gbps (312 MB/s)

# 10GbE link (Dell node)
Dell → Storage: ~10 Gbps (1250 MB/s) theoretical
Dell → Storage: ~9 Gbps (1125 MB/s) realistic with overhead
Dell → Storage: ~9+ Gbps with jumbo frames enabled

# Storage (RAID 5 limited)
TrueNAS Read: ~200-300 MB/s (bottleneck)
TrueNAS Write: ~150-200 MB/s (bottleneck)

# Note: Dell's 10GbE won't fully saturate due to TrueNAS RAID 5 limits
# But provides future headroom and better concurrent access
```

### Latency
```bash
Intra-VLAN: <1ms
Cross-VLAN: <2ms (via pfSense)
Dell 10GbE: <0.5ms (lower latency than 2.5GbE)
```

---

## 🔗 References

- [MikroTik VLAN Guide](https://wiki.mikrotik.com/wiki/Manual:Interface/VLAN)
- [pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)
- [Proxmox Network Configuration](https://pve.proxmox.com/wiki/Network_Configuration)
- [10GbE Tuning Guide](https://fasterdata.es.net/host-tuning/linux/)

---

**Last Updated:** [DATE]  
**Network uptime:** [DAYS]
