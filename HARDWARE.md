# 🖥️ Hardware Specifications & Bill of Materials

Complete hardware list for my homelab build with specs, pricing, and purchase links.

---

## 📊 Quick Summary

| Category | Count | Total Cost (CAD) |
|----------|-------|------------------|
| Compute Nodes | 4 | $425.00 |
| NAS Components | 1 complete build | $979.30 |
| Firewall | 1 | $330.46 |
| Raspberry Pi Setup | 1 | $201.55 |
| Network Equipment | Switch + accessories | $328.31 |
| Rack Infrastructure | Cabinet + accessories | $598.16 |
| Storage Drives | Various SSDs/HDDs | $304.61 |
| **TOTAL** | - | **~$3,167.39** |

---

## 💻 Compute Hardware (Proxmox Cluster)

### ThinkCentre Nodes

| Component | Specs | Price (CAD) | Link | Notes |
|-----------|-------|-------------|------|-------|
| Lenovo ThinkCentre M710Q | i5-7400T 2.4GHz, 16GB DDR4 RAM, 500GB SSD SATA | $102.14 | [eBay.ca](https://www.ebay.ca/itm/197901648050) | Node 1 (upgraded RAM) |
| Lenovo ThinkCentre M710Q | i5-7500T, 16GB DDR4 RAM, 500GB SSD SATA | $121.74 | [eBay.ca](https://www.ebay.ca/itm/376693720411) | Node 2 (upgraded RAM) |
| Lenovo ThinkCentre M715Q | AMD Pro A10-9700E, 16GB DDR4 RAM, 128GB SSD SATA + 1TB NVMe | $99.98 | [eBay.ca](https://www.ebay.ca/itm/155474578754) | Node 3 (upgraded storage) |
| Lenovo ThinkCentre M710Q | i5-7500T, 16GB DDR4, 500GB SSD SATA | $121.74 | [eBay.ca](https://www.ebay.ca/itm/376693720411) | Node 4 (upgraded RAM) |

**Notes:**
- All nodes upgraded to 16GB DDR4 for better VM performance
- 7th gen Intel/AMD CPUs support virtualization extensions
- Power consumption: ~10-15W idle per node

---

## 💾 Storage Components

### SSD Storage

| Component | Specs | Price (CAD) | Link | Purpose |
|-----------|-------|-------------|------|---------|
| SanDisk SSD Plus 500GB | SATA (opened box) | $59.27 | [Amazon.ca](https://www.amazon.ca/dp/B0F4Y2VR8S) | General storage |
| SanDisk SSD Plus 500GB | SATA (new) | $64.99 | [Amazon.ca](https://www.amazon.ca/dp/B0F4Y2VR8S) | General storage |
| Patriot P220 128GB | SATA SSD | $24.99 | [Amazon.ca](https://www.amazon.ca/dp/B0BS9W3T48) | Boot drive |
| CUSU M.2 SSD 2TB | PCIe 3.0x4 NVMe | $165.36 | [AliExpress](https://www.aliexpress.com/item/1005008785149427.html) | High-speed storage |

### HDD Storage

| Component | Specs | Price (CAD) | Link | Purpose |
|-----------|-------|-------------|------|---------|
| 1TB 3.5" HDD | SATA (x5 for NAS) | $75 ($15 each) | MDG Computers (Local) | RAID 5 array |

---

## 🎮 Raspberry Pi 5 Setup (Monitoring Node)

| Component | Specs | Price (CAD) | Link | Purpose |
|-----------|-------|-------------|------|---------|
| Raspberry Pi 5 8GB | 2023 model, 8GB RAM | $133.69 | [Amazon.ca](https://www.amazon.ca/dp/B0CK2FCG1K) | Monitoring host |
| GeeekPi P33 M.2 NVMe PoE+ HAT | M.2 M-Key with PoE+ | $48.98 | [Amazon.ca](https://www.amazon.ca/dp/B0D8JC3MXQ) | PoE power + storage |
| Gigabit PoE+ Injector | IEEE 802.3at | $18.88 | [Amazon.ca](https://www.amazon.ca/Injector-IEEE802-3at-Replacement-TPE-115GI-TL-PoE160S/dp/B00NRF9GQO) | Power over Ethernet |

**Total:** $201.55

**Configuration:**
- 512GB NVMe storage via PoE+ HAT
- 7" touchscreen attached (see Rack Infrastructure)
- Runs: Grafana, Prometheus, Scaphandre, NUT-Exporter

---

## 🔥 Firewall Hardware (pfSense/OPNsense)

| Component | Specs | Price (CAD) | Link | Purpose |
|-----------|-------|-------------|------|---------|
| Fanless N150 Mini PC | 4x 2.5GbE, DDR4, NVMe, Type-C (barebone) | $245.99 | [AliExpress](https://www.aliexpress.com/item/1005010292635214.html) | Firewall appliance |
| Patriot P320 128GB | NVMe SSD | $33.99 | [Amazon.ca](https://www.amazon.ca/dp/B0D4RD18YV) | Boot drive |
| Crucial 8GB DDR4 3200MHz | SO-DIMM RAM | $50.48 | [eBay.ca](https://www.ebay.ca/itm/365735557774) | System memory |

**Total:** $330.46

**Features:**
- 4x Intel 2.5GbE NICs (for WAN, LAN, DMZ, Management)
- Fanless design for silent operation
- Low power consumption (~6-8W idle)
- Sits on top of rack

---

## 🌐 Network Equipment

| Component | Specs | Price (CAD) | Link | Purpose |
|-----------|-------|-------------|------|---------|
| MikroTik CRS310-8G+2S+IN | 8x 2.5GbE + 2x SFP+ managed switch | $276.00 | [Amazon.ca](https://www.amazon.ca/dp/B0CH9NHFHS) | Core switch |
| MikroTik RMK-2/10 | Rack mount ears kit | $33.52 | [WirelessNetware.ca](https://www.shop.wirelessnetware.ca/accessories/516-rmk-210-4752224008688.html) | Rack mounting |
| 2.5Gb Network Adapter | M.2 A+E/Mini PCIe to RJ45 | $18.79 | [AliExpress](https://www.aliexpress.com/item/1005008820106326.html) | NIC upgrade |

**Total:** $328.31

**Existing Equipment:**
- Eero 6+ Mesh Pod (WiFi 6 mesh system)
- Netgear Router AC1750 (1750Mbps from Xplorer ISP - used as AP)

**Network Design:**
- All inter-rack communication at 2.5GbE
- SFP+ ports reserved for future 10GbE uplink
- VLANs: Management, LAN, DMZ, Storage

---

## 🏢 Rack Infrastructure

| Component | Specs | Price (CAD) | Link | Purpose |
|-----------|-------|-------------|------|---------|
| GeeekPi 12U Server Cabinet | 12U rackmount cabinet | $209.94 | [Amazon.ca](https://www.amazon.ca/GeeekPi-Cabinet-Equipment-RackMate-Rackmount/dp/B0DT2XM22G) | Main enclosure |
| GeeekPi 12-Port Patch Panel | Cat6 patch panel | $23.79 | [Amazon.ca](https://www.amazon.ca/dp/B0D5XPNHHF) | Cable management |
| GeeekPi 1U Server Rack Shelf | Universal 1U shelf (x2) | $56.08 ($28.04 each) | [Amazon.ca](https://www.amazon.ca/dp/B0D5XGSPXD) | Node mounting |
| PDU 3-Socket | Rackmount power distribution | $42.72 | [AliExpress](https://www.aliexpress.com/item/1005005963996490.html) | Power distribution |
| 3-Prong 1-to-4 Power Splitter | Power cord splitter | $23.99 | [Amazon.ca](https://www.amazon.ca/dp/B0CH9NHFHS) | Additional outlets |
| Freenove 7" Touchscreen | 1024x600 HDMI display | $47.96 | [Amazon.ca](https://www.amazon.ca/dp/B0BPP6MFFJ) | Pi monitoring display |
| CyberPower CST150UC-FC | UPS 1500VA / 900W | $219.99 | [Costco.ca](https://www.costco.ca/.product.1615528.html) | Battery backup |

**Total:** $624.47

**Rack Layout:** See README.md for complete top-down diagram

---

## 💾 NAS Specifications (TrueNAS Build)

| Component | Specs | Price (CAD) | Link | Purpose |
|-----------|-------|-------------|------|---------|
| Jonsbo N2 | 5-bay NAS case | $169.05 | [AliExpress](https://www.aliexpress.com/item/1005007766481140.html) | Case |
| N5105 Industrial Board | Motherboard + CPU combo | $160.38 | [AliExpress](https://www.aliexpress.com/item/1005006221619148.html) | System board |
| Noctua NF-A12x15 PWM | 120mm slim case fan | $27.95 | [Amazon.ca](https://www.amazon.ca/Noctua-NF-A12x15-PWM-4-Pin-Premium/dp/B072Q2547D) | Case cooling |
| Noctua NF-A4x20 | 40mm CPU fan | $29.79 | [AliExpress](https://www.aliexpress.com/item/1005006690066510.html) | CPU cooling |
| GAMEMAX 650W 80+ Gold | Fully modular SFX PSU | $144.49 | [Amazon.ca](https://www.amazon.ca/dp/B0F9NR7G17) | Power supply |
| Crucial 8GB x2 DDR4 3200MHz | SO-DIMM RAM (16GB total) | $93.69 | [eBay.ca](https://www.ebay.ca/itm/365735557774) | System memory |
| 6-pack SATA Cables | SATA III cables | $6.79 | [AliExpress](https://www.aliexpress.com/item/1005007523507363.html) | Drive connections |
| 1TB NVMe M.2 PCIe 3.0x4 | 3,500MB/s read | $107.99 | [Amazon.ca](https://www.amazon.ca/dp/B0DTY976K3) | ZFS L2ARC cache |
| Kingston A400 120GB | 2.5" SATA SSD | $30.58 | [AliExpress](https://www.aliexpress.com/item/1005007438137775.html) | Boot drive |
| 1TB 3.5" HDD x5 | SATA storage drives | $75.00 | MDG Computers (Local) | RAID 5 array |

**Total:** $845.71

**NAS Configuration:**
- **CPU:** Intel N5105 (4C/4T, 10W TDP)
- **RAM:** 16GB DDR4 (ECC not supported, but adequate for home use)
- **Storage:** 5x 1TB in RAID 5 = 4TB usable capacity
- **Cache:** 1TB NVMe for ZFS L2ARC (read cache)
- **Boot:** 120GB SSD for TrueNAS OS
- **Power:** 650W 80+ Gold (overkill for future expansion)

**Expected Performance:**
- Sequential Read: 200-300 MB/s (RAID 5 limited)
- Sequential Write: 150-200 MB/s
- Network bottleneck at 2.5GbE (~312 MB/s)

---

## 🔌 Power Consumption Estimates

| Device | Idle (W) | Load (W) | Notes |
|--------|----------|----------|-------|
| ThinkCentre x4 | 40-60W | 120-160W | ~10-15W each idle |
| NAS (Jonsbo N2) | 15-20W | 40-60W | 5x HDD spinning |
| Firewall (N150) | 6-8W | 12-15W | Fanless design |
| Raspberry Pi 5 | 5-8W | 8-12W | With screen & NVMe |
| MikroTik Switch | 8-12W | 15-20W | No PoE devices |
| **Total System** | **74-108W** | **195-267W** | Estimated |

**Annual Cost (@ $0.10/kWh):**
- Idle: $65-95/year
- 24/7 average load: $170-235/year

**UPS Runtime:**
- 900W UPS capacity
- ~100W system draw
- Estimated runtime: 60-90 minutes

---

## 📦 Purchase Timeline

**Phase 1 - Foundation (Week 1)**
- ✅ Rack cabinet & infrastructure
- ✅ Switch & network equipment
- ✅ UPS

**Phase 2 - Compute (Week 2)**
- ✅ ThinkCentre nodes x4
- ✅ RAM upgrades
- ✅ Storage drives

**Phase 3 - Storage (Week 3)**
- ✅ NAS case & components
- ✅ HDDs for RAID array

**Phase 4 - Peripherals (Week 4)**
- ✅ Raspberry Pi setup
- ✅ Firewall hardware
- ✅ Cables & accessories

---

## 🛠️ Future Upgrades

**Short Term (6 months):**
- [ ] Additional 2TB NVMe for compute nodes
- [ ] 10GbE SFP+ DAC cables (when budget allows)
- [ ] UPS network management card

**Medium Term (1 year):**
- [ ] Replace 1TB HDDs with 4TB drives
- [ ] Add 5th node for Proxmox quorum
- [ ] Dedicated backup server

**Long Term (2+ years):**
- [ ] 10GbE network upgrade
- [ ] Larger rack (20U or 25U)
- [ ] Enterprise-grade UPS

---

## 🔗 Vendor Notes

**Recommended Canadian Vendors:**
- **Amazon.ca** - Fast shipping, easy returns
- **Costco.ca** - Best prices on UPS units
- **eBay.ca** - Great for used enterprise hardware
- **MDG Computers (Local)** - Reliable for used HDDs

**International Vendors:**
- **AliExpress** - Cheap cases/components, 2-4 week shipping
- Be aware of duties/taxes on orders over $150 CAD

---

## 📝 Notes

- All prices in Canadian Dollars (CAD)
- Prices accurate as of purchase date (2024-2025)
- Used equipment prices may vary significantly
- Budget was flexible—optimize for performance over cost
- Total investment: **~$3,167** for complete homelab

**Lessons Learned:**
- Buy quality fans (Noctua) - worth the premium for silence
- Used ThinkCentre Tiny PCs are incredible value
- 2.5GbE is the sweet spot for homelab (affordable & fast)
- Don't skimp on the UPS - protect your investment!
