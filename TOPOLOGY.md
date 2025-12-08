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

### Miscellaneous INFO
Netgear ProSafe GS724T V2 24-Port Gigabit Smart Switch | 43.06$

Eero 6+ Mesh - Wi-Fi 6 (AX3000) 1GB/s
My ISP is 50Mb/s download and 15Mb/s upload
Netgear Router AC1750, 1750Mbps
2.5GBE backbone 
SMB vs NFS vs iSCI 
TrueNAS + iSCI = Zvol
PXE Boot (Diskless boot)
SAN (Storage Area Network) => VM's hardrive is on the NAS
NFS on TrueNAS => for Backups and ISO library
SMB => Jellyfin + Documents + Prism
Quorum (2-1 majority vote from nodes)
DDNS (Pfsense) => if WAN IP changes you can still VPN in
Tailscale => If I mess up a firewall config and DNS fails. I can use as backdoor (don't need port forwarding)
Raspberry-Pi => Ubuntu-Server,Tailscale, PiVPN, Fail2Ban
Tiering => Asymmetric cluster
Deduplication Ratio: Usually 15:1 or better for VM backups
NFS Subdir External Provisioner/ TrueNAS-CSI
Cilium handles the network between your Tiny Nodes and Dell
Renovate keeps your GitOps repo updated automatically.
Upgrade Wifi for 2.5GBE and VLAN (so it dosen't strip)
VLAN 666 for Tor Node, Revers Proxy ingress 
Guest-wifi



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

______________________________________________________________________


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

##### Sofwares to look into
Cillum/Calico
CoreDNS
Rook/Ceph 
homepage
Isto 
Falco/Dynatrace 
ELK stack
External DNS Kubernetes 
Renavate 
Gata
Full Packet Capture (PCAP)
Suricata Zeek
ntopng
Security Onion
glutun 
deluge? 

______________________________________________________________________

