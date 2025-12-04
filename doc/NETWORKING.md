# 🌐 Network Configuration

Complete network topology, VLAN configuration, and firewall rules.

---

## 📊 Network Overview

```
Internet → ISP Router (AP Mode) → N150 Firewall → MikroTik Switch → Devices
                                       ↓
                                   VLANs: Management, LAN, DMZ, Storage
```

---

## 🏷️ VLAN Design

| VLAN ID | Name | Subnet | Gateway | Purpose |
|---------|------|--------|---------|---------|
| 1 | Native/Unused | - | - | Disabled for security |
| 10 | Management | 10.0.99.0/24 | 10.0.99.1 | Infrastructure management |
| 20 | LAN | 192.168.1.0/24 | 192.168.1.1 | User devices, WiFi |
| 30 | DMZ | 10.0.10.0/24 | 10.0.10.1 | Public-facing services |
| 40 | Storage | 10.0.40.0/24 | 10.0.40.1 | NAS, iSCSI, NFS traffic |

---

## 🖥️ IP Address Allocation

### Management VLAN (10.0.99.0/24)

| Device | IP | Hostname | Notes |
|--------|----|----- |-------|
| Firewall | 10.0.99.1 | fw.homelab.local | Gateway |
| MikroTik Switch | 10.0.99.2 | switch.homelab.local | Management |
| Proxmox Node 1 | 10.0.99.11 | pve-node1.homelab.local | AMD A10-9700E |
| Proxmox Node 2 | 10.0.99.12 | pve-node2.homelab.local | i5-7400T |
| Proxmox Node 3 | 10.0.99.13 | pve-node3.homelab.local | i5-7500T |
| Proxmox Node 4 | 10.0.99.14 | pve-node4.homelab.local | i5-7500T |
| Raspberry Pi | 10.0.99.20 | pi-monitor.homelab.local | Monitoring |
| Reserved | 10.0.99.30-99 | - | Future devices |

### LAN VLAN (192.168.1.0/24)

| Device | IP | Hostname | Notes |
|--------|----|----- |-------|
| Gateway | 192.168.1.1 | - | Firewall interface |
| DHCP Pool | 192.168.1.100-200 | - | Dynamic assignments |
| Reserved | 192.168.1.10-50 | - | Static client IPs |

### DMZ VLAN (10.0.10.0/24)

| Device | IP | Hostname | Notes |
|--------|----|----- |-------|
| Gateway | 10.0.10.1 | - | Firewall interface |
| Web Server | 10.0.10.10 | web.homelab.local | Nginx/WordPress |
| VPN Server | 10.0.10.11 | vpn.homelab.local | WireGuard |
| Reserved | 10.0.10.20-100 | - | Public services |

### Storage VLAN (10.0.40.0/24)

| Device | IP | Hostname | Notes |
|--------|----|----- |-------|
| Gateway | 10.0.40.1 | - | Firewall interface |
| TrueNAS | 10.0.40.10 | truenas.homelab.local | NAS |
| Reserved | 10.0.40.20-50 | - | Future storage |

---

## 🔥 Firewall Configuration (pfSense/OPNsense)

### Interface Assignments

```bash
# WAN Interface
igb0 → DHCP from ISP (Xplorer)
# ISP provides public IP

# LAN Interface  
igb1 → 192.168.1.1/24 (VLAN 20)

# Management Interface
igb2 → 10.0.99.1/24 (VLAN 10)

# DMZ Interface
igb3 → 10.0.10.1/24 (VLAN 30)
```

### Firewall Rules

#### WAN Rules
```bash
# Block all inbound by default
# Allow established/related connections
# Allow WireGuard VPN (UDP 51820)

Rule 1: Block All
Action: Block
Protocol: Any
Source: Any
Destination: Any

Rule 2: Allow VPN
Action: Pass
Protocol: UDP
Source: Any
Destination: WAN Address
Port: 51820
```

#### LAN Rules
```bash
# Allow LAN to Management (for web UIs)
Rule 1: LAN to Management
Action: Pass
Protocol: Any
Source: 192.168.1.0/24
Destination: 10.0.99.0/24

# Allow LAN to Storage (SMB/NFS)
Rule 2: LAN to Storage
Action: Pass
Protocol: Any
Source: 192.168.1.0/24
Destination: 10.0.40.0/24

# Allow LAN to Internet
Rule 3: LAN to WAN
Action: Pass
Protocol: Any
Source: 192.168.1.0/24
Destination: Any
```

#### Management Rules
```bash
# Allow Management to Everything (admin access)
Rule 1: Management to Any
Action: Pass
Protocol: Any
Source: 10.0.99.0/24
Destination: Any

# Deny other VLANs to Management
Rule 2: Block to Management
Action: Block
Protocol: Any
Source: !10.0.99.0/24
Destination: 10.0.99.0/24
```

#### DMZ Rules
```bash
# Allow DMZ to Internet only
Rule 1: DMZ to WAN
Action: Pass
Protocol: Any
Source: 10.0.10.0/24
Destination: WAN

# Block DMZ to all internal networks
Rule 2: Block DMZ to Internal
Action: Block
Protocol: Any
Source: 10.0.10.0/24
Destination: 10.0.0.0/8, 192.168.0.0/16
```

#### Storage Rules
```bash
# Allow Storage to Management (for monitoring)
Rule 1: Storage to Management
Action: Pass
Protocol: Any
Source: 10.0.40.0/24
Destination: 10.0.99.0/24

# Block Storage to other VLANs
Rule 2: Isolate Storage
Action: Block
Protocol: Any
Source: 10.0.40.0/24
Destination: !10.0.99.0/24
```

### NAT / Port Forwarding

```bash
# WireGuard VPN
WAN:51820 → 10.0.10.11:51820 (UDP)

# HTTPS (if hosting public services)
WAN:443 → 10.0.10.10:443 (TCP)

# [Add other port forwards as needed]
```

---

## 🔧 MikroTik Switch Configuration

### Initial Setup
```bash
# Access via WinBox or WebFig
# Default IP: 192.168.88.1
# Change to: 10.0.99.2

/ip address
add address=10.0.99.2/24 interface=bridge

# Update admin password
/user set admin password=<strong_password>
```

### VLAN Configuration

```routeros
# Create bridge for all ports
/interface bridge
add name=bridge1 vlan-filtering=yes

# Add all ports to bridge
/interface bridge port
add bridge=bridge1 interface=sfp-sfpplus1 pvid=10
add bridge=bridge1 interface=sfp-sfpplus2 pvid=10
add bridge=bridge1 interface=ether1 pvid=10
add bridge=bridge1 interface=ether2 pvid=10
add bridge=bridge1 interface=ether3 pvid=40
add bridge=bridge1 interface=ether4 pvid=99
add bridge=bridge1 interface=ether5 pvid=20
add bridge=bridge1 interface=ether6 pvid=20
add bridge=bridge1 interface=ether7 pvid=30
add bridge=bridge1 interface=ether8 pvid=20

# Create VLANs
/interface vlan
add interface=bridge1 name=vlan10-mgmt vlan-id=10
add interface=bridge1 name=vlan20-lan vlan-id=20
add interface=bridge1 name=vlan30-dmz vlan-id=30
add interface=bridge1 name=vlan40-storage vlan-id=40

# Tag VLANs on bridge
/interface bridge vlan
add bridge=bridge1 tagged=sfp-sfpplus1,sfp-sfpplus2,ether1 untagged=ether4 vlan-ids=10
add bridge=bridge1 tagged=sfp-sfpplus1,sfp-sfpplus2,ether1 untagged=ether5,ether6,ether8 vlan-ids=20
add bridge=bridge1 tagged=sfp-sfpplus1,sfp-sfpplus2,ether1 untagged=ether7 vlan-ids=30
add bridge=bridge1 tagged=sfp-sfpplus1,sfp-sfpplus2,ether1 untagged=ether3 vlan-ids=40
```

### Port Assignment

| Port | VLAN | Purpose | Speed |
|------|------|---------|-------|
| ether1 | Trunk | → Firewall (all VLANs) | 2.5GbE |
| ether2 | 10 (Tagged) | → Proxmox Node 1 | 2.5GbE |
| ether3 | 40 (Untagged) | → TrueNAS | 2.5GbE |
| ether4 | 99 (Untagged) | → Raspberry Pi (PoE) | 2.5GbE |
| ether5 | 20 (Untagged) | → ISP Router (AP mode) | 2.5GbE |
| ether6 | 20 (Untagged) | Reserved LAN | 2.5GbE |
| ether7 | 30 (Untagged) | Reserved DMZ | 2.5GbE |
| ether8 | 20 (Untagged) | Reserved LAN | 2.5GbE |
| sfp-sfpplus1 | Trunk | Reserved (future 10GbE) | 10GbE SFP+ |
| sfp-sfpplus2 | Trunk | Reserved (future 10GbE) | 10GbE SFP+ |

---

## 🛡️ DNS Configuration (Pi-Hole)

### Installation
```bash
# Install Pi-Hole on Raspberry Pi or VM
curl -sSL https://install.pi-hole.net | bash

# Set static IP: 10.0.99.20
# Web interface: http://10.0.99.20/admin
```

### Custom DNS Records
```bash
# Add local domain records
# Via Pi-Hole web UI → Local DNS → DNS Records

10.0.99.11    pve-node1.homelab.local
10.0.99.12    pve-node2.homelab.local
10.0.99.13    pve-node3.homelab.local
10.0.99.14    pve-node4.homelab.local
10.0.99.20    pi-monitor.homelab.local
10.0.40.10    truenas.homelab.local
10.0.10.10    web.homelab.local
192.168.1.1   fw.homelab.local
```

### Upstream DNS
```bash
# Configure upstream DNS servers
1.1.1.1          # Cloudflare
1.0.0.1          # Cloudflare secondary
9.9.9.9          # Quad9 (privacy-focused)
```

### Blocklists
```bash
# Default blocklists + additional ones
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
https://v.firebog.net/hosts/lists.php?type=tick
```

---

## 🔐 VPN Configuration (WireGuard)

### Server Setup (on Firewall or VM)
```bash
# Generate server keys
wg genkey | tee privatekey | wg pubkey > publickey
```

**Server config:** `/etc/wireguard/wg0.conf`
```ini
[Interface]
Address = 10.0.50.1/24
ListenPort = 51820
PrivateKey = <server_private_key>

# Client 1 (Phone)
[Peer]
PublicKey = <client1_public_key>
AllowedIPs = 10.0.50.2/32

# Client 2 (Laptop)
[Peer]
PublicKey = <client2_public_key>
AllowedIPs = 10.0.50.3/32
```

```bash
# Start WireGuard
wg-quick up wg0
systemctl enable wg-quick@wg0
```

### Client Config Example
```ini
[Interface]
PrivateKey = <client_private_key>
Address = 10.0.50.2/32
DNS = 10.0.99.20  # Pi-Hole

[Peer]
PublicKey = <server_public_key>
Endpoint = <public_ip>:51820
AllowedIPs = 10.0.0.0/8, 192.168.1.0/24  # Route homelab traffic
PersistentKeepalive = 25
```

---

## 📈 Network Performance

### Expected Throughput
```bash
# Internet (ISP limited)
Download: ~500 Mbps
Upload: ~50 Mbps

# Internal (2.5GbE limited)
LAN → Storage: ~2.5 Gbps (312 MB/s)
Node → Node: ~2.5 Gbps (312 MB/s)

# Storage (RAID 5 limited)
TrueNAS Read: ~200-300 MB/s
TrueNAS Write: ~150-200 MB/s
```

### Latency Tests
```bash
# From workstation
ping 192.168.1.1   # Firewall: <1ms
ping 10.0.99.11    # Proxmox: <1ms
ping 10.0.40.10    # TrueNAS: <1ms
```

---

## 🆘 Troubleshooting

### VLAN Issues
```routeros
# Check VLAN interfaces on switch
/interface vlan print

# Verify VLAN tagging
/interface bridge vlan print
```

```bash
# Test VLAN connectivity from Linux
ping -I vlan10 10.0.99.1

# Verify VLAN tagging with tcpdump
tcpdump -i eth0 -e vlan
```

### Firewall Issues
```bash
# Check firewall logs
tail -f /var/log/filter.log

# Test port forwarding
nc -zv <public_ip> 51820

# Verify NAT rules
pfctl -sn
```

### DNS Issues
```bash
# Test DNS resolution
nslookup pve-node1.homelab.local 10.0.99.20

# Check Pi-Hole logs
tail -f /var/log/pihole.log

# Flush DNS cache (on clients)
sudo systemd-resolve --flush-caches
```

---

## 🔗 References

- [MikroTik VLAN Guide](https://wiki.mikrotik.com/wiki/Manual:Interface/VLAN)
- [pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)
- [WireGuard Quick Start](https://www.wireguard.com/quickstart/)

---

**Last Updated:** [DATE]  
**Network uptime:** [DAYS]
