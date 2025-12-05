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
Dell Precision 3620 Tower i7-7700 Quad Core 3.6Ghz 4x8GB RAM 

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
Tower 
Mini PC Cluster
Raspberry-Pi
UPS 

##### Networking
Modem
Router/L3 Switch
Firewall 
Switch 
Jump Host 


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

Eero 6+ Mesh - Wi-Fi 6 (AX3000) 1GB/s
My ISP is 50Mb/s download and 15Mb/s upload
Netgear Router AC1750, 1750Mbps

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
Out-of-band management (OOB) => remotly rebooting

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
Caching Proxy 

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
Hardware Watchdog HA







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

Here are the updated tables reflecting your **Final Architecture**: A **4-Node Heterogeneous Cluster** (3 Tiny + 1 Dell) with the Raspberry Pi acting as the 5th voter (QDevice).

### 🧠 The Concept: The Heterogeneous 4-Node Cluster

> *"Ok now that the pi and the dell are part of the cluster..."*

**You have moved from "Split" to "Unified."**

*   **Single Pane of Glass:** You log into `https://10.0.10.11:8006` and you see **Node 1, Node 2, Node 3, and Dell-Node-4**.
*   **The Voting Logic (Quorum):**
    *   3 Tiny Nodes + 1 Dell = 4 Votes.
    *   Raspberry Pi (QDevice) = 1 Vote.
    *   **Total = 5 Votes.** (You can lose any 2 nodes and the cluster stays alive).
*   **Storage Strategy:**
    *   **Tiny Nodes:** Run VMs on their local 500GB SSDs (Replicated to each other).
    *   **Dell Node:** Runs VMs on its **1TB NVMe** (No Replication to Tiny Nodes to avoid filling them up).
    *   **Shared:** All 4 nodes mount the **TrueNAS NFS** share for ISOs and Backups.

---

### 📝 Updated Service Matrix

#### 🖧 Physical Network & Security Layer

| Machine / Hardware | OS | Services & Software Stack |
| :--- | :--- | :--- |
| **N150 Firewall**<br>*(The Edge)* | **pfSense CE** | • **FreeRADIUS:** Enterprise Auth for WiFi/VPN.<br>• **Avahi:** mDNS/Bonjour Reflector.<br>• **SNMP Agent:** Grafana monitoring metrics.<br>• **pfBlockerNG-Devel:** Ad-blocking & GeoIP filtering.<br>• **CrowdSec Agent:** IPS/IDS.<br>• **ntopng:** Deep Packet Inspection.<br>• **WireGuard:** VPN Server.<br>• **HAProxy:** Reverse Proxy Ingress.<br>• **Kea DHCP:** DHCP Server. |
| **MikroTik CRS310**<br>*(The Backbone)* | **RouterOS v7** | • **Port SFP+1:** **10GbE Uplink** to Dell Precision.<br>• **L3 HW Offloading:** Wire-speed Routing.<br>• **ACLs:** Hardware-level traffic blocking.<br>• **Port Mirroring:** Sends traffic copy to Pi for analysis.<br>• **IGMP Snooping:** Multicast optimization. |
| **Raspberry Pi 5**<br>*(Jump Host / OOB)* | **Ubuntu Server 24.04**<br>*(128GB NVMe)* | • **Corosync QDevice:** **(CRITICAL)** 5th Voter for Cluster Quorum.<br>• **MeshCentral:** Central OOB Mgmt (Intel AMT).<br>• **Tailscale:** Backup VPN.<br>• **NUT Master:** UPS Monitor.<br>• **RustDesk Server:** RDP Relay.<br>• **Uptime Kuma:** Service Dashboard.<br>• **Apt-Cacher-NG:** Update Mirror.<br>• **Syslog Receiver:** Forensic Logs.<br>• **Tcpdump:** Rolling PCAP. |

#### ⚙️ Compute Layer (4-Node Heterogeneous Cluster)

| Machine | Connectivity | Role | OS | Services & Software Stack |
| :--- | :--- | :--- | :--- | :--- |
| **ThinkCentre M710Q**<br>*(Nodes 1, 2, 3)* | **2.5GbE** | **Standard Cluster**<br>*(HA Group 1)* | **Proxmox VE 8.2**<br>*(500GB SSD)* | • **Corosync:** Cluster HA Member.<br>• **QEMU/KVM:** Standard Virtualization.<br>• **LXC:** Containerization.<br>• **ZFS:** **Replication Target** (Between Nodes 1-3 only).<br>• **NUT Client:** Auto-shutdown.<br>• **CrowdSec Agent:** Host Security.<br>• **Node-Exporter:** Metrics. |
| **Dell Precision 3620**<br>*(Node 4)* | **10GbE (SFP+)** | **Performance Node**<br>*(HA Group 2)* | **Proxmox VE 8.2**<br>*(1TB NVMe)* | • **Corosync:** Cluster HA Member.<br>• **QEMU/KVM:** Heavy/Windows/Gaming VMs.<br>• **ZFS (Local):** **1TB NVMe** (High IOPS for VDI).<br>• **Bulk Storage:** **500GB HDD** (Steam Cache).<br>• **Intel iGPU:** Passed through to Jellyfin.<br>• **Node-Exporter:** Metrics. |

#### 💾 Storage & Backup Layer

| Machine | Connectivity | OS | Services & Software Stack |
| :--- | :--- | :--- | :--- |
| **Jonsbo N2**<br>*(NAS)* | **2.5GbE** | **TrueNAS Scale** | • **NFSv4:** VM Storage (Shared to all 4 Nodes).<br>• **SMB:** Media Shares.<br>• **Rclone:** Cloud Sync (Proton).<br>• **ZFS Scrub:** Data Integrity.<br>• **SMARTd:** Disk Health.<br>• **Node-Exporter:** Metrics. |
| **ThinkCentre M715Q**<br>*(Backup)* | **2.5GbE** | **Proxmox Backup Server** | • **Backup Daemon:** Deduplication engine.<br>• **Garbage Collection:** Pruning old data.<br>• **Datastore:** 1TB NVMe.<br>• **Node-Exporter:** Metrics. |

---

### ☁️ Virtual Workloads Matrix (Optimized Placement)

*Note: Since this is now one cluster, "Host Preference" refers to Proxmox **High Availability Groups**.*

| Container / VM Name | Host Preference | Base OS | Runtime | Workload & Services Details |
| :--- | :--- | :--- | :--- | :--- |
| **LXC-Mgmt-DevOps** | **Tiny Nodes (1-3)** | **Debian 12** | **Podman** | **Management & Automation Stack**<br>• **Ansible:** Configuration management.<br>• **Terraform:** IaC (VM Spawning).<br>• **n8n:** Workflow Automation.<br>• **GitLab Runner:** CI/CD Agent.<br>• **Cloud-Init:** Provisioning Templates. |
| **LXC-Identity-Proxy** | **Tiny Nodes (1-3)** | **Debian 12** | **Podman** | **Security & Access Stack**<br>• **Authentik:** MFA, SSO, User Mgmt.<br>• **Nginx Proxy Manager:** Reverse Proxy (SSL).<br>• **Vaultwarden:** Password Manager.<br>• **CrowdSec Agent:** Log Analysis/Banning. |
| **LXC-Net-Services** | **Tiny Nodes (1-3)** | **Debian 12** | **Podman** | **Network Utilities Stack**<br>• **Pi-Hole:** Ad Blocking & Local DNS.<br>• **Unbound:** Recursive DNS.<br>• **SearXNG:** Meta-search Engine.<br>• **Uptime Kuma:** Internal Monitoring. |
| **VM-K8s-Cluster**<br>*(3x Nodes)* | **Tiny Nodes (1-3)** | **Talos Linux** | **Containerd** | **Kubernetes Cluster (K3s)**<br>• **LGTM Stack:** Loki, Grafana, Tempo, Mimir.<br>• **Prometheus:** Metrics.<br>• **Cert-Manager:** SSL Mgmt.<br>• **Test Lab:** Orchestration Playground. |
| **LXC-Media-Stack** | **Dell (Node 4)** | **Debian 12** | **Podman** | **Media & Gaming Stack (10GbE)**<br>• **Jellyfin:** Media Server (**Uses Dell iGPU**).<br>• **Gluetun:** VPN Killswitch (ProtonVPN).<br>• **qBittorrent / Prowlarr:** Torrenting.<br>• **Radarr / Sonarr:** Media Managers.<br>• **SteamCMD:** Downloads to 500GB HDD. |
| **LXC-Minecraft** | **Dell (Node 4)** | **Debian 12** | **Podman** | **Game Server**<br>• **Minecraft Java:** Uses high clock speed (3.6GHz).<br>• **Itzg-Docker:** Containerized Server. |
| **VM-Active-Directory** | **Dell (Node 4)** | **Win Server** | **Native** | **Enterprise Identity Lab**<br>• **AD DS:** Domain Services (Uses high RAM).<br>• **DNS/DHCP:** Windows Networking.<br>• **LDAP:** Integration Testing. |
| **VM-Win11-VDI** | **Dell (Node 4)** | **Win 11 Pro** | **Native** | **Virtual Desktop Interface**<br>• **RustDesk Client:** Remote Access.<br>• **Workstation:** General Usage.|

