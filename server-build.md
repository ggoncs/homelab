## Current Spend (Not Accurate)
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




#### Miscellaneous INFO
473$ for the xmas-list + rj45 headers
Netgear ProSafe GS724T V2 24-Port Gigabit Smart Switch | 43.06$
Netgear Router AC1750, 1750Mbps
Xplornet Hub ZTE ZXHN H298N 
Eero 6+ Mesh - Wi-Fi 6 (AX3000) 1GB/s
Netgear R6200 Wifi Router 803.11ac 1200Mibs


#### Features I can't use
iSCSI / Diskless Booting (Sanboot)
Local LLM

#### Storage
##### SATA SSD

SanDisk 500GB
SanDisk 500GB
SanDisk 500GB
Patriot 128GB 
Kingston 128GB 
Cusu 2TB

##### NVMe
RP3500 1TB 
MSI 1TB 
G15 512GB
Patriot 128GB 
Cusu 2TB


##### High Availability (HA) 
3-2-1 Rule
Uninterruptible Power Suppy (UPS)
Proxmox Backup Server (PBS)
Network UPS Tools (NUT)
Data (Offsite Backup) (proton)
DNS Reduncancy (Pi-Hole) 



### Logical Design 
Eero (Router) 
DMZ (Firewall)
VLAN Tagging (Switch)
Raspberry-Pi (jump host)



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

### My Network

Eero (WAN)
DMZ (Firewall)
Microtik (Layer 3 managed switch)
Raspberry-Pi (jump host)



### Research 
https://loganmarchione.com/2022/09/homelab-10-mini-rack-shelves/
https://linuxblog.io/home-lab-beginners-guide-hardware/
https://linuxblog.io/pfsense-firewall-config-settings/
https://www.youtube.com/watch?v=lUzSsX4T4WQ
https://www.cdw.ca/product/netgear-gs308e-ethernet-switch/8434280
https://www.amazon.ca/UBIQUITI-ER-X-SFP-Edgerouter-X-Sfp/dp/B012X45WH6



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


##### Drive Shuffle 
G15 laptop 1TB 
NAS 120GB SATA + 1TB 
ThinkCentre Intel 512GB SATA
ThinkCentre Intel 512GB SATA
ThinkCentre Intel 512GB SATA
ThinkCentre AMD 128GB SATA + 512GB Nvme (don't have)
Pi 5 512GB 
N150 128GB 
Dell Hardrives 500GB HD



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


### Other Software (INFO DUMP)
glusterfs, ha,

I'm running a single Proxmox unprivileged LXC container using Docker to run the following services: Plex, Overseerr, Radarr, Sonarr, Prowlarr, SABnzb, qBittorrent, Bazarr, Tuatulli, Bookshelf (Readarr fork), Calibre. Happy to share the docker compose file to anyone who requests it (this was easily the longest and hardest part of the project, getting all the services working together and automated, especially Readarr and Calibre).

###### Reddit guy info: (not included into the article) 
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



https://www.reddit.com/r/minilab/comments/1on32ah/joined_the_club/
https://www.wirelessnetware.ca/blog/why-more-canadians-are-choosing-wireless-netware-for-mikrotik-solutions/
https://imgur.com/a/homelab-v3-Vkc1EBI
https://www.reddit.com/r/sffpc/comments/16jq9hn/jonsbo_n2_replacing_stock_fan_with_nuctoa_nfa12x25/
https://www.reddit.com/r/minilab/comments/1otqsva/home_lab_v3_i_swear_im_done_upgrading/


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



## Fixes needed from Claude

For some reason I don't see I don't see the network config file you made? can you re display it?

Can you change the order of the rack layout 

the screen is attached to Pi! 
so raberry Pi and firewall are on top (there is only 1 Pi)
You forgot the N150 firewall (it is sitting on top also)
Remove the Eero Mesh pod from the diagram
the NAS is 5U total 
my thinkcenter nodes 
3 intel and at the bottom would be amd one

Add this to Mobile section
PAM + Aegis 2FA

Software to the table list 
XFC
LACE

When would I use cloud-init for this homelab 
is it used in the inital setup, or when I spin up virtual machines after? 

So for my backup solution 
TrueNAS  
ZFS Replication (only replicating snapshots)
And its encrypted (NAS and the PBS)
Pull replication
(no rsnapshot)

add r/minilab - Used it for hardware inspiration


I'm trying to use ansible, so do factor that in the template files 
(or is it better to do everything manually to learn the commands)

Ansible:
Phase 4	Clustering/Networking (Proxmox)
Phase 5-6	OS and Services (LXC/VM)	
Ongoing	Consistency/Updates	
