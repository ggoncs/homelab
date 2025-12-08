# 💻⚡ Raspberry Pi 5 Homelab Jump Host Setup

![Raspberry Pi 5](YOUR_IMAGE_URL_HERE)

A comprehensive guide for setting up a Raspberry Pi 5 as a high-performance homelab jump host with NVMe boot and PCIe Gen 3 speeds.

## 📚 Reference Documentation

- [Raspberry Pi Update Bootloader ](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#bootloader_update_stable)
- [PCIe Gen 3 Speed Configuration](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#pcie-gen-3-0)
- [NVMe SSD Boot Guide by Jeff Geerling](https://www.jeffgeerling.com/blog/2023/nvme-ssd-boot-raspberry-pi-5)
- [LED Warning Flash Codes](https://www.raspberrypi.com/documentation/computers/configuration.html#led-warning-flash-codes)

---

## 1. Initial Setup and OS Installation (Pre-SSH)

### 1.1 Flash Bootloader

**Action:** Use Raspberry Pi Imager

```bash
flatpak run org.raspberrypi.rpi-imager
```

- Select: **Raspberry Pi 5**
- Choose: **Misc utility images → Bootloader (Pi 5 family) → SD card boot**
- Flash to MicroSD card

> **Note:** This updates the Pi's internal firmware (EEPROM).

### 1.2 Update EEPROM

**Action:** Boot from SD card
- Insert the flashed SD card
- Power on the Pi
- Wait for the green LED to stop flashing

> **Note:** This ensures the Pi's internal memory is ready for NVMe/USB boot.

### 1.3 Flash Ubuntu

**Action:** Use Raspberry Pi Imager again

- Select: **Raspberry Pi 5**
- Choose: **Other general-purpose OS → Ubuntu**
- Flash to the same MicroSD card

### 1.4 Pre-Configuration Settings

**Action:** Press `Ctrl + Shift + X` in Imager to configure:

- Set hostname: `raspberrypi`
- Set username and password: `admin` / `password`
- Configure locale settings
- Enable SSH under Services

### 1.5 First Boot

**Action:** Insert SD card and power on the Pi

---

## 2. Establishing SSH Connection

### 2.1 Scan Network for Pi

```bash
nmap -p 22 192.168.1.0/24
```

**Expected Output:**
```
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 2C:CF:67:88:AA:61 (Raspberry Pi Trading)
```

> Replace `192.168.1.0/24` with your network range.

### 2.2 Verify SSH Port

```bash
nmap -p 22 <PI_IP_ADDRESS>
```

**Expected Output:**
```
PORT   STATE SERVICE
22/tcp open  ssh
```

### 2.3 Connect via SSH

```bash
ssh admin@192.168.1.200
```

> Replace `192.168.1.200` with your Pi's actual IP address.

---

## 3. System Updates and Essential Packages

### 3.1 Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### 3.2 Install Essential Tools

```bash
sudo apt install git curl wget htop net-tools unzip pciutils -y
```

> `pciutils` is required for `lspci` command.

---

## 4. User Management

### 4.1 Create New Administrative User

```bash
sudo adduser jump-adm
sudo usermod -aG sudo jump-adm
```

### 4.2 Switch to New User

```bash
exit
ssh jump-adm@<PI_IP_ADDRESS>
```

### 4.3 Remove Default User

```bash
sudo deluser --remove-home admin
```

---

## 5. NVMe Migration and Boot Configuration

### 5.1 Verify NVMe Detection

```bash
lsblk
```

### 5.2 Clone SD Card to NVMe

```bash
sudo dd if=/dev/mmcblk0 of=/dev/nvme0n1 bs=4M status=progress
```

> **Warning:** This will overwrite all data on the NVMe drive.

### 5.3 Randomize NVMe GUID

Prevents boot conflicts with the original SD card.

```bash
sudo gdisk /dev/nvme0n1
```

**Commands in gdisk:**
1. Enter `x` (extra functionality)
2. Enter `g` (change disk GUID)
3. Enter `R` (recovery and transformation options)
4. Enter `w` (write table to disk)
5. Enter `Y` (confirm)

### 5.4 Configure Boot Order

```bash
sudo raspi-config
```

- Select: **6 Advanced Options**
- Choose: **A6 Boot Order**
- Select: **NVMe/USB Boot**

### 5.5 Power Off and Remove SD Card

```bash
sudo poweroff
```

> Remove the SD card before powering back on.

### 5.6 Expand Filesystem

After booting from NVMe:

```bash
sudo raspi-config
```

- Select: **6 Advanced Options**
- Choose: **A1 Expand Filesystem**
- Reboot when prompted

---

## 6. PCIe Gen 3 Performance Configuration

### 6.1 Enable PCIe Gen 3

```bash
sudo vim /boot/firmware/config.txt
```

Add the following under `[all]`:

```ini
[all]
dtparam=pciex1_gen=3
```

### 6.2 Reboot

```bash
sudo reboot
```

### 6.3 Verify PCIe Speed

```bash
sudo lspci -vv | grep -E "LnkCap|LnkSta"
```

**Expected Output:**
```
LnkSta: Speed 8GT/s (overdriven), Width x1
```

### 6.4 Benchmark NVMe Performance

```bash
sudo hdparm -tT /dev/nvme0n1
```

**Expected Output:**
```
/dev/nvme0n1:
 Timing cached reads:   3600 MB in  2.00 seconds = 1801.89 MB/sec
 Timing buffered disk reads: 2534 MB in  3.00 seconds = 844.25 MB/sec
```

---

## 7. Hostname Configuration

### 7.1 Set Hostname

```bash
sudo hostnamectl set-hostname pi-jump-homelab
```

### 7.2 Update Hosts File

```bash
sudo vim /etc/hosts
```

Update to:

```
127.0.0.1 localhost
127.0.1.1 pi-jump-homelab
```

### 7.3 Verify Hostname

```bash
ping -c 4 pi-jump-homelab.local
```

---

## 🎉 Setup Complete!

Your Raspberry Pi 5 is now configured as a high-performance jump host with:

- ✅ NVMe boot
- ✅ PCIe Gen 3 speeds (up to ~850 MB/s)
- ✅ Secure SSH access
- ✅ Custom administrative user
- ✅ Optimized hostname configuration

## 🔧 Troubleshooting

If you encounter boot issues, refer to the [LED Warning Flash Codes](https://www.raspberrypi.com/documentation/computers/configuration.html#led-warning-flash-codes) documentation for diagnostics.

---

<center>Made with ❤️</center>
