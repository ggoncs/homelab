# 🏠 My Homelab Journey

<div align="center">
  
![pfSense](https://img.shields.io/badge/pfSense-212121?style=for-the-badge&logo=pfsense&logoColor=white)
![Proxmox](https://img.shields.io/badge/Proxmox-E57000?style=for-the-badge&logo=proxmox&logoColor=white)
![TrueNAS](https://img.shields.io/badge/TrueNAS-0095D5?style=for-the-badge&logo=truenas&logoColor=white)
![MikroTik](https://img.shields.io/badge/MikroTik-293239?style=for-the-badge&logo=mikrotik&logoColor=white)

</div>

---

## 📖 My Story

This is my second homelab—a direct upgrade from a single Dell Tower running Debian + CasaOS. While the original setup was capable (especially with 32GB DDR4 RAM), I faced critical issues with data redundancy, backups, and storage. I had filled the 1TB NVMe completely, storing all my games for potential re-downloads. This made it impossible to use Timeshift for backups, forcing me to rely on Proton Drive, which defeated the entire purpose of having a local server.

It was time for an upgrade. This time, I wanted a dedicated NAS to host game downloads, media files, and run a full-fledged Plex server. I also had a Raspberry Pi 5 and Lenovo ThinkCentre mini collecting dust—unused because I only had two Ethernet ports available in the basement from my Eero mesh (one for Xbox, one for the Dell tower). I needed a proper server rack with networking infrastructure.

### 🚨 Software Limitations

On the software side, CasaOS showed serious limitations. While it's pretty, free, and a great gateway to IT without paying premium prices for commercial NAS solutions, it had issues. CasaOS excels at spinning up known Docker containers through its app menu, but experimenting with custom containers and practicing DNS/Bash scripting/Apache caused frequent breakage requiring rebuilds. CasaOS broke my Nextcloud installation four separate times after updates, forcing me to export and migrate to new containers each time. I eventually started using Portainer as my primary dashboard—not a good sign.

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

## 🎯 Core Technology Stack

This homelab is built on four pillars:

- **🔥 pfSense** - Network security and routing
- **🔧 MikroTik RouterOS** - Managed switching with 2.5GbE
- **☁️ Proxmox VE** - Virtualization and cluster management
- **💾 TrueNAS** - ZFS-based storage with data redundancy

---

## 📊 Network Diagrams

### Logical Network Diagram
*[Diagram Coming Soon]*

### Physical Rack Layout

```
┌─────────────────────────────────────────┐
│  N150 Firewall (on top of rack)        │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  7" Screen + Eero Mesh Pod (on top)     │  0U
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  Venting Panel                          │  U12
├─────────────────────────────────────────┤
│  MikroTik CRS310-8G+2S+IN              │  U11
│  8x 2.5GbE + 2x SFP+                   │
├─────────────────────────────────────────┤
│  12-Port Patch Panel                    │  U10
├─────────────────────────────────────────┤
│  Raspberry Pi 5 (with 7" screen)       │  U09
│  8GB RAM, 512GB NVMe, PoE+             │
├─────────────────────────────────────────┤
│  ThinkCentre M710Q (Node 4)            │  U08
│  i5-7500T, 16GB RAM, 500GB SSD         │
├─────────────────────────────────────────┤
│  ThinkCentre M715Q (Node 3)            │  U07
│  AMD A10-9700E, 16GB RAM, 128GB+1TB    │
├─────────────────────────────────────────┤
│  ThinkCentre M710Q (Node 2)            │  U06
│  i5-7500T, 16GB RAM, 500GB SSD         │
├─────────────────────────────────────────┤
│  ThinkCentre M710Q (Node 1)            │  U05
│  i5-7400T, 16GB RAM, 500GB SSD         │
├─────────────────────────────────────────┤
│                                         │  U04
│  Jonsbo N2 NAS                         │  U03
│  N5105, 16GB RAM                       │  U02
│  5x 1TB HDD (RAID 5)                   │  U01
│  1TB NVMe Cache, 650W PSU              │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  CyberPower UPS 1500VA                  │
└─────────────────────────────────────────┘
```

**Equipment Summary:**
- **Raspberry Pi 5** - 8GB RAM, PoE, 512GB NVMe, 7" Touchscreen (monitoring/services)
- **N150 Firewall** - 4x 2.5GbE, 8GB DDR4, 128GB NVMe (pfSense)
- **MikroTik CRS310** - 8x 2.5GbE + 2x SFP+ managed switch
- **ThinkCentre Nodes (4x)** - Proxmox cluster with 16GB RAM each
- **Jonsbo N2 NAS** - TrueNAS with RAID 5 storage
- **CyberPower UPS** - 1500VA battery backup

---

## ✅ Requirements

### Core Requirements
- ✅ RAID 5 storage
- ✅ 8-port managed 2.5GbE switch
- ✅ Jump server
- ✅ 12U rack server
- ✅ Firewall (DMZ)
- ✅ Clustering
- ❌ **Ethernet over Power (EoP)** - Since I wanted a 10-inch managed switch, only Mikrotik and QNAP are well-known high-quality options. QNAP supports PoE and 2.5GbE but costs double. A PoE+ injector is affordable, and I only need it for the Raspberry Pi (which maxes at 1Gbps anyway).

### 📊 Analysis: Current Setup Issues

**Hardware:**
- Server uses mesh network instead of direct router connection (could use EoP or MoCa)
- Eero mesh pod has only two Ethernet ports—not enough for expansion
- Current hardware can't communicate effectively (solution: Proxmox cluster)
- No data redundancy (solution: RAID 5+1)
- Out of space, couldn't follow industry-standard 3-2-1 backup strategy
- Server location in living room requires low noise
- Expanding network means low idle power consumption is critical
- Need to segregate storage from main hardware while learning (solution: Proxmox + TrueNAS)
- Avoid as much network bottleneck, aim for 2.5Gb/s as it's more standardized

**Current Software Stack:**
- CasaOS
- NextCloud
- Docker
- Timeshift
- Fish Shell
- UFW
- Portainer
- Syncthing

---

## 🛠️ New Software Stack

> **📋 Note:** For detailed hardware specifications and pricing, see [HARDWARE.md](./HARDWARE.md)

### 🥧 Raspberry Pi 5 (Service Node)

| Technology | Purpose | Description |
|------------|---------|-------------|
| **Grafana** | Visualization | Real-time monitoring dashboards for infrastructure metrics |
| **Scaphandre** | Power Monitoring | Tracks energy consumption of servers and services |
| **Prometheus** | Metrics Collection | Time-series database for monitoring and alerting |
| **NUT-Exporter** | UPS Monitoring | Exposes UPS metrics to Prometheus for power monitoring |

### 🚀 DevOps & Automation

| Technology | Purpose | Description |
|------------|---------|-------------|
| **Kubernetes (Talos)** | Container Orchestration | Minimal, immutable OS for running containerized workloads |
| **Ansible** | Configuration Management | Automates server provisioning and configuration |
| **ArgoCD** | GitOps CD | Continuous deployment using Git as source of truth |
| **GitLab** | CI/CD & Git | Self-hosted Git repos with built-in CI/CD pipelines |
| **CI Runners** | Build Automation | Execute GitLab CI/CD jobs for automated testing and deployment |
| **Podman** | Container Runtime | Daemonless container engine as Docker alternative |

### 🌐 Networking

| Technology | Purpose | Description |
|------------|---------|-------------|
| **Netgear Router** | Access Point | ISP router configured in AP mode for WiFi |
| **Router-on-a-stick** | VLAN Routing | Single interface routing multiple VLANs through firewall |
| **Pi-Hole** | DNS & Ad Blocking | Network-wide ad blocking and DNS caching |
| **PiVPN (WireGuard)** | VPN Server | Secure remote access to home network |

### 🎯 Compute Nodes (Proxmox Cluster)

| Technology | Purpose | Description |
|------------|---------|-------------|
| **Proxmox VE (PVE)** | Hypervisor | Open-source virtualization platform for VMs and containers |
| **Proxmox Backup Server** | Backup Solution | Deduplicated, incremental backups for VMs and containers |
| **Tor Node** | Privacy | Contribute to Tor network anonymity |
| **VDI (AD)** | Virtual Desktop | Windows virtual desktop with Active Directory integration |
| **WordPress** | Web Server | Self-hosted blog/website platform |
| **Minecraft** | Game Server | Self-hosted Minecraft server for friends |
| **SearxNG** | Meta-Search | Privacy-respecting meta-search engine |
| **Samba** | File Sharing | Network file sharing compatible with Windows/Mac/Linux |
| **Nginx** | Reverse Proxy | Web server and reverse proxy for services |
| **NUT Client** | UPS Management | Network UPS Tools for graceful shutdowns |
| **cloud-init** | VM Provisioning | Automated initial VM configuration |
| **n8n** | Workflow Automation | Self-hosted alternative to Zapier for automations |
| **Corosync** | Cluster Management | High-availability cluster communication for Proxmox |
| **NFS/iSCSI** | Network Storage | Network file system and block storage protocols |

### 💾 NAS (TrueNAS)

| Technology | Purpose | Description |
|------------|---------|-------------|
| **TrueNAS** | Storage OS | FreeBSD-based NAS with ZFS filesystem |
| **Jellyfin** | Media Server | Open-source media streaming (movies, TV, music) |
| **ZFS Snapshots** | Backup | Filesystem-level snapshots for point-in-time recovery |
| **rsnapshot** | Alternative Backup | Incremental backup solution (evaluating vs ZFS snapshots) |

### 📱 Mobile Management

| Technology | Purpose | Description |
|------------|---------|-------------|
| **Filebrowser** | File Access | Web-based file manager for remote access to NAS |
| **Proxmobo** | Mobile Proxmox | Android app for managing Proxmox from phone |

---

## 📚 Configuration & Setup Guides

- [Initial Setup Guide](./docs/SETUP.md) - Step-by-step installation and configuration
- [Network Configuration](./docs/NETWORK.md) - VLANs, firewall rules, and routing
- [Proxmox Cluster Setup](./docs/PROXMOX.md) - Cluster configuration and HA setup
- [TrueNAS Configuration](./docs/TRUENAS.md) - ZFS pools, shares, and snapshots
- [Monitoring Setup](./docs/MONITORING.md) - Grafana, Prometheus, and alerting

---

## 💰 Total Investment

Building a homelab that balances performance, expandability, and power efficiency (I was not budget conscious lmao)

**Total Hardware Cost:** See [HARDWARE.md](./HARDWARE.md) for complete breakdown

---

## 🔗 Resources

- [r/homelab](https://reddit.com/r/homelab) - Community for homelab enthusiasts
- [Proxmox Documentation](https://pve.proxmox.com/wiki/Main_Page)
- [TrueNAS Documentation](https://www.truenas.com/docs/)
- [pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)

---

## 📝 License

This documentation is shared for educational purposes. Feel free to use and adapt for your own homelab!
