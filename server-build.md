# Server Build

## What is the point 
- Torrent Node and Seeding 
- Local Backups 
- Streaming Movies 
- Virtualized enviroments (testing stuff) (ADDS, CTFs)
- Searxng 
- Vault Warden?
- Private VPN
- Game file Server 
- RDP to my PC

### Current Spend (Not Accurate)
#### Ebay
- 118.69$ 
- 112.98$ 
- 93.69$ (N) 
- 271.17$ 
- 55.40$ (X)(E)
- 88.23$ (X)(E)
#### Amazon
- 1064.43$ 
- 28.4$  
- 474.59$ (X)
- 206.42$ (X)
- 220.45 (N)
#### Aliexpress
- 362.49$ (N) 
- 50.44$ 
- 49.10$ 
- 325.62$ 
#### Misc 
- Wireless Netware: 37.88$  
- MDG Computers Local 90$ (N)
- Home Depot: 25.96$ (E)


Daniel -> 2851.30$
Xmas -> 824.64$
Grand Total -> 3675.94$

------------------------
Server -> 2739.72$
NAS -> 766.63$
Equipment -> 169.59$

------------------------
Legend
(NOT X) -> Daniel
(X) Xmas expense 
(E) Equipment 
(N) NAS 
(NOT N AND NOT E) Server

-----------------------

### High Availability (HA) - INFO DUMP
Redundancy Recommendations (The "3-2-1" Rule)
You have High Availability (HA) for compute, but you need redundancy for Data and Power.
A. Power (The Auto-Shutdown)
Network UPS Tools (NUT): You have a UPS. But if the power goes out for 30 minutes, does your cluster know?
The Fix: Configure NUT (software) on your Pi or PBS.
Scenario: Power fails -> UPS battery hits 20% -> NUT tells Proxmox "Shut down VMs now" -> Cluster safely powers off.
Without this: Power cuts instantly -> ZFS corruption -> Data loss.
B. Data (Offsite Backup)
The Risk: Fire, Flood, or Electrical Surge destroys the rack. Your Cluster, your NAS, and your Backup Server (AMD) are all physically in the same square meter.
The Fix: Cloud Sync.
In Proxmox Backup Server, configure a "Remote" or use Rclone on your NAS to encrypt and upload your most critical data (photos, documents) to Backblaze B2, AWS S3, or even a Google Drive.
Cost: pennies per GB.
C. DNS Redundancy
The Risk: Your Pi 5 (Pi-hole) dies. The internet works, but nobody can load websites because DNS is down. "Wife/Parent Acceptance Factor" drops to zero.
The Fix: Run a secondary Pi-hole container on one of the Intel Nodes (in Proxmox). Set your DHCP to hand out both IP addresses.


#### Corosync "Split Brain" Prevention

Step 2: The Software Fix (Token Rotation)
By default, Corosync expects a response in ~100ms. If the network is clogged by ZFS, it panics. We will change that timeout to 3-5 seconds.
Do this after installing Proxmox:
On one node, create a file: /etc/pve/corosync.conf (Proxmox manages this file).
You will edit the totem section to increase the token timeout.
code
Bash
totem {
    version: 2
    secauth: on
    cluster_name: my-cluster
    config_version: 1
    # CHANGE THIS LINE BELOW
    token: 5000  
    # 5000ms = 5 seconds tolerance before declaring a node dead
    interface {
        linknumber: 0
    }
}
Why this works:
Even if ZFS Replication floods your 2.5GbE cable, it won't block the line for a full 5 seconds. The Corosync heartbeat will eventually squeeze through, and the cluster will stay stable.


### Other Software (INFO DUMP)
glusterfs, ha,

I'm running a single Proxmox unprivileged LXC container using Docker to run the following services: Plex, Overseerr, Radarr, Sonarr, Prowlarr, SABnzb, qBittorrent, Bazarr, Tuatulli, Bookshelf (Readarr fork), Calibre. Happy to share the docker compose file to anyone who requests it (this was easily the longest and hardest part of the project, getting all the services working together and automated, especially Readarr and Calibre).

#### Reddit guy info: (not included into the article) 
A bit about software:
- basically everything is configured via ansible
- all services (adguard, rsnapshot, samba, nginx, searxng, baikal, forgejo, syncthing, mc, etc...) are running in rootless podman containers.
- zfs for nas drives 

Scaphandre. It acts as a Prometheus exporter that tracks power consumption metrics at the process and host level, which you can visualize in Grafana.


### Jump Server Stack (INFO DUMP)
Fail2Ban
MFA for SSH: Consider setting up libpam-google-authenticator
LGTM Stack

oki: Install Loki on this Pi (or your NAS).
Promtail: Install Promtail on your Cluster nodes and Firewall.
Why: If a server crashes, Prometheus tells you when it stopped responding. Loki tells you why (by showing you the error logs right before the crash in the same Grafana dashboard).
3. Uptime Kuma
Purpose: Simple "Up/Down" monitoring.
Why: Prometheus is great for deep metrics (CPU usage, RAM), but Uptime Kuma gives you a beautiful "Status Page" that shows if your Internet, Gateway, or DNS is down. It’s easier to read at a glance than a complex Grafana dashboard.
4. Tailscale (As a Backdoor)
Purpose: Emergency Access.
Why: You are hosting WireGuard (PiVPN) yourself. If you mess up the Firewall config, or if your ISP changes your IP and DDNS fails, you are locked out. Keep Tailscale installed on the Pi as a backup entry method that doesn't require port forwarding.
Revised Service List for Pi 5
Category	Service	Status	Notes
Access	WireGuard	Planned	Primary VPN.
Access	Tailscale	Recommended	Backup VPN (Emergency access).
Security	Fail2Ban	Recommended	Ban bots attacking your VPN/SSH.
Monitoring	Grafana	Planned	The Dashboard.
Monitoring	Prometheus	Planned	The Database for numbers.
Monitoring	Loki	Recommended	The Database for Logs.
Monitoring	Uptime Kuma	Recommended	"Is the internet down?" dashboard.
Exporters	NUT-Exporter	Planned	UPS stats.
Exporters	Node-Exporter	Recommended	Monitor the Pi's own heat/CPU.
Technical Note on the GeeekPi HAT & Ubuntu
When you install Ubuntu on the Pi 5 with the GeeekPi PoE+ HAT, the fan might run at 100% or 0% by default. You may need to edit /boot/firmware/config.txt to enable the official Pi fan control overlay:
code
Bash
dtparam=fan_temp0=50000
dtparam=fan_temp0_hyst=2000
dtparam=fan_temp1=60000
dtparam=fan_temp1_hyst=2000


### WAN configuration INFO DUMP

2. The Configuration Steps
Step A: Eero Configuration (The Edge)
Open Eero App.
DHCP: Let Eero handle DHCP for the "House" (e.g., 192.168.4.x).
DMZ: Go to Settings > Network Settings > Reservations & Port Forwarding.
Assign a Static IP to the N150 (e.g., 192.168.4.2).
Enable DMZ for this IP.
Result: The Eero forwards all incoming internet ports to your N150. This minimizes the "Double NAT" issues for your servers.
Step B: Netgear AC1750 (The Lab AP)
Plug your laptop directly into the Netgear (disconnected from the rest of the network).
Log in to the Web UI.
Enable AP Mode (Access Point Mode).
This disables the DHCP server and NAT on the Netgear. It becomes a "dumb" WiFi antenna.
Set the SSID to something unique (e.g., Homelab_5G).
Step C: N150 Firewall (The Router)
Port 1 (WAN): Connects to Eero.
Type: Static IPv4 (192.168.4.2).
Gateway: Eero IP (192.168.4.1).
Port 2 (LAN): Connects to MikroTik Switch Port 1.
Subnet: 10.0.10.1/24 (Management/Servers).
Port 3 (OPT1/WIFI): Connects to Netgear AC1750.
Subnet: 10.0.20.1/24 (Lab WiFi).
Enable DHCP Server on this interface so your laptop gets an IP when you join WiFi.
3. The Updated Physical Map (Ports Saved!)
By moving the Netgear AP directly to the N150 Firewall, you save a port on your precious MikroTik switch.
N150 Firewall Connections:
Port 1: Cable to Eero LAN.
Port 2: Cable to MikroTik Port 1.
Port 3: Cable to Netgear AC1750 (WAN/Internet port).
Port 4: Empty (Emergency Laptop access).
MikroTik CRS310 Port Map (Updated):
Port	Speed	Device
Port 1	2.5GbE	N150 Firewall (LAN Uplink)
Port 2	2.5GbE	Node 1 (Intel)
Port 3	2.5GbE	Node 2 (Intel)
Port 4	2.5GbE	Node 3 (Intel)
Port 5	2.5GbE	PBS Node (AMD)
Port 6	2.5GbE	TrueNAS
Port 7	1GbE	Raspberry Pi 5 (via PoE Injector)
Port 8	1GbE	EMPTY / SPARE (Available for future expansion!)


### UPS monitoring - INFO DUMP

Step B: UPS Monitoring (Graceful Shutdown)
This is critical. You need NUT (Network UPS Tools).
Physical: USB cable from CyberPower UPS to the Raspberry Pi 5 (or TrueNAS).
Master (Server): Configure the Pi as the netserver. It reads the USB data.
Slaves (Clients): Install nut-client on Proxmox Nodes and TrueNAS.
Logic:
Power fails -> UPS signals Pi.
Pi waits (e.g., 5 mins on battery).
Pi sends FSD (Forced Shutdown) flag to network.
Proxmox nodes receive flag -> Stop VMs -> Shutdown OS.
TrueNAS receives flag -> Unmount ZFS -> Shutdown.
Pi shuts down last

### Automation INFO DUMP 

Step C: Automation (Ansible & Terraform)
Terraform: Use this to create the VM.
Example: "I want a VM with 2 vCPUs, 4GB RAM, named 'Web-Server'."
Ansible: Use this to configure the VM.
Example: "I want Nginx installed, my user created, and the timezone set to EST."
Logical Workflow:
You push code to GitHub.
GitLab CI/CD runner (on your cluster) sees the change.
Runner executes terraform apply -> Proxmox creates VM.
Runner executes ansible-playbook -> Configures VM.

What is a Killswitch?
A safety mechanism that ensures traffic only flows through the VPN interface (tun0). If the VPN connection drops, the network interface blocks all traffic, rather than defaulting back to your unprotected ISP connection (which would expose your IP).

Gluetun is a "VPN Client in a Container."

You don't configure the VPN inside qBittorrent. Instead, you tell Docker/Podman to route qBittorrent's network through Gluetun.
Docker Compose Example (Visualized):

@code yaml
services:
  gluetun:
    image: qmcgaw/gluetun
    cap_add:
      - NET_ADMIN
    environment:
      - VPN_SERVICE_PROVIDER=protonvpn
      - VPN_TYPE=wireguard
    ports:
      - 8080:8080 # qBittorrent WebUI routed through here

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent
    network_mode: "service:gluetun" # <--- THE MAGIC LINE
@end


### Uptime Kuma - INFO DUMP

1. How to set up Telegram Notifications
You will use Uptime Kuma (running on your Raspberry Pi) to handle this. It is the central nervous system that checks if everything else is alive.
The Setup Steps:
Create a Bot: Open Telegram on your phone, search for @BotFather, and type /newbot. Name it (e.g., Homelab_Alert_Bot).
Get the Keys: BotFather will give you an API Token. Copy this.
Get your ID: Search for @userinfobot in Telegram and click Start. It will give you your Chat ID.
Configure Uptime Kuma:
Go to your Pi's dashboard (Uptime Kuma).
Settings -> Notifications -> Setup Notification.
Type: Telegram.
Paste your Bot Token and Chat ID.
Click "Test". You will get a ping on your phone instantly.
Apply it: Edit your monitors (e.g., "Proxmox Node 1", "Plex") and enable this notification for them.



### File Tree

homelab/
├── README.md                 # Main overview (what I just created)
├── HARDWARE.md              # All hardware specs, prices, and links
├── docs/
│   ├── SETUP.md            # Initial setup walkthrough
│   ├── NETWORK.md          # Network configuration & commands
│   ├── PROXMOX.md          # Proxmox cluster setup
│   ├── TRUENAS.md          # TrueNAS configuration
│   └── MONITORING.md       # Grafana/Prometheus setup
└── configs/
    ├── pfsense/            # pfSense config backups
    ├── proxmox/            # Proxmox configs
    ├── ansible/            # Ansible playbooks
    └── kubernetes/         # K8s manifests


### Research 
https://loganmarchione.com/2022/09/homelab-10-mini-rack-shelves/
https://linuxblog.io/home-lab-beginners-guide-hardware/
https://linuxblog.io/pfsense-firewall-config-settings/
https://www.youtube.com/watch?v=lUzSsX4T4WQ
https://www.cdw.ca/product/netgear-gs308e-ethernet-switch/8434280
https://www.amazon.ca/UBIQUITI-ER-X-SFP-Edgerouter-X-Sfp/dp/B012X45WH6

#### NAS Research
https://www.youtube.com/watch?v=XXKppFyHtHk&t=234s
https://linustechtips.com/topic/1590875-up-to-12-internal-drives-in-a-jonsbo-n2/
https://hunio.org/posts/homelab/my-new-downsized-homelab/
https://www.youtube.com/watch?v=C6hf3ddtNCs


#### More Research
https://www.reddit.com/r/minilab/comments/1on90up/first_minilab_only_powered_by_one_laptop_power/
https://www.reddit.com/r/minilab/comments/1ohr6ay/my_new_mini_rack_downsized_from_12u/
https://www.reddit.com/r/minilab/comments/1obaiao/first_minilab/
https://github.com/geerlingguy/mini-rack?tab=readme-ov-file
https://www.reddit.com/r/homelab/comments/1m7onp8/lenovo_thinkcentere_25_gb_ethernet_upgrade/
https://www.reddit.com/r/minilab/comments/1hhm9ar/power_strip_for_10_rack/


#### Seperate Session
https://www.reddit.com/r/minilab/comments/1on32ah/joined_the_club/
https://www.wirelessnetware.ca/blog/why-more-canadians-are-choosing-wireless-netware-for-mikrotik-solutions/
https://imgur.com/a/homelab-v3-Vkc1EBI
https://www.reddit.com/r/sffpc/comments/16jq9hn/jonsbo_n2_replacing_stock_fan_with_nuctoa_nfa12x25/
https://www.reddit.com/r/minilab/comments/1otqsva/home_lab_v3_i_swear_im_done_upgrading/

#### Topology Research 

https://github.com/daviaraujocc/lgtm-stack
https://medium.com/@asingh.etw/building-observability-opentelemetry-platform-with-the-grafana-lgtm-stack-and-agent-88437e94e005
https://github.com/daviaraujocc/lgtm-stack

#### Features to Consider
iSCSI / Diskless Booting (Sanboot)
Local LLM
Keycloak (redhat)


### Usernames
Admin Usernames 
switch = ctr-adm38
Pi = jump-adm64
Proxmox = cluster-adm92
TrueNas = nas-admin11
non-admin = dlaurin


## TERMS + Notes
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



