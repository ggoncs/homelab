# Mini Rack Server - Proxmox Cluster + TrueNAS 

## My Homelab Experience/Story

This is my Second Homelab, direct upgrade of the single Dell Tower hosting on Debian + CasaOS. Though very capable still especially the 32GB DDR4 RAM. I was facing issue with 
data redundancy, backups and storage. I would store all my games incase I wanted to download them again. I filled the 1TB nvme completely full. making it impossible to backup 
with timeshift. At that point I was starting to host all my backups on Proton drive for data security. And it was starting to defeat the hole point of my server

It was time for an upgrade, This time I will get a dedicated NAS to host my games-download, media files, and also a full fledge plex server for movies. Also, a while ago I bought a rasberry pi 5 and a lenovo thinkcentre mini that were collecting dust not being used for anything, as I didin't have enough ethernet ports in the basement to support more than 2 computers (xbox and dell tower) from the eero mesh. It was clear to me that I needed a full fledge server rack with networking. 

On the software side, I was seeing some serious limitation to my setup, CasaOS is super pretty, free and great-gateway drug to IT without needing fork over a money to UGREEN for their NAS. I needed change. CasaOS. it very good at spining up known docker containers though their app menu, but as soon you started experimenting and spinning up you own ones, practicing DNS/Bash scripting/Apache. They started to break, needing me to rebuild them often. CasaOS breaking Nextcloud four seperate time after updating. needing me to zip file and export to new container. I was starting to use Portainer as my dashboard to managed my containers instead of CasaOS... yeah not good

Unrelated Issue, I was having a bit of trouble hosting steam-cli for my games. So I decided to install a GUI (KDE) to just have steam so I can install my games. HUGE issues started to happen. My GPU NvS310 stopped working, and steam was very unhappy that it couldn't use gpu accelration properly, Not only that, because I intalled KDE, it uses a different network manager called "Network Manager" (you don't say) and debian server uses ifupdown. This broke so many things; one of the most annoying is that my containers didin't know anymore what default gateway to use anymore, so they would just make one. Making them not possible to connect to the internet anymore for the server. So everytime I updated and rebooted. I would need physically run these commands: 

- 
- 
- 

So stupid and annoying as I couln't SSH and fix it. Well now I'm embarking a new journay with my now upgraded Homelab!



##### Requirements:

RAID 5 | ✅
8 port managed switch 2.5GBs ✅
bastion host (can be run in a container with opensense) ✅
12U rack server ✅
PFsense (bonus) ❌ 
Buying a firewall with pfsense built-in is really expensive, and honestly overkill for now, also right now at home I am not full admin of the network so I currently do not have need for it.

EoP as my server would be in the basement ❌ 
Since I wanted a managed switch 10inch, there is only two option of well known brands, Mintrokit and QNAP. QNAP supports PoE and 2.5GBs but, is double the price. therefore a deal breaker and buying a PoE+ injector is not that expensive and I don't loose that much perfomance as the only device needing it is Rasberry Pi that only has 1GB/s speed



Requirements Gathering 

- Eero Mesh pod
- Dell Precision 3620 Tower 
- Lenovo ThinkCentre M710Q  
- Raspberry Pi 5 

- The Server is using the Mesh network instead of being connected directly to the router, (Can use EoP) 

- Eero mesh pod only have two ethernet ports which is not enough to connect more devices and giving room to upgrade horizontally (use a switch)

- Current hardware cannot communicate to each other (Can use Proxmox cluster)

- No data redundancy (Use RAID 5+1)

- Ran out of space, couldn't do industry standard 3-2-1 backups (multiple machine to host backups)

- Server location is near a desk in the living room, low noise needs to be a high priority 

- Since expanding the amount of computers on the network, Low idle Watt consomption need to be considered 

- Segregating storage computer with main hardware, as I'm still learning (Use proxmox + TrueNAS)



(Analysis)
###### Current Software 




###### Capable to run these service: 
- proxmox 
- kubernetes + TalOS
- wordpress 
- minecraft 
- ansible
- nginx 
- podman 
- searxng 
- samba 
- steam-cli
- zfs snapshot / rsnapshot



###### Reddit guy info: (not included into the article) 
A bit about software:
- basically everything is configured via ansible
- all services (adguard, rsnapshot, samba, nginx, searxng, baikal, forgejo, syncthing, mc, etc...) are running in rootless podman containers.
- zfs for nas drives 

Logical Design 
Physical Design 

### Top down of the server rack
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


##### U-Tetris Math
1U (4.45 cm)

22.25 (NAS) + 17.8 (Nodes) + 4.45 (Switch) + 4.45 (Patch) + 4.45 (Vent) = 53.4 cm.
Total Rack Height Available (12U): 53.4 cm.

### Software 
Proxmox 
TrueNAS
Kubernetes Cluster + Talos
Active Directory 
Virtualized Linux VMs 
Torr Node 
Nextcloud 
Plex Media Streaming 
Scaphandre. It acts as a Prometheus exporter that tracks power consumption metrics at the process and host level, which you can visualize in Grafana.


#### Current Hardware
Connected to Eero Mesh pod

Dell Precision 3620 Tower i7-7700 Quad Core 3.6Ghz 32B 1TB NVME GPU NvS310 | 419$ amazon.ca | [link][https://www.amazon.ca/Dell-Optiplex-7050-Excellent-Condition/dp/B0F854GHFB/ref=sr_1_1?sr=8-1]

Lenovo ThinkCentre M710Q Tiny Micro i5-7400T 2.4Ghz 8GB RAM with 500GB HDD | 102.14$ [link][https://www.ebay.ca/itm/197901648050?mkrid=711-127632-2357-0&sssrc=4429486&stype=1&var=]

Lenovo ThinkCentre M715Q AMD Pro A10-9700E 8GB 500GB HD | ebay.ca [link][https://www.ebay.ca/itm/155474578754?mkrid=711-127632-2357-0&sssrc=4429486&stype=1&var=]

Raspberry Pi 5 8GB 2023 | 133.69$ | amazon.ca [link][https://www.amazon.ca/dp/B0CK2FCG1K?ref_=cm_sw_r_cso_cp_apan_dp_M2HK0C5AXXZCHN42H14E]

GeeekPi P33 M.2 NVME M-Key PoE+ | 48.98$ | amazon.ca [link][https://www.amazon.ca/dp/B0D8JC3MXQ?ref_=cm_sw_r_cso_cp_apan_dp_TY1G9N40WHAF650DFV1E]

Installation and Maintenance
#### Server
GeeekPi 12U Server Cabinet | 209.94$ | amazon.ca [link][https://www.amazon.ca/GeeekPi-Cabinet-Equipment-RackMate-Rackmount/dp/B0DT2XM22G/ref=sr_1_1_sspa?s=electronics&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1]

GeeekPi 12 Port Patch Panel | 23.79$ | amazon.ca [link][https://www.amazon.ca/dp/B0D5XPNHHF?ref_=cm_sw_r_cso_cp_apan_dp_TMZ63EQ1FNC02H6FC7M3]

GeeekPi 1U Server Rack Shelf | 28.04$ | amazon.ca [link][https://www.amazon.ca/dp/B0D5XGSPXD?ref_=cm_sw_r_cso_cp_apan_dp_TT262C4WDSHKJ7W1254J]

PDU 3 Socket 42.72$ | aliexpress.com [link][https://www.aliexpress.com/item/1005005963996490.html]

3 Prong 1-to-4 Power Cord Splitter Cable | 23.99$ | amazon.ca [link][https://www.amazon.ca/Mikrotik-CRS310-8G-2S-IN-MikroTik/dp/B0CH9NHFHS?source=ps-sl-shoppingads-lpcontext&ref_=fplfs&psc=1&smid=A1DVY69FJDQ80C]

Freenove 7 Inch Touchscreen Monitor | 47.96$ | amazon.ca [link][https://www.amazon.ca/dp/B0BPP6MFFJ?ref_=cm_sw_r_cso_cp_apan_dp_AH6BYX1167Z1GPQD24CM]


SANDISK SSD Plus 500GB Opened Box | 59.27$ | amazon.ca [link][https://www.amazon.ca/dp/B0F4Y2VR8S?ref_=cm_sw_r_cso_cp_apan_dp_KNQYPGB8N57W8ANDHRKR]

SANDISK SSD Plus 500GB | 64.99$ | amazon.ca [link][https://www.amazon.ca/dp/B0F4Y2VR8S?ref_=cm_sw_r_cso_cp_apan_dp_KNQYPGB8N57W8ANDHRKR]


###### Network
MikroTik CRS310-8G+2S+in 276$ | amazon.ca [link][https://www.amazon.ca/Mikrotik-CRS310-8G-2S-IN-MikroTik/dp/B0CH9NHFHS?source=ps-sl-shoppingads-lpcontext&ref_=fplfs&psc=1&smid=A1DVY69FJDQ80C]

Mikrotik RMK-2/10 Rack Mount | 33.52$ | wirelessnetware.ca [link][https://www.shop.wirelessnetware.ca/accessories/516-rmk-210-4752224008688.html]

Mellanox MCX311A-XCAT 10G Ethernet 10GbE SFP+ PCI-E NIC | 33.41$ | ebay.ca [link][https://www.ebay.ca/itm/297438405607?_skw=ConnectX-3+MCX311A&epid=11030189803&itmmeta=01KB8TJV9W92ADMM5SSMV38FNP&hash=item4540b5e3e7:g:xZoAAOSwizdoXkxh&itmprp=enc%3AAQAKAAAA8FkggFvd1GGDu0w3yXCmi1eTct2eNn9h6OhuqC%2FjnAYutFJm2Ep%2Fj8fpI1E%2BFjMXRYG3iWEjEGe6bQDvXEDajpdzkE8HiSsnx5X74b7OYmO0wE4zeD%2BxMUunZeruxNpChEme7uVZ4d1yPbkgebyZnMYtkIn2oq46sg89aw3z0LW76ToXTz9B%2FD9SsT3Bc0Nvo59%2Bl49ONf%2BOt%2BmXnc3k6TANXKgXbvjP2uGduV0m%2BP%2BUy1yPUdIewALMSiK03%2FjDaA5zkGy%2FhSC8XznlbYUAE4jT7HouMpAL6MjRudKGEAQcWboFEMPDyFw%2BJeuB3mgJ4w%3D%3D%7Ctkp%3ABFBMiLXLmtpm]

Gigabit PoE+ Injector 18.88$ | amazon.ca [link][https://www.amazon.ca/Injector-IEEE802-3at-Replacement-TPE-115GI-TL-PoE160S/dp/B00NRF9GQO/ref=asc_df_B00NRF9GQO?mcid=c5b2d3709dca3350af73dc68296d4834&hvadid=706724917389&hvpos=&hvnetw=g&hvrand=14741287802779362230&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9220713&hvtargid=pla-568844063746&psc=1&hvocijid=14741287802779362230-B00NRF9GQO-&hvexpln=0]

2.5Gb Network Adapter M.2 A+E/Mini PCIE to RJ45 | 18.79$ | aliexpress.com [link][https://www.aliexpress.com/item/1005008820106326.html?spm=a2g0o.productlist.main.4.14f3CSoWCSoW7p&algo_pvid=9ea1b4c1-c266-4f9c-a9bd-bf1601744de0&pdp_ext_f=%7B%22order%22%3A%2210%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005008820106326%7C_p_origin_prod%3A]





##### NAS Specs 
Case: JONSBO N2 NAS | 169.05$ | aliexpress.com [link][]
Motherboard + CPU: N5105 Industrial Motherboard | 160.38$ | aliexpress.com [link][https://www.aliexpress.com/item/1005006221619148.html?invitationCode=ZG9ZZGs2a29nUm1rUTRnemdvenZRc2NTMG56SUU2RytUNDgxazRGL2NkS2VQemFTZUJrNWVWT0s1MU1hdTAyWg&srcSns=sns_Copy&spreadType=socialShare&social_params=21997576728&bizType=ProductDetail&spreadCode=ZG9ZZGs2a29nUm1rUTRnemdvenZRc2NTMG56SUU2RytUNDgxazRGL2NkS2VQemFTZUJrNWVWT0s1MU1hdTAyWg&aff_fcid=11d34d826be44bdca3ce4ea5b11fa659-1764475371352-07846-_mtpTlVJ&tt=MG&aff_fsk=_mtpTlVJ&aff_platform=default&sk=_mtpTlVJ&aff_trace_key=11d34d826be44bdca3ce4ea5b11fa659-1764475371352-07846-_mtpTlVJ&shareId=21997576728&businessType=ProductDetail&platform=AE&terminal_id=5a00cd0046d34917859afd9a27ad68fd&afSmartRedirect=y]


Case Fan: Noctua NF-A9 PWM | 25.05$  | amazon.ca [link][https://www.amazon.ca/dp/B00RUZ059O?ref_=cm_sw_r_cso_cp_apan_dp_TNAJA8BGTY87N7G4GYGD]


CPU Fan: Noctua NF-A4x20 | 29.79$ | aliexpress.com [link][https://www.aliexpress.com/item/1005006690066510.html?invitationCode=ZG9ZZGs2a29nUm1BaG9tTldBMThUTWNTMG56SUU2RytUNDgxazRGL2NkS2VQemFTZUJrNWVWT0s1MU1hdTAyWg&srcSns=sns_Copy&spreadType=socialShare&social_params=21993743625&bizType=ProductDetail&spreadCode=ZG9ZZGs2a29nUm1BaG9tTldBMThUTWNTMG56SUU2RytUNDgxazRGL2NkS2VQemFTZUJrNWVWT0s1MU1hdTAyWg&aff_fcid=a7ed093179cd4545aa99603d06564bc5-1764475372039-00341-_mNTMNy9&tt=MG&aff_fsk=_mNTMNy9&aff_platform=default&sk=_mNTMNy9&aff_trace_key=a7ed093179cd4545aa99603d06564bc5-1764475372039-00341-_mNTMNy9&shareId=21993743625&businessType=ProductDetail&platform=AE&terminal_id=5a00cd0046d34917859afd9a27ad68fd&afSmartRedirect=y]


PSU: GAMEMAX 650W 80+ Gold Fully Modular SFX | 144.49$ | amazon.ca [link][https://www.amazon.ca/dp/B0F9NR7G17?ref_=cm_sw_r_cso_cp_apan_dp_PH67FNBZWJ218P1PK691]

Memory: Crucial 8GBx2 DDR4 3200 MHz SO-DIMM | 93.69$ | ebay.ca  [link][https://www.ebay.ca/itm/365735557774?mkrid=711-127632-2357-0&sssrc=4429486&stype=1&var=635510335212]

Sata Cables: 6pcs Cable | 6.79$ | aliexpress.com [link][https://www.aliexpress.com/item/1005007523507363.html?invitationCode=ZG9ZZGs2a29nUmxjRVY1UDFjN0dOY2NTMG56SUU2RytUNDgxazRGL2NkS2VQemFTZUJrNWVWT0s1MU1hdTAyWg&srcSns=sns_Copy&spreadType=socialShare&social_params=21993743699&bizType=ProductDetail&spreadCode=ZG9ZZGs2a29nUmxjRVY1UDFjN0dOY2NTMG56SUU2RytUNDgxazRGL2NkS2VQemFTZUJrNWVWT0s1MU1hdTAyWg&aff_fcid=707041ef4f354b0ebe1121e792c1b2e9-1764475372743-01332-_mOqgIE5&tt=MG&aff_fsk=_mOqgIE5&aff_platform=default&sk=_mOqgIE5&aff_trace_key=707041ef4f354b0ebe1121e792c1b2e9-1764475372743-01332-_mOqgIE5&shareId=21993743699&businessType=ProductDetail&platform=AE&terminal_id=5a00cd0046d34917859afd9a27ad68fd&afSmartRedirect=y]

Hard Drive 1TB 3.5inch | 25$ | MDG Computers 

1TB NVMe SSD M.2 SSD PCIe 3.0x4 Read 3,500MB/s | 107.99$ | amazon.ca [link][https://www.amazon.ca/dp/B0DTY976K3?ref_=cm_sw_r_cso_cp_apan_dp_ESB6ZQZRBSZYSZWYZ2DC]










