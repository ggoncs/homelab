# 🤖 Ansible Automation Structure

Complete Ansible setup for homelab automation (Phase 4+).

---

## 📁 Directory Structure

```
ansible/
├── ansible.cfg
├── inventory/
│   ├── hosts.yml
│   └── group_vars/
│       ├── all.yml
│       ├── proxmox.yml
│       ├── pbs.yml
│       └── services.yml
├── roles/
│   ├── common/
│   ├── proxmox/
│   ├── pbs/
│   ├── truenas/
│   ├── pihole/
│   ├── monitoring/
│   └── wireguard/
├── playbooks/
│   ├── 01-initial-setup.yml
│   ├── 02-proxmox-cluster.yml
│   ├── 03-storage-setup.yml
│   ├── 04-deploy-services.yml
│   └── 05-monitoring.yml
├── templates/
│   ├── interfaces/
│   ├── systemd/
│   └── configs/
└── files/
    ├── ssh_keys/
    └── certificates/
```

---

## ⚙️ ansible.cfg

```ini
[defaults]
inventory = inventory/hosts.yml
remote_user = root
host_key_checking = False
retry_files_enabled = False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 3600

# Use Python 3
interpreter_python = auto_silent

# Privilege escalation
become = True
become_method = sudo
become_user = root
become_ask_pass = False

# SSH settings
timeout = 30
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no

# Logging
log_path = ./ansible.log

[ssh_connection]
pipelining = True
control_path = /tmp/ansible-ssh-%%h-%%p-%%r
```

---

## 📋 inventory/hosts.yml

```yaml
---
all:
  children:
    homelab:
      children:
        proxmox_cluster:
          hosts:
            pve-node1:
              ansible_host: 10.0.10.11
              node_id: 1
              storage_type: zfs
            pve-node2:
              ansible_host: 10.0.10.12
              node_id: 2
              storage_type: zfs
            pve-node3:
              ansible_host: 10.0.10.13
              node_id: 3
              storage_type: zfs
          vars:
            cluster_name: homelab-cluster
            zfs_pool: rpool
            
        pbs:
          hosts:
            pbs-node:
              ansible_host: 10.0.10.14
              storage_type: ext4
              backup_dir: /backup
              
        storage:
          hosts:
            truenas:
              ansible_host: 10.0.10.20
              ansible_python_interpreter: /usr/bin/python3
              
        services:
          hosts:
            raspberry-pi:
              ansible_host: 10.0.10.30
              services:
                - wireguard
                - pihole_backup
                - nut_server
                - ddns
                - grafana
                - prometheus
              
        vms:
          hosts:
            pihole-primary:
              ansible_host: 10.0.30.10
              vm_id: 100
              parent_node: pve-node1
              
            pihole-secondary:
              ansible_host: 10.0.30.11
              vm_id: 101
              parent_node: pve-node2
              
            jellyfin:
              ansible_host: 10.0.30.20
              vm_id: 200
              parent_node: pve-node1
              
            nginx:
              ansible_host: 10.0.30.25
              vm_id: 201
              parent_node: pve-node2
```

---

## 🌍 inventory/group_vars/all.yml

```yaml
---
# Global variables for all hosts

# Network configuration
homelab_domain: homelab.local
dns_servers:
  - 10.0.30.10  # Primary Pi-Hole
  - 10.0.10.30  # Secondary Pi-Hole

ntp_servers:
  - 0.north-america.pool.ntp.org
  - 1.north-america.pool.ntp.org

timezone: America/Toronto

# VLANs
vlans:
  management:
    id: 10
    subnet: 10.0.10.0/24
    gateway: 10.0.10.1
  users:
    id: 20
    subnet: 10.0.20.0/24
    gateway: 10.0.20.1
  services:
    id: 30
    subnet: 10.0.30.0/24
    gateway: 10.0.30.1
  storage:
    id: 40
    subnet: 10.0.40.0/24
    gateway: 10.0.40.1
  experimental:
    id: 50
    subnet: 10.0.50.0/24
    gateway: 10.0.50.1

# SSH configuration
ssh_port: 22
ssh_key_path: "{{ playbook_dir }}/../files/ssh_keys/id_ed25519.pub"

# System packages (common)
common_packages:
  - vim
  - htop
  - iotop
  - curl
  - wget
  - git
  - tmux
  - ncdu
  - net-tools
  - rsync

# Monitoring
prometheus_port: 9090
grafana_port: 3000
node_exporter_port: 9100
```

---

## 🔧 inventory/group_vars/proxmox.yml

```yaml
---
# Proxmox cluster variables

proxmox_version: "8.2"
cluster_name: homelab-cluster

# ZFS configuration
zfs_pool_name: rpool
zfs_compression: lz4
zfs_atime: off
zfs_arc_max: 8G  # 8GB max for 16GB RAM nodes

# Corosync configuration
corosync_ring0_addr: 10.0.10.0/24
corosync_bindnet_addr: 10.0.10.0
corosync_expected_votes: 3

# Storage configuration
storage_dirs:
  - name: local
    path: /var/lib/vz
    content: images,rootdir,vztmpl,iso
    
  - name: local-zfs
    pool: rpool/data
    content: images,rootdir

# NFS mounts from TrueNAS
nfs_mounts:
  - name: truenas-isos
    server: 10.0.40.10
    export: /mnt/tank/proxmox-isos
    content: iso,vztmpl
    
  - name: truenas-backups
    server: 10.0.40.10
    export: /mnt/tank/proxmox-backups
    content: backup

# Proxmox repositories
proxmox_repos:
  enterprise: false  # Disable enterprise repo
  no_subscription: true  # Enable no-subscription repo
  
# Backup schedule
backup_schedule:
  enabled: true
  storage: truenas-backups
  mode: snapshot
  compress: zstd
  dow: mon,wed,fri  # Monday, Wednesday, Friday
  starttime: "02:00"
  mailnotification: always
  mailto: your-email@example.com
```

---

## 💾 inventory/group_vars/pbs.yml

```yaml
---
# Proxmox Backup Server variables

pbs_datastore:
  name: backup
  path: /backup
  gc_schedule: "daily"
  prune_schedule: "daily"
  
# Prune settings (3-2-1 strategy)
prune_settings:
  keep_last: 3
  keep_hourly: 24
  keep_daily: 7
  keep_weekly: 4
  keep_monthly: 6
  
# ZFS replication from TrueNAS
zfs_replication:
  enabled: true
  source_host: 10.0.40.10
  source_dataset: tank
  pull_replication: true
  encrypted: true
  schedule: "0 3 * * *"  # 3 AM daily

# Cloud backup to Proton Drive (offsite)
cloud_backup:
  enabled: true
  provider: protondrive
  schedule: "0 4 * * 0"  # Sunday 4 AM weekly
  retention: 4  # Keep 4 weeks
```

---

## 📝 playbooks/01-initial-setup.yml

```yaml
---
- name: Initial Homelab Setup
  hosts: all
  become: true
  tasks:
  
    - name: Update APT cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
        
    - name: Upgrade all packages
      apt:
        upgrade: dist
        autoremove: yes
        autoclean: yes
        
    - name: Install common packages
      apt:
        name: "{{ common_packages }}"
        state: present
        
    - name: Set timezone
      timezone:
        name: "{{ timezone }}"
        
    - name: Configure NTP
      template:
        src: timesyncd.conf.j2
        dest: /etc/systemd/timesyncd.conf
      notify: restart timesyncd
      
    - name: Create admin user
      user:
        name: admin
        shell: /bin/bash
        groups: sudo
        append: yes
        create_home: yes
        
    - name: Add SSH public key
      authorized_key:
        user: admin
        key: "{{ lookup('file', ssh_key_path) }}"
        
    - name: Disable password authentication
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PasswordAuthentication'
        line: 'PasswordAuthentication no'
      notify: restart sshd
      
    - name: Install node_exporter
      include_role:
        name: node_exporter
        
  handlers:
    - name: restart timesyncd
      systemd:
        name: systemd-timesyncd
        state: restarted
        
    - name: restart sshd
      systemd:
        name: sshd
        state: restarted
```

---

## ☁️ playbooks/02-proxmox-cluster.yml

```yaml
---
- name: Configure Proxmox Cluster
  hosts: proxmox_cluster
  become: true
  serial: 1  # Run one node at a time
  tasks:
  
    - name: Remove enterprise repository
      apt_repository:
        repo: "deb https://enterprise.proxmox.com/debian/pve {{ ansible_distribution_release }} pve-enterprise"
        state: absent
        
    - name: Add no-subscription repository
      apt_repository:
        repo: "deb http://download.proxmox.com/debian/pve {{ ansible_distribution_release }} pve-no-subscription"
        state: present
        filename: pve-no-subscription
        
    - name: Update and upgrade Proxmox
      apt:
        update_cache: yes
        upgrade: dist
        
    - name: Install ZFS utilities
      apt:
        name:
          - zfsutils-linux
          - zfs-dkms
        state: present
        
    - name: Configure ZFS ARC
      lineinfile:
        path: /etc/modprobe.d/zfs.conf
        line: "options zfs zfs_arc_max={{ zfs_arc_max | regex_replace('G', '000000000') }}"
        create: yes
      notify: update initramfs
      
    - name: Create cluster (node1 only)
      command: pvecm create {{ cluster_name }}
      when: inventory_hostname == 'pve-node1'
      ignore_errors: yes
      
    - name: Join cluster (node2 and node3)
      command: pvecm add 10.0.10.11
      when: inventory_hostname != 'pve-node1'
      ignore_errors: yes
      
    - name: Configure NFS storage
      include_tasks: tasks/configure_nfs.yml
      
  handlers:
    - name: update initramfs
      command: update-initramfs -u
```

---

## 💾 playbooks/03-storage-setup.yml

```yaml
---
- name: Configure TrueNAS NFS Shares
  hosts: truenas
  tasks:
  
    - name: Ensure NFS service is running
      service:
        name: nfs-server
        state: started
        enabled: yes
        
    - name: Create datasets for Proxmox
      command: >
        zfs create -o compression=lz4 tank/{{ item }}
      loop:
        - proxmox-isos
        - proxmox-backups
      ignore_errors: yes
      
    - name: Create NFS shares
      # This would use TrueNAS API or manual configuration
      # Placeholder for actual TrueNAS API calls
      debug:
        msg: "Configure NFS shares via TrueNAS web UI"

- name: Mount NFS on Proxmox nodes
  hosts: proxmox_cluster
  become: true
  tasks:
  
    - name: Create mount points
      file:
        path: "/mnt/pve/{{ item.name }}"
        state: directory
      loop: "{{ nfs_mounts }}"
      
    - name: Mount NFS shares
      mount:
        path: "/mnt/pve/{{ item.name }}"
        src: "{{ item.server }}:{{ item.export }}"
        fstype: nfs
        opts: defaults
        state: mounted
      loop: "{{ nfs_mounts }}"
```

---

## 🎯 playbooks/04-deploy-services.yml

```yaml
---
- name: Deploy Pi-Hole VMs
  hosts: pve-node1
  tasks:
  
    - name: Clone Pi-Hole template
      proxmox_kvm:
        api_host: "{{ ansible_host }}"
        api_user: root@pam
        api_password: "{{ proxmox_password }}"
        vmid: 100
        clone: 9000  # Ubuntu template
        name: pihole-primary
        newid: 100
        full: yes
        
    - name: Configure Pi-Hole VM
      proxmox_kvm:
        api_host: "{{ ansible_host }}"
        api_user: root@pam
        api_password: "{{ proxmox_password }}"
        vmid: 100
        cores: 2
        memory: 2048
        net:
          net0: "virtio,bridge=vmbr0,tag=30"
        ipconfig:
          ipconfig0: "ip=10.0.30.10/24,gw=10.0.30.1"
        nameserver: 1.1.1.1
        
    - name: Start Pi-Hole VM
      proxmox_kvm:
        api_host: "{{ ansible_host }}"
        api_user: root@pam
        api_password: "{{ proxmox_password }}"
        vmid: 100
        state: started

- name: Configure Pi-Hole
  hosts: pihole-primary
  become: true
  tasks:
  
    - name: Install Pi-Hole
      shell: curl -sSL https://install.pi-hole.net | bash /dev/stdin --unattended
      args:
        creates: /usr/local/bin/pihole
        
    - name: Set Pi-Hole password
      command: pihole -a -p "{{ pihole_password }}"
      
    - name: Add custom DNS records
      blockinfile:
        path: /etc/pihole/custom.list
        block: |
          10.0.10.11 pve-node1.homelab.local
          10.0.10.12 pve-node2.homelab.local
          10.0.10.13 pve-node3.homelab.local
          10.0.10.14 pbs.homelab.local
          10.0.10.20 truenas.homelab.local
          10.0.10.30 pi-jump.homelab.local
      notify: restart pihole
      
  handlers:
    - name: restart pihole
      command: pihole restartdns
```

---

## 📊 playbooks/05-monitoring.yml

```yaml
---
- name: Deploy Monitoring Stack on Raspberry Pi
  hosts: raspberry-pi
  become: true
  tasks:
  
    - name: Install Docker
      shell: curl -fsSL https://get.docker.com | sh
      args:
        creates: /usr/bin/docker
        
    - name: Add admin user to docker group
      user:
        name: admin
        groups: docker
        append: yes
        
    - name: Create monitoring directory
      file:
        path: /home/admin/monitoring
        state: directory
        owner: admin
        group: admin
        
    - name: Copy docker-compose.yml
      template:
        src: docker-compose-monitoring.yml.j2
        dest: /home/admin/monitoring/docker-compose.yml
        owner: admin
        group: admin
        
    - name: Copy Prometheus config
      template:
        src: prometheus.yml.j2
        dest: /home/admin/monitoring/prometheus.yml
        owner: admin
        group: admin
        
    - name: Start monitoring stack
      docker_compose:
        project_src: /home/admin/monitoring
        state: present
      become_user: admin

- name: Install node_exporter on all hosts
  hosts: all
  become: true
  tasks:
  
    - name: Download node_exporter
      get_url:
        url: https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-linux-amd64.tar.gz
        dest: /tmp/node_exporter.tar.gz
        
    - name: Extract node_exporter
      unarchive:
        src: /tmp/node_exporter.tar.gz
        dest: /usr/local/bin/
        remote_src: yes
        extra_opts: [--strip-components=1]
        
    - name: Create node_exporter systemd service
      template:
        src: node_exporter.service.j2
        dest: /etc/systemd/system/node_exporter.service
      notify: restart node_exporter
      
  handlers:
    - name: restart node_exporter
      systemd:
        name: node_exporter
        state: restarted
        enabled: yes
        daemon_reload: yes
```

---

## 🚀 Usage

### Run playbooks in order:

```bash
# Phase 1: Initial setup (run manually first)
ansible-playbook playbooks/01-initial-setup.yml

# Phase 4: Cluster setup
ansible-playbook playbooks/02-proxmox-cluster.yml

# Phase 5: Storage setup
ansible-playbook playbooks/03-storage-setup.yml

# Phase 6: Deploy services
ansible-playbook playbooks/04-deploy-services.yml

# Ongoing: Monitoring
ansible-playbook playbooks/05-monitoring.yml
```

### Run specific roles:

```bash
# Update all nodes
ansible-playbook playbooks/01-initial-setup.yml --tags update

# Reconfigure firewall only
ansible-playbook playbooks/firewall.yml

# Deploy single service
ansible-playbook playbooks/04-deploy-services.yml --limit pihole-primary
```

---

## 📋 TODO: Additional Playbooks Needed

- [ ] WireGuard VPN setup on Raspberry Pi
- [ ] Jellyfin deployment
- [ ] Nginx reverse proxy configuration
- [ ] Backup automation (ZFS replication)
- [ ] Certificate management (Let's Encrypt)
- [ ] Alert configuration (email/telegram)

---

**Last Updated:** [DATE]
