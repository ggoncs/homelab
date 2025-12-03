# 📊 Monitoring Setup

Complete monitoring stack with Grafana, Prometheus, and various exporters.

---

## 🎯 Monitoring Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Raspberry Pi 5                           │
│                  (10.0.99.20)                              │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   Grafana    │  │  Prometheus  │  │ Scaphandre   │    │
│  │   :3000      │  │    :9090     │  │   :8080      │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
│         │                  │                  │            │
│         └──────────────────┴──────────────────┘            │
└─────────────────────────────────────────────────────────────┘
                              │
         ┌────────────────────┼────────────────────┐
         │                    │                    │
    ┌────▼─────┐       ┌──────▼──────┐     ┌──────▼──────┐
    │ Proxmox  │       │  TrueNAS    │     │     UPS     │
    │ Exporter │       │  Exporter   │     │  NUT-Export │
    │  :9221   │       │   :9100     │     │   :9199     │
    └──────────┘       └─────────────┘     └─────────────┘
```

**Components:**
- **Grafana** - Visualization and dashboards
- **Prometheus** - Time-series metrics database
- **Scaphandre** - Power consumption monitoring
- **Node Exporter** - System metrics (CPU, RAM, disk, network)
- **NUT Exporter** - UPS monitoring
- **Proxmox Exporter** - VM/container metrics
- **TrueNAS Exporter** - Storage pool metrics

---

## 🐳 Docker Compose Setup

### Create Directory Structure
```bash
# On Raspberry Pi
mkdir -p ~/monitoring/{prometheus,grafana,scaphandre}
cd ~/monitoring
```

### Docker Compose File
```yaml
# ~/monitoring/docker-compose.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    ports:
      - "9090:9090"
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=<your_password>
      - GF_INSTALL_PLUGINS=grafana-piechart-panel
    ports:
      - "3000:3000"
    networks:
      - monitoring
    depends_on:
      - prometheus

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    command:
      - '--path.rootfs=/host'
    volumes:
      - /:/host:ro,rslave
    ports:
      - "9100:9100"
    networks:
      - monitoring

  scaphandre:
    image: hubblo/scaphandre:latest
    container_name: scaphandre
    restart: unless-stopped
    privileged: true
    volumes:
      - /sys/class/powercap:/sys/class/powercap:ro
      - /proc:/proc:ro
    ports:
      - "8080:8080"
    networks:
      - monitoring
    command: prometheus

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus-data:
  grafana-data:
```

### Start Stack
```bash
docker-compose up -d
docker-compose ps
```

---

## 📈 Prometheus Configuration

### Main Config File
```yaml
# ~/monitoring/prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'homelab'
    environment: 'production'

# Alertmanager configuration (optional)
alerting:
  alertmanagers:
    - static_configs:
        - targets: []

# Load rules once and periodically evaluate them
rule_files:
  - "alerts.yml"

# Scrape configurations
scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Raspberry Pi (Node Exporter)
  - job_name: 'raspberry-pi'
    static_configs:
      - targets: ['node-exporter:9100']
        labels:
          instance: 'pi-monitor'

  # Proxmox Cluster
  - job_name: 'proxmox'
    static_configs:
      - targets: 
        - '10.0.99.11:9221'
        - '10.0.99.12:9221'
        - '10.0.99.13:9221'
        - '10.0.99.14:9221'
        labels:
          cluster: 'homelab-cluster'
    relabel_configs:
      - source_labels: [__address__]
        regex: '10\.0\.99\.(\d+):9221'
        target_label: node
        replacement: 'pve-node${1}'

  # TrueNAS
  - job_name: 'truenas'
    static_configs:
      - targets: ['10.0.40.10:9100']
        labels:
          instance: 'truenas'
          type: 'storage'

  # UPS (NUT Exporter)
  - job_name: 'ups'
    static_configs:
      - targets: ['10.0.99.20:9199']
        labels:
          instance: 'cyberpower-ups'

  # Power Monitoring (Scaphandre)
  - job_name: 'power'
    static_configs:
      - targets: ['scaphandre:8080']
        labels:
          type: 'power-consumption'

  # MikroTik Switch (SNMP - optional)
  - job_name: 'mikrotik-switch'
    static_configs:
      - targets:
        - 10.0.99.2
    metrics_path: /snmp
    params:
      module: [if_mib]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9116  # SNMP exporter address
```

### Alert Rules
```yaml
# ~/monitoring/prometheus/alerts.yml
groups:
  - name: homelab_alerts
    interval: 30s
    rules:
      # Node down
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"
          description: "{{ $labels.instance }} has been down for more than 5 minutes."

      # High CPU
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is above 80% for 10 minutes."

      # High Memory
      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is above 85%."

      # Disk Space
      - alert: LowDiskSpace
        expr: (node_filesystem_avail_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="tmpfs"}) * 100 < 15
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Filesystem {{ $labels.mountpoint }} has less than 15% space remaining."

      # UPS Battery
      - alert: UPSOnBattery
        expr: network_ups_tools_ups_status{flag="OB"} == 1
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "UPS is running on battery"
          description: "Power failure detected - UPS switched to battery."

      # ZFS Pool
      - alert: ZFSPoolDegraded
        expr: node_zfs_zpool_state{state="DEGRADED"} == 1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "ZFS pool degraded"
          description: "ZFS pool {{ $labels.pool }} is in degraded state."
```

---

## 📦 Install Exporters

### Node Exporter (on each Proxmox node)
```bash
# On each Proxmox node
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*-amd64.tar.gz
tar xvfz node_exporter-*-amd64.tar.gz
sudo cp node_exporter-*/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter

# Create systemd service
sudo nano /etc/systemd/system/node_exporter.service
```

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

```bash
# Start service
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
sudo systemctl status node_exporter
```

### Proxmox Exporter (on each Proxmox node)
```bash
# Install dependencies
apt install -y python3 python3-pip

# Install exporter
pip3 install prometheus-pve-exporter

# Create config
sudo nano /etc/prometheus/pve.yml
```

```yaml
default:
  user: monitoring@pve
  password: <monitoring_user_password>
  verify_ssl: false
```

```bash
# Create monitoring user in Proxmox
pveum user add monitoring@pve
pveum passwd monitoring@pve
pveum aclmod / -user monitoring@pve -role PVEAuditor

# Create systemd service
sudo nano /etc/systemd/system/pve_exporter.service
```

```ini
[Unit]
Description=Proxmox VE Exporter
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/pve_exporter /etc/prometheus/pve.yml
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now pve_exporter
```

### NUT Exporter (for UPS monitoring)
```bash
# On Raspberry Pi or dedicated host

# Install NUT (Network UPS Tools)
sudo apt install -y nut nut-client nut-server

# Configure NUT
sudo nano /etc/nut/ups.conf
```

```ini
[cyberpower]
    driver = usbhid-ups
    port = auto
    desc = "CyberPower UPS"
```

```bash
# Configure upsd
sudo nano /etc/nut/upsd.conf
```

```ini
LISTEN 0.0.0.0 3493
```

```bash
# Configure users
sudo nano /etc/nut/upsd.users
```

```ini
[monuser]
    password = <your_password>
    upsmon master
```

```bash
# Start NUT services
sudo systemctl enable --now nut-server
sudo systemctl enable --now nut-client

# Install NUT exporter
docker run -d \
  --name nut-exporter \
  --restart unless-stopped \
  -p 9199:9199 \
  -e NUT_EXPORTER_SERVERS=localhost \
  -e NUT_EXPORTER_USERNAME=monuser \
  -e NUT_EXPORTER_PASSWORD=<your_password> \
  hon95/prometheus-nut-exporter
```

---

## 📊 Grafana Dashboards

### Initial Setup
```bash
# Access Grafana: http://10.0.99.20:3000
# Login: admin / <your_password>

# Add Prometheus data source
Configuration → Data Sources → Add data source
- Type: Prometheus
- URL: http://prometheus:9090
- Save & Test
```

### Import Pre-built Dashboards

#### Node Exporter Full
```bash
# Dashboard ID: 1860
# Grafana → + → Import → 1860
# Select Prometheus data source
```

#### Proxmox Cluster
```bash
# Dashboard ID: 10347
# Shows VM/CT metrics, resource usage
```

#### ZFS Monitoring
```bash
# Dashboard ID: 12620
# TrueNAS pool health, capacity, performance
```

#### UPS Monitoring
```bash
# Create custom dashboard with panels:
- Battery charge %
- Load %
- Runtime remaining
- Input/Output voltage
- Power consumption (watts)
```

### Custom Dashboard - Homelab Overview

**Create New Dashboard:**

**Panel 1: Cluster Status**
```promql
# Total CPU cores
sum(node_cpu_seconds_total) by (instance)

# Total RAM
sum(node_memory_MemTotal_bytes) by (instance)

# Online nodes
up{job="proxmox"}
```

**Panel 2: Storage Usage**
```promql
# TrueNAS pool usage
100 - (node_filesystem_avail_bytes{instance="truenas",mountpoint="/mnt/tank"} / node_filesystem_size_bytes{instance="truenas",mountpoint="/mnt/tank"} * 100)
```

**Panel 3: Network Traffic**
```promql
# Inbound traffic
rate(node_network_receive_bytes_total{device="eth0"}[5m])

# Outbound traffic
rate(node_network_transmit_bytes_total{device="eth0"}[5m])
```

**Panel 4: Power Consumption**
```promql
# Total power draw
sum(scaphandre_host_power_watts)
```

**Panel 5: Temperature**
```promql
# CPU temps
node_hwmon_temp_celsius{chip="coretemp-*"}
```

---

## 🖥️ 7" Display Configuration

### Auto-start Grafana on Boot
```bash
# On Raspberry Pi
sudo apt install -y chromium-browser unclutter

# Create autostart script
mkdir -p ~/.config/autostart
nano ~/.config/autostart/grafana-kiosk.desktop
```

```ini
[Desktop Entry]
Type=Application
Name=Grafana Kiosk
Exec=/usr/bin/chromium-browser --kiosk --noerrdialogs --disable-infobars --no-first-run --incognito http://localhost:3000/d/homelab-overview
```

```bash
# Hide mouse cursor
unclutter -idle 0 &

# Disable screen blanking
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
# Add:
@xset s off
@xset -dpms
@xset s noblank
```

### Rotate Display (if needed)
```bash
sudo nano /boot/config.txt
# Add:
display_rotate=1  # 90 degrees
# Or
display_rotate=3  # 270 degrees
```

---

## 📱 Mobile Access

### Grafana Mobile App
```bash
# Download Grafana Mobile app (iOS/Android)
# Add server: http://10.0.99.20:3000
# Login with credentials
# Access dashboards on the go
```

### VPN Access
```bash
# Connect to WireGuard VPN
# Access Grafana at: http://10.0.99.20:3000
```

---

## 🔔 Alert Configuration

### Email Alerts (via Gmail)
```bash
# Grafana → Alerting → Contact points → Add contact point

Name: Email
Integration: Email
Addresses: your-email@gmail.com

# SMTP settings:
Host: smtp.gmail.com:587
User: your-email@gmail.com
Password: <app-specific-password>
From: your-email@gmail.com
```

### Telegram Alerts (Optional)
```bash
# Create Telegram bot via @BotFather
# Get bot token and chat ID

# Grafana → Contact points → Add
Integration: Telegram
Bot Token: <your_bot_token>
Chat ID: <your_chat_id>
```

### Configure Alert Rules
```bash
# Grafana → Alerting → Alert rules → New alert rule

# Example: High CPU Alert
Query A: avg(rate(node_cpu_seconds_total{mode!="idle"}[5m])) by (instance) * 100
Condition: WHEN avg() IS ABOVE 80
For: 10m
Labels: severity=warning
Annotations: High CPU usage detected
```

---

## 🆘 Troubleshooting

### Prometheus Not Scraping
```bash
# Check Prometheus targets
http://10.0.99.20:9090/targets

# If target is down:
# 1. Check if exporter is running
systemctl status node_exporter

# 2. Check firewall
sudo ufw allow 9100/tcp

# 3. Test connectivity
curl http://10.0.99.11:9100/metrics
```

### Grafana Dashboard Not Loading
```bash
# Check Grafana logs
docker logs grafana

# Check data source connection
# Grafana → Configuration → Data Sources → Test
```

### High Memory Usage
```bash
# Prometheus memory usage
# Reduce retention time in docker-compose.yml
--storage.tsdb.retention.time=15d  # Instead of 30d

# Restart Prometheus
docker-compose restart prometheus
```

---

## 📚 Useful Queries

```promql
# CPU usage by node
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage by node
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk I/O
rate(node_disk_read_bytes_total[5m])
rate(node_disk_written_bytes_total[5m])

# Network bandwidth
rate(node_network_receive_bytes_total{device="eth0"}[5m]) * 8
rate(node_network_transmit_bytes_total{device="eth0"}[5m]) * 8

# Proxmox VM count
count(pve_up{type="qemu"})

# ZFS pool capacity
node_zfs_zpool_used_bytes / node_zfs_zpool_size_bytes * 100

# UPS load
network_ups_tools_ups_load
```

---

## 🔗 References

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Node Exporter](https://github.com/prometheus/node_exporter)
- [Proxmox Exporter](https://github.com/prometheus-pve/prometheus-pve-exporter)

---

**Last Updated:** [DATE]  
**Metrics Retention:** 30 days  
**Uptime:** [DAYS]
