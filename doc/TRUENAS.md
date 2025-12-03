# 💾 TrueNAS Configuration

Complete TrueNAS SCALE setup, ZFS pool management, shares, and backup strategies.

---

## 📋 System Overview

**Hardware:** Jonsbo N2 NAS (Units U01-U05)
- **CPU:** Intel N5105 (4C/4T, 10W TDP)
- **RAM:** 16GB DDR4 3200MHz (non-ECC)
- **Boot Drive:** 120GB Kingston A400 SSD
- **Cache:** 1TB NVMe M.2 (L2ARC read cache)
- **Data Drives:** 5x 1TB 3.5" SATA HDD (RAID-Z1)
- **PSU:** GAMEMAX 650W 80+ Gold
- **Network:** Single 2.5GbE connection

**IP Address:** 10.0.40.10/24 (Storage VLAN)

---

## 🚀 Initial Setup

### Installation
1. Download TrueNAS SCALE ISO
2. Flash to USB drive with Rufus/Etcher
3. Boot Jonsbo N2 from USB
4. Install to 120GB Kingston SSD
5. Reboot and remove USB

### First Boot Configuration
```bash
# Access web UI: http://10.0.40.10

# Login with credentials set during install
Username: admin
Password: <your_password>

# System → General Settings
- Hostname: truenas
- Domain: homelab.local
- DNS: 10.0.99.20 (Pi-Hole)
- Timezone: America/Toronto
```

### Network Configuration
```bash
# Network → Interfaces → Edit enp1s0
- Type: Static
- IP Address: 10.0.40.10/24
- Gateway: 10.0.40.1
- MTU: 1500 (or 9000 for jumbo frames if switch supports)
```

### Update System
```bash
# System → Update
- Check for updates
- Apply latest stable release
```

---

## 💿 ZFS Pool Creation

### Create RAID-Z1 Pool

```bash
# Via Web UI: Storage → Pools → Add

Pool Name: tank
Layout: RAID-Z1 (RAID 5 equivalent)
Disks: Select 5x 1TB HDDs

# Advanced options:
Ashift: 12 (4K sectors)
Compression: lz4 (enabled)
Deduplication: Off (requires lots of RAM)
```

**Pool Characteristics:**
- **Raw Capacity:** 5TB
- **Usable Capacity:** ~4TB (1 disk parity)
- **Fault Tolerance:** 1 disk failure
- **Compression Ratio:** ~1.2-1.5x (with lz4)

### Add L2ARC Cache
```bash
# Storage → Pools → tank → Add Vdev

Type: Cache (L2ARC)
Disk: 1TB NVMe M.2

# This accelerates read performance for frequently accessed data
```

### Configure Pool Settings
```bash
# Storage → Pools → tank → Edit

# Recommended settings:
Auto TRIM: Enabled
Compression: lz4
Checksum: SHA256
Atime: Off (better performance)
Sync: Standard
```

---

## 📁 Dataset Structure

### Create Datasets
```bash
# Storage → Pools → tank → Add Dataset

# Main datasets:
tank/media          # Jellyfin media files
tank/backups        # Proxmox backups
tank/documents      # General file storage
tank/isos           # OS installation ISOs
tank/downloads      # Download staging
tank/photos         # Photo library
```

### Configure Dataset Properties

#### Media Dataset (for Jellyfin)
```bash
# tank/media settings:
Compression: lz4
Record Size: 1M (large files like videos)
Quota: None
Snapshot: Enabled
```

#### Backups Dataset (for Proxmox)
```bash
# tank/backups settings:
Compression: zstd (better compression for backups)
Record Size: 128K (mixed file sizes)
Quota: 1TB (limit backup size)
Snapshot: Enabled (weekly)
```

#### Documents Dataset
```bash
# tank/documents settings:
Compression: lz4
Record Size: 128K (mixed file sizes)
Quota: 500GB
Snapshot: Enabled (daily)
```

---

## 🌐 SMB Shares (Windows/Mac/Linux)

### Enable SMB Service
```bash
# Services → SMB → Configure

NetBIOS Name: TRUENAS
Workgroup: WORKGROUP
Description: Homelab NAS

# Services → SMB → Start Automatically: Enabled
```

### Create SMB Share
```bash
# Sharing → Windows (SMB) → Add

# Media Share
Path: /mnt/tank/media
Name: media
Purpose: Default share parameters
Description: Media files for Jellyfin

# Advanced Options:
Enable ACL: Yes
Browsable to Network Clients: Yes
Export Recycle Bin: Yes
```

### Configure SMB Permissions
```bash
# Datasets → tank/media → Edit Permissions

User: media_user (create in Accounts → Users)
Group: media_group
Mode: 770
ACL Type: POSIX
```

### Additional Shares
```bash
# Create similar shares for:
- documents → /mnt/tank/documents
- downloads → /mnt/tank/downloads
- photos → /mnt/tank/photos
```

### Access from Windows
```bash
\\10.0.40.10\media
\\truenas.homelab.local\media
```

---

## 📡 NFS Shares (Linux/Proxmox)

### Enable NFS Service
```bash
# Services → NFS → Configure

NFSv4: Enabled
Bind IP Addresses: 10.0.40.10
Allow non-root mount: Enabled
```

### Create NFS Share for Proxmox ISOs
```bash
# Sharing → Unix (NFS) → Add

Path: /mnt/tank/isos
Description: Proxmox ISO storage

# Access:
Authorized Networks: 10.0.99.0/24 (Management VLAN)
MapAll User: root
MapAll Group: root

# Advanced Options:
Quiet: Yes
Read Only: No
```

### Create NFS Share for Proxmox Backups
```bash
# Sharing → Unix (NFS) → Add

Path: /mnt/tank/backups
Description: Proxmox backups

# Access:
Authorized Networks: 10.0.99.0/24
MapAll User: root
MapAll Group: root
```

### Mount from Linux
```bash
# On Proxmox or Linux client
mount -t nfs 10.0.40.10:/mnt/tank/isos /mnt/isos

# Add to /etc/fstab for persistent mount
10.0.40.10:/mnt/tank/isos /mnt/isos nfs defaults 0 0
```

---

## 🎬 Jellyfin Configuration

### Install Jellyfin
```bash
# Apps → Available Applications → Search "Jellyfin"
# Click Install

# Configuration:
Name: jellyfin
Host Path (Media): /mnt/tank/media
Host Path (Config): /mnt/tank/jellyfin-config
Network: Bridge
Port: 8096
```

### Access Jellyfin
```bash
# Web UI: http://10.0.40.10:8096

# Initial setup:
- Create admin account
- Add media library: /media
- Configure metadata providers
- Set transcoding preferences
```

### Media Organization
```bash
# Recommended structure in /mnt/tank/media:
/mnt/tank/media/Movies/
  ├── Movie Name (Year)/
  │   └── Movie Name (Year).mkv
/mnt/tank/media/TV Shows/
  ├── Show Name/
  │   ├── Season 01/
  │   │   ├── Show Name - S01E01.mkv
  │   │   └── Show Name - S01E02.mkv
/mnt/tank/media/Music/
  ├── Artist/
  │   └── Album/
  │       └── Track.mp3
```

---

## 📸 Snapshots & Replication

### Automatic Snapshots
```bash
# Data Protection → Periodic Snapshot Tasks → Add

Dataset: tank/documents
Recursive: Yes
Lifetime: 2 weeks
Naming Schema: auto-%Y%m%d-%H%M
Schedule: Daily at 2:00 AM

# Create similar tasks for:
- tank/media (weekly, keep 4 weeks)
- tank/backups (weekly, keep 2 weeks)
- tank/photos (daily, keep 1 month)
```

### Manual Snapshots
```bash
# Storage → Pools → tank/documents → Create Snapshot

Name: pre-migration
Recursive: Yes
```

### Snapshot Management
```bash
# View snapshots
# Storage → Pools → tank → Snapshots

# Rollback to snapshot
# Select snapshot → Rollback

# Clone snapshot
# Select snapshot → Clone to New Dataset
```

### Snapshot Cleanup
```bash
# Data Protection → Periodic Snapshot Tasks → Edit
# Adjust "Lifetime" to auto-delete old snapshots
```

---

## 🔄 Backup Strategies

### 3-2-1 Backup Rule
- **3 copies** of data
- **2 different** storage mediums
- **1 offsite** backup

### Local Snapshots (Copy 1)
```bash
# ZFS snapshots on TrueNAS
# Hourly/Daily/Weekly retention
```

### Proxmox Backups (Copy 2)
```bash
# Proxmox VMs backed up to /mnt/tank/backups
# Different storage medium (NAS vs. compute nodes)
```

### Cloud Backup (Copy 3 - Optional)
```bash
# Data Protection → Cloud Sync Tasks → Add

# Backup to cloud storage:
- Backblaze B2
- Wasabi
- AWS S3

# Schedule: Weekly
# Encryption: Enabled
```

---

## 📊 Monitoring & Alerts

### Enable S.M.A.R.T. Monitoring
```bash
# Storage → Disks → Select disk → Edit

# Enable S.M.A.R.T.
# Configure S.M.A.R.T. tests (short, long)
```

### S.M.A.R.T. Test Schedule
```bash
# Tasks → S.M.A.R.T. Tests → Add

# Short test: Weekly
# Long test: Monthly
# All disks: Selected
```

### Email Alerts
```bash
# System → Email

# Configure SMTP settings:
Outgoing Mail Server: smtp.gmail.com
Port: 587
Security: TLS
Username: your-email@gmail.com
Password: app-specific-password

# Test email
# Send Test Mail
```

### Alert Configuration
```bash
# System → Alert Settings

# Configure alerts for:
- Pool status warnings
- Disk failures
- High temperature
- Low disk space (<10%)
```

### Integrate with Prometheus
```bash
# Install Prometheus exporter
# Export metrics to Grafana dashboard
# See MONITORING.md for details
```

---

## 🔧 Maintenance Tasks

### Scrub Schedule
```bash
# Tasks → Scrub Tasks → Add

Pool: tank
Threshold: 35 days
Schedule: Monthly (first Sunday at 2 AM)
```

**Manual Scrub:**
```bash
# Storage → Pools → tank → Scrub Pool
```

### TRIM Schedule (for SSDs)
```bash
# Automatically enabled if "Auto TRIM" is on
# Runs weekly by default
```

### Update TrueNAS
```bash
# System → Update

# Check for updates: Weekly
# Apply during maintenance window
# Always backup pool before major updates
```

---

## ⚡ Performance Tuning

### Network Performance
```bash
# Network → Global Configuration

# Increase network buffers
Sysctl:
net.inet.tcp.sendbuf_max=16777216
net.inet.tcp.recvbuf_max=16777216

# Enable jumbo frames (if switch supports)
# Network → Interfaces → Edit enp1s0
MTU: 9000
```

### ZFS ARC Tuning
```bash
# System → Advanced → Sysctl

# Limit ARC to 8GB (leave 8GB for system)
vfs.zfs.arc_max: 8589934592

# Minimum ARC: 4GB
vfs.zfs.arc_min: 4294967296
```

### Disk Performance
```bash
# Check disk performance
iostat -x 1

# Expected RAID-Z1 performance:
Sequential Read: 200-300 MB/s
Sequential Write: 150-200 MB/s
```

---

## 🆘 Troubleshooting

### Pool Degraded
```bash
# Check pool status
zpool status tank

# If disk failed, replace:
# Storage → Pools → tank → Status
# Click degraded device → Replace

# Insert new disk
# System will resilver automatically
```

### High Memory Usage
```bash
# Check ARC stats
arc_summary

# Reduce ARC size if needed (see Performance Tuning)
```

### Slow Performance
```bash
# Check for:
1. Disk issues (S.M.A.R.T. errors)
2. High ARC miss rate
3. Network bottlenecks
4. Fragmentation

# Defragment pool (if needed)
# NOTE: ZFS doesn't need traditional defrag
# If fragmentation is high, consider rebuilding pool
```

### SMB Connection Issues
```bash
# Check SMB service
systemctl status smb

# Restart service
systemctl restart smb

# Check firewall rules
# Ensure ports 139, 445 are open
```

### NFS Mount Failures
```bash
# Check NFS service
systemctl status nfs

# Verify exports
showmount -e 10.0.40.10

# Check network connectivity
ping 10.0.40.10
```

---

## 📚 Useful Commands

```bash
# Pool management
zpool status                  # Show all pool status
zpool list                    # List pools and usage
zpool scrub tank             # Start scrub
zpool clear tank             # Clear errors

# Dataset management
zfs list                     # List all datasets
zfs get all tank/media       # Show all properties
zfs set compression=lz4 tank/media  # Set property

# Snapshots
zfs snapshot tank/media@backup-2024-01-01
zfs list -t snapshot         # List all snapshots
zfs rollback tank/media@backup-2024-01-01
zfs destroy tank/media@backup-2024-01-01

# Performance
zpool iostat -v 1            # I/O statistics
arc_summary                  # ARC cache statistics
```

---

## 🔗 References

- [TrueNAS Documentation](https://www.truenas.com/docs/)
- [OpenZFS Documentation](https://openzfs.github.io/openzfs-docs/)
- [TrueNAS Forum](https://www.truenas.com/community/)

---

**Last Updated:** [DATE]  
**Pool Health:** [STATUS]  
**Uptime:** [DAYS]
