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
VLAN 20: Storage (SAN) - Specialized traffic for iSCSI/NFS. Crucial for your 2.5GbE backbone.
Hosts: TrueNAS, Proxmox Storage Interfaces, PBS.
VLAN 30: Server/Services - Where your VMs and Containers live.
Hosts: Kubernetes Nodes, Docker Hosts, Web Servers.
VLAN 40: Trusted LAN - Your PC, Phone, WiFi.
VLAN 50: IoT/Untrusted - Smart bulbs, Guest WiFi.
VLAN 666: DMZ - Exposed services (Tor Node, Reverse Proxy ingress).

Here is the configuration map for your Mini-Enterprise Homelab. This setup separates **Management** (Pi), **Security/Edge** (N150), **Compute** (Intel Cluster), **Backup** (AMD), and **Storage** (Jonsbo).

### 📋 Homelab Service & OS Matrix

| Machine / Hardware | Role | Operating System | Active Services & Software Stack |
| :--- | :--- | :--- | :--- |
| **N150 Firewall**<br>*(4x 2.5GbE)* | **Edge Router & Security** | **pfSense CE** (or OPNsense) | • **Firewall (pf):** L3 Rules, VLAN Routing, NAT.<br>• **WireGuard Server:** Primary VPN (Road Warrior access).<br>• **Unbound:** DNS Resolver (DNSSEC enabled).<br>• **HAProxy:** Reverse Proxy (exposed ports 80/443).<br>• **pfBlockerNG:** Network-wide Ad/Geo-IP blocking.<br>• **LLDP:** Link Layer Discovery (for Switch). |
| **Raspberry Pi 5**<br>*(8GB, NVMe, Touchscreen)* | **Jump Host & Monitor** | **Ubuntu Server 24.04**<br>*(Headless w/ Kiosk)* | • **Tailscale:** Backup/Emergency VPN (Subnet Router).<br>• **NUT Master:** UPS Server (USB connection to CyberPower).<br>• **OpenSSH:** Secure Gateway (Jump Host).<br>• **LGTM Stack:** Loki (Logs), Grafana (Visuals), Tempo, Mimir.<br>• **Uptime Kuma:** Dashboard for Service Status (displayed on 7" screen).<br>• **Wazuh Agent:** Security Monitoring. |
| **MikroTik CRS310**<br>*(8x 2.5GbE + SFP+)* | **Core Switch** | **RouterOS v7** | • **VLANs:** Tagging (Mgmt, Storage, IoT, Trusted).<br>• **IGMP Snooping:** Optimizing multicast.<br>• **Jumbo Frames:** MTU 9000 (for Storage/VLAN 20).<br>• **LACP/BOND:** Link Aggregation (if applicable). |
| **ThinkCentre M710Q**<br>*(Node 1, 2, 3)* | **Compute Cluster (HA)** | **Proxmox VE 8.2** | • **Corosync:** High Availability Clustering.<br>• **QEMU/KVM:** Hosting VMs.<br>• **LXC:** Hosting Containers.<br>• **Replication:** ZFS Replication (Fast migration).<br>• **NUT Client:** Auto-shutdown listener.<br>• **Wazuh Agent:** Security Monitoring. |
| **ThinkCentre M715Q**<br>*(AMD A10)* | **Backup Server** | **Proxmox Backup Server**<br>*(Bare Metal)* | • **Backup Daemon:** Deduplicated backup handling.<br>• **Datastore:** 1TB NVMe (Hot backups).<br>• **Garbage Collection:** Pruning old backups.<br>• **Sync Jobs:** Offsite replication (Cloud).<br>• **NUT Client:** Auto-shutdown listener. |
| **Jonsbo N2**<br>*(N5105, 5x HDD)* | **Storage (SAN/NAS)** | **TrueNAS Scale** | • **ZFS:** RAIDZ1 (Parity Storage).<br>• **NFSv4:** Shared Storage for Proxmox VMs/ISOs.<br>• **SMB (Samba):** Shares for Windows/Mac (Media, Docs).<br>• **Smartd:** HDD Health Monitoring.<br>• **Scrub Tasks:** Data integrity checks.<br>• **NUT Client:** Auto-shutdown listener. |

---

### 📦 Workload Breakdown (Virtualization Layer)

These services run **inside** the Proxmox Cluster (Nodes 1-3). You will distribute these across your 3 nodes using **LXC Containers** (with Podman inside) or **VMs**.

| LXC / VM Name | Purpose | Software / Docker Containers Running Inside |
| :--- | :--- | :--- |
| **LXC-Docker-Media** | **Media Stack** | • **Gluetun:** VPN Client (Killswitch).<br>• **qBittorrent:** Torrent Client (Routed via Gluetun).<br>• **Prowlarr:** Index Manager (Routed via Gluetun).<br>• **Radarr/Sonarr:** Media Management.<br>• **Jellyfin:** Media Streaming (Mounts SMB share). |
| **LXC-Docker-Mgmt** | **DevOps & Auth** | • **Authentik:** SSO & Identity Provider.<br>• **Nginx Proxy Manager:** Easy SSL/Subdomains.<br>• **Ansible:** Configuration Management Controller.<br>• **GitLab Runner:** CI/CD Job Executor. |
| **VM-SIEM** | **Security** | • **Wazuh Manager:** Central Security Server (Ingests logs).<br>• **CrowdSec:** Intrusion Prevention System. |
| **LXC-PiHole** | **DNS (Internal)** | • **Pi-Hole (Primary):** Ad-blocking DNS for VLANs.<br>• **Unbound:** Recursive DNS. |
| **VM-Docker-Lab** | **Testing** | • **K3s / Talos:** Kubernetes playground.<br>• **Terraform State:** Infrastructure tracking. |

### 🔌 Physical Connections (Critical for this Setup)

1.  **UPS USB Cable:** Connects directly to **Raspberry Pi 5**.
2.  **NFS Traffic:** Must flow over **VLAN 20** (2.5GbE) between **Proxmox Nodes** and **Jonsbo NAS**.
3.  **Backup Traffic:** Flows from **Proxmox Nodes** to **AMD M715Q** (PBS).
4.  **Touchscreen:** Connects to Pi 5 (HDMI/DSI) to display the **Uptime Kuma** or **Grafana** dashboard permanently.
