# Topology
## Physcial Topology
### Rack Diagrams
#### OLD VERSION 
Screen - Mesh } On top 
PDU } Behind U12/U11 (Top)
1U Venting Pannel
Switch } 1U Mounted
Patch Pannel } 0.5U 
Raspberry PI } O.5U Rack Shelf 
ThinkCentre } 1U Rack Shelf
ThinkCentre } 1U Rack Shelf
NAS } 5U At the bottom
UPS - Dell Tower } Sitting Next to it 


Unit	Device	Mounting Method	Notes
Top	Pi 5 + Freenove Screen	Sitting on top	moved from U09
Top	Mesh Pod	Sitting on top	
U12	Venting Panel	Screwed into rails	
U11	MikroTik Switch	Hard Mount	
U10	Patch Panel	Screwed into rails	
U09	Services Shelf	N150 Firewall (Left) + ThinkCentre PBS (Right)	Perfect fit!
U08	ThinkCentre Stack	Node 3 (Top)	
U07	ThinkCentre Stack	Node 2 (Middle)	
U06	ThinkCentre Stack	Node 1 (Bottom)	
U01-05	NAS (Jonsbo N2)	Floor/Body	



### Hardware
Raspberry Pi 5 8GB, PoE, 512GB NVMe with 7inch Touchscreen
N150 4x 2.5GbE, 8GB DDR4, NVMe 128GB
MikroTik CRS310-8G+2S+IN 8x 2.5GbE + 2x SFP+ managed switch
ThinkCentre M710Q i5-7400T, 2.5GbE, 16GB RAM, 500GB SSD SATA
ThinkCentre M710Q i5-7500T, 2.5GbE, 16GB RAM, 500GB SSD SATA
ThinkCentre M710Q i5-7500T, 2.5GbE, 16GB RAM, 500GB SSD SATA
ThinkCentre M715Q AMD Pro A10-9700E, 16GB DDR4 RAM, 128GB SSD SATA Boot-drive + 1TB NVMe
Jonsbo N2, N5105, 16GB DDR4 RAM, 1TB NVMe, 128GB SSD SATA boot-drive, 650W PSU, 5x1TB HDD
CyberPower CST150UC-FC UPS 1500VA


### Homelab Parts SAMPLE - Physical Design
#### Parts: 
##### Rack 
cabinet 
mouting ears
rails 
patch panel 
fan panels
cable management guard
lacing bar
brush panel
PDU 
display panel

##### Hardware
Server 
Mini PC
Raspberry-Pi
Arduino
UPS 
KVM 
GPU cluster

##### Networking
Router/L3 Switch
Modem
Firewall 
Switch managed/unamaged
Jump Host 
Bastion Host 
Hub

### Storage
#### Where it is going
G15 laptop 1TB 
NAS 120GB + 1TB NVMe
ThinkCentre Intel 500GB 
ThinkCentre Intel 500GB
ThinkCentre Intel 512GB
ThinkCentre AMD 128GB + 2TB NVME
Pi 5 512GB 
N150 128GB 
Dell Hardrives 500GB HD

### Miscellaneous INFO
Netgear ProSafe GS724T V2 24-Port Gigabit Smart Switch | 43.06$
Netgear Router AC1750, 1750Mbps
Xplornet Hub ZTE ZXHN H298N 
Eero 6+ Mesh - Wi-Fi 6 (AX3000) 1GB/s
Netgear R6200 Wifi Router 803.11ac 1200Mibs
My ISP is 50Mb/s download and 15Mb/s upload


## Logical Topology
### Logical Design 
Eero (WAN) 
DMZ (Firewall)
VLAN Tagging (Switch)
Raspberry-Pi (jump host)

#### HA Notes 
- Corosync => Cluster Manager
- ZFS(RAID0) replication on nodes
- Quorum (2-1 majority vote from nodes)
- Seperate VLAN for Management(VLAN1) and for Storage/Migration(VLAN2)
- Backup DNS in a container
- Hardware Watchdog in ThinkCentre BIOS => if proxmox freezes the hello packet (ACK), reboots it 
High Availability (HA)
Network UPS Tools (NUT)
Maybe put NUT on the PBS instead
CloudSync => RClone 

##### High Availability (HA) 
3-2-1 Rule
Uninterruptible Power Suppy (UPS)
Proxmox Backup Server (PBS)
Network UPS Tools (NUT)
Data (Offsite Backup) (proton)
DNS Reduncancy (Pi-Hole) 

#### Log Aggregation Notes 
LGTM Stack (OpenTelemetry) => Loki,Grafana,Tempo,Mimir
Uptime Kuma 
Loki => logs agreggator 
mimir => metric agreggator 
tempo => trace aggregator 
grafana => visualization, testing, actionable insights
Kuma => simplified dashboard on 7-inch screen
grafana alloy=> automate finding OpenTelemetry
promtail => if a server crashes, Prometheus says it crashed. Loki tells you why, Grafana displays it

##### Exporters
NUT-Exporter => planned UPS stats.
Node-Exporter => recommended monitor the Pi's own heat/CPU. 

______________________________________________________________________

## Attempt 2 
### Notes 
VLAN 666 for Tor Node, Revers Proxy ingress 
Guest-wifi

#### Terms
Hyper-Converged Infrastructure (HCI) => All one computer managing resources
Disaggregated HCI (dHCI) => independent scaling HCI 
Compute-Storage Split => SAN, NAS and servers are all seperate
Virtual Desktop Infrastructure (VDI)
I/O-intensitivy => data speed and accesible
CapEx => Higher initial capital expenditure
Remote/Branch Offices (ROBO) 
Edge Computing
OLTP
operational expenses (OpEx)
cost of ownership (TCO)
Software Defined Storage (SDS) 
Type-1 Hypervisor
Deduplication
Incremental Backup
Immutable Backups
Split Tunneling
XDR
Stateful Packet Inspection 
L3 Routing
Infrastructure as Code (IaC)
NUT (Network UPS Tools)
LXC => Linux Container

##### Services
Hypervisor => PVE
Storage => TrueNAS Scale
Backup => PBS
VPN => Wireguard
Authentication => Authentik
SIEM => Wazuh
IPS => CrowdSec
Firewall => pfSense
Log aggregation => Loki
IaC => Terraform 
Config Mgmt => Ansible 
Orchestration => Kubernetes (k3s) 
GitOps => ArgoCD (not very relevant for my main tasks)
Torrenting => Radarr (Movies), Sonarr (TV), Bazarr (Subtitles), Jellyfin (Streaming)
UPS Monitoring => NUT
Container => Podman

##### Software
VLANs 
Subnetting 
PVE 
BMC/IPMI
SAN 
WinBox 
iSCSI/NFS
PBS
Kubernetes 
Docker 
Web Servers
Tor Node 
Reverse Proxy ingress
ZFS
SIEM 
IPS/IDS 
GitOps 
gluetun 
qBittorrent 
Prowlarr
LXC



______________________________________________________________________

## Architecture
### Logical Architecture & Topology
VLAN 10: Management (Mgmt) - Only VPN access - PVE
Hosts: PVE, Switch GUI (WinBox), UPS Mgmt, BMC/IPMI.
VLAN 20: Storage (SAN) - Specialized traffic for NFS/SMB. Crucial for your 2.5GbE backbone.
Hosts: TrueNAS, Proxmox Storage Interfaces, PBS.
VLAN 30: Server/Services - Where your VMs and Containers live.
Hosts: Kubernetes Nodes, Docker Hosts, Web Servers.
VLAN 40: Trusted LAN - Your PC, Phone, WiFi.
VLAN 50: IoT/Untrusted - Smart bulbs, Guest WiFi.
VLAN 666: DMZ - Exposed services (Tor Node, Reverse Proxy ingress).



### 🏗️ The "Mini-Enterprise" Service Matrix

Here is where every single piece of software lives.

#### 🖧 Physical Network & Security Layer

| Machine / Hardware | OS | Services & Software Stack |
| :--- | :--- | :--- |
| **N150 Firewall**<br>*(The Edge)* | **OPNsense**<br>*(Preferred over pfSense for plugins)* | • **CrowdSec Bouncer:** (Modern Fail2Ban) Detects attacks & blocks IPs firewall-wide.<br>• **WireGuard Server:** Your entry point (VPN In).<br>• **HAProxy:** Reverse Proxy Ingress (Port 80/443 handling).<br>• **Unbound DNS:** Recursive DNS.<br>• **Kea DHCP:** Managing IPs.<br>• **IGMP Proxy:** For multicast traffic. |
| **MikroTik CRS310**<br>*(The Backbone)* | **RouterOS v7** | • **VLAN Filtering:** Segregating traffic.<br>• **L3 Hardware Offloading:** Routing traffic at wire speed. |
| **Raspberry Pi 5**<br>*(Jump Host / OOB)* | **Ubuntu Server 24.04** | • **Tailscale:** Emergency Backdoor VPN.<br>• **NUT Master:** Monitors CyberPower UPS via USB.<br>• **RustDesk Server:** Self-hosted "TeamViewer" (RDP) Relay server.<br>• **Uptime Kuma:** Dashboard on 7" Screen.<br>• **Wake-on-LAN (etherwake):** Script to wake up Intel nodes if off. |

#### ⚙️ Compute Cluster Layer (The Intel Nodes)

These machines run the Hypervisor. They utilize **Intel vPro (AMT)** for BMC/IPMI-like remote power control (since they are Enterprise Desktops).

| Machine | OS | Infrastructure Services (Running on Host) |
| :--- | :--- | :--- |
| **ThinkCentre M710Q**<br>*(Nodes 1, 2, 3)* | **Proxmox VE 8.2** | • **Corosync:** Cluster Communication.<br>• **QEMU/KVM:** Virtualization.<br>• **LXC:** Containerization.<br>• **ZFS:** Local Storage replication.<br>• **NUT Client:** Auto-shutdown listener.<br>• **CrowdSec Agent:** Host protection. |

#### 💾 Storage & Backup Layer

| Machine | OS | Services & Software Stack |
| :--- | :--- | :--- |
| **Jonsbo N2**<br>*(NAS)* | **TrueNAS Scale** | • **NFSv4:** Shared storage for Proxmox.<br>• **SMB:** Shares for Media/Docs.<br>• **Rclone:** **Cronjob** task to Sync encrypted data to **Proton Drive**.<br>• **ZFS Scrub:** Data integrity.<br>• **SMARTd:** Disk health. |
| **ThinkCentre M715Q**<br>*(Backup)* | **Proxmox Backup Server** | • **Backup Daemon:** Handling incremental backups.<br>• **Garbage Collection:** Deleting old backups.<br>• **Datastore:** 1TB NVMe. |

---

### ☁️ Virtual Workloads (The "Software Stack")

These are the VMs and Containers running **inside** the Proxmox Cluster.

#### 1. The DevOps & Management Stack (LXC)
*   **Operating System:** Alpine Linux or Debian (LXC)
*   **Services:**
    *   **Ansible Controller:** Runs the playbooks to update all other servers.
    *   **Terraform:** Manages the creation of VMs.
    *   **n8n:** Workflow automation (e.g., "If Download finishes, send me a generic notification").
    *   **GitLab Runner:** Executes your CI/CD pipelines.
    *   **Cloud-Init:** Used here to template new VMs.

#### 2. The Identity & Security Stack (Docker in LXC)
*   **Operating System:** Ubuntu Server 22.04 (LXC)
*   **Services:**
    *   **Authentik:** **MFA** (TOTP/WebAuthn) and SSO Provider. Handles users.
    *   **Nginx Proxy Manager (or Traefik):** Internal Reverse Proxy.
    *   **Bitwarden (Vaultwarden):** Password Manager.

#### 3. The "Arr" Media Stack (Podman in LXC)
*   **Operating System:** Debian 12 (LXC)
*   **Services:**
    *   **Gluetun:** **VPN Killswitch** (Connected to ProtonVPN).
    *   **qBittorrent:** Torrent client (Routed *through* Gluetun).
    *   **Prowlarr:** Indexer manager (Routed *through* Gluetun).
    *   **Radarr/Sonarr:** Movie/TV collection managers.
    *   **Jellyfin:** Media Player.

#### 4. The Kubernetes Cluster (3x VMs)
*   **Operating System:** Talos Linux or Ubuntu Server (VMs)
*   **Cluster Type:** K3s (Lightweight Kubernetes)
*   **Services (The LGTM Stack):**
    *   **Loki:** Log aggregation.
    *   **Grafana:** Visualization Dashboards.
    *   **Tempo:** Tracing.
    *   **Mimir:** Metrics.
    *   **Prometheus:** Scrapes data from node-exporters.

#### 5. Windows VDI (VM)
*   **Operating System:** Windows 10/11 Pro
*   **Services:**
    *   **RustDesk Client:** Allows you to RDP into this machine from outside via your Pi Relay.
    *   **BlueIris:** (Optional) If you ever do Security Cameras.

---

### 🕒 Automation & Cronjobs

To make this "Industry Standard," you don't just run things manually. You set schedules.

1.  **NAS (TrueNAS):**
    *   `0 3 * * *` (3 AM): **Rclone** sync critical documents to Proton Drive.
    *   `0 5 * * 0` (Sunday): ZFS Scrub (Check for bit rot).

2.  **Proxmox Cluster:**
    *   `0 2 * * *` (2 AM): Backup all VMs to PBS (AMD Machine).

3.  **Ansible Node:**
    *   `0 4 * * 6` (Saturday): Run `apt update && apt upgrade` on all Linux nodes (Automated Patching).

4.  **Raspberry Pi (UPS):**
    *   **Daemon (Not Cron):** Polls UPS every 5 seconds. If battery < 20%, triggers `upsmon -c fsd` (Forced Shutdown).
