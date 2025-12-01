# 🏠 My Homelab Journey

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

## ✅ Requirements

### Core Requirements
- ✅ RAID 5 storage
- ✅ 8-port managed 2.5GbE switch
- ✅ Bastion host (can run in container with OPNsense)
- ✅ 12U rack server
- ❌ **pfSense** - Buying a firewall with pfSense is expensive and overkill for now. I'm not the network admin at home, so I don't currently need it.
- ❌ **Ethernet over Power (EoP)** - Since I wanted a 10-inch managed switch, only Mikrotik and QNAP are well-known options. QNAP supports PoE and 2.5GbE but costs double. A PoE+ injector is affordable, and I only need it for the Raspberry Pi (which maxes at 1Gbps anyway).

### 📊 Analysis: Current Setup Issues

**Hardware:**
- Server uses mesh network instead of direct router connection (could use EoP)
- Eero mesh pod has only two Ethernet ports—not enough for expansion
- Current hardware can't communicate effectively (solution: Proxmox cluster)
- No data redundancy (solution: RAID 5+1)
- Out of space, couldn't follow industry-standard 3-2-1 backup strategy
- Server location in living room requires low noise
- Expanding network means low idle power consumption is critical
- Need to segregate storage from main hardware while learning (solution: Proxmox + TrueNAS)

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

### 🥧 Raspberry Pi 5 (Service Node)
- Grafana
- Scaphandre
- Prometheus
- NUT-Exporter

### 🚀 DevOps
- Kubernetes (Talos)
- ArgoCD
- GitLab
- CI runners
- Podman

### 🌐 Networking
- Netgear Router (AP mode)
- Router-on-a-stick
- Pi-Hole (DNS)
- PiVPN (WireGuard)

### 🎯 Compute Nodes
- Proxmox VE (PVE)
- Tor Node
- VDI (AD)
- WordPress
- Minecraft
- SearxNG
- Samba
- Nginx
- NUT client 
- cloud.init

### 💾 NAS
- TrueNAS
- Plex/Jellyfin
- ZFS snapshots / rsnapshot

### Mobile 
- Filebrowser

### Don't know what it is 
- Ansible 
- Corosync 
- NFS/iSCSI

---

## 🏗️ Rack Layout (Top to Bottom)

| Unit | Device | Mounting Method | Height |
|------|--------|----------------|--------|
| Top | 7" Screen + Mesh Pod | Sitting on top of cabinet | 0U |
| U12 | Venting Panel | Screwed into rails | 1U |
| U11 | MikroTik Switch | Hard Mount (RMK-2/10 Ears) | 1U |
| U10 | Patch Panel | Screwed into rails | 1U |
| U09 | Services Shelf | 1U shelf: N100 Firewall (left) + Pi 5 (right) | 1U |
| U08-U06 | ThinkCentre Stack | Nodes 3, 2, 1 on 1U shelf | 3U |
| U05-U01 | NAS (Jonsbo N2) | Sits on cabinet floor | 5U |

### 📐 Rack Space Calculation
- 1U = 4.45 cm
- NAS: 22.25 cm (5U)
- Nodes: 17.8 cm (3U)
- Switch: 4.45 cm (1U)
- Patch Panel: 4.45 cm (1U)
- Vent: 4.45 cm (1U)
- **Total: 53.4 cm (12U)**

---

## 🖥️ Hardware Components

### 💻 Existing Hardware
- **Eero 6+ Mesh Pod**
- **Netgear Router AC1750** (1750Mbps, from Explorer ISP)
- **Dell Precision 3620 Tower** - i7-7700 Quad Core 3.6GHz, 32GB RAM, 1TB NVMe, NVS310 GPU  
  [$419 CAD](https://www.amazon.ca/Dell-Optiplex-7050-Excellent-Condition/dp/B0F854GHFB)
- **Lenovo ThinkCentre M710Q** - i5-7400T 2.4GHz, 8GB RAM, 500GB HDD  
  [$102.14 CAD](https://www.ebay.ca/itm/197901648050)
- **Lenovo ThinkCentre M715Q** - AMD Pro A10-9700E, 8GB RAM, 500GB HD  
  [$99.98 CAD](https://www.ebay.ca/itm/155474578754)
- **Lenovo ThinkCentre M710q** - i5-7500T, 8GB DDR4, Intel AC8265 (no HDD)  
  [$121.74 CAD](https://www.ebay.ca/itm/376693720411)

### 💿 Storage
- **SanDisk SSD Plus 500GB** (opened box) - [$59.27 CAD](https://www.amazon.ca/dp/B0F4Y2VR8S)
- **SanDisk SSD Plus 500GB** (new) - [$64.99 CAD](https://www.amazon.ca/dp/B0F4Y2VR8S)
- **Patriot P220 128GB SATA SSD** - [$24.99 CAD](https://www.amazon.ca/dp/B0BS9W3T48)

### 🎮 Raspberry Pi 5 Setup
- **Raspberry Pi 5 8GB** (2023 model) - [$133.69 CAD](https://www.amazon.ca/dp/B0CK2FCG1K)
- **GeeekPi P33 M.2 NVMe PoE+ HAT** - [$48.98 CAD](https://www.amazon.ca/dp/B0D8JC3MXQ)

---

## 🏢 Rack Infrastructure

- **GeeekPi 12U Server Cabinet** - [$209.94 CAD](https://www.amazon.ca/GeeekPi-Cabinet-Equipment-RackMate-Rackmount/dp/B0DT2XM22G)
- **GeeekPi 12-Port Patch Panel** - [$23.79 CAD](https://www.amazon.ca/dp/B0D5XPNHHF)
- **GeeekPi 1U Server Rack Shelf** - [$28.04 CAD](https://www.amazon.ca/dp/B0D5XGSPXD)
- **PDU 3-Socket** - [$42.72 CAD](https://www.aliexpress.com/item/1005005963996490.html)
- **3-Prong 1-to-4 Power Splitter** - [$23.99 CAD](https://www.amazon.ca/dp/B0CH9NHFHS)
- **Freenove 7" Touchscreen Monitor** - [$47.96 CAD](https://www.amazon.ca/dp/B0BPP6MFFJ)

---

## 🌐 Network Equipment

- **MikroTik CRS310-8G+2S+IN** (8x 2.5GbE + 2x SFP+) - [$276 CAD](https://www.amazon.ca/dp/B0CH9NHFHS)
- **MikroTik RMK-2/10 Rack Mount Kit** - [$33.52 CAD](https://www.shop.wirelessnetware.ca/accessories/516-rmk-210-4752224008688.html)
- **Gigabit PoE+ Injector** - [$18.88 CAD](https://www.amazon.ca/Injector-IEEE802-3at-Replacement-TPE-115GI-TL-PoE160S/dp/B00NRF9GQO)

### 🔥 Firewall Hardware
- **Fanless N100 Mini PC** - 4x 2.5GbE, DDR4, NVMe, Type-C - [$245.99 USD](https://www.aliexpress.com/item/1005010292635214.html)
- **Patriot P320 128GB NVMe** - [$33.99 CAD](https://www.amazon.ca/dp/B0D4RD18YV)
- **Crucial 8GB DDR4 3200MHz SO-DIMM** - [eBay.ca](https://www.ebay.ca/itm/365735557774)
- **2.5Gb Network Adapter** (M.2 A+E/Mini PCIe to RJ45) - [$18.79 USD](https://www.aliexpress.com/item/1005008820106326.html)

### 🖥️ Alternative N100 Option
- **N100 Linux Industrial Embedded PC** - 4x 2.5GbE - [$283.29 USD](https://www.aliexpress.com/item/1005005404120290.html)

---

## 💾 NAS Specifications

- **Case:** Jonsbo N2 NAS - [$169.05 USD](https://www.aliexpress.com/item/1005007766481140.html)
- **Motherboard + CPU:** N5105 Industrial Board - [$160.38 USD](https://www.aliexpress.com/item/1005006221619148.html)
- **Case Fan:** Noctua NF-A9 PWM - [$25.05 CAD](https://www.amazon.ca/dp/B00RUZ059O)
- **CPU Fan:** Noctua NF-A4x20 - [$29.79 USD](https://www.aliexpress.com/item/1005006690066510.html)
- **PSU:** GAMEMAX 650W 80+ Gold Fully Modular SFX - [$144.49 CAD](https://www.amazon.ca/dp/B0F9NR7G17)
- **Memory:** Crucial 8GB x2 DDR4 3200MHz SO-DIMM - [$93.69 CAD](https://www.ebay.ca/itm/365735557774)
- **SATA Cables:** 6-pack - [$6.79 USD](https://www.aliexpress.com/item/1005007523507363.html)
- **Boot Drive:** 1TB NVMe M.2 PCIe 3.0x4 (3,500MB/s) - [$107.99 CAD](https://www.amazon.ca/dp/B0DTY976K3)
- **Storage:** 1TB 3.5" HDD - $25 CAD (MDG Computers)

---

## 💰 Total Investment

Building a homelab that balances performance, expandability, and power efficiency 
