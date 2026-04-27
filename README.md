# 🏠 My Homelab Journey

<div align="center">
  
![pfSense](https://img.shields.io/badge/pfSense-212121?style=for-the-badge&logo=pfsense&logoColor=white)
![Proxmox](https://img.shields.io/badge/Proxmox-E57000?style=for-the-badge&logo=proxmox&logoColor=white)
![TrueNAS](https://img.shields.io/badge/TrueNAS-0095D5?style=for-the-badge&logo=truenas&logoColor=white)
![MikroTik](https://img.shields.io/badge/MikroTik-293239?style=for-the-badge&logo=mikrotik&logoColor=white)

</div>

---

## 📖 My Story

This is my second homelab a direct upgrade from a single Dell Tower running Debian + CasaOS. While the original setup was capable (especially with 32GB DDR4 RAM), I faced critical issues with data redundancy, backups, and storage. I had filled the 1TB NVMe completely, storing all games for potential re-downloads. This made it impossible to use Timeshift for backups, forcing me to rely on Proton Drive, which defeated the entire purpose of having a local server.

It was time for an upgrade. This time, I wanted a dedicated NAS to host game downloads, media files, and run a full-fledged Jellyfin server (with arr stack) . I also had a Raspberry Pi 5 and Lenovo ThinkCentre minis collecting dust, unused because I only had two Ethernet ports available from the home network. I needed a proper server rack with networking infrastructure a complete DMZ within my home network.

### 🚨 Software Limitations

On the software side, CasaOS showed serious limitations. While it's pretty, free, and a great gateway to IT without paying premium prices for commercial NAS solutions, it had issues. CasaOS excels at spinning up known Docker containers through its app menu, but experimenting with custom containers and practicing DNS/Bash scripting/Apache caused frequent breakage requiring rebuilds. CasaOS broke my Nextcloud installation four separate times after updates, forcing me to export and migrate to new containers each time. I eventually started using Portainer as my primary dashboard, not a good sign.

### 💥 The Breaking Point

I encountered trouble hosting Steam-CLI for my games, so I installed a GUI (KDE) to run Steam directly. This caused massive issues:
- My NVS310 GPU stopped working
- Steam couldn't use GPU acceleration properly
- KDE uses Network Manager while Debian Server uses ifupdown, creating conflicts
- Containers lost their default gateway configuration and couldn't connect to the internet

Every time I updated and rebooted, I had to physically run these commands:

```bash
nmcli device show enp0s31f6
sudo ip route flush dev vethe90e8b0
```

This was incredibly frustrating since I couldn't SSH in to fix it remotely. Time for a new journey with my upgraded homelab!

---

## 🎯 Core Architecture: Heterogeneous 4-Node Cluster

This homelab uses a **unified heterogeneous cluster** approach:

### The Concept

- **4 Compute Nodes** in a single Proxmox cluster:
  - 3x Intel ThinkCentre Tiny (2.5GbE, 500GB SSD each)
  - 1x Dell Precision Tower (10GbE SFP+, 1TB NVMe + 500GB HDD)
- **Raspberry Pi 5** as Corosync QDevice (5th voter for quorum)
- **Single pane of glass** management at `https://10.0.10.11:8006`

### Voting Logic (Quorum)
- 3 Tiny Nodes + 1 Dell = **4 Votes**
- Raspberry Pi QDevice = **1 Vote**
- **Total = 5 Votes** (can lose any 2 nodes and cluster stays alive)

### Storage Strategy
- **Intel Nodes (1-3):** Run VMs on local 500GB SSDs with ZFS replication between themselves
- **Dell Node (4):** Runs VMs on 1TB NVMe (no replication to avoid filling Tiny nodes)
- **Shared Storage:** All 4 nodes mount TrueNAS NFS for ISOs, backups, and media
- **PBS:** Separate AMD node (not in cluster) for deduplicated backups

---

## 🎯 Core Technology Stack

This homelab is built on four pillars:

- **🔥 pfSense** - Network security, routing, and VPN (DMZ gateway)
- **🦄 MikroTik RouterOS** - Managed switching with 2.5GbE + 10GbE SFP+
- **☁️ Proxmox VE** - 4-node heterogeneous cluster with HA
- **💾 TrueNAS** - ZFS-based centralized storage with data redundancy

---

## 📊 Network Architecture

### Network Topology Overview

```
Internet (50/15 Mbps)
    ↓
Home Router (Not Admin)
    ↓
Eero Mesh Network (192.168.4.0/22)
    ↓
    └─── pfSense Firewall WAN (single uplink)
             ↓
        [HOMELAB DMZ - 10.0.0.0/8]
             ↓
        MikroTik Switch (2.5GbE + 10GbE SFP+)
             ↓
        ┌────┴────┬────────┬─────────┬────────┬───────────┐
        │         │        │         │        │           │
    Node 1-3  Dell Node  TrueNAS   PBS   Raspberry Pi  Netgear AP
    (2.5GbE)  (10GbE)   (2.5GbE) (2.5GbE)  (2.5GbE)   (homelab-5G)
        │         │        │         │        │
        └─────────┴────────┴─────────┴────────┘
              Proxmox Cluster (4 nodes)
           + QDevice on Pi (5th voter)
```

### VLAN Design Philosophy

**6 VLANs for Complete Network Segregation:**

| VLAN | Name | Purpose | Security Level |
|------|------|---------|----------------|
| 10 | Management | Proxmox, Switch, IPMI, BMC - VPN access only | 🔴 Critical |
| 20 | Storage/SAN | NFS, SMB, - 2.5GbE/10GbE backbone | 🟡 Isolated |
| 30 | Services | VMs, Containers, Kubernetes, Web Services | 🟢 Standard |
| 40 | Trusted LAN | laptop, phone, gaming-pc  homelab-5G WiFi | 🟢 Standard |
| 50 | IoT/Untrusted | homelab-guest, Smart devices, Testing | 🟠 Restricted |
| 666 | DMZ | Tor node, Reverse proxy, Public-facing services | 🔴 Exposed |

---

## 🏗️ Physical Rack Layout

```
┌─────────────────────────────────────────┐
│  Eero Mesh Pod (WAN)                    │
│  Uplink to home network                 │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  N150 Firewall (on top of rack)         │
│  4x 2.5GbE, 8GB DDR4, 128GB NVMe        │
│  WAN → 192.168.4.x (home network)       │
│  LAN → 10.0.0.0/8 (homelab DMZ)         │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  Raspberry Pi 5 (on top of rack)        │
│  8GB RAM, 512GB NVMe, PoE+              │
│  + 7" Touchscreen attached              │
│  Jump Host + QDevice, UPS monitor       │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  Netgear AC1750 AP 1GbE (homelab-5G)    │  
│  VLAN-aware AP for DMZ wireless         │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  Venting Panel                          │  U12
├─────────────────────────────────────────┤
│  MikroTik CRS310-8G+2S+IN               │  U11
│  8x 2.5GbE + 2x SFP+ (10GbE)            │
├─────────────────────────────────────────┤
│  12-Port Patch Panel                    │  U10
├─────────────────────────────────────────┤
│  ThinkCentre M715Q (PBS Node)           │  U05
│  AMD A10-9700E, 16GB RAM, 2TB (2.5GbE)  │ 
├─────────────────────────────────────────┤
│  ThinkCentre M710Q (Node 3)             │  U08
│  i5-7500T, 16GB, 500GB SSD (2.5GbE)     │ 
├─────────────────────────────────────────┤
│  ThinkCentre M710Q (Node 2)             │  U07
│  i5-7500T, 16GB, 500GB SSD (2.5GbE)     │
├─────────────────────────────────────────┤
│  ThinkCentre M710Q (Node 1)             │  U06
│  i5-7400T, 16GB, 500GB SSD (2.5GbE)     │
├─────────────────────────────────────────┤
│                                         │  U05
│  Jonsbo N2 NAS                          │  U04
│  N5105, 16GB RAM, TrueNAS               │  U03
│  5x 1TB HDD (RAID 5)                    │  U02
│  1TB NVMe Cache, 650W PSU               │  U01
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  CyberPower UPS 1500VA (Next to Rack)   │
└─────────────────────────────────────────┘

╔═══════════════════════════════════╗
║  Dell Precision 3620 Tower        ║
║  (Node 4 - Next to Rack)          ║
║  ════════════════════════════════ ║
║  CPU: i7-7700 @ 3.6GHz (4C/8T)    ║
║  RAM: 32GB DDR4                   ║
║  SSD: 1TB NVMe (VMs)              ║
║  HDD: 500GB (Steam cache)         ║
║  NIC: 10GbE SFP+                  ║
║  GPU: Intel HD 630 (iGPU)         ║
╚═══════════════════════════════════╝
```

## ✅ Requirements & Features

### Core Requirements
- ✅ **4-node heterogeneous cluster** with HA (3 Intel + 1 Dell)
- ✅ **10GbE SFP+ uplink** for Dell performance node
- ✅ **Corosync QDevice** on Raspberry Pi (5th voter for quorum)
- ✅ **ZFS replication** between Intel nodes (500GB SSDs)
- ✅ **RAID 5 storage** on TrueNAS
- ✅ **6 VLAN design** for complete network segregation
- ✅ **Jump server** via Raspberry Pi + VPN
- ✅ **DMZ network** (10.0.0.0/8 isolated from home 192.168.4.0/22)
- ✅ **Wireless AP** for homelab (Netgear AC1750)
- ✅ **UPS with NUT** for graceful shutdowns

### Network Constraints
- ❌ **No admin access to main router** - Single uplink to home network
- ✅ **Port forwarding** - Coordinated with network admin
- ⚠️ **Limited bandwidth** - 50 Mbps down / 15 Mbps up (shared)

---

## 🛠️ Software Stack by Layer

### 🖧 Physical Network & Security Layer

| Component | OS/Platform | Key Services |
|-----------|-------------|--------------|
| **N150 Firewall** | pfSense CE | WireGuard VPN, HAProxy, pfBlockerNG, CrowdSec, ntopng, FreeRADIUS, Avahi |
| **MikroTik CRS310** | RouterOS v7 | L3 HW offload, ACLs, Port mirroring, IGMP snooping, 10GbE SFP+ |
| **Raspberry Pi 5** | Ubuntu 24.04 | **Corosync QDevice**, MeshCentral, Tailscale, NUT Master, Uptime Kuma, Apt-Cacher-NG |

### ⚙️ Compute Layer (4-Node Cluster)

| Node | Connectivity | Role | Services |
|------|--------------|------|----------|
| **ThinkCentre Nodes 1-3** | 2.5GbE | HA Group 1 (Standard) | Proxmox VE 8.2, Corosync, QEMU/KVM, LXC, ZFS replication, NUT client |
| **Dell Tower (Node 4)** | **10GbE SFP+** | HA Group 2 (Performance) | Proxmox VE 8.2, Heavy VMs with GUI, GPU passthrough (Intel HD 630 iGPU), 1TB NVMe for high IOPS |

### 💾 Storage & Backup Layer

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| **TrueNAS (Jonsbo N2)** | Centralized storage | NFSv4 (VM storage), SMB (media), Rclone (Proton), ZFS scrub |
| **PBS (AMD ThinkCentre)** | Backup server | Deduplication, garbage collection, 1TB NVMe datastore |

### ☁️ Virtual Workloads (Optimized Placement)

**Intel Nodes (HA Group 1):**
- LXC-Mgmt-DevOps (Ansible, Terraform, n8n, GitLab Runner)
- LXC-Identity-Proxy (Authentik, Nginx Proxy Manager, Vaultwarden, CrowdSec)
- LXC-Net-Services (Pi-Hole, Unbound, SearXNG, Uptime Kuma)
- VM-K8s-Cluster (3x Talos nodes - LGTM stack, Prometheus, Cert-Manager)

**Dell Node (HA Group 2 - High Performance):**
- **LXC-Media-Stack** (Jellyfin w/ iGPU passthrough, Gluetun, qBittorrent, *arr suite)
- **VM-Active-Directory** (Windows Server 2022, AD DS, DNS/DHCP, LDAP testing)
- **VM-Win11-VDI** (Windows 11 Pro, RustDesk, general desktop usage)
- **VM-Kali-Linux** (Pentesting lab, isolated in IoT VLAN for safety)
- **VM-CTF-Labs** (Capture The Flag practice environments, HackTheBox VMs)
- **LXC-Minecraft** (Java server using high clock speed i7-7700)

---

## 🔒 3-2-1 Backup Strategy

### Backup Architecture

**3 Copies of Data:**
1. **Production** - TrueNAS (ZFS RAID 5)
2. **Backup #1** - Proxmox Backup Server (encrypted, deduplicated)
3. **Backup #2** - Proton Drive (offsite, encrypted via Rclone)

**2 Different Storage Types:**
- **Onsite #1:** TrueNAS (ZFS on HDDs)
- **Onsite #2:** PBS (ext4 on NVMe)

**1 Offsite Copy:**
- **Proton Drive** (cloud storage, encrypted)

### Backup Flow
```
TrueNAS (Production) 
    ├─→ ZFS Snapshots (hourly/daily)
    ├─→ VM Storage (NFS) → Proxmox → PBS (nightly backups)
    └─→ Cloud Sync → Proton Drive (weekly, encrypted)

Intel Nodes (1-3)
    └─→ ZFS Replication (between nodes for HA)
```

---

## 📚 Configuration & Setup Guides

### Ansible Automation Strategy

**Manual Setup (Phases 1-3):** Learn the commands hands-on
- Phase 1: Physical rack assembly
- Phase 2: Network bootstrap (pfSense, MikroTik, VLANs)
- Phase 3: Proxmox installation on all 4 nodes

**Ansible Automation (Phase 4+):** Maintain consistency
- **Phase 4:** Clustering (4-node cluster + QDevice setup)
- **Phase 5:** Storage (TrueNAS shares, NFS mounts)
- **Phase 6:** Services (Deploy VMs/LXCs, Kubernetes)
- **Ongoing:** Updates, monitoring, configuration drift prevention

### Documentation Files

- [Hardware Specifications](./HARDWARE.md) - Complete BOM with pricing
- [Initial Setup Guide](./docs/SETUP.md) - Step-by-step installation
- [Network Configuration](./docs/NETWORK.md) - 6-VLAN design, pfSense rules
- [Proxmox Cluster Setup](./docs/PROXMOX.md) - 4-node heterogeneous cluster + QDevice
- [TrueNAS Configuration](./docs/TRUENAS.md) - ZFS pools, shares, cloud sync
- [Monitoring Setup](./docs/MONITORING.md) - LGTM stack, UPS monitoring
- [pfSense Configuration](./docs/PFSENSE.md) - Complete firewall setup
- [Ansible Playbooks](./ansible/) - Automation for Phase 4+ configuration

---

## 💰 Total Investment

Building a homelab that balances performance, expandability, and power efficiency (I was not budget conscious lmao)

**Total Hardware Cost:** See [HARDWARE.md](./HARDWARE.md) for complete breakdown

---

## 🔗 Resources

- [r/homelab](https://reddit.com/r/homelab) - Community for homelab enthusiasts
- [r/minilab](https://reddit.com/r/minilab) - Inspiration for compact homelab builds
- [Proxmox Documentation](https://pve.proxmox.com/wiki/Main_Page)
- [TrueNAS Documentation](https://www.truenas.com/docs/)
- [pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)
- [MikroTik Wiki](https://wiki.mikrotik.com/)

---

## 📝 License

This documentation is shared for educational purposes. Feel free to use and adapt for your own homelab!
