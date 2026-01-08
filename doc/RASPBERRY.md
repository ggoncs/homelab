# 💻⚡ Raspberry Pi 5 Homelab Jump Host Setup

![Raspberry Pi 5](YOUR_IMAGE_URL_HERE)

A comprehensive guide for setting up a Raspberry Pi 5 as a high-performance homelab jump host with NVMe boot, PCIe Gen 3 speeds, Docker, and Chromium kiosk mode.

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

### 5.2 Install rpi-clone

```bash
git clone https://github.com/geerlingguy/rpi-clone.git
cd rpi-clone/
sudo cp rpi-clone rpi-clone-setup /usr/local/sbin
```

### 5.3 Clone SD Card to NVMe

```bash
sudo rpi-clone nvme0n1
```

> **Note:** `rpi-clone` is more reliable than `dd` as it handles partitions and filesystem expansion automatically.

### 5.4 Clean NVMe (if needed)

If you encounter errors, clean the NVMe first:

```bash
sudo umount /dev/nvme0n1p*
sudo wipefs --all --force /dev/nvme0n1
```

Then run `sudo rpi-clone nvme0n1` again.

### 5.5 Randomize NVMe GUID

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

### 5.6 Configure Boot Order

```bash
sudo raspi-config
```

- Select: **6 Advanced Options**
- Choose: **A6 Boot Order**
- Select: **NVMe/USB Boot**

### 5.7 Configure EEPROM Boot Order (Alternative Method)

You can also edit the EEPROM configuration directly:

```bash
export EDITOR=vim
sudo -E rpi-eeprom-config --edit
```

Verify the configuration:

```bash
rpi-eeprom-config
```

### 5.8 Power Off and Remove SD Card

```bash
sudo poweroff
```

> Remove the SD card before powering back on.

### 5.9 Expand Filesystem

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

## 7. Docker Installation

### 7.1 Install Docker

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### 7.2 Add User to Docker Group

```bash
sudo usermod -aG docker $USER
```

### 7.3 Log Out and Back In

```bash
logout
```

Then reconnect via SSH to apply group changes.

### 7.4 Verify Docker Installation

```bash
docker --version
docker run hello-world
```

---

## 8. Chromium Kiosk Mode Setup (Display Configuration)

### 8.1 Install Required Packages

```bash
sudo apt update
sudo apt install xserver-xorg xinit x11-xserver-utils chromium unclutter -y
```

### 8.2 Configure X Server Startup

Create `.xinitrc`:

```bash
vim ~/.xinitrc
```

Add the following content:

```bash
#!/bin/bash
# 1. Disable the screen saver and power management (so the screen stays ON)
xset s off
xset -dpms
xset s noblank

# 2. Hide the mouse cursor after 0.5 seconds of inactivity
unclutter -idle 0.5 -root &

# 3. Launch Chromium in Kiosk Mode
# --kiosk: Full screen, no address bar
# --incognito: Don't save history/cache
# --noerrdialogs: Don't show "Chromium didn't shut down correctly"
exec chromium --noerrdialogs --disable-infobars --kiosk --incognito "http://localhost:3001"
```

Make it executable:

```bash
chmod +x ~/.xinitrc
```

### 8.3 Configure Auto Login

```bash
sudo raspi-config
```

- Select: **1 System Options**
- Choose: **S5 Boot / Auto Login**
- Select: **B2 Console Autologin**
- Finish and reboot

### 8.4 Auto-start X Server on Login

Create `.bash_profile`:

```bash
vim ~/.bash_profile
```

Add the following:

```bash
if [[ -z $DISPLAY ]] && [[ $(tty) = /dev/tty1 ]]; then
    startx -- -nocursor
fi
```

### 8.5 Add User to Video Groups

```bash
sudo usermod -aG video,render,tty jump-adm
```

### 8.6 Configure GPU Driver

Create driver configuration:

```bash
sudo mkdir -p /etc/X11/xorg.conf.d
sudo vim /etc/X11/xorg.conf.d/99-vc4.conf
```

Add the following:

```
Section "OutputClass"
    Identifier "vc4"
    MatchDriver "vc4"
    Driver "modesetting"
    Option "PrimaryGPU" "true"
EndSection
```

### 8.7 Remove Conflicting Driver

```bash
sudo apt remove xserver-xorg-video-fbturbo
```

### 8.8 Verify X Server Configuration

Check the X wrapper configuration:

```bash
cat /etc/X11/Xwrapper.config
```

Should contain:

```
allowed_users=console
needs_root_rights=yes
```

### 8.9 Reboot

```bash
sudo reboot
```

After reboot, Chromium should automatically launch in kiosk mode displaying your application at `http://localhost:3001`.

---

## 9. Hostname Configuration

### 9.1 Set Hostname

```bash
sudo hostnamectl set-hostname pi-jump-homelab
```

### 9.2 Update Hosts File

```bash
sudo vim /etc/hosts
```

Update to:

```
127.0.0.1 localhost
127.0.1.1 pi-jump-homelab
```

### 9.3 Verify Hostname

```bash
ping -c 4 pi-jump-homelab.local
```

---

## 🎉 Setup Complete!

Your Raspberry Pi 5 is now configured as a high-performance jump host with:

- ✅ NVMe boot (using rpi-clone)
- ✅ PCIe Gen 3 speeds (up to ~850 MB/s)
- ✅ Secure SSH access
- ✅ Custom administrative user
- ✅ Docker container support
- ✅ Chromium kiosk mode with auto-start
- ✅ Optimized hostname configuration

## 🔧 Troubleshooting

If you encounter boot issues, refer to the [LED Warning Flash Codes](https://www.raspberrypi.com/documentation/computers/configuration.html#led-warning-flash-codes) documentation for diagnostics.

---

<center>Made with ❤️</center>
