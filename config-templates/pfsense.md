# 🔥 pfSense Firewall Configuration Template

Complete configuration guide for N150 4x 2.5GbE pfSense firewall.

---

## 🖥️ Hardware Overview

**N150 Mini PC:**
- **CPU:** Intel N150 (fanless)
- **RAM:** 8GB DDR4
- **Storage:** 128GB NVMe SSD
- **NICs:** 4x Intel 2.5GbE ports

**Port Assignment:**
- **igb0:** WAN (to Eero mesh - 192.168.4.0/22)
- **igb1:** LAN trunk (to MikroTik - all VLANs tagged)
- **igb2:** Unused (future expansion)
- **igb3:** Unused (future expansion)

---

## 📥 Initial Installation

### 1. Download pfSense
```bash
# Download latest pfSense CE from:
https://www.pfsense.org/download/

# Version: Community Edition (CE)
# Architecture: AMD64 (64-bit)
# Installer: USB Memstick Installer
# Console: VGA
# Mirror: Choose closest location
```

### 2. Flash to USB
```bash
# On Linux/Mac:
sudo dd if=pfSense-CE-X.X-RELEASE-amd64.img of=/dev/sdX bs=4M status=progress

# On Windows: Use Rufus or Etcher
```

### 3. Install to N150
```bash
# Boot from USB
# Follow installer prompts:
# - Accept license
# - Install pfSense
# - Select 128GB NVMe as destination
# - Reboot

# Initial console setup:
# Should VLANs be set up now? n (we'll do this later)
# WAN interface: igb0
# LAN interface: igb1
# Enter LAN IPv4 address: 10.0.10.1
# Enter LAN subnet: 24
# Enable DHCP on LAN? n (we'll configure later)
```

---

## 🌐 Interface Configuration

### Access Web Interface
```bash
# From a device on the LAN (plugged into MikroTik):
https://10.0.10.1

# Default credentials:
Username: admin
Password: pfsense
```

### Configure WAN Interface
```bash
# Interfaces → WAN
Type: DHCP
MTU: 1500
Block private networks: UNCHECKED (you're behind another router)
Block bogon networks: CHECKED
```

### Create VLAN Interfaces
```bash
# Interfaces → Assignments → VLANs → Add

VLAN 10 (Management):
Parent Interface: igb1
VLAN Tag: 10
Description: VLAN10-Management

VLAN 20 (Users):
Parent Interface: igb1
VLAN Tag: 20
Description: VLAN20-Users

VLAN 30 (Services):
Parent Interface: igb1
VLAN Tag: 30
Description: VLAN30-Services

VLAN 40 (Storage):
Parent Interface: igb1
VLAN Tag: 40
Description: VLAN40-Storage

VLAN 50 (Experimental):
Parent Interface: igb1
VLAN Tag: 50
Description: VLAN50-Experimental
```

### Assign VLAN Interfaces
```bash
# Interfaces → Assignments → Add each VLAN

# Then configure each VLAN interface:

# OPT1 → Rename to "MANAGEMENT"
Interfaces → OPT1
Enable: CHECKED
Description: MANAGEMENT
IPv4 Configuration Type: Static IPv4
IPv4 Address: 10.0.10.1 / 24

# OPT2 → Rename to "USERS"
Interfaces → OPT2
Enable: CHECKED
Description: USERS
IPv4 Configuration Type: Static IPv4
IPv4 Address: 10.0.20.1 / 24

# OPT3 → Rename to "SERVICES"
Interfaces → OPT3
Enable: CHECKED
Description: SERVICES
IPv4 Configuration Type: Static IPv4
IPv4 Address: 10.0.30.1 / 24

# OPT4 → Rename to "STORAGE"
Interfaces → OPT4
Enable: CHECKED
Description: STORAGE
IPv4 Configuration Type: Static IPv4
IPv4 Address: 10.0.40.1 / 24

# OPT5 → Rename to "EXPERIMENTAL"
Interfaces → OPT5
Enable: CHECKED
Description: EXPERIMENTAL
IPv4 Configuration Type: Static IPv4
IPv4 Address: 10.0.50.1 / 24
```

---

## 🔧 DHCP Server Configuration

### Management VLAN (Static IPs Only)
```bash
# Services → DHCP Server → MANAGEMENT
Enable: UNCHECKED (all management IPs are static)
```

### Users VLAN
```bash
# Services → DHCP Server → USERS
Enable: CHECKED
Range: 10.0.20.100 to 10.0.20.200
DNS Servers: 10.0.30.10, 10.0.10.30 (Pi-Hole primary and secondary)
Gateway: 10.0.20.1
Domain name: homelab.local
```

### Services VLAN
```bash
# Services → DHCP Server → SERVICES
Enable: UNCHECKED (VMs get static IPs)
```

### Storage VLAN
```bash
# Services → DHCP Server → STORAGE
Enable: UNCHECKED (storage devices get static IPs)
```

### Experimental VLAN
```bash
# Services → DHCP Server → EXPERIMENTAL
Enable: CHECKED
Range: 10.0.50.100 to 10.0.50.200
DNS Servers: 10.0.30.10, 10.0.10.30
Gateway: 10.0.50.1
Domain name: homelab.local
```

---

## 🛡️ Firewall Rules

### WAN Rules
```bash
# Firewall → Rules → WAN

# Rule 1: Block all inbound (default)
Action: Block
Protocol: Any
Source: Any
Destination: Any
Description: Block all inbound

# Rule 2: Allow WireGuard
Action: Pass
Protocol: UDP
Source: Any
Destination: WAN address
Destination Port: 51820
Description: WireGuard VPN

# Rule 3: Allow established connections
Action: Pass
Protocol: Any
Source: Any
Destination: Any
Advanced: State Type = keep state
Description: Allow established connections
```

### MANAGEMENT Rules
```bash
# Firewall → Rules → MANAGEMENT

# Rule 1: Allow Management to Any
Action: Pass
Protocol: Any
Source: 10.0.10.0/24
Destination: Any
Description: Full admin access

# Rule 2: Allow pfSense WebGUI access
Action: Pass
Protocol: TCP
Source: 10.0.10.0/24
Destination: 10.0.10.1
Destination Port: 443
Description: pfSense WebGUI

# Rule 3: Allow DNS
Action: Pass
Protocol: TCP/UDP
Source: 10.0.10.0/24
Destination: 10.0.30.10, 10.0.10.30
Destination Port: 53
Description: DNS to Pi-Hole
```

### USERS Rules
```bash
# Firewall → Rules → USERS

# Rule 1: Allow DNS
Action: Pass
Protocol: TCP/UDP
Source: 10.0.20.0/24
Destination: 10.0.30.10, 10.0.10.30
Destination Port: 53
Description: DNS queries

# Rule 2: Allow to Services
Action: Pass
Protocol: Any
Source: 10.0.20.0/24
Destination: 10.0.30.0/24
Description: Access to Jellyfin, web services

# Rule 3: Allow SMB/NFS to Storage
Action: Pass
Protocol: TCP/UDP
Source: 10.0.20.0/24
Destination: 10.0.40.0/24
Destination Port: 445, 139, 2049, 111
Description: File shares

# Rule 4: Allow Internet
Action: Pass
Protocol: Any
Source: 10.0.20.0/24
Destination: Any
Description: Internet access

# Rule 5: Block to Management
Action: Block
Protocol: Any
Source: 10.0.20.0/24
Destination: 10.0.10.0/24
Description: No access to management

# Rule 6: Block to Experimental
Action: Block
Protocol: Any
Source: 10.0.20.0/24
Destination: 10.0.50.0/24
Description: No access to experimental
```

### SERVICES Rules
```bash
# Firewall → Rules → SERVICES

# Rule 1: Allow to Storage
Action: Pass
Protocol: Any
Source: 10.0.30.0/24
Destination: 10.0.40.0/24
Description: Services access storage

# Rule 2: Allow to Management (monitoring)
Action: Pass
Protocol: Any
Source: 10.0.30.0/24
Destination: 10.0.10.0/24
Description: Monitoring access

# Rule 3: Allow Internet
Action: Pass
Protocol: Any
Source: 10.0.30.0/24
Destination: Any
Description: Updates and external services

# Rule 4: Block to Users
Action: Block
Protocol: Any
Source: 10.0.30.0/24
Destination: 10.0.20.0/24
Description: Isolate from users

# Rule 5: Block to Experimental
Action: Block
Protocol: Any
Source: 10.0.30.0/24
Destination: 10.0.50.0/24
Description: Isolate from experimental
```

### STORAGE Rules
```bash
# Firewall → Rules → STORAGE

# Rule 1: Allow to Management
Action: Pass
Protocol: Any
Source: 10.0.40.0/24
Destination: 10.0.10.0/24
Description: Storage monitoring and backups

# Rule 2: Allow Internet (cloud sync, updates)
Action: Pass
Protocol: Any
Source: 10.0.40.0/24
Destination: Any
Description: Cloud backup and updates

# Rule 3: Block to all other VLANs
Action: Block
Protocol: Any
Source: 10.0.40.0/24
Destination: 10.0.20.0/24, 10.0.30.0/24, 10.0.50.0/24
Description: Isolate storage
```

### EXPERIMENTAL Rules
```bash
# Firewall → Rules → EXPERIMENTAL

# Rule 1: Allow Internet ONLY
Action: Pass
Protocol: Any
Source: 10.0.50.0/24
Destination: Any
Description: Experimental to Internet

# Rule 2: Block ALL internal networks
Action: Block
Protocol: Any
Source: 10.0.50.0/24
Destination: 10.0.0.0/8
Description: Complete isolation from homelab

# Rule 3: Allow Management to Experimental
Action: Pass
Protocol: Any
Source: 10.0.10.0/24
Destination: 10.0.50.0/24
Description: Admin access to experimental
```

---

## 🔄 NAT Configuration

### Outbound NAT
```bash
# Firewall → NAT → Outbound
Mode: Automatic outbound NAT rule generation (default)

# This will automatically NAT all internal traffic to WAN
```

### Port Forwarding (Inbound NAT)
```bash
# Firewall → NAT → Port Forward

# WireGuard VPN to Raspberry Pi
Interface: WAN
Protocol: UDP
Destination: WAN address
Destination Port: 51820
Redirect Target IP: 10.0.10.30 (Raspberry Pi)
Redirect Target Port: 51820
Description: WireGuard VPN to Pi

# Optional: HTTPS to Nginx reverse proxy
Interface: WAN
Protocol: TCP
Destination: WAN address
Destination Port: 443
Redirect Target IP: 10.0.30.25
Redirect Target Port: 443
Description: HTTPS to reverse proxy

# Optional: SSH to jump host
Interface: WAN
Protocol: TCP
Destination: WAN address
Destination Port: 2222
Redirect Target IP: 10.0.10.30
Redirect Target Port: 22
Description: SSH to Raspberry Pi
```

---

## 🔐 WireGuard VPN Configuration

### Install WireGuard Package
```bash
# System → Package Manager → Available Packages
Search: WireGuard
Install: pfSense-pkg-WireGuard

# After install, go to:
# VPN → WireGuard
```

### Create WireGuard Tunnel
```bash
# VPN → WireGuard → Tunnels → Add Tunnel

Enabled: CHECKED
Description: Homelab VPN
Listen Port: 51820
Interface Keys: Generate (click to auto-generate)

# Note the PUBLIC KEY - you'll need this for clients

Interface Addresses: 10.0.60.1/24
# This creates a new subnet just for VPN clients
```

### Create WireGuard Peers (Clients)

**Client 1 - Laptop:**
```bash
# VPN → WireGuard → Peers → Add Peer

Enabled: CHECKED
Tunnel: Homelab VPN
Description: Laptop
Dynamic Endpoint: CHECKED (client's IP changes)
Public Key: <paste_client_public_key>
Allowed IPs: 10.0.60.2/32
```

**Client 2 - Phone:**
```bash
Enabled: CHECKED
Tunnel: Homelab VPN
Description: Phone
Dynamic Endpoint: CHECKED
Public Key: <paste_client_public_key>
Allowed IPs: 10.0.60.3/32
```

### WireGuard Interface Assignment
```bash
# Interfaces → Assignments
Available network ports: wg0 (Homelab VPN)
Click: Add

# Interfaces → WG0
Enable: CHECKED
Description: WireGuard
IPv4 Configuration Type: None (already configured in tunnel)
```

### WireGuard Firewall Rules
```bash
# Firewall → Rules → WireGuard

# Rule 1: Allow VPN clients to entire homelab
Action: Pass
Protocol: Any
Source: 10.0.60.0/24
Destination: 10.0.0.0/8
Description: VPN full access to homelab

# Rule 2: Allow VPN clients to Internet
Action: Pass
Protocol: Any
Source: 10.0.60.0/24
Destination: Any
Description: VPN to Internet
```

### WireGuard Client Configuration Files

**Laptop config (`laptop.conf`):**
```ini
[Interface]
PrivateKey = <laptop_private_key>
Address = 10.0.60.2/32
DNS = 10.0.30.10

[Peer]
PublicKey = <pfsense_public_key>
Endpoint = <your_ddns_hostname_or_ip>:51820
AllowedIPs = 10.0.0.0/8, 192.168.4.0/22
# Routes homelab and home network through VPN
PersistentKeepalive = 25
```

**Phone config (QR code or file):**
```ini
[Interface]
PrivateKey = <phone_private_key>
Address = 10.0.60.3/32
DNS = 10.0.30.10

[Peer]
PublicKey = <pfsense_public_key>
Endpoint = <your_ddns_hostname_or_ip>:51820
AllowedIPs = 10.0.0.0/8
# Only route homelab through VPN (split tunnel)
PersistentKeepalive = 25
```

### Generate Client Keys (on Linux/Mac)
```bash
# Install WireGuard tools
# Ubuntu/Debian:
sudo apt install wireguard-tools

# Generate keys for each client
wg genkey | tee laptop_privatekey | wg pubkey > laptop_publickey
wg genkey | tee phone_privatekey | wg pubkey > phone_publickey

# Use private key in client config
# Use public key in pfSense peer config
```

---

## 🌍 Dynamic DNS Configuration

### Option 1: DDNS on pfSense (Recommended)

```bash
# Services → Dynamic DNS → Add

Service Type: Cloudflare / No-IP / DuckDNS (choose your provider)
Interface to monitor: WAN
Hostname: homelab.yourdomain.com
Username: <your_username_or_email>
Password: <api_token>
Description: Homelab DDNS

# Test: Click the refresh icon to force update
```

### Option 2: DDNS on Raspberry Pi (Alternative)

```bash
# If pfSense doesn't support your provider
# Install ddclient on Raspberry Pi instead
# See network config for details
```

---

## ⚙️ System Settings

### General Setup
```bash
# System → General Setup

Hostname: fw
Domain: homelab.local
DNS Servers: 
  - 10.0.30.10 (Pi-Hole primary)
  - 10.0.10.30 (Pi-Hole secondary)
  
Timezone: America/Toronto (adjust for your location)
```

### Advanced Settings
```bash
# System → Advanced → Admin Access

Protocol: HTTPS
SSL/TLS Certificate: WebGUI default
TCP Port: 443
WebGUI Redirect: CHECKED (auto-redirect HTTP to HTTPS)

# Security
Anti-lockout rule: CHECKED (prevents accidental lockout)
DNS Rebinding Checks: UNCHECKED (allows homelab.local access)

# SSH Access (optional, for CLI management)
Secure Shell Server: CHECKED
SSH Port: 22
Authentication Method: Public Key Only (after setting up keys)
```

### Notifications (Email Alerts)
```bash
# System → Advanced → Notifications

E-Mail Server: smtp.gmail.com
SMTP Port: 587
Secure SMTP Connection: Enable STARTTLS
From e-mail address: your-email@gmail.com
Notification E-Mail address: your-email@gmail.com
Notification E-Mail auth username: your-email@gmail.com
Notification E-Mail auth password: <app-specific-password>

Test: Send Test E-Mail
```

---

## 📊 Monitoring & Logging

### Enable Logging
```bash
# Status → System Logs → Settings

Log Message Format: syslog
Reverse Log Order: CHECKED
GUI Log Entries: 500

# Forward logs to Raspberry Pi (optional)
Enable Remote Logging: CHECKED
Remote Log Servers: 10.0.10.30:514 (syslog)
```

### Dashboard Widgets
```bash
# Dashboard → Configure widgets:

- Interfaces (WAN, VLANs status)
- System Information
- Traffic Graphs
- Gateway Status
- Services Status
- Firewall Logs
```

---

## 🔄 Backup Configuration

### Manual Backup
```bash
# Diagnostics → Backup & Restore

# Download Configuration
Area: All
Backup Configuration: Click to download XML file

# Save to: ~/homelab/configs/pfsense/pfsense-backup-YYYY-MM-DD.xml
```

### Automatic Backup (via AutoConfigBackup)
```bash
# System → Package Manager → Available Packages
Install: AutoConfigBackup

# Services → Auto Config Backup → Settings
Backup on every config change: CHECKED
```

---

## 🆘 Recovery Procedures

### Reset to Factory
```bash
# If you get locked out:
# 1. Connect keyboard/monitor to N150
# 2. Boot to console
# 3. Select option 4: Reset webConfigurator password
# 4. Reset to default: admin / pfsense
```

### Restore Configuration
```bash
# Diagnostics → Backup & Restore
# Choose File: pfsense-backup-YYYY-MM-DD.xml
# Restore Configuration
# Reboot after restore
```

---

## 📋 Post-Configuration Checklist

- [ ] WAN gets DHCP from Eero (192.168.4.x)
- [ ] All 5 VLAN interfaces are up
- [ ] DHCP working on Users and Experimental VLANs
- [ ] Firewall rules allow traffic per design
- [ ] WireGuard VPN accessible from outside
- [ ] DDNS updating public IP correctly
- [ ] Port forwarding working (test from outside network)
- [ ] Can ping internet from all VLANs
- [ ] DNS resolving via Pi-Hole
- [ ] Backups scheduled/saved
- [ ] Email alerts configured and tested

---

## 🔗 Quick Reference

**WebGUI:** https://10.0.10.1  
**Default Credentials:** admin / pfsense  
**Console Access:** Keyboard + monitor on N150  
**SSH Access:** ssh admin@10.0.10.1 (if enabled)

---

**Last Updated:** [DATE]
