# Immich on a Home Server: Complete Setup Guide

> **Target audience:** This guide is written for someone who is **new to Linux**, but **resourceful, patient, and comfortable following step-by-step instructions**. You do *not* need prior server or Docker experience.

If something feels unfamiliar, that's normal. The guide intentionally explains *why* you're doing things and links out to trusted resources when deeper understanding helps.

This guide walks through setting up **Immich** (self‑hosted photo & video backup) on a home server using **Ubuntu + Docker**, with optional remote access and Google Photos import.

---

## Table of Contents

- [High-Level Architecture](#high-level-architecture)
- [Design Decisions and Optional Enhancements](#design-decisions-and-optional-enhancements)
  - [Backups](#backups)
  - [HTTPS and Remote Access](#https-and-remote-access)
  - [Security Basics](#security-basics)
  - [Storage Growth](#storage-growth)
- [Step-by-Step Instructions](#step-by-step-instructions)
- [1. Install Latest Ubuntu Server](#1-install-latest-ubuntu-server)
- [2. Set a Static IP Address](#2-set-a-static-ip-address)
- [3. Configure Firewall (UFW)](#3-configure-firewall-ufw)
- [4. Install Docker and Docker Compose](#4-install-docker-and-docker-compose)
- [5. Set Up Storage for Immich](#5-set-up-storage-for-immich)
- [6. Install and Configure Immich](#6-install-and-configure-immich)
- [7. Set Up Nginx Reverse Proxy (Strongly Recommended)](#7-set-up-nginx-reverse-proxy-strongly-recommended)
- [8. Purchase a Domain Name (Required for Remote Access)](#8-purchase-a-domain-name-required-for-remote-access)
- [9. Set Up DNS and Port Forwarding](#9-set-up-dns-and-port-forwarding)
- [10. Set Up HTTPS with Let's Encrypt](#10-set-up-https-with-lets-encrypt)
- [11. Install Immich Mobile Apps](#11-install-immich-mobile-apps)
- [12. Configure Mobile Apps and Set Up Backup](#12-configure-mobile-apps-and-set-up-backup)
- [13. Create Convenience Scripts](#13-create-convenience-scripts)
- [14. Import Photos from Google Photos (Optional)](#14-import-photos-from-google-photos-optional)
- [15. Backup Strategy (Strongly Recommended)](#15-backup-strategy-strongly-recommended)
- [16. Optional: Google OAuth Integration](#16-optional-google-oauth-integration)
- [17. Maintenance and Updates](#17-maintenance-and-updates)
- [Beginner Tips: How to Think While Using Linux](#beginner-tips-how-to-think-while-using-linux)
- [Common Beginner Mistakes (And How to Avoid Them)](#common-beginner-mistakes-and-how-to-avoid-them)
- [Advanced Storage Setup (Optional)](#advanced-storage-setup-optional)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)
- [Conclusion](#conclusion)

---

## High‑Level Architecture

* Ubuntu Server (bare metal or VM)
* Docker + Docker Compose
* Immich (server, machine learning, database, Redis)
* Local access via static LAN IP
* Remote access via domain + DNS (Cloudflare)
* Optional OAuth + Google Photos import

---

## Design Decisions and Optional Enhancements

This guide focuses on a **safe, sensible default setup** that works well for most home users.

There are a few areas where you may choose to go further depending on your needs. These are not required to get started, but are worth knowing about.

### Backups

* Immich manages your photos, but does not automatically protect them from disk failure
* A simple backup strategy is strongly recommended and covered later in this guide

### HTTPS and Remote Access

* Mobile apps work best with HTTPS
* This guide introduces Nginx first, then explains HTTPS options later
* DNS + port forwarding is often the easiest choice for home servers

### Security Basics

* Firewall setup is included
* Strong passwords and minimal exposure are assumed

### Storage Growth

* Photo libraries grow over time
* Early storage planning avoids painful migrations later

If you're following the guide top to bottom, you don't need to make any decisions here yet — everything will be explained when it's time.

---

## Step‑by‑Step Instructions

---

## 1. Install Latest Ubuntu Server

**📋 What You'll Accomplish:**
* Install Ubuntu Server 25.10 on your hardware
* Enable SSH for remote access
* Update the system to the latest security patches
* Verify you can log in and run basic commands

**⏱ Estimated Time:** 30-45 minutes (mostly waiting for installation)

**📦 Prerequisites:**
* A computer to use as your server (can be old hardware, 4GB+ RAM recommended)
* 8GB+ USB drive for installation
* Another computer to create the bootable USB and connect via SSH (optional but recommended)

---

### Step 1.1: Download Ubuntu Server

**Recommended:** Ubuntu Server **25.10 (non-LTS)**

#### Why 25.10 Instead of LTS?

If you're new to Linux, you may see advice online to *always* use LTS releases. That's good advice for enterprise servers, but **home servers are different**.

Ubuntu **25.10** is recommended here because:

* It supports **newer consumer hardware** out of the box
* It includes a **newer Linux kernel**, which improves disk, USB, and network reliability
* Docker and container tooling tend to work better on newer releases

For a single-purpose home server like Immich, this usually results in *fewer* problems, not more.

**Trade-offs:**

* 9‑month support lifecycle (LTS gets 5 years)
* Requires upgrading ~once per year

For a single-purpose home server (Immich), this is usually a good trade.

**💡 Tip:** If you prefer the stability of LTS, Ubuntu Server 24.04 LTS will also work. Just use 24.04 instead of 25.10 throughout this guide.

#### Download the ISO

1. Go to [ubuntu.com/download/server](https://ubuntu.com/download/server)
2. Download **Ubuntu Server 25.10**
3. The file will be named something like `ubuntu-25.10-live-server-amd64.iso` (approximately 2GB)

**✓ Success Criteria:**
* You have a `.iso` file in your Downloads folder
* File size is around 2GB

---

### Step 1.2: Create Bootable USB Drive

You'll create a USB drive that can boot and install Ubuntu.

**Windows users:** Use [Rufus](https://rufus.ie/)
**Mac/Linux users:** Use [balenaEtcher](https://www.balena.io/etcher/)

#### Using Rufus (Windows)

1. Download and run Rufus
2. Insert your USB drive
3. Rufus should auto-detect your USB drive in the "Device" dropdown
4. Click "SELECT" and choose the Ubuntu ISO you downloaded
5. Leave all other settings at default
6. Click "START"
7. If asked about ISO mode vs DD mode, choose **ISO mode**
8. Wait for the process to complete (5-10 minutes)

#### Using balenaEtcher (Mac/Linux)

1. Download and run balenaEtcher
2. Click "Flash from file" and select the Ubuntu ISO
3. Click "Select target" and choose your USB drive
4. Click "Flash!"
5. Enter your password if prompted
6. Wait for the process to complete (5-10 minutes)

**✓ Success Criteria:**
* The tool shows "Success" or "Complete"
* Your USB drive is now labeled "Ubuntu Server" or similar

**⚠ Warning:** This process will erase everything on the USB drive. Make sure you've backed up any important files first.

---

### Step 1.3: Boot and Install Ubuntu

Now you'll boot your server from the USB drive and install Ubuntu.

1. **Insert the USB drive** into your server computer
2. **Turn on the computer** and immediately press the boot menu key:
   * Common keys: `F12`, `F2`, `ESC`, `F10`, or `DEL`
   * Look for a message like "Press F12 for Boot Menu" when the computer starts
   * If you miss it, restart and try again
3. **Select the USB drive** from the boot menu
   * Look for "USB", "Removable Device", or "Ubuntu"
4. **Select "Install Ubuntu Server"** from the first menu (not "Try Ubuntu")

#### Follow the Installation Wizard

The installer will ask you several questions. Here's what to choose:

**Language & Keyboard:**
* Choose your language and keyboard layout
* Use arrow keys and Enter to navigate

**Network:**
* If you have a wired connection, it should auto-configure
* **✓ Success:** You should see an IP address (like `192.168.1.X`) appear
* **⚠ If no network:** That's okay, you can configure it later. For now, continue without network.

**Proxy:**
* Leave blank and press Enter (unless your network requires a proxy)

**Mirror:**
* Use the default Ubuntu archive mirror (just press Enter)

**Storage:**
* **Choose "Use an entire disk"** (recommended for beginners)
* Select your hard drive from the list
* **⚠ Warning:** This will erase everything on the selected drive
* Select "Done" and confirm when prompted

**Profile Setup:**
* **Your name:** Your real name (e.g., "Jane Doe")
* **Server name:** A name for this server (e.g., "immich-server")
* **Username:** Your login name (e.g., "jane") - use lowercase, no spaces
* **Password:** Create a strong password and remember it
* **⚠ Important:** Write down your username and password - you'll need them constantly

**SSH Setup:**
* **✓ CRITICAL:** Use spacebar to check the box: **"Install OpenSSH server"**
* This lets you manage the server remotely
* If you forget this, you'll have to configure it manually later

**Featured Server Snaps:**
* Don't select anything (we'll use Docker instead)
* Just press "Done"

**Installation Progress:**
* The installer will now copy files (this takes 10-20 minutes)
* You'll see a progress bar and log messages
* **✓ Success:** When you see "Installation complete!", continue

**Reboot:**
* Select "Reboot Now"
* **Remove the USB drive** when prompted
* The system will restart

**✓ Success Criteria:**
* System reboots and shows a login prompt
* The login prompt shows your server name (e.g., `immich-server login:`)

---

### Step 1.4: First Login

After reboot, you'll see a screen like this:

```
Ubuntu 25.10 immich-server tty1

immich-server login: _
```

1. **Type your username** (the one you created during installation)
2. **Press Enter**
3. **Type your password** (you won't see any characters as you type - this is normal)
4. **Press Enter**

**✓ Success Criteria:**

You should see a welcome message and a command prompt like this:

```
Welcome to Ubuntu 25.10 (GNU/Linux 6.11.0-9-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Last login: Mon Feb  3 10:23:45 2025
username@immich-server:~$ _
```

The `$` symbol means you're ready to type commands!

**⚠ Troubleshooting:**

* **"Login incorrect":** You mistyped your username or password. Try again slowly.
* **Nothing happens when typing password:** This is normal - Linux hides passwords for security. Just type carefully and press Enter.
* **Can't remember password:** You'll need to reinstall Ubuntu (there's no easy recovery for beginners).

---

### Step 1.5: Update the System

Now that you're logged in, you'll update Ubuntu to get the latest security patches.

**💡 Why update now?** The installation media might be a few weeks old. Updating ensures you have all security fixes before connecting to the internet.

Run these commands one at a time:

```bash
sudo apt update
```

**What this does:** Downloads the list of available software updates.

**✓ Expected output:**

```
Hit:1 http://archive.ubuntu.com/ubuntu oracular InRelease
Get:2 http://archive.ubuntu.com/ubuntu oracular-updates InRelease [126 kB]
Get:3 http://security.ubuntu.com/ubuntu oracular-security InRelease [126 kB]
...
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
123 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

The key phrase: **"X packages can be upgraded"** (number varies).

**⚠ If you see errors:**
* **"Could not resolve archive.ubuntu.com":** You're not connected to the internet. Check your network cable.
* **"Permission denied":** You forgot `sudo` at the beginning.

---

Now upgrade the packages:

```bash
sudo apt upgrade -y
```

**What this does:** Installs all available updates. The `-y` flag automatically says "yes" to all prompts.

**✓ Expected output:**

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following packages will be upgraded:
  base-files bind9-dnsutils bind9-host bind9-libs ...
123 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 456 MB of archives.
After this operation, 12.3 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu oracular-updates/main amd64 base-files ...
...
Setting up base-files (13.5ubuntu1.1) ...
...
Processing triggers for man-db (2.12.1-2) ...
```

This will take 5-15 minutes depending on how many updates are available and your internet speed.

**✓ Success Criteria:**
* No error messages (warnings in yellow are usually okay)
* Ends with something like "Processing triggers..." and returns to the command prompt

---

Now reboot to apply kernel updates:

```bash
sudo reboot
```

**What this does:** Restarts the server. Any kernel (core operating system) updates require a reboot to take effect.

The connection will drop if you're using SSH. **This is normal.**

**Wait 1-2 minutes** for the server to restart.

---

### Step 1.6: Log Back In

After reboot, log back in the same way:

1. At the login prompt, enter your **username**
2. Enter your **password**

**✓ Success Criteria:**

You should see the welcome message again, and the command prompt:

```
username@immich-server:~$ _
```

---

### Step 1.7: Verify Your Installation

Let's confirm everything is working correctly.

**Check Ubuntu version:**

```bash
lsb_release -a
```

**✓ Expected output:**

```
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 25.10
Release:        25.10
Codename:       oracular
```

The key line: **"Description: Ubuntu 25.10"**

---

**Check for updates (should be clean now):**

```bash
sudo apt update
```

**✓ Expected output:**

```
...
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
```

The key phrase: **"All packages are up to date."**

---

**Verify SSH is running:**

```bash
sudo systemctl status ssh
```

**✓ Expected output:**

```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-02-03 10:23:45 UTC; 2min 15s ago
...
```

The key phrase: **"Active: active (running)"** in green.

Press `q` to exit this view.

**⚠ If SSH is not running:**

```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```

---

### ✅ Checkpoint: Section 1 Complete

**You should now have:**

* ✓ Ubuntu Server 25.10 installed on your hardware
* ✓ A user account you can log in with
* ✓ System fully updated with latest security patches
* ✓ SSH server running (for remote access in the next section)
* ✓ Ability to run basic commands like `sudo apt update`

**What you learned:**

* How to boot from USB and install Ubuntu
* How to use `sudo` (run commands as administrator)
* How to update your system with `apt`
* How to check service status with `systemctl`

**Next steps:**

In Section 2, you'll configure a static IP address so you can reliably connect to your server from other computers on your network.

**💡 Pro Tip:** If you have another computer on the same network, you can now connect via SSH instead of typing directly on the server. This makes copy-pasting commands much easier. We'll cover this in Section 2.

---

## 2. Set a Static IP Address

**📋 What You'll Accomplish:**
* Find your server's current IP address
* Configure a static (fixed) IP address using Netplan
* Connect to your server via SSH from another computer
* Verify the network configuration is working correctly

**⏱ Estimated Time:** 10-15 minutes

**📦 Prerequisites:**
* ✓ Section 1 complete (Ubuntu Server installed and updated)
* ✓ Server connected to your network via ethernet cable (WiFi setup is more complex)
* ✓ Access to your router settings (optional, but helpful for choosing an IP)

**💡 Why a static IP?**

Right now, your server probably has a dynamic IP that could change when you restart it or your router. A static IP ensures you can always find your server at the same address (like a permanent home address vs. a temporary hotel room).

---

### Step 2.1: Find Your Current Network Information

First, let's see what network settings your server has right now.

Run this command:

```bash
ip addr show
```

**✓ Expected output:**

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:3f:2e:91 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.147/24 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 86395sec preferred_lft 86395sec
    inet6 fe80::a00:27ff:fe3f:2e91/64 scope link
       valid_lft forever preferred_lft forever
```

**📝 What to note:**

1. **Interface name:** Look for the section that shows your wired connection (usually `enp0s3`, `eth0`, `ens33`, or similar)
   - Ignore `lo` (loopback - that's internal only)
   - In the example above, it's **enp0s3**

2. **Current IP address:** Look for the line starting with `inet` under your interface
   - In the example above, it's **192.168.1.147/24**
   - Your IP will be different

3. **Network range:** The `/24` means your network uses IPs from `192.168.1.1` to `192.168.1.254`

**✍️ Write down:**
- Your interface name: `_____________` (e.g., enp0s3)
- Your current IP: `_____________` (e.g., 192.168.1.147)

**⚠ Troubleshooting:**

* **No `inet` line under your interface:** Your network cable might be unplugged, or your network didn't assign an IP. Check the cable and try `sudo dhclient` to request an IP.
* **Only see 127.0.0.1:** That's the loopback address - keep looking for another interface.

---

### Step 2.2: Choose Your Static IP Address

You need to pick an IP address that:
- Is in your network's range (usually 192.168.1.X or 192.168.0.X)
- Won't conflict with other devices
- Is outside your router's DHCP range (to avoid collisions)

**💡 Recommended approach:**

Most home routers assign DHCP addresses in the range **192.168.1.100-192.168.1.254**.

**Safe choices for static IPs:**
- `192.168.1.50` (this guide's example)
- `192.168.1.10` through `192.168.1.99`

**✍️ Write down your chosen static IP:** `_____________`

**🔍 Optional: Check your router's DHCP range**

If you want to be certain:
1. Log into your router (usually at http://192.168.1.1 or http://192.168.0.1)
2. Look for "DHCP Settings" or "LAN Settings"
3. Note the DHCP range (e.g., "100-254")
4. Choose a static IP outside that range

---

### Step 2.3: Find Your Router's Gateway IP

You need to know your router's IP address (called the "gateway").

Run this command:

```bash
ip route show
```

**✓ Expected output:**

```
default via 192.168.1.1 dev enp0s3 proto dhcp src 192.168.1.147 metric 100
192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.147 metric 100
```

**📝 What to note:**

The line starting with `default via` shows your gateway IP.
- In the example above: **192.168.1.1**
- This is almost always your router's address

**✍️ Write down your gateway IP:** `_____________` (e.g., 192.168.1.1)

---

### Step 2.4: Find Your Netplan Configuration File

Ubuntu uses **Netplan** to configure networking. The configuration is stored in a YAML file.

Run this command:

```bash
ls /etc/netplan/
```

**✓ Expected output:**

```
00-installer-config.yaml
```

Or you might see:
```
01-netcfg.yaml
```

Or:
```
50-cloud-init.yaml
```

**📝 Note the filename.** You'll need it in the next step.

**✍️ Write down your Netplan filename:** `_____________`

---

### Step 2.5: Edit the Netplan Configuration

Now you'll edit the configuration file to set a static IP.

**⚠ Important:** Take your time with this step. YAML files are sensitive to spacing and indentation.

Open the file (replace the filename with yours):

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

**What this does:** Opens the Netplan configuration file in a text editor.

You'll see something like this:

```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: true
  version: 2
```

**Now, replace the entire contents** with this template:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:  # Replace with YOUR interface name
      dhcp4: no
      addresses:
        - 192.168.1.50/24  # Replace with YOUR chosen static IP
      routes:
        - to: default
          via: 192.168.1.1  # Replace with YOUR gateway IP
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

**🔧 Customize these lines:**

1. **Line 4:** `enp0s3` → Replace with your interface name from Step 2.1
2. **Line 7:** `192.168.1.50/24` → Replace with your chosen static IP from Step 2.2
3. **Line 10:** `192.168.1.1` → Replace with your gateway IP from Step 2.3

**⚠ Critical YAML rules:**
- Use **spaces**, not tabs (press spacebar 2 times for each indent level)
- Keep the exact indentation shown (2 spaces, then 4 spaces, then 6 spaces, etc.)
- Don't add extra spaces at the end of lines
- The dash `-` is important - don't forget it

**Example for a different network:**

If your network is `192.168.0.X` (instead of `192.168.1.X`), it would look like:

```yaml
network:
  version: 2
  ethernets:
    eth0:  # Different interface name
      dhcp4: no
      addresses:
        - 192.168.0.50/24  # Different network range
      routes:
        - to: default
          via: 192.168.0.1  # Different gateway
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

**💡 What do these settings mean?**

- `dhcp4: no` - Don't get IP automatically (we're setting it manually)
- `addresses` - Your static IP address
- `routes` - Where to send internet traffic (to your router)
- `nameservers` - DNS servers to use (8.8.8.8 is Google's public DNS)

**Save the file:**

1. Press `Ctrl + O` (that's the letter O, not zero)
2. Press `Enter` to confirm
3. Press `Ctrl + X` to exit

**✓ Success Criteria:**
* You see "Wrote X lines" at the bottom
* You're back at the command prompt

---

### Step 2.6: Test the Configuration (Without Applying)

Before applying the changes, let's check for syntax errors.

Run this command:

```bash
sudo netplan try
```

**What this does:** Tests the configuration and automatically reverts if something goes wrong (it gives you 120 seconds to confirm).

**✓ Expected output:**

```
Do you want to keep these settings?

Press ENTER before the timeout to accept the new configuration

Changes will revert in 120 seconds
```

**If you see this:** Your configuration is valid! Press `Enter` to accept.

**⚠ If you see errors:**

Common error messages:

**"Invalid YAML"** or **"mapping values are not allowed"**
- You have a spacing/indentation problem
- Open the file again: `sudo nano /etc/netplan/00-installer-config.yaml`
- Check that you're using spaces (not tabs)
- Check that colons have a space after them: `version: 2` not `version:2`
- Compare carefully with the template above

**"Permissions denied"**
- You forgot `sudo`

**If it's not working:**
- Wait for the 120-second timeout - your old configuration will automatically restore
- Try editing again, being extra careful with spacing

---

### Step 2.7: Verify the New IP Address

Check if the static IP is now assigned:

```bash
ip addr show
```

**✓ Expected output:**

Look for your interface (e.g., `enp0s3`) and find the `inet` line:

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:3f:2e:91 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.50/24 brd 192.168.1.255 scope global enp0s3
       valid_lft forever preferred_lft forever
```

**📝 Key changes:**

- **IP address:** Should now show your static IP (e.g., `192.168.1.50`)
- **"scope global":** Confirms it's a network-accessible address
- **No "dynamic":** The word "dynamic" is gone (compare to Step 2.1 output)

**✓ Success Criteria:**
* Your chosen static IP appears in the `inet` line
* The word "dynamic" is no longer there

---

### Step 2.8: Test Internet Connectivity

Make sure you can still reach the internet.

```bash
ping -c 4 google.com
```

**What this does:** Sends 4 test packets to Google and reports if they arrive.

**✓ Expected output:**

```
PING google.com (142.250.190.46) 56(84) bytes of data.
64 bytes from lga25s83-in-f14.1e100.net (142.250.190.46): icmp_seq=1 ttl=117 time=12.3 ms
64 bytes from lga25s83-in-f14.1e100.net (142.250.190.46): icmp_seq=2 ttl=117 time=11.8 ms
64 bytes from lga25s83-in-f14.1e100.net (142.250.190.46): icmp_seq=3 ttl=117 time=12.1 ms
64 bytes from lga25s83-in-f14.1e100.net (142.250.190.46): icmp_seq=4 ttl=117 time=11.9 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 11.782/12.025/12.283/0.193 ms
```

**📝 Key phrase:** `4 packets transmitted, 4 received, 0% packet loss`

**✓ Success Criteria:**
* You see responses from google.com
* 0% packet loss
* Times are shown (usually 1-50ms for home networks)

Press `Ctrl + C` if it doesn't stop automatically.

**⚠ If ping fails:**

**"Temporary failure in name resolution"**
- DNS isn't working - check your `nameservers` section in netplan
- Make sure you have `8.8.8.8` and `8.8.4.4` listed

**"Network is unreachable"**
- Your gateway is wrong - check the `via` line in your routes
- It should be your router's IP (usually 192.168.1.1)

**"Destination Host Unreachable"**
- Your IP address might conflict with another device
- Try a different static IP in the same range

To fix errors:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
# Fix the problem
sudo netplan apply
```

---

### Step 2.9: Connect via SSH (Optional but Recommended)

Now that you have a static IP, you can connect from another computer on your network. This makes copy-pasting commands much easier.

**From another computer on the same network:**

**On Windows:** Use PowerShell or [PuTTY](https://www.putty.org/)
**On Mac/Linux:** Use the Terminal

Run this command (replace `username` and IP with yours):

```bash
ssh username@192.168.1.50
```

Example:
```bash
ssh jane@192.168.1.50
```

**What this does:** Connects to your server remotely over SSH.

**First time connecting:**

You'll see a message like:

```
The authenticity of host '192.168.1.50 (192.168.1.50)' can't be established.
ED25519 key fingerprint is SHA256:abc123def456...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

**Type `yes` and press Enter.** This is normal for first-time connections.

**✓ Expected output:**

```
Warning: Permanently added '192.168.1.50' (ED25519) to the list of known hosts.
username@192.168.1.50's password:
```

Enter your password (you won't see it as you type).

**✓ Success Criteria:**

You should see the Ubuntu welcome message and be at the command prompt:

```
Welcome to Ubuntu 25.10 (GNU/Linux 6.11.0-9-generic x86_64)
...
username@immich-server:~$ _
```

**💡 Pro Tips:**

- **You're now controlling the server from another computer!**
- You can copy commands from this guide and paste them into the SSH session
- You can leave the SSH session open while working through the guide
- To disconnect: type `exit` or press `Ctrl + D`

**⚠ Troubleshooting SSH:**

**"Connection refused"**
- SSH might not be running: `sudo systemctl start ssh`
- Check Section 1.7 to verify SSH is enabled

**"No route to host"**
- You're not on the same network
- Double-check the IP address

**"Permission denied"**
- Wrong username or password
- Remember: usernames are case-sensitive

---

### ✅ Checkpoint: Section 2 Complete

**You should now have:**

* ✓ A static IP address configured (e.g., 192.168.1.50)
* ✓ Network connectivity working (can ping google.com)
* ✓ Ability to SSH to your server from another computer
* ✓ Understanding of your network configuration (interface name, gateway, IP range)

**What you learned:**

* How to view network configuration with `ip addr show` and `ip route`
* How to edit YAML configuration files with nano
* How to configure static networking with Netplan
* How to test configurations safely with `netplan try`
* How to connect remotely via SSH

**Network details you documented:**

* Interface name: `_____________`
* Static IP: `_____________`
* Gateway: `_____________`
* Netplan file: `_____________`

**💡 Important:** From now on, always use this static IP to connect to your server. If you restart your server or router, the IP will stay the same.

**Next steps:**

In Section 3, you'll configure a firewall to protect your server, allowing only the services you need (SSH and Immich) while blocking everything else.

**🔖 Bookmark this:** You can now access your server at `http://192.168.1.50:2283` (once Immich is installed in Section 6).

---

## 3. Configure Firewall (UFW)

**📋 What You'll Accomplish:**
* Enable Ubuntu's built-in firewall (UFW)
* Configure rules to allow SSH access
* Configure rules to allow Immich access
* Verify the firewall is protecting your server
* Understand basic firewall concepts

**⏱ Estimated Time:** 5 minutes

**📦 Prerequisites:**
* ✓ Section 1 complete (Ubuntu Server installed)
* ✓ Section 2 complete (Static IP configured)
* ✓ Connected via SSH or at the server console

**💡 Why configure a firewall?**

A firewall blocks unwanted network traffic while allowing services you need. Think of it like a bouncer at a club - only people on the list get in.

Without a firewall, all ports on your server are potentially accessible. With UFW (Uncomplicated Firewall), you'll explicitly allow only:
- **Port 22:** SSH (so you can connect remotely)
- **Port 2283:** Immich (so you can access your photos)

Everything else will be blocked by default.

**⚠ Important:** We'll add SSH rules BEFORE enabling the firewall. If you enable the firewall first without allowing SSH, you'll lock yourself out!

---

### Step 3.1: Check UFW Status

First, let's see if UFW is already running.

```bash
sudo ufw status
```

**✓ Expected output:**

```
Status: inactive
```

This means the firewall exists but isn't running yet. This is normal for a fresh Ubuntu install.

**⚠ If you see "Status: active":**

That's fine - it means UFW is already enabled. Continue with the next steps to add rules.

---

### Step 3.2: Allow SSH (Port 22)

**⚠ CRITICAL: Do this FIRST before enabling the firewall!**

If you're connected via SSH and enable the firewall without allowing SSH first, you'll be disconnected and unable to reconnect.

```bash
sudo ufw allow 22/tcp
```

**What this does:** Creates a rule allowing incoming connections on port 22 (SSH).

**✓ Expected output:**

```
Rules updated
Rules updated (v6)
```

The "(v6)" line means it also added the rule for IPv6 (modern internet addresses). This is automatic and good.

**📝 Understanding the command:**
- `ufw` - The firewall program
- `allow` - Permit this traffic
- `22` - Port number (SSH uses port 22)
- `/tcp` - Protocol (SSH uses TCP)

---

### Step 3.3: Allow Immich (Port 2283)

Now allow traffic for Immich.

```bash
sudo ufw allow 2283/tcp
```

**What this does:** Creates a rule allowing incoming connections on port 2283 (Immich's default port).

**✓ Expected output:**

```
Rules updated
Rules updated (v6)
```

**💡 Why port 2283?**

Immich chose this non-standard port to avoid conflicts. Standard web ports are 80 (HTTP) and 443 (HTTPS), which we'll add later when setting up remote access.

---

### Step 3.4: Review Rules Before Enabling

Let's check what rules we've added before turning on the firewall.

```bash
sudo ufw show added
```

**✓ Expected output:**

```
Added user rules (see 'ufw status' for running firewall):
ufw allow 22/tcp
ufw allow 2283/tcp
```

This shows the rules you've created. They're not active yet, but they're ready.

**✓ Success Criteria:**
* You see both `22/tcp` and `2283/tcp` listed
* No error messages

**⚠ If you don't see SSH (22/tcp):**

**DO NOT** enable the firewall yet! Add it now:

```bash
sudo ufw allow 22/tcp
```

---

### Step 3.5: Enable the Firewall

Now that SSH is allowed, it's safe to enable the firewall.

```bash
sudo ufw enable
```

**What this does:** Activates the firewall with the rules you've configured.

**You'll see a warning:**

```
Command may disrupt existing ssh connections. Proceed with operation (y|n)?
```

**Type `y` and press Enter.**

This warning appears every time, but it's safe because you already added the SSH rule.

**✓ Expected output:**

```
Firewall is active and enabled on system startup
```

**📝 Key phrase:** "enabled on system startup" - This means UFW will automatically start when you reboot.

---

### Step 3.6: Verify Firewall Status

Check that the firewall is running with the correct rules.

```bash
sudo ufw status
```

**✓ Expected output:**

```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
2283/tcp                   ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
2283/tcp (v6)              ALLOW       Anywhere (v6)
```

**📝 What this means:**

| Column | Meaning | Example |
|--------|---------|---------|
| **To** | Port/service being accessed | `22/tcp` (SSH) |
| **Action** | What happens to traffic | `ALLOW` (permitted) |
| **From** | Where traffic can come from | `Anywhere` (any IP address) |

**✓ Success Criteria:**
* Status shows "active"
* You see rules for both `22/tcp` and `2283/tcp`
* Both IPv4 and IPv6 versions are listed

---

### Step 3.7: Test SSH Connection Still Works

Let's make sure the firewall didn't break your SSH connection.

**If you're already connected via SSH:**

Your current connection should still work (existing connections aren't affected).

**Open a NEW terminal/SSH window** and try connecting again:

```bash
ssh username@192.168.1.50
```

(Replace with your username and IP from Section 2)

**✓ Expected result:**

You should connect successfully and see the command prompt.

**✓ Success Criteria:**
* You can open a new SSH connection
* You can log in with your password
* You see the Ubuntu welcome message

**⚠ If you can't connect:**

**Don't panic!** Your existing SSH window should still be open.

In your working SSH window, check the firewall rules:

```bash
sudo ufw status
```

If you don't see `22/tcp ALLOW Anywhere`, add it:

```bash
sudo ufw allow 22/tcp
```

Then try the new connection again.

**🚨 If you're completely locked out:**

You'll need physical access to the server. At the server console:

```bash
sudo ufw allow 22/tcp
```

Or temporarily disable the firewall:

```bash
sudo ufw disable
```

---

### Step 3.8: Verify Detailed Firewall Settings

For a more detailed view of your firewall configuration:

```bash
sudo ufw status verbose
```

**✓ Expected output:**

```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
2283/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
2283/tcp (v6)              ALLOW IN    Anywhere (v6)
```

**📝 Understanding the output:**

- **Logging: on (low)** - UFW is logging blocked connections (for debugging)
- **Default: deny (incoming)** - Block all incoming traffic by default (good!)
- **Default: allow (outgoing)** - Allow all outgoing traffic (so you can browse the web, download updates, etc.)
- **ALLOW IN** - These specific ports are exceptions to the deny rule

**💡 What this security model means:**

- ✅ People can connect TO your server on ports 22 and 2283
- ✅ Your server can connect OUT to anywhere (for updates, etc.)
- ❌ Everything else is blocked (no one can access random ports)

---

### Step 3.9: Understanding What's Blocked

Let's confirm that other ports are actually blocked.

**From another computer on your network**, try connecting to a random port:

```bash
telnet 192.168.1.50 12345
```

(Replace IP with your server's IP)

**✓ Expected result:**

```
Trying 192.168.1.50...
telnet: Unable to connect to remote host: Connection refused
```

Or it will just hang and timeout.

**📝 This is GOOD!** It means random ports are blocked.

Press `Ctrl + C` to cancel if it hangs.

**💡 If you don't have telnet:**

That's fine - just trust that UFW is doing its job. The default "deny incoming" rule blocks everything except what you explicitly allowed.

---

### Step 3.10: Understanding UFW Logging (Optional)

UFW logs blocked connection attempts. This is useful for security monitoring.

View recent blocked connections:

```bash
sudo tail -20 /var/log/ufw.log
```

**✓ Expected output:**

```
Feb  3 12:34:56 immich-server kernel: [UFW BLOCK] IN=enp0s3 OUT= MAC=... SRC=192.168.1.100 DST=192.168.1.50 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=12345 PROTO=TCP SPT=54321 DPT=12345 ...
```

**📝 What this means:**

- **[UFW BLOCK]** - UFW blocked this connection
- **SRC=192.168.1.100** - Source IP (who tried to connect)
- **DST=192.168.1.50** - Destination (your server)
- **DPT=12345** - Destination port they tried to access

**⚠ If the file doesn't exist:**

That's fine - it means nothing has been blocked yet, or logging isn't configured. This isn't a problem.

**💡 You don't need to monitor this actively.** It's just here if you want to investigate suspicious activity later.

---

### 📚 Common UFW Commands Reference

Here are useful commands for managing UFW in the future:

**Check status:**
```bash
sudo ufw status
sudo ufw status verbose    # More details
sudo ufw status numbered   # Shows rule numbers
```

**Add rules:**
```bash
sudo ufw allow 80/tcp      # Allow HTTP
sudo ufw allow 443/tcp     # Allow HTTPS
sudo ufw allow from 192.168.1.100   # Allow all traffic from specific IP
```

**Remove rules:**
```bash
sudo ufw status numbered   # See rule numbers
sudo ufw delete 3          # Delete rule #3
```

**Disable/enable firewall:**
```bash
sudo ufw disable   # Turn off (emergency use only)
sudo ufw enable    # Turn on
```

**Reset firewall (start over):**
```bash
sudo ufw reset     # Removes all rules
```

**💡 You don't need to memorize these.** Just know you can refer back to this section when needed.

---

### ✅ Checkpoint: Section 3 Complete

**You should now have:**

* ✓ UFW firewall enabled and active
* ✓ SSH access allowed (port 22)
* ✓ Immich access allowed (port 2283)
* ✓ All other ports blocked by default
* ✓ Firewall configured to start automatically on boot
* ✓ Ability to connect via SSH through the firewall

**What you learned:**

* Why firewalls are important (block unwanted access)
* How to use UFW (Ubuntu's firewall)
* The importance of allowing SSH BEFORE enabling the firewall
* Default deny policy (block everything except what you allow)
* How to check firewall status and logs
* Basic UFW commands for future reference

**Security status:**

| Port | Status | Purpose |
|------|--------|---------|
| 22 | ✅ Open | SSH (remote access) |
| 2283 | ✅ Open | Immich (will install in Section 6) |
| 80 | ❌ Closed | HTTP (will open in Section 9 for remote access) |
| 443 | ❌ Closed | HTTPS (will open in Section 10 for SSL) |
| All others | ❌ Closed | Default deny |

**💡 Important notes:**

* Don't disable UFW unless absolutely necessary
* If you add services later, remember to add firewall rules
* The firewall only affects incoming connections - you can still browse the web, download updates, etc.

**Next steps:**

In Section 4, you'll install Docker and Docker Compose, which are required to run Immich and its dependencies in containers.

**🔖 Quick reference:**

To check firewall status anytime: `sudo ufw status`

---

## 4. Install Docker and Docker Compose

**📋 What You'll Accomplish:**
* Install Docker Engine from the official Docker repository
* Install Docker Compose plugin for managing multi-container applications
* Configure your user account to run Docker commands without sudo
* Verify Docker is working correctly
* Understand what Docker containers are and why we use them

**⏱ Estimated Time:** 10-15 minutes

**📦 Prerequisites:**
* ✓ Section 1 complete (Ubuntu Server installed and updated)
* ✓ Section 2 complete (Static IP configured)
* ✓ Section 3 complete (Firewall configured)
* ✓ Internet connection working

**💡 What is Docker and why do we need it?**

**Docker** packages applications and their dependencies into isolated "containers." Think of containers like self-contained apartments in a building - each has everything it needs to run, but they share the building's foundation (the operating system).

**Why use Docker for Immich?**

Without Docker, you'd need to manually install:
- PostgreSQL database
- Redis cache server
- Python with machine learning libraries
- Node.js server
- FFmpeg for video processing
- Specific versions that all work together

With Docker, all of this is packaged and tested. You just run one command and everything works together.

**Benefits for beginners:**
- ✅ One-command installation
- ✅ One-command updates
- ✅ Can't break your system (containers are isolated)
- ✅ Easy to uninstall (just delete containers)

---

### Step 4.1: Remove Old Docker Versions (If Any)

First, let's remove any old Docker packages that might conflict.

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

**What this does:** Removes outdated Docker packages from Ubuntu's default repositories.

**✓ Expected output:**

Either:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package 'docker' is not installed, so not removed
Package 'docker-engine' is not installed, so not removed
...
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

Or if old versions exist:
```
The following packages will be REMOVED:
  docker.io containerd runc
...
Removing docker.io ...
```

**📝 Both outputs are fine.** The first means no old versions exist (most common for fresh installs). The second means old versions were removed.

---

### Step 4.2: Install Prerequisites

Docker needs a few tools to download and verify packages securely.

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
```

**What this does:** Installs SSL certificates and tools needed for secure downloads.

**✓ Expected output:**

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20240203)
curl is already the newest version (8.5.0-2ubuntu10.5)
gnupg is already the newest version (2.4.4-2ubuntu17)
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

**📝 "already the newest version"** is normal - these tools are usually pre-installed. No action needed.

---

### Step 4.3: Add Docker's Official GPG Key

This verifies that Docker packages you download are authentic and haven't been tampered with.

**Create a directory for keyrings:**

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

**What this does:** Creates a secure directory for storing package verification keys.

**✓ Expected output:** No output (silent success is normal for `install` command).

---

**Download Docker's GPG key:**

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

**What this does:** Downloads Docker's public key and saves it in a format Ubuntu can use.

**✓ Expected output:** No output (silent success).

**⚠ If you see an error:**

**"curl: command not found"**
- You skipped Step 4.2. Go back and install curl.

**"Failed to connect"**
- Check your internet connection: `ping -c 4 google.com`

---

**Make the key readable:**

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

**What this does:** Sets permissions so the package manager can read the key.

**✓ Expected output:** No output (silent success).

---

### Step 4.4: Add Docker Repository

Now add Docker's official repository to Ubuntu's package sources.

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**What this does:** Adds a configuration file telling Ubuntu where to download Docker packages from.

**✓ Expected output:** No visible output (redirected to `/dev/null`).

**📝 Understanding this command:**
- `dpkg --print-architecture` - Gets your system architecture (amd64 for most PCs)
- `$VERSION_CODENAME` - Gets your Ubuntu version name (e.g., "oracular" for 25.10)
- `tee` - Writes to a file with sudo permissions
- `> /dev/null` - Hides the output to keep things clean

**Verify the repository was added:**

```bash
cat /etc/apt/sources.list.d/docker.list
```

**✓ Expected output:**

```
deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu oracular stable
```

Your architecture and codename might differ (that's fine).

---

### Step 4.5: Update Package Index

Now that the Docker repository is added, update the package list.

```bash
sudo apt update
```

**What this does:** Downloads the list of available packages from Docker's repository.

**✓ Expected output:**

```
Hit:1 http://archive.ubuntu.com/ubuntu oracular InRelease
Get:2 https://download.docker.com/linux/ubuntu oracular InRelease [48.8 kB]
Get:3 https://download.docker.com/linux/ubuntu oracular/stable amd64 Packages [35.5 kB]
...
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
```

**📝 Key indicator:** You should see a line mentioning `download.docker.com` - this confirms the Docker repository was added successfully.

---

### Step 4.6: Install Docker Engine and Docker Compose

Now install Docker and all required components.

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**What this does:** Installs Docker Engine, command-line tools, and Docker Compose.

**📝 What each package does:**
- **docker-ce** - Docker Engine (the core daemon that runs containers)
- **docker-ce-cli** - Command-line interface (the `docker` command)
- **containerd.io** - Container runtime (low-level tool that actually runs containers)
- **docker-buildx-plugin** - Tool for building container images
- **docker-compose-plugin** - Tool for running multi-container apps (what Immich uses)

**✓ Expected output:**

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  docker-ce-rootless-extras libslirp0 pigz slirp4netns
The following NEW packages will be installed:
  containerd.io docker-buildx-plugin docker-ce docker-ce-cli
  docker-ce-rootless-extras docker-compose-plugin libslirp0 pigz slirp4netns
0 upgraded, 9 newly installed, 0 to remove and 0 not upgraded.
Need to get 123 MB of archives.
After this operation, 456 MB of additional disk space will be used.
Get:1 https://download.docker.com/linux/ubuntu oracular/stable amd64 containerd.io ...
...
Setting up docker-ce (5:27.3.1-1~ubuntu.25.10~oracular) ...
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /usr/lib/systemd/system/docker.socket.
Processing triggers for man-db (2.12.1-2) ...
```

**⏱ This takes 2-5 minutes** depending on your internet speed (downloading ~123 MB).

**✓ Success Criteria:**
* No error messages
* Ends with "Processing triggers..." and returns to prompt
* You see "Created symlink" messages (Docker service is auto-starting)

**⚠ If you see errors:**

**"Unable to locate package docker-ce"**
- The repository wasn't added correctly
- Go back to Step 4.4 and verify the repository file exists
- Run `sudo apt update` again

**"Hash Sum mismatch"**
- Network issue during download
- Run `sudo apt update` and try the install command again

---

### Step 4.7: Verify Docker Installation

Check that Docker installed correctly and is running.

**Check Docker version:**

```bash
sudo docker --version
```

**✓ Expected output:**

```
Docker version 27.3.1, build ce12230
```

Your version number might be different (that's fine - as long as it's 20.x or higher).

---

**Check Docker Compose version:**

```bash
sudo docker compose version
```

**✓ Expected output:**

```
Docker Compose version v2.29.7
```

Your version might differ (that's fine - as long as it's 2.x or higher).

**💡 Note:** The command is `docker compose` (with a space), not `docker-compose` (with a hyphen). Version 2.x uses the space format.

---

**Check Docker service status:**

```bash
sudo systemctl status docker
```

**✓ Expected output:**

```
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-02-03 14:23:45 UTC; 2min 15s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 12345 (dockerd)
      Tasks: 8
     Memory: 32.5M (peak: 35.2M)
        CPU: 156ms
     CGroup: /system.slice/docker.service
             └─12345 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

**📝 Key indicators:**
- **Loaded: loaded** - Docker service configuration loaded
- **Active: active (running)** - Docker is running right now (shown in green)
- **enabled** - Docker will start automatically on boot

Press `q` to exit this view.

**✓ Success Criteria:**
* Status shows "active (running)" in green
* Service is "enabled"

**⚠ If Docker is not running:**

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

---

### Step 4.8: Test Docker with Hello World

Let's run a test container to make sure Docker works.

```bash
sudo docker run hello-world
```

**What this does:** Downloads a tiny test container and runs it.

**✓ Expected output:**

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete
Digest: sha256:d37ada95d47ad12224c205a938129df7a3e52345828b4fa27318f5e5514d2b1e
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

**📝 Understanding the output:**
- **"Unable to find image locally"** - Normal first time running a container
- **"Pulling from library/hello-world"** - Downloading the test container
- **"Hello from Docker!"** - Success! Docker is working

**✓ Success Criteria:**
* You see "Hello from Docker!"
* No error messages

**⚠ If you see errors:**

**"Cannot connect to the Docker daemon"**
- Docker service isn't running
- Run: `sudo systemctl start docker`

**"Permission denied"**
- You need sudo (we'll fix this in the next step)

---

### Step 4.9: Add Your User to the Docker Group

Right now, you need `sudo` for every Docker command. Let's fix that.

**Add yourself to the docker group:**

```bash
sudo usermod -aG docker $USER
```

**What this does:** Adds your user account to the "docker" group, which has permission to run Docker commands.

**📝 Understanding the command:**
- `usermod` - Modify a user account
- `-aG` - Append to group (don't remove from other groups)
- `docker` - The group name
- `$USER` - Your current username (auto-filled)

**✓ Expected output:** No output (silent success).

---

**Verify the change:**

```bash
groups $USER
```

**✓ Expected output:**

```
username : username adm cdrom sudo dip plugdev lxd docker
```

**📝 Key indicator:** You should see `docker` at the end of the list.

**⚠ If you don't see "docker":**
- The command might have failed silently
- Try running it again with your username explicitly: `sudo usermod -aG docker yourusername`

---

### Step 4.10: Log Out and Back In

Group membership changes don't take effect until you log out and back in.

**Log out:**

```bash
exit
```

**What this does:** Closes your current SSH session (or console login).

---

**Log back in:**

If you're using SSH from another computer:

```bash
ssh username@192.168.1.50
```

If you're at the server console:
- Type your username and press Enter
- Type your password and press Enter

---

### Step 4.11: Verify Docker Works Without Sudo

Now test that you can run Docker without `sudo`.

```bash
docker ps
```

**What this does:** Lists running Docker containers (should be empty right now).

**✓ Expected output:**

```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

**📝 Key indicator:** An empty table with column headers - this means Docker is working and you have no containers running yet.

**✓ Success Criteria:**
* No "permission denied" error
* You see the column headers even though the list is empty
* No `sudo` needed!

---

**⚠ If you see "permission denied":**

The group change didn't take effect. Try these steps:

1. **Check groups again:**
   ```bash
   groups
   ```
   If you don't see `docker` in the list, you need to reboot.

2. **Reboot the server:**
   ```bash
   sudo reboot
   ```

3. **Log back in and test again:**
   ```bash
   docker ps
   ```

**🔧 Alternative (if reboot didn't help):**

Log out completely, then SSH back in:
```bash
exit
ssh username@192.168.1.50
docker ps
```

---

### Step 4.12: Run Hello World Without Sudo

Let's confirm everything works without sudo.

```bash
docker run hello-world
```

**✓ Expected output:**

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

**✓ Success Criteria:**
* You see "Hello from Docker!"
* No permission errors
* No `sudo` needed

---

### Step 4.13: Clean Up Test Container

The hello-world container is still stored on your system. Let's remove it to keep things clean.

**List all containers (including stopped ones):**

```bash
docker ps -a
```

**✓ Expected output:**

```
CONTAINER ID   IMAGE           COMMAND      CREATED          STATUS                      PORTS     NAMES
a1b2c3d4e5f6   hello-world     "/hello"     2 minutes ago    Exited (0) 2 minutes ago              eloquent_darwin
f6e5d4c3b2a1   hello-world     "/hello"     5 minutes ago    Exited (0) 5 minutes ago              brave_tesla
```

**📝 What this shows:**
- **CONTAINER ID** - Unique ID for each container
- **IMAGE** - The container image used
- **STATUS** - "Exited" means it ran and stopped
- **NAMES** - Random name assigned by Docker

---

**Remove the test containers:**

```bash
docker container prune
```

**What this does:** Removes all stopped containers.

**You'll see a prompt:**

```
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N]
```

**Type `y` and press Enter.**

**✓ Expected output:**

```
Deleted Containers:
a1b2c3d4e5f6
f6e5d4c3b2a1

Total reclaimed space: 0B
```

---

**Also remove the test image:**

```bash
docker image rm hello-world
```

**✓ Expected output:**

```
Untagged: hello-world:latest
Untagged: hello-world@sha256:d37ada95d47ad12224c205a938129df7a3e52345828b4fa27318f5e5514d2b1e
Deleted: sha256:d2c94e258dcb3c5ac2798d32e1249e42ef01cba4841c2234249495f87264ac5a
Deleted: sha256:ac28800ec8bb38d5c35b49d45a6ac4777544941199075dff8c4eb63e093aa81e
```

---

**Verify cleanup:**

```bash
docker ps -a
docker images
```

**✓ Expected output:**

```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

And:

```
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

Both should be empty (just headers).

---

### ✅ Checkpoint: Section 4 Complete

**You should now have:**

* ✓ Docker Engine installed and running
* ✓ Docker Compose plugin installed
* ✓ Your user added to the docker group (no sudo needed)
* ✓ Ability to run Docker commands: `docker ps`, `docker run`, etc.
* ✓ Docker service configured to start automatically on boot
* ✓ Test container successfully run and cleaned up

**What you learned:**

* What Docker is and why it's useful (containerization)
* How to add external package repositories to Ubuntu
* How to verify package signatures with GPG keys
* How to manage systemd services (`systemctl`)
* How Linux groups and permissions work (`usermod -aG`)
* Basic Docker commands: `docker run`, `docker ps`, `docker images`
* How to clean up unused containers and images

**Docker installation summary:**

| Component | Version | Status |
|-----------|---------|--------|
| Docker Engine | 27.x+ | ✅ Installed |
| Docker CLI | 27.x+ | ✅ Installed |
| Containerd | Latest | ✅ Installed |
| Docker Compose | 2.x+ | ✅ Installed |
| User permissions | docker group | ✅ Configured |

**Useful commands reference:**

```bash
# Check Docker status
docker ps                    # Running containers
docker ps -a                 # All containers
docker images                # Downloaded images
systemctl status docker      # Docker service status

# Clean up
docker container prune       # Remove stopped containers
docker image prune           # Remove unused images
docker system prune          # Remove everything unused

# Get help
docker --help                # List all commands
docker run --help            # Help for specific command
```

**Next steps:**

In Section 5, you'll set up storage for Immich by creating a directory where your photos will be stored, and you'll check that you have enough disk space for your photo library.

**🔖 Docker quick test:** Anytime you want to verify Docker is working, run: `docker run --rm hello-world` (the `--rm` flag automatically removes the container after it runs).

---

## 5. Set Up Storage for Immich

**📋 What You'll Accomplish:**
* Understand storage planning for photo libraries
* Create a dedicated directory for Immich to store photos
* Set correct ownership and permissions
* Check available disk space
* Calculate if you have enough space for your photo library

**⏱ Estimated Time:** 5 minutes

**📦 Prerequisites:**
* ✓ Section 1 complete (Ubuntu Server installed)
* ✓ Section 4 complete (Docker installed)

**💡 Why plan storage now?**

Photo libraries grow over time. Starting with the right storage setup prevents painful migrations later. If you run out of space or need to move photos to a bigger drive after uploading thousands of photos, it's time-consuming and risky.

**Common beginner mistake:** Using the root partition (`/`) and running out of space, which can crash the entire system. We'll avoid this by planning ahead.

---

### Step 5.1: Understanding Storage Options

Before creating directories, let's understand the two main approaches:

**Option A: Use the main system drive** (what this guide uses)
* ✅ Simple - no extra hardware needed
* ✅ Good for: Libraries under 500GB
* ⚠️ Risk: If you fill the root partition, the whole system becomes unstable
* 💡 Best for: Small to medium libraries, or testing Immich

**Option B: Use a separate drive/partition**
* ✅ System and photos are isolated
* ✅ Good for: Libraries over 500GB or unlimited growth
* ✅ Can easily replace/upgrade drive later
* ⚠️ Complexity: Requires mounting drives, more configuration
* 💡 Best for: Large libraries (100,000+ photos), or serious long-term use

**For this guide:** We'll use **Option A** with a dedicated directory on the main system drive. This is perfect for beginners and small-to-medium libraries.

**🔍 If you want Option B:** See the "Advanced Storage Setup" section at the end of this guide. Complete this section first, then come back and modify the storage path.

---

### Step 5.2: Check Current Disk Usage

Before creating directories, let's see how much space you have.

```bash
df -h /
```

**What this does:** Shows disk space for the root partition (where we'll store photos).

**✓ Expected output:**

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       100G  8.2G   87G   9% /
```

**📝 Understanding the output:**

| Column | Meaning | Example |
|--------|---------|---------|
| **Filesystem** | The disk device | `/dev/sda2` |
| **Size** | Total disk size | `100G` (100 GB) |
| **Used** | Space currently used | `8.2G` (8.2 GB) |
| **Avail** | Space available for use | `87G` (87 GB) |
| **Use%** | Percentage used | `9%` |
| **Mounted on** | Where it's accessible | `/` (root) |

**✓ Success Criteria:**
* You see a filesystem mounted on `/`
* "Avail" shows free space available
* "Use%" is under 80% (healthy)

**📝 In this example:** You have 87 GB available for photos.

**⚠ If Use% is over 80%:**
- You're running low on space
- Consider using a separate drive (see "Advanced Storage Setup" section)
- Or clean up unnecessary files

---

### Step 5.3: Estimate Your Storage Needs

Let's calculate how much space your photo library will need.

**📊 Storage estimation guide:**

| Library Size | Average Space Needed | Notes |
|--------------|---------------------|--------|
| 1,000 photos | ~5 GB | Small collection |
| 5,000 photos | ~25 GB | Moderate collection |
| 10,000 photos | ~50 GB | Large collection |
| 25,000 photos | ~125 GB | Very large |
| 50,000 photos | ~250 GB | Serious photographer |
| 100,000 photos | ~500 GB | Professional/multi-year archive |

**📝 These are estimates assuming:**
- Modern smartphone photos (12-24 MP, JPEG compressed)
- Mix of photos and some videos
- No RAW files (RAW files are 3-5x larger)

**Multiply by 1.5-2x if:**
- You shoot RAW format
- You have many 4K videos
- You want room for growth

**✍️ Calculate your needs:**

How many photos do you have? `_____________`

Estimated space needed: `_____________` GB

Available space from Step 5.2: `_____________` GB

**✓ You're good to proceed if:** Estimated space < (Available space ÷ 2)

The "÷ 2" gives you a safety margin for system updates, Docker images, and database.

**⚠ If you don't have enough space:**
- Consider using a separate drive
- Or reduce your library size (keep only recent photos)
- See "Advanced Storage Setup" section at the end

---

### Step 5.4: Create the Immich Directory

Now create the directory where Immich will store your photos.

```bash
sudo mkdir -p /var/lib/immich
```

**What this does:** Creates the directory `/var/lib/immich` for photo storage.

**📝 Understanding the command:**
- `mkdir` - Make directory
- `-p` - Create parent directories if needed (and don't error if it exists)
- `/var/lib/immich` - The path to create

**💡 Why `/var/lib/immich`?**
- `/var/lib` is the standard Linux location for application data
- Keeps photos separate from system files
- Easy to find and back up

**✓ Expected output:** No output (silent success).

---

### Step 5.5: Verify the Directory Was Created

Check that the directory exists.

```bash
ls -ld /var/lib/immich
```

**What this does:** Lists details about the directory.

**✓ Expected output:**

```
drwxr-xr-x 2 root root 4096 Feb  3 15:23 /var/lib/immich
```

**📝 Understanding the output:**
- **d** - This is a directory (not a file)
- **rwxr-xr-x** - Permission settings (we'll change these next)
- **root root** - Owner and group (currently root - we'll fix this)
- **4096** - Size in bytes (empty directory)
- **Feb 3 15:23** - Creation date/time
- **/var/lib/immich** - The directory path

**✓ Success Criteria:**
* First character is `d` (directory)
* Path ends with `/var/lib/immich`
* No "No such file or directory" error

---

### Step 5.6: Set Directory Ownership

The directory is currently owned by root. Let's change it to your user so Docker can write to it.

```bash
sudo chown -R $USER:$USER /var/lib/immich
```

**What this does:** Changes the owner and group of the directory to your user account.

**📝 Understanding the command:**
- `chown` - Change ownership
- `-R` - Recursive (apply to directory and all contents)
- `$USER:$USER` - Set owner and group to your username (auto-filled)
- `/var/lib/immich` - The directory to change

**✓ Expected output:** No output (silent success).

---

### Step 5.7: Verify Ownership Change

Check that the ownership was changed correctly.

```bash
ls -ld /var/lib/immich
```

**✓ Expected output:**

```
drwxr-xr-x 2 username username 4096 Feb  3 15:23 /var/lib/immich
```

**📝 Key change:**
- **Before:** `root root`
- **After:** `username username` (your actual username)

**✓ Success Criteria:**
* Owner shows your username (not root)
* Group shows your username (not root)

**⚠ If you still see "root root":**

The command might have failed. Try running it explicitly with your username:

```bash
sudo chown -R yourusername:yourusername /var/lib/immich
```

Replace `yourusername` with your actual username.

---

### Step 5.8: Verify Permissions

Let's make sure the permissions allow reading and writing.

```bash
stat /var/lib/immich
```

**What this does:** Shows detailed information about the directory.

**✓ Expected output:**

```
  File: /var/lib/immich
  Size: 4096      	Blocks: 8          IO Block: 4096   directory
Device: 8,2	Inode: 1234567     Links: 2
Access: (0755/drwxr-xr-x)  Uid: ( 1000/username)   Gid: ( 1000/username)
Access: 2025-02-03 15:23:45.123456789 +0000
Modify: 2025-02-03 15:23:45.123456789 +0000
Change: 2025-02-03 15:24:12.987654321 +0000
 Birth: 2025-02-03 15:23:45.123456789 +0000
```

**📝 Key information:**
- **Access: (0755/drwxr-xr-x)** - Permission mode
  - Owner can read, write, execute (`rwx`)
  - Group can read and execute (`r-x`)
  - Others can read and execute (`r-x`)
- **Uid: (1000/username)** - Owner is your user
- **Gid: (1000/username)** - Group is your user

**💡 These permissions are perfect for Immich:**
- Your user (running Docker) can read/write
- Others on the system can read (but not modify)

**✓ Success Criteria:**
* Access shows `0755` or `drwxr-xr-x`
* Uid and Gid show your username

---

### Step 5.9: Test Write Access

Let's confirm you can actually write to the directory.

```bash
touch /var/lib/immich/test.txt
```

**What this does:** Creates an empty file to test write permissions.

**✓ Expected output:** No output (silent success).

---

**Verify the test file was created:**

```bash
ls -l /var/lib/immich/
```

**✓ Expected output:**

```
total 0
-rw-r--r-- 1 username username 0 Feb  3 15:25 test.txt
```

**✓ Success Criteria:**
* You see `test.txt` in the listing
* Owner is your username
* No permission errors

---

**Remove the test file:**

```bash
rm /var/lib/immich/test.txt
```

**✓ Expected output:** No output (silent success).

---

**Verify it's gone:**

```bash
ls -l /var/lib/immich/
```

**✓ Expected output:**

```
total 0
```

The directory should be empty again.

---

### Step 5.10: Check Final Disk Space Available

Now that the directory is set up, let's do a final check on available space.

```bash
df -h /var/lib/immich
```

**✓ Expected output:**

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       100G  8.2G   87G   9% /
```

**📝 This should match Step 5.2** (we haven't used any space yet - just created an empty directory).

**✍️ Document your storage:**

| Information | Your Value |
|-------------|------------|
| Storage location | `/var/lib/immich` |
| Total available space | `_____________` GB |
| Estimated photo library size | `_____________` GB |
| Safety margin remaining | `_____________` GB |

**💡 Monitor this space periodically:**

As you upload photos, check disk usage:

```bash
du -sh /var/lib/immich
```

This shows how much space Immich is using.

---

### Step 5.11: Understanding Directory Structure (Preview)

After Immich is installed, this directory will contain:

```
/var/lib/immich/
├── library/           # Your actual photo/video files
├── upload/            # Temporary upload staging
├── thumbs/            # Generated thumbnails
├── encoded-video/     # Transcoded videos
└── profile/           # User profile pictures
```

**💡 You don't need to create these.** Immich creates them automatically when it starts.

**📝 For backups:** You'll back up the entire `/var/lib/immich` directory (covered in Section 15).

---

### ✅ Checkpoint: Section 5 Complete

**You should now have:**

* ✓ Understanding of storage planning (Option A vs Option B)
* ✓ Knowledge of your available disk space
* ✓ Estimated storage needs for your photo library
* ✓ Created directory: `/var/lib/immich`
* ✓ Correct ownership set (your user, not root)
* ✓ Verified write permissions work
* ✓ Confirmation that you have enough space

**What you learned:**

* How to check disk space with `df -h`
* How to estimate photo library size
* Where Linux applications store data (`/var/lib`)
* How to create directories with `mkdir -p`
* How to change ownership with `chown`
* How to check permissions with `ls -l` and `stat`
* How to test write access with `touch`
* The importance of leaving safety margin (50% free space)

**Storage configuration summary:**

| Setting | Value |
|---------|-------|
| Storage path | `/var/lib/immich` |
| Owner | Your username |
| Permissions | `0755` (rwxr-xr-x) |
| Location | Root partition `/` |
| Type | Option A (main system drive) |

**Useful commands reference:**

```bash
# Check disk space
df -h /var/lib/immich              # Available space
du -sh /var/lib/immich             # Used space (after uploading photos)
du -h /var/lib/immich | sort -rh   # Largest subdirectories

# Check directory details
ls -ld /var/lib/immich             # Directory info
ls -lh /var/lib/immich             # Contents
stat /var/lib/immich               # Detailed stats
```

**⚠️ Important reminders:**

* Keep disk usage under 80% (system stability)
* Monitor space as you upload photos
* Plan for growth (photos accumulate over time)
* Back up this directory regularly (Section 15)

**Next steps:**

In Section 6, you'll install Immich using Docker Compose. You'll download the configuration files, customize settings, and start all the Immich containers. By the end of Section 6, you'll have Immich running and accessible on your local network!

**🔖 Storage quick check:** Run `df -h /var/lib/immich` anytime to check available space.

---

## 6. Install and Configure Immich

**📋 What You'll Accomplish:**
* Download Immich's Docker Compose configuration
* Configure environment variables (storage location, database password, timezone)
* Download all required Docker images
* Start all Immich services (server, database, machine learning, etc.)
* Create your admin account
* Access Immich's web interface

**⏱ Estimated Time:** 15-20 minutes (mostly waiting for downloads)

**📦 Prerequisites:**
* ✓ Section 4 complete (Docker installed)
* ✓ Section 5 complete (Storage directory created at `/var/lib/immich`)
* ✓ Internet connection working
* ✓ At least 10 GB free disk space (for Docker images and database)

**💡 What you're about to install:**

Immich consists of multiple services that work together:
- **immich-server** - Web interface and API
- **immich-microservices** - Background tasks (thumbnail generation, etc.)
- **immich-machine-learning** - AI features (face recognition, object detection)
- **postgres** - Database (stores metadata about your photos)
- **redis** - Cache (makes the app faster)

Docker Compose manages all of these for you with a single configuration file.

---

### Step 6.1: Create Immich Configuration Directory

Create a directory in your home folder to store Immich's configuration files.

```bash
mkdir -p ~/immich
```

**What this does:** Creates a directory called `immich` in your home directory.

**✓ Expected output:** No output (silent success).

---

**Navigate to the directory:**

```bash
cd ~/immich
```

**What this does:** Changes your working directory to `~/immich`.

**✓ Expected output:** No output (your prompt might change to show `~/immich`).

---

**Verify you're in the right place:**

```bash
pwd
```

**✓ Expected output:**

```
/home/username/immich
```

(Replace `username` with your actual username)

**📝 Understanding paths:**
- `~` is shorthand for your home directory (`/home/username`)
- `~/immich` expands to `/home/username/immich`
- This is separate from `/var/lib/immich` (where photos are stored)

---

### Step 6.2: Download Docker Compose Configuration

Download Immich's official Docker Compose file from GitHub.

```bash
curl -o docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
```

**What this does:** Downloads the latest `docker-compose.yml` file and saves it in the current directory.

**📝 Understanding the command:**
- `curl` - Tool for downloading files from the internet
- `-o docker-compose.yml` - Save as this filename
- `https://github.com/...` - Where to download from

**✓ Expected output:**

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100  4521  100  4521    0     0  12345      0 --:--:-- --:--:-- --:--:-- 12345
```

**📝 Key indicator:** You see a progress bar showing 100% completion.

**⚠ If you see an error:**

**"curl: command not found"**
- Install curl: `sudo apt install curl`

**"Failed to connect" or "Could not resolve host"**
- Check your internet connection: `ping -c 4 google.com`

**"404 Not Found"**
- The URL might have changed. Visit https://github.com/immich-app/immich and check the latest release instructions.

---

**Verify the file was downloaded:**

```bash
ls -lh docker-compose.yml
```

**✓ Expected output:**

```
-rw-r--r-- 1 username username 4.5K Feb  3 16:45 docker-compose.yml
```

**✓ Success Criteria:**
* File exists
* Size is around 4-5 KB
* Owned by your user

---

### Step 6.3: Download Environment File Template

Download the example environment configuration file.

```bash
curl -o .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

**What this does:** Downloads the `.env` template file with default settings.

**💡 About `.env` files:**
- Store configuration variables (like passwords, paths)
- The dot (.) makes it a hidden file
- Docker Compose automatically reads this file

**✓ Expected output:**

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1234  100  1234    0     0   5678      0 --:--:-- --:--:-- --:--:--  5678
```

---

**Verify both files exist:**

```bash
ls -lha
```

**✓ Expected output:**

```
total 16K
drwxr-xr-x  2 username username 4.0K Feb  3 16:45 .
drwxr-x--- 15 username username 4.0K Feb  3 16:40 ..
-rw-r--r--  1 username username 1.3K Feb  3 16:45 .env
-rw-r--r--  1 username username 4.5K Feb  3 16:44 docker-compose.yml
```

**📝 Note the `-a` flag** shows hidden files (files starting with `.`).

**✓ Success Criteria:**
* You see `.env` (environment file)
* You see `docker-compose.yml` (Docker configuration)

---

### Step 6.4: Configure Environment Variables

Now customize the `.env` file with your settings.

**Open the file in nano:**

```bash
nano .env
```

**What this does:** Opens the environment file for editing.

You'll see a file with many settings. Most have good defaults, but you need to change a few critical ones.

---

**Look for these settings and modify them:**

**1. UPLOAD_LOCATION** (where photos are stored)

Find the line:
```bash
UPLOAD_LOCATION=./library
```

Change it to:
```bash
UPLOAD_LOCATION=/var/lib/immich
```

**💡 Why:** This tells Immich to store photos in the directory you created in Section 5.

---

**2. DB_PASSWORD** (database security)

Find the line:
```bash
DB_PASSWORD=postgres
```

Change it to a strong password:
```bash
DB_PASSWORD=MyStr0ng!Passw0rd2025
```

**⚠ Important password rules:**
- Use at least 16 characters
- Mix uppercase, lowercase, numbers, symbols
- Don't use common words
- Write it down somewhere safe (you'll need it for backups)

**✍️ Write down your DB password:** `_________________________`

**💡 You won't need to type this password regularly.** It's only for the database that Immich uses internally.

---

**3. TZ** (timezone for timestamps)

Find the line:
```bash
TZ=Etc/UTC
```

Change it to your timezone:
```bash
TZ=America/New_York
```

**📍 Find your timezone:**
- North America: `America/New_York`, `America/Chicago`, `America/Denver`, `America/Los_Angeles`
- Europe: `Europe/London`, `Europe/Paris`, `Europe/Berlin`
- Asia: `Asia/Tokyo`, `Asia/Shanghai`, `Asia/Dubai`
- Full list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones

**💡 Why this matters:** Photo timestamps will display in your local timezone instead of UTC.

---

**Optional settings you can customize:**

**4. IMMICH_VERSION** (which version to use)

```bash
IMMICH_VERSION=release
```

**Leave this as `release`** to always get the latest stable version.

---

**5. DB_HOSTNAME, DB_USERNAME, DB_DATABASE_NAME**

**Leave these at defaults** unless you know what you're doing.

---

**Save the file:**

1. Press `Ctrl + O` (write out)
2. Press `Enter` (confirm filename)
3. Press `Ctrl + X` (exit nano)

**✓ Success Criteria:**
* You see "Wrote X lines" at the bottom
* You're back at the command prompt

---

### Step 6.5: Verify Environment Configuration

Let's check that your changes were saved correctly.

```bash
grep -E "^(UPLOAD_LOCATION|DB_PASSWORD|TZ)=" .env
```

**What this does:** Shows only the lines you modified (ignoring comments).

**✓ Expected output:**

```
UPLOAD_LOCATION=/var/lib/immich
DB_PASSWORD=MyStr0ng!Passw0rd2025
TZ=America/New_York
```

**✓ Success Criteria:**
* `UPLOAD_LOCATION` shows `/var/lib/immich` (not `./library`)
* `DB_PASSWORD` is NOT `postgres` (you changed it)
* `TZ` shows your timezone (not `Etc/UTC`)

**⚠ If the values are wrong:**

Open the file again and fix them:
```bash
nano .env
```

---

### Step 6.6: Review Docker Compose File (Optional)

Let's quickly look at what services will be started.

```bash
cat docker-compose.yml
```

**What this does:** Displays the Docker Compose configuration.

You'll see a long YAML file. Don't worry about understanding all of it, but notice the main services:

```yaml
services:
  immich-server:
    ...
  immich-microservices:
    ...
  immich-machine-learning:
    ...
  redis:
    ...
  postgres:
    ...
```

**💡 You don't need to edit this file.** The official configuration is designed to work out of the box.

Press `q` if the output is long, or just let it scroll.

---

### Step 6.7: Pull Docker Images

Before starting Immich, download all required Docker images.

```bash
docker compose pull
```

**What this does:** Downloads all the container images needed to run Immich.

**⏱ This takes 5-15 minutes** depending on your internet speed (downloading 2-4 GB total).

**✓ Expected output:**

```
[+] Pulling 43/43
 ✔ redis 7 layers [⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled           5.2s
 ✔ postgres 13 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled    12.3s
 ✔ immich-server 15 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled    45.6s
 ✔ immich-microservices Skipped - same as immich-server
 ✔ immich-machine-learning 8 layers [⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled    78.9s
```

**📝 Progress indicators:**
- Each service shows a progress bar
- "Pulled" means download complete
- "Skipped" means image already exists or is reused

**✓ Success Criteria:**
* All services show "Pulled" or "Skipped"
* No error messages
* Returns to command prompt

**⚠ If you see errors:**

**"error pulling image configuration"**
- Network interruption during download
- Run the command again: `docker compose pull`

**"no space left on device"**
- Not enough disk space for Docker images
- Check space: `df -h /`
- Need at least 10 GB free

**"Cannot connect to the Docker daemon"**
- Docker isn't running: `sudo systemctl start docker`

---

**Verify images were downloaded:**

```bash
docker images
```

**✓ Expected output:**

```
REPOSITORY                             TAG       IMAGE ID       CREATED        SIZE
ghcr.io/immich-app/immich-server       release   a1b2c3d4e5f6   2 days ago     234MB
ghcr.io/immich-app/immich-machine-learning release   f6e5d4c3b2a1   2 days ago     1.89GB
tensorchord/pgvecto-rs                 pg16      9a8b7c6d5e4f   1 week ago     345MB
redis                                  6.2-alpine c4d5e6f7a8b9   2 weeks ago    32.3MB
```

**📝 Your output might differ** (different image IDs, dates, sizes) - that's fine as long as you see immich-related images.

**✓ Success Criteria:**
* You see `immich-server`, `immich-machine-learning`
* You see `postgres` or `pgvecto-rs`
* You see `redis`

---

### Step 6.8: Start Immich Services

Now start all Immich containers in detached mode (background).

```bash
docker compose up -d
```

**What this does:** Starts all services defined in `docker-compose.yml` in the background.

**📝 Understanding the command:**
- `docker compose` - Multi-container Docker command
- `up` - Create and start containers
- `-d` - Detached mode (run in background)

**⏱ This takes 1-2 minutes** for first-time initialization (database setup, etc.).

**✓ Expected output:**

```
[+] Running 6/6
 ✔ Network immich_default                 Created   0.1s
 ✔ Container immich-redis                 Started   1.2s
 ✔ Container immich-postgres              Started   1.5s
 ✔ Container immich-machine-learning      Started   2.3s
 ✔ Container immich-server                Started   3.1s
 ✔ Container immich-microservices         Started   3.4s
```

**📝 What's happening:**
- **Network created** - Docker creates a private network for containers
- **Containers started** - Each service starts in order
- **Times shown** - How long each took to start

**✓ Success Criteria:**
* All containers show "Started" or "Created"
* No error messages
* Returns to command prompt

**⚠ Common errors:**

**"port is already allocated"**
- Port 2283 is being used by something else
- Check what's using it: `sudo lsof -i :2283`
- Either stop that service or change Immich's port in docker-compose.yml

**"permission denied while trying to connect to the Docker daemon"**
- You're not in the docker group yet
- Go back to Section 4.10 and complete the group setup

**"Error response from daemon: pull access denied"**
- Image pull failed
- Run `docker compose pull` again

---

### Step 6.9: Check Container Status

Verify all containers are running.

```bash
docker compose ps
```

**What this does:** Shows the status of all Immich containers.

**✓ Expected output:**

```
NAME                      COMMAND                  SERVICE               STATUS              PORTS
immich-machine-learning   "/bin/sh -c 'python …"   immich-machine-learning   Up 30 seconds       3003/tcp
immich-microservices      "/bin/sh -c './start…"   immich-microservices      Up 30 seconds
immich-postgres           "docker-entrypoint.s…"   postgres              Up 35 seconds       5432/tcp
immich-redis              "docker-entrypoint.s…"   redis                 Up 35 seconds       6379/tcp
immich-server             "/bin/sh -c './start…"   immich-server         Up 30 seconds       0.0.0.0:2283->3001/tcp
```

**📝 Key indicators:**
- **STATUS** column should show "Up" for all containers
- **immich-server** should show `0.0.0.0:2283->3001/tcp` (port mapping)

**✓ Success Criteria:**
* All 5 containers are listed
* All show "Up" status (not "Exited" or "Restarting")
* immich-server shows port 2283 mapping

**⚠ If a container shows "Exited" or is missing:**

Check its logs to see what went wrong:

```bash
docker compose logs immich-server
```

(Replace `immich-server` with whichever container failed)

Common issues:
- Database initialization failed (check postgres logs)
- Port conflict (something else using port 2283)
- Permission issue with `/var/lib/immich`

---

### Step 6.10: Watch Initialization Logs

Let's watch the server logs to see Immich start up.

```bash
docker compose logs -f immich-server
```

**What this does:** Shows real-time logs from the Immich server container.

**📝 Understanding the command:**
- `logs` - Show container logs
- `-f` - Follow (keep watching for new logs)
- `immich-server` - Which container to watch

**✓ Expected output:**

You'll see a stream of log messages. Look for these key indicators:

```
[Nest] 1  - 02/03/2025, 4:56:23 PM     LOG [Bootstrap] Immich Server is listening on 0.0.0.0:3001 [v1.94.1]
[Nest] 1  - 02/03/2025, 4:56:23 PM     LOG [DatabaseService] Database initialized
[Nest] 1  - 02/03/2025, 4:56:24 PM     LOG [StorageCore] Storage initialized
[Nest] 1  - 02/03/2025, 4:56:24 PM     LOG [MicroservicesService] Microservices initialized
```

**📝 Key phrases to look for:**
- "Immich Server is listening" - Server started
- "Database initialized" - Database connection working
- "Storage initialized" - Can access `/var/lib/immich`
- No errors in red

**✓ Success Criteria:**
* You see "Immich Server is listening"
* No red error messages
* Logs are flowing (new lines appearing)

**Press `Ctrl + C`** to stop following logs (containers keep running).

**⚠ If you see errors:**

**"EACCES: permission denied, mkdir '/usr/src/app/upload'"**
- Permission problem with storage directory
- Fix: `sudo chown -R $USER:$USER /var/lib/immich`

**"Connection refused to postgres"**
- Database container isn't ready yet
- Wait 30 seconds and check logs again

**"UPLOAD_LOCATION must be set"**
- .env file wasn't read correctly
- Check that `.env` exists: `ls -la .env`
- Restart: `docker compose restart`

---

### Step 6.11: Access Immich Web Interface

Now the moment of truth - access Immich in your web browser!

**From a computer on the same network:**

1. Open your web browser (Chrome, Firefox, Safari, etc.)
2. Go to: `http://192.168.1.50:2283`

**(Replace `192.168.1.50` with your server's static IP from Section 2)**

**✓ Expected result:**

You should see the Immich welcome/setup page with a clean interface and a "Getting Started" section.

**🎉 If you see the Immich interface - SUCCESS!** Immich is running!

**⚠ If you can't connect:**

**"This site can't be reached" or "Connection refused"**
- Check the server is running: `docker compose ps`
- Check port in URL (should be `:2283`)
- Verify you're on the same network as the server
- Check firewall: `sudo ufw status` (should allow 2283/tcp)

**"Connection timed out"**
- Firewall blocking port 2283
- Add rule: `sudo ufw allow 2283/tcp`

**Browser shows a different website**
- Wrong IP address
- Verify server IP: `ip addr show`

---

### Step 6.12: Create Your Admin Account

Once you can access Immich's web interface, create your admin account.

You'll see a sign-up form. Fill it out:

1. **Email address:** Use a valid email (you can use it for password recovery later)
   - Example: `admin@yourdomain.com` or your personal email

2. **Password:** Create a strong password for your Immich account
   - At least 8 characters
   - Use mix of letters, numbers, symbols
   - **⚠ This is different from your DB_PASSWORD** - this is your login password

3. **First name:** Your first name

4. **Last name:** Your last name

**Click "Sign Up"**

**✓ Expected result:**

You're logged in and see the Immich dashboard with:
- A welcome message
- Empty photo library
- Navigation menu (Photos, Sharing, Albums, etc.)
- Upload button

**🎉 Congratulations! Immich is fully installed and configured!**

**✍️ Document your admin credentials:**

| Credential | Value |
|------------|-------|
| Admin email | `_________________________` |
| Admin password | `_________________________` |
| Web URL (local) | `http://192.168.1.50:2283` |

**⚠ Keep these safe!** You'll need them to log in.

---

### Step 6.13: Quick Tour of Immich Interface

Take a moment to explore the interface:

**📸 Main navigation (left sidebar):**
- **Photos** - View all your photos (empty for now)
- **Albums** - Organize photos into albums
- **Sharing** - Share photos with others
- **Archive** - Hide photos from main view
- **Favorites** - Your starred photos
- **Trash** - Deleted photos (recoverable for 30 days)

**⚙️ Settings (top right, gear icon):**
- **Account Settings** - Change password, email, etc.
- **App Settings** - Theme, language, defaults
- **Library** - Storage statistics
- **User Management** - Add family members (admin only)

**Don't upload photos yet!** You'll do that after completing the remaining sections (especially backups).

---

### ✅ Checkpoint: Section 6 Complete

**You should now have:**

* ✓ Docker Compose configuration downloaded
* ✓ Environment variables configured (storage path, DB password, timezone)
* ✓ All Docker images downloaded (2-4 GB)
* ✓ All 5 Immich containers running (server, microservices, ML, postgres, redis)
* ✓ Immich accessible at `http://192.168.1.50:2283`
* ✓ Admin account created and working
* ✓ Understanding of the Immich interface

**What you learned:**

* How to use Docker Compose for multi-container applications
* How to configure applications with `.env` files
* How to check container status with `docker compose ps`
* How to view container logs with `docker compose logs`
* The difference between service passwords (DB_PASSWORD) and user passwords
* How to access web services on your local network

**Immich installation summary:**

| Component | Status | Details |
|-----------|--------|---------|
| Docker images | ✅ Downloaded | 2-4 GB total |
| Configuration | ✅ Customized | Storage, password, timezone |
| Containers | ✅ Running | 5 services up |
| Database | ✅ Initialized | PostgreSQL with pgvecto-rs |
| Web interface | ✅ Accessible | Port 2283 |
| Admin account | ✅ Created | Ready to use |

**Useful commands reference:**

```bash
# Container management
docker compose ps              # Check status
docker compose logs -f         # Watch all logs
docker compose logs immich-server  # Specific container
docker compose restart         # Restart all containers
docker compose stop            # Stop all containers
docker compose start           # Start stopped containers
docker compose down            # Stop and remove containers
docker compose up -d           # Start containers

# Maintenance
docker compose pull            # Update to latest images
docker compose up -d           # Restart with new images
docker system prune            # Clean up unused data

# Configuration location
~/immich/docker-compose.yml    # Service configuration
~/immich/.env                  # Environment variables
/var/lib/immich/               # Photo storage
```

**Next steps:**

Your Immich installation is working, but it's only accessible on your local network. The remaining sections cover:

- **Section 7:** Nginx reverse proxy (optional, for HTTPS)
- **Section 8:** Domain name setup (required for remote access)
- **Section 9:** DNS and port forwarding (access from anywhere)
- **Section 10:** HTTPS with Let's Encrypt (secure remote access)
- **Sections 11-12:** Mobile apps
- **Section 13:** Convenience scripts (updates, backups)
- **Section 14:** Google Photos import
- **Section 15:** Backup strategy (**highly recommended before uploading photos**)

**🔖 Quick access:** Bookmark `http://192.168.1.50:2283` in your browser for easy access while on your home network.

**💡 Try it out:** Upload a test photo from the web interface to make sure everything works, then delete it. Don't upload your full library until you've set up backups (Section 15)!

---

## 7. Set Up Nginx Reverse Proxy (Strongly Recommended)

**📋 What You'll Accomplish:**
* Install Nginx web server
* Configure Nginx as a reverse proxy for Immich
* Enable HTTP access on port 80 (standard web port)
* Prepare for HTTPS setup in Section 10
* Understand how reverse proxies work

**⏱ Estimated Time:** 10-15 minutes

**📦 Prerequisites:**
* ✓ Section 6 complete (Immich installed and running)
* ✓ Immich accessible at `http://192.168.1.50:2283`

**💡 Should you do this section?**

**Yes, if you want:**
- ✅ Remote access to your photos (away from home)
- ✅ Mobile app auto-upload (works best with HTTPS)
- ✅ HTTPS encryption (required for modern browsers and mobile apps)
- ✅ Access via a domain name instead of IP:port
- ✅ Professional setup

**You can skip if:**
- ❌ You only want local network access (home WiFi only)
- ❌ You're comfortable typing `http://192.168.1.50:2283` every time
- ❌ You don't plan to use mobile apps much

**⚠ Our recommendation: Complete this section.** Even if you only want local access now, it's much easier to set up Nginx early than to add it later after you've been using Immich for months.

---

### What is a Reverse Proxy?

**Without a reverse proxy:**
```
Your Phone → Internet → Router → Your Server:2283 → Immich
```
- You access: `http://192.168.1.50:2283`
- No HTTPS encryption
- Awkward port number
- Immich is directly exposed

**With a reverse proxy (Nginx):**
```
Your Phone → Internet → Router → Nginx:443 → Immich:2283
```
- You access: `https://photos.yourdomain.com`
- HTTPS encryption
- Standard port (443)
- Nginx provides security layer

**💡 Think of Nginx as a receptionist:**
- Sits at the front desk (port 80/443)
- Receives all visitors
- Forwards them to the right department (Immich on port 2283)
- Handles security checks (HTTPS)

---

### Step 7.1: Install Nginx

Install the Nginx web server.

```bash
sudo apt update
sudo apt install -y nginx
```

**What this does:** Installs Nginx from Ubuntu's official repositories.

**✓ Expected output:**

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  fontconfig-config fonts-dejavu-core libdeflate0 libfontconfig1
  libgd3 libjbig0 libjpeg-turbo8 libjpeg8 libnginx-mod-http-geoip2
  ...
The following NEW packages will be installed:
  fontconfig-config fonts-dejavu-core libdeflate0 libfontconfig1
  libgd3 libjbig0 libjpeg-turbo8 libjpeg8 libnginx-mod-http-geoip2
  ...
  nginx nginx-common nginx-core
0 upgraded, 25 newly installed, 0 to remove and 0 not upgraded.
Need to get 3,456 kB of archives.
After this operation, 12.3 MB of additional disk space will be used.
...
Setting up nginx (1.24.0-2ubuntu7) ...
```

**⏱ This takes 1-2 minutes.**

**✓ Success Criteria:**
* Nginx and related packages installed
* No error messages
* Ends with "Setting up nginx..."

---

### Step 7.2: Start and Enable Nginx

Start the Nginx service and configure it to start on boot.

```bash
sudo systemctl start nginx
```

**What this does:** Starts the Nginx web server immediately.

**✓ Expected output:** No output (silent success).

---

```bash
sudo systemctl enable nginx
```

**What this does:** Configures Nginx to start automatically when the server boots.

**✓ Expected output:**

```
Synchronizing state of nginx.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable nginx
```

Or just:
```
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.
```

Both are fine.

---

### Step 7.3: Verify Nginx is Running

Check that Nginx started successfully.

```bash
sudo systemctl status nginx
```

**✓ Expected output:**

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-02-03 17:12:34 UTC; 15s ago
       Docs: man:nginx(8)
    Process: 12345 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 12346 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 12347 (nginx)
      Tasks: 3 (limit: 4567)
     Memory: 4.2M
        CPU: 45ms
     CGroup: /system.slice/nginx.service
             ├─12347 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─12348 "nginx: worker process" "" "" "" "" "" "" "" ""
             └─12349 "nginx: worker process" "" "" "" "" "" "" "" ""
```

**📝 Key indicators:**
- **Loaded: loaded** - Nginx service configuration loaded
- **enabled** - Will start on boot
- **Active: active (running)** - Currently running (shown in green)
- **Main PID:** - Process ID shows it's running

Press `q` to exit.

**✓ Success Criteria:**
* Status shows "active (running)" in green
* Service is "enabled"

**⚠ If Nginx is not running:**

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

---

### Step 7.4: Test Nginx Welcome Page

Nginx should now be serving the default welcome page.

**From another computer on your network**, open a web browser and go to:

```
http://192.168.1.50
```

(Replace with your server's IP)

**✓ Expected result:**

You should see the **Nginx welcome page** with:
- "Welcome to nginx!"
- "If you see this page, the nginx web server is successfully installed"
- Additional information about configuration

**🎉 If you see this page - Nginx is working!**

**⚠ If you don't see the Nginx page:**

**"This site can't be reached"**
- Nginx isn't running: `sudo systemctl start nginx`
- Wrong IP address: verify with `ip addr show`

**Connection timeout**
- Firewall blocking port 80
- We'll fix this in the next step

---

### Step 7.5: Update Firewall Rules

Allow HTTP traffic through the firewall.

```bash
sudo ufw allow 80/tcp
```

**What this does:** Opens port 80 (HTTP) for incoming connections.

**✓ Expected output:**

```
Rule added
Rule added (v6)
```

---

**Verify the firewall rule:**

```bash
sudo ufw status
```

**✓ Expected output:**

```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
2283/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
2283/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
```

**✓ Success Criteria:**
* You see `80/tcp ALLOW Anywhere`
* Previous rules (22, 2283) are still there

---

### Step 7.6: Create Nginx Configuration for Immich

Now configure Nginx to proxy requests to Immich.

**Create a new configuration file:**

```bash
sudo nano /etc/nginx/sites-available/immich
```

**What this does:** Creates a new Nginx configuration file specifically for Immich.

**📝 Understanding Nginx structure:**
- `/etc/nginx/sites-available/` - Configuration files (not yet active)
- `/etc/nginx/sites-enabled/` - Active configurations (symbolic links)
- This separation lets you prepare configs without activating them

---

**Add this configuration:**

```nginx
server {
    listen 80;
    server_name _;  # Accept any domain/IP for now

    # For Let's Encrypt certificate validation (Section 10)
    location /.well-known/acme-challenge/ {
        root /var/www/html;
    }

    # Proxy all other requests to Immich
    location / {
        proxy_pass http://localhost:2283;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support (required for Immich live updates)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Support large file uploads (photos/videos)
        client_max_body_size 50000M;

        # Increase timeouts for large uploads
        proxy_read_timeout 600s;
        proxy_send_timeout 600s;
        send_timeout 600s;
    }
}
```

**📝 Understanding this configuration:**

| Setting | What it does |
|---------|--------------|
| `listen 80` | Listen for HTTP requests on port 80 |
| `server_name _` | Accept any domain/IP (we'll change this in Section 8 if you have a domain) |
| `proxy_pass http://localhost:2283` | Forward requests to Immich on port 2283 |
| `proxy_set_header` lines | Pass client information to Immich |
| `Upgrade` + `Connection` | Enable WebSocket (for real-time updates) |
| `client_max_body_size 50000M` | Allow uploads up to 50GB (large videos) |
| `proxy_read_timeout 600s` | Wait up to 10 minutes for uploads to complete |

**💡 Why these settings matter:**
- **WebSocket** - Lets Immich push updates to your browser in real-time
- **Large uploads** - Videos can be several GB each
- **Long timeouts** - Slow connections need time to upload large files

---

**Save the file:**

1. Press `Ctrl + O` (write out)
2. Press `Enter` (confirm filename)
3. Press `Ctrl + X` (exit)

---

### Step 7.7: Enable the Immich Configuration

Activate the configuration by creating a symbolic link.

```bash
sudo ln -s /etc/nginx/sites-available/immich /etc/nginx/sites-enabled/
```

**What this does:** Creates a link from `sites-enabled` to `sites-available`, activating the configuration.

**✓ Expected output:** No output (silent success).

---

**Verify the link was created:**

```bash
ls -l /etc/nginx/sites-enabled/
```

**✓ Expected output:**

```
total 0
lrwxrwxrwx 1 root root 34 Feb  3 17:25 default -> /etc/nginx/sites-available/default
lrwxrwxrwx 1 root root 34 Feb  3 17:30 immich -> /etc/nginx/sites-available/immich
```

**📝 Note:** You should see both `default` and `immich`. We'll remove `default` next.

---

### Step 7.8: Remove Default Nginx Site

Remove the default Nginx welcome page.

```bash
sudo rm /etc/nginx/sites-enabled/default
```

**What this does:** Removes the default site so Nginx serves Immich instead.

**✓ Expected output:** No output (silent success).

---

**Verify only Immich config remains:**

```bash
ls -l /etc/nginx/sites-enabled/
```

**✓ Expected output:**

```
total 0
lrwxrwxrwx 1 root root 34 Feb  3 17:30 immich -> /etc/nginx/sites-available/immich
```

**✓ Success Criteria:**
* Only `immich` link exists
* `default` is gone

---

### Step 7.9: Test Nginx Configuration

Before reloading Nginx, test that the configuration syntax is correct.

```bash
sudo nginx -t
```

**What this does:** Tests all Nginx configuration files for syntax errors.

**✓ Expected output:**

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**📝 Key phrases:**
- "syntax is ok"
- "test is successful"

**✓ Success Criteria:**
* Both lines show success
* No error messages

**⚠ If you see errors:**

Common error messages:

**"unknown directive"**
- You have a typo in the configuration file
- Open it again: `sudo nano /etc/nginx/sites-available/immich`
- Check for missing semicolons (`;`) at end of lines
- Verify quotes are balanced (`"..."`)

**"conflicting server name"**
- You didn't remove the default site
- Run: `sudo rm /etc/nginx/sites-enabled/default`

**"could not open configuration file"**
- The symbolic link wasn't created
- Run: `sudo ln -s /etc/nginx/sites-available/immich /etc/nginx/sites-enabled/`

---

### Step 7.10: Reload Nginx Configuration

Apply the new configuration.

```bash
sudo systemctl reload nginx
```

**What this does:** Reloads Nginx configuration without dropping existing connections.

**📝 `reload` vs `restart`:**
- **reload** - Graceful reload (doesn't interrupt active uploads)
- **restart** - Full restart (drops all connections)
- Always use `reload` when just changing configuration

**✓ Expected output:** No output (silent success).

---

**Verify Nginx is still running:**

```bash
sudo systemctl status nginx
```

**✓ Expected output:**

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-02-03 17:12:34 UTC; 5min ago
...
```

**✓ Success Criteria:**
* Still shows "active (running)"
* No errors in the log output

Press `q` to exit.

---

### Step 7.11: Test Nginx Proxy to Immich

Now test that Nginx is successfully proxying requests to Immich.

**From another computer on your network**, open a web browser and go to:

```
http://192.168.1.50
```

(Replace with your server's IP - **note: no `:2283` port needed anymore!**)

**✓ Expected result:**

You should see the **Immich login page** (not the Nginx welcome page).

**🎉 Success!** You can now access Immich without typing the port number!

**📝 What changed:**
- **Before:** `http://192.168.1.50:2283`
- **After:** `http://192.168.1.50` (port 80 is default for HTTP)

**⚠ If you still see the Nginx welcome page:**
- You didn't remove the default site
- Run: `sudo rm /etc/nginx/sites-enabled/default`
- Then: `sudo systemctl reload nginx`

**⚠ If you see "502 Bad Gateway":**
- Immich isn't running
- Check: `docker compose ps` in `~/immich/`
- If containers are stopped: `docker compose up -d`

**⚠ If connection refused:**
- Nginx isn't running
- Start it: `sudo systemctl start nginx`

---

### Step 7.12: Verify WebSocket Functionality

WebSockets enable real-time updates in Immich. Let's verify they work through Nginx.

1. **Log in to Immich** at `http://192.168.1.50`
2. **Open your browser's developer console:**
   - Chrome/Edge: Press `F12` or `Ctrl+Shift+I`
   - Firefox: Press `F12` or `Ctrl+Shift+K`
   - Safari: Enable Developer menu, then press `Cmd+Option+I`
3. **Click the "Console" tab**
4. **Look for WebSocket messages**

**✓ Expected result:**

You should see messages like:
```
WebSocket connection established
Connected to server
```

Or you might just see no errors (which is good).

**⚠ If you see WebSocket errors:**
```
WebSocket connection failed
```

This means the WebSocket configuration isn't working. Check:
1. The `Upgrade` and `Connection` headers are in your Nginx config
2. Nginx configuration syntax: `sudo nginx -t`
3. Reload Nginx: `sudo systemctl reload nginx`

**💡 You can close the developer console** - just wanted to verify WebSockets work.

---

### Step 7.13: Test Large File Upload Support

Verify that large file uploads will work (important for videos).

**In Immich:**
1. Click the upload button (cloud with up arrow)
2. Try to select a large file (>100MB if you have one, or just a normal photo)

**✓ Expected result:**
* File upload starts (progress bar appears)
* No "413 Request Entity Too Large" error

**If you don't have a large file to test:**
- That's fine, the configuration supports it
- You'll test this naturally when uploading photos/videos later

---

### ✅ Checkpoint: Section 7 Complete

**You should now have:**

* ✓ Nginx installed and running
* ✓ Nginx configured as reverse proxy for Immich
* ✓ Firewall rule allowing HTTP (port 80)
* ✓ Immich accessible at `http://192.168.1.50` (no port needed)
* ✓ WebSocket support working (real-time updates)
* ✓ Large file upload support configured (up to 50GB)
* ✓ Nginx auto-starts on boot

**What you learned:**

* What a reverse proxy is and why it's useful
* How Nginx configuration files are structured
* The difference between `sites-available` and `sites-enabled`
* How to test Nginx configuration with `nginx -t`
* The difference between `reload` and `restart`
* How to configure WebSocket proxying
* How to allow large file uploads through a proxy

**Access methods comparison:**

| Method | URL | When to use |
|--------|-----|-------------|
| Direct to Immich | `http://192.168.1.50:2283` | Testing, troubleshooting |
| Through Nginx | `http://192.168.1.50` | Normal use (cleaner) |
| With domain (Section 8+) | `https://photos.yourdomain.com` | Remote access |

**Nginx configuration summary:**

| Setting | Value | Purpose |
|---------|-------|---------|
| Configuration file | `/etc/nginx/sites-available/immich` | Main config |
| Enabled link | `/etc/nginx/sites-enabled/immich` | Active config |
| Listen port | 80 (HTTP) | Standard web port |
| Proxy target | `localhost:2283` | Immich server |
| Max upload size | 50000M (50GB) | Large video support |
| Timeout | 600s (10 min) | Slow upload support |

**Useful Nginx commands:**

```bash
# Service management
sudo systemctl status nginx          # Check if running
sudo systemctl start nginx           # Start
sudo systemctl stop nginx            # Stop
sudo systemctl restart nginx         # Full restart
sudo systemctl reload nginx          # Reload config (preferred)

# Configuration
sudo nginx -t                        # Test configuration
sudo nano /etc/nginx/sites-available/immich  # Edit config

# Logs
sudo tail -f /var/log/nginx/access.log   # Access log (requests)
sudo tail -f /var/log/nginx/error.log    # Error log (problems)

# Troubleshooting
sudo systemctl status nginx          # Check status
sudo nginx -t                        # Test config syntax
docker compose ps                    # Check Immich is running
```

**Next steps:**

Now that Nginx is set up, you're ready for remote access. The next sections will:

- **Section 8:** Purchase a domain name (required for HTTPS and clean remote access)
- **Section 9:** Configure DNS and port forwarding (make Immich accessible from anywhere)
- **Section 10:** Set up HTTPS with Let's Encrypt (secure your connection)

**💡 Important:** Sections 8-10 work together. If you want remote access, you'll need to complete all three. If you only want local access, you can stop here (but we recommend continuing).

**🔖 Quick test:** Try accessing Immich at `http://192.168.1.50` instead of `http://192.168.1.50:2283` - it should work!
sudo systemctl reload nginx
```

### Update Firewall

```bash
# Allow HTTP
sudo ufw allow 80/tcp

# Check status
sudo ufw status
```

---

## 8. Purchase a Domain Name (Required for Remote Access)

**📋 What You'll Accomplish:**
* Understand what domain names are and why you need one
* Choose a domain registrar (we recommend Cloudflare)
* Search for and purchase an available domain name
* Configure basic DNS settings
* Prepare for remote access setup

**⏱ Estimated Time:** 15-20 minutes

**💰 Cost:** $10-15/year (varies by domain)

**📦 Prerequisites:**
* ✓ Section 7 complete (Nginx configured)
* ✓ Credit/debit card for domain purchase
* ✓ Decision on whether you want remote access

**💡 Do you need a domain?**

**Yes, if you want:**
- ✅ Access your photos from anywhere (not just home WiFi)
- ✅ HTTPS/SSL encryption (required for modern mobile apps)
- ✅ A memorable URL like `photos.yourdomain.com` instead of `192.168.1.50:2283`
- ✅ Mobile app auto-backup over cellular data

**You can skip if:**
- ❌ You only access Immich from home WiFi
- ❌ You're comfortable using local IP addresses
- ❌ You don't need HTTPS

**⚠ Note:** Sections 8, 9, and 10 work together. If you want remote access, you need all three.

---

### What is a Domain Name?

**Without a domain:**
```
You type: http://203.0.113.45:2283
```
- Hard to remember
- Changes if your ISP assigns a new IP
- Can't use HTTPS easily

**With a domain:**
```
You type: https://photos.yourdomain.com
```
- Easy to remember
- Never changes (even if IP changes)
- Works with HTTPS/SSL certificates
- Professional appearance

**💡 Think of it like a phone contact:**
- **IP address** = Phone number (hard to remember, can change)
- **Domain name** = Contact name (easy to remember, stays the same)

---

### Step 8.1: Understand Domain Naming

Before buying, understand what makes a good domain name for Immich.

**Domain structure:**
```
photos.example.com
  ↑       ↑      ↑
  |       |      └─ TLD (Top-Level Domain)
  |       └─ Your domain name
  └─ Subdomain (optional)
```

**Examples:**
- `myphotos.com` - Simple, direct
- `johndoe-photos.com` - Personal name
- `familyarchive.net` - Descriptive
- `photos.smithfamily.org` - Subdomain approach

**💡 Naming tips:**

**Good practices:**
- ✅ Short and memorable
- ✅ Easy to spell
- ✅ No special characters or numbers (unless meaningful)
- ✅ Use `.com`, `.net`, or `.org` (widely recognized)

**Avoid:**
- ❌ Very long names (hard to type on mobile)
- ❌ Confusing spellings
- ❌ Trademarked names
- ❌ Personal information you want private

**⚠ Privacy consideration:**

Domain ownership is **public information**. Your name, email, and address can be looked up unless you use privacy protection.

**Solution:** Most registrars (including Cloudflare) include **free privacy protection** (also called "WHOIS privacy") that hides your personal information.

**✍️ Brainstorm 3-5 domain ideas:**

1. `_________________________`
2. `_________________________`
3. `_________________________`
4. `_________________________`
5. `_________________________`

---

### Step 8.2: Why We Recommend Cloudflare

There are many domain registrars (GoDaddy, Namecheap, Google Domains, etc.). We recommend **Cloudflare** for beginners because:

**Advantages:**
- ✅ **At-cost pricing** ($10-15/year, no markup)
- ✅ **Free WHOIS privacy** (protects your personal info)
- ✅ **Built-in DNS management** (no separate service needed)
- ✅ **Free DDoS protection** (protects your server from attacks)
- ✅ **Simple interface** (beginner-friendly)
- ✅ **No upsells** (other registrars push expensive add-ons)

**Disadvantages:**
- ⚠️ Fewer TLDs available (no exotic extensions like `.photography`)
- ⚠️ DNS-only (can't use Cloudflare's proxy feature with home servers)

**Alternatives if Cloudflare doesn't have your desired TLD:**
- **Namecheap** - Wide TLD selection, affordable
- **Porkbun** - Good pricing, free WHOIS privacy
- **Google Domains** (now Squarespace) - Simple, but more expensive

**💡 You can use any registrar.** The DNS setup in Section 9 works the same way.

---

### Step 8.3: Create Cloudflare Account

Go to Cloudflare and create an account.

1. **Visit:** [https://www.cloudflare.com/](https://www.cloudflare.com/)
2. **Click "Sign Up"** (top right)
3. **Enter your email address** and create a strong password
4. **Verify your email** (check your inbox for confirmation link)

**✓ Expected result:**

After verification, you'll see the Cloudflare dashboard.

**✍️ Write down your Cloudflare credentials:**

| Credential | Value |
|------------|-------|
| Email | `_________________________` |
| Password | `_________________________` |

---

### Step 8.4: Navigate to Domain Registration

Find the domain registration section.

1. **Log in to Cloudflare** if not already logged in
2. **Click "Domain Registration"** in the left sidebar

**📸 What you'll see:**
- A search box with "Register a new domain"
- List of popular TLDs (.com, .net, .org)
- Pricing information

**⚠ If you don't see "Domain Registration":**

- You might need to verify your email first
- Or Cloudflare might have changed the interface - look for "Domains" or "Register"

---

### Step 8.5: Search for Available Domains

Search for your desired domain name.

1. **Enter your domain idea** in the search box (without the TLD)
   - Example: Type `johnsmith-photos` not `johnsmith-photos.com`

2. **Click "Search"** or press Enter

**✓ Expected result:**

Cloudflare shows available domains with prices:

```
johnsmith-photos.com    $13.20/year    ✅ Available
johnsmith-photos.net    $12.85/year    ✅ Available
johnsmith-photos.org    $13.50/year    ✅ Available
johnsmith-photos.co     $31.00/year    ✅ Available
```

**If your domain is taken:**

```
johnsmith.com          ❌ Unavailable
```

**💡 Tips for finding available domains:**

- Add descriptive words: `photos`, `pics`, `archive`, `memories`
- Try different TLDs: `.net`, `.org`, `.cloud`
- Add year: `smithphotos2025.com`
- Use initials: `jsphotos.com`
- Be creative but memorable

**Common available patterns:**
- `firstname-photos.com`
- `familyname-pics.net`
- `yourname-immich.com`

---

### Step 8.6: Select and Purchase Domain

Choose your domain and complete the purchase.

1. **Click the "Purchase" button** next to your chosen domain

2. **Review the order summary:**
   - Domain name: `yourname-photos.com`
   - Price: ~$10-15/year
   - Privacy protection: Included (free)
   - Auto-renew: Enabled (recommended)

3. **Enter payment information:**
   - Credit/debit card
   - Billing address

4. **Review Terms of Service** (domain registration is a contract)

5. **Click "Complete Purchase"** or "Purchase Domain"

**✓ Expected result:**

You'll see a confirmation message:
- "Domain registered successfully!"
- "Your domain: yourname-photos.com"
- Email confirmation sent

**✍️ Document your domain:**

| Information | Value |
|-------------|-------|
| Domain name | `_________________________` |
| Registrar | Cloudflare |
| Registration date | `_________________________` |
| Renewal date | `_________________________` |
| Annual cost | `_________________________` |

**⚠ Important reminders:**

- **Auto-renew:** Make sure it's enabled so your domain doesn't expire
- **Email notifications:** Watch for renewal reminders
- **Payment method:** Keep your card info up to date

---

### Step 8.7: Verify Domain Ownership

Confirm the domain is in your account.

1. **Go to Cloudflare dashboard**
2. **Click "Websites"** in the left sidebar
3. **You should see your domain listed**

**✓ Expected result:**

Your domain appears with:
- Domain name: `yourname-photos.com`
- Status: Active
- Plan: Free (Cloudflare offers a free DNS plan)

**If you don't see your domain:**
- Wait a few minutes (registration can take 5-10 minutes)
- Check your email for confirmation
- Refresh the page

---

### Step 8.8: Access DNS Settings

Navigate to the DNS management page.

1. **Click on your domain** in the Cloudflare dashboard
2. **Click "DNS"** in the top menu (or left sidebar)
3. **You should see "DNS Records" section**

**✓ Expected result:**

You see a DNS management page with:
- "Add record" button
- Empty list (or a few default records)
- Cloudflare nameservers listed

**📝 What you'll see:**

```
DNS Records for yourname-photos.com

Cloudflare nameservers:
- cash.ns.cloudflare.com
- deb.ns.cloudflare.com

Type    Name    Content         Proxy status    TTL
(empty list for now)
```

**💡 What are nameservers?**

Nameservers are like the "phonebook" for your domain. They tell the internet where to look for your domain's DNS records.

When you buy a domain from Cloudflare, it automatically uses Cloudflare's nameservers (which is what we want).

---

### Step 8.9: Verify Nameservers (Important)

Confirm your domain is using Cloudflare nameservers.

**On the DNS page**, look for a section showing nameservers.

**✓ Expected result:**

```
Nameservers
✅ Cloudflare nameservers active

cash.ns.cloudflare.com
deb.ns.cloudflare.com
```

(The exact nameserver names may vary, but they should end with `cloudflare.com`)

**✓ Success Criteria:**
- Shows "Cloudflare nameservers active" or similar
- Lists 2+ nameservers ending in `cloudflare.com`

**⚠ If you bought from Cloudflare:**

This should be automatic. If you see "Pending" or "Update nameservers," wait 10-15 minutes and refresh.

**⚠ If you bought from a different registrar:**

You'll need to change nameservers at your registrar to point to Cloudflare's nameservers. This is beyond the scope of this guide, but Cloudflare provides instructions when you add the domain.

---

### Step 8.10: Understand DNS Propagation

DNS changes take time to spread across the internet.

**What is DNS propagation?**

When you make DNS changes, they need to spread to DNS servers worldwide. This takes time.

**Typical timeline:**
- **Immediate:** Cloudflare's servers (you can test right away)
- **5-30 minutes:** Most of the internet
- **24-48 hours:** Complete worldwide propagation (worst case)

**💡 What this means for you:**

After adding DNS records in Section 9:
- Your server might work immediately for you
- It might take an hour for your phone on cellular
- It might take up to 24 hours for some locations

**This is normal.** Be patient.

---

### Step 8.11: Optional: Set Up Email Forwarding

Cloudflare offers free email forwarding for your domain.

**What is email forwarding?**

Receive email at `admin@yourname-photos.com` and have it forwarded to your personal email.

**Benefits:**
- Professional appearance
- Separate contact for Immich
- Can change where it forwards without updating everywhere

**To set up (optional):**

1. **Click "Email"** in Cloudflare dashboard (for your domain)
2. **Click "Email Routing"**
3. **Add destination address** (your personal email)
4. **Add routing rule:**
   - `admin@yourname-photos.com` → `yourpersonal@email.com`
5. **Verify** your personal email (Cloudflare sends a confirmation)

**💡 This is completely optional.** You can use your regular email for Immich.

---

### ✅ Checkpoint: Section 8 Complete

**You should now have:**

* ✓ Understanding of domain names and why they're needed
* ✓ Cloudflare account created
* ✓ Domain name purchased (~$10-15/year)
* ✓ Domain showing in your Cloudflare dashboard
* ✓ Cloudflare nameservers active
* ✓ Access to DNS management page
* ✓ WHOIS privacy protection enabled (automatic)

**What you learned:**

* What domain names are (friendly names for IP addresses)
* How domain structure works (subdomain.domain.tld)
* Why Cloudflare is good for beginners
* The domain registration process
* What nameservers do
* What DNS propagation means (and why changes take time)
* How to access DNS management

**Domain registration summary:**

| Item | Status | Details |
|------|--------|---------|
| Domain name | ✅ Registered | yourname-photos.com |
| Registrar | ✅ Cloudflare | Account created |
| Nameservers | ✅ Active | Cloudflare DNS |
| Privacy protection | ✅ Enabled | WHOIS privacy included |
| DNS access | ✅ Ready | Can add records |
| Cost | 💰 $10-15/year | Auto-renew recommended |

**Important reminders:**

* **Auto-renew:** Keep it enabled to prevent domain expiration
* **Email notifications:** Watch for renewal reminders
* **Payment method:** Keep card info current
* **Don't let it expire:** If your domain expires, someone else can buy it

**Useful Cloudflare features (for future):**

```
Available in dashboard:
- DNS Records (Section 9)
- SSL/TLS settings (Section 10)
- Email Routing (optional)
- Analytics (traffic stats)
- Page Rules (advanced)
```

**Next steps:**

In Section 9, you'll:
1. Find your home's public IP address
2. Create a DNS A record pointing your domain to your IP
3. Configure port forwarding on your router
4. Test that your domain reaches your server

This connects your domain name to your home server, making it accessible from anywhere on the internet.

**🔖 Keep this info handy:** Your domain name and Cloudflare login - you'll need them in the next sections.

**💡 Domain is registered but not working yet.** That's normal! Section 9 connects it to your server.

---

## 9. Set Up DNS and Port Forwarding

**📋 What You'll Accomplish:**
* Find your home's public IP address
* Create a DNS record pointing your domain to your IP
* Configure your router to forward traffic to your server
* Update firewall rules for HTTPS
* Test that your domain works from outside your network
* Understand dynamic IP limitations and solutions

**⏱ Estimated Time:** 20-30 minutes

**📦 Prerequisites:**
* ✓ Section 8 complete (Domain purchased and active)
* ✓ Section 7 complete (Nginx configured)
* ✓ Access to your router's admin page
* ✓ Your server's static IP (from Section 2)

**💡 What this section does:**

This section connects three pieces:
1. **Your domain** (`photos.yourdomain.com`)
2. **Your home's public IP** (how the internet finds your house)
3. **Your server** (inside your house on local network)

**The flow after this section:**
```
Internet → Domain → Public IP → Router → Port Forward → Server → Nginx → Immich
```

---

### Understanding Public vs Private IPs

Before we start, understand the two types of IP addresses:

**Private IP** (local network only):
- Example: `192.168.1.50`
- What you set in Section 2
- Only works on your home WiFi
- Can't be reached from the internet

**Public IP** (internet-facing):
- Example: `203.0.113.45`
- Assigned by your ISP to your router
- How the internet finds your home
- Shared by all devices in your house

**💡 Analogy:**
- **Public IP** = Your home's street address
- **Private IP** = Room number inside your house
- **Port forwarding** = Directing visitors to the right room

---

### Step 9.1: Find Your Public IP Address

First, discover what public IP address your ISP assigned to your home.

**From your server (or any computer on your home network):**

```bash
curl ifconfig.me
```

**What this does:** Asks an external service "what IP address am I coming from?"

**✓ Expected output:**

```
203.0.113.45
```

A single IP address (4 numbers separated by dots).

**✍️ Write down your public IP:** `_____________`

---

**Alternative methods to find your public IP:**

**Method 2: Using a web browser**

Visit any of these sites:
- https://whatismyipaddress.com/
- https://www.whatismyip.com/
- https://ipinfo.io/

**Method 3: Using Google**

Just search "what is my ip" in Google - it shows your public IP at the top.

---

**📝 Understanding dynamic vs static public IPs:**

**Dynamic IP (most home internet):**
- ✅ Free (included with internet service)
- ⚠️ Can change when you restart your router
- ⚠️ Can change periodically (weekly/monthly)
- 💡 Solution: Dynamic DNS (covered later)

**Static IP (business internet):**
- ✅ Never changes
- ⚠️ Costs extra ($10-50/month)
- 💡 Usually not worth it for home use

**For now:** We'll use your current public IP. If it changes later, you'll just update the DNS record (takes 5 minutes).

---

### Step 9.2: Create DNS A Record in Cloudflare

Now point your domain to your public IP.

**Log in to Cloudflare:**

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com/)
2. Log in with your credentials
3. **Click on your domain** (e.g., `yourname-photos.com`)

---

**Navigate to DNS settings:**

1. **Click "DNS"** in the top menu
2. You should see "DNS Records" section

**✓ Expected result:**

You see a page with:
- "Add record" button
- Table of DNS records (might be empty or have a few defaults)
- Cloudflare nameservers listed below

---

**Add a new A record:**

1. **Click "Add record"**

2. **Fill in the form:**

| Field | Value | Explanation |
|-------|-------|-------------|
| **Type** | `A` | Maps domain to IPv4 address |
| **Name** | `@` or `photos` | See below for choosing |
| **IPv4 address** | Your public IP | From Step 9.1 |
| **Proxy status** | 🔘 DNS only (gray cloud) | **Critical - must be gray!** |
| **TTL** | Auto | How long to cache (automatic is fine) |

**🤔 Choosing the Name field:**

**Option 1: Use `@` (root domain)**
- Creates: `yourname-photos.com`
- Pro: Shorter to type
- Con: Less flexible for other services

**Option 2: Use `photos` (subdomain)**
- Creates: `photos.yourname-photos.com`
- Pro: Can add other services (`blog.yourname-photos.com`, etc.)
- Con: Slightly longer

**💡 We recommend:** Use `photos` (subdomain approach)

---

**⚠ CRITICAL: Disable Cloudflare Proxy (Gray Cloud)**

Look for the **"Proxy status"** toggle:

```
🟠 Proxied (orange cloud)    ← DO NOT USE
🔘 DNS only (gray cloud)     ← USE THIS
```

**Click the cloud icon** until it's **gray** (DNS only).

**Why disable proxy?**
- Cloudflare proxy doesn't work well with home servers
- We're using Let's Encrypt for SSL (Section 10)
- Proxied mode requires Cloudflare SSL settings we won't use

---

**Example configuration:**

```
Type:     A
Name:     photos
IPv4:     203.0.113.45  (your actual public IP)
Proxy:    🔘 DNS only (gray cloud)
TTL:      Auto
```

3. **Click "Save"**

**✓ Expected result:**

The record appears in the DNS records table:

```
Type    Name      Content         Proxy status    TTL
A       photos    203.0.113.45    DNS only        Auto
```

**✓ Success Criteria:**
* Record shows in table
* Proxy status shows gray cloud "DNS only"
* Content shows your public IP

---

### Step 9.3: Verify DNS Record Was Created

Confirm the record is saved correctly.

**In Cloudflare dashboard:**

Look at the DNS records table and verify you see:

```
A    photos    203.0.113.45    DNS only    Auto
```

(With your actual public IP)

---

**Optional: Test DNS immediately**

While DNS propagation takes time globally, Cloudflare's servers are updated immediately.

```bash
nslookup photos.yourname-photos.com 1.1.1.1
```

**What this does:** Queries Cloudflare's DNS directly (1.1.1.1 is Cloudflare's DNS server).

**✓ Expected output:**

```
Server:  1.1.1.1
Address: 1.1.1.1#53

Non-authoritative answer:
Name:    photos.yourname-photos.com
Address: 203.0.113.45
```

**📝 Key indicator:** "Address" shows your public IP.

**⚠ If you see "can't find":**
- Wait 2-3 minutes and try again (record just created)
- Check spelling of domain
- Make sure you saved the record in Cloudflare

---

### Step 9.4: Update Nginx Server Name

Update your Nginx configuration to respond to your domain name.

**Open Nginx config:**

```bash
sudo nano /etc/nginx/sites-available/immich
```

**Find the line:**

```nginx
server_name _;
```

**Change it to your domain:**

```nginx
server_name photos.yourname-photos.com;
```

(Use your actual domain)

**💡 If you used `@` instead of `photos`:**

```nginx
server_name yourname-photos.com;
```

---

**Save the file:**

1. Press `Ctrl + O` (write out)
2. Press `Enter`
3. Press `Ctrl + X` (exit)

---

**Test and reload Nginx:**

```bash
sudo nginx -t
```

**✓ Expected output:**

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

```bash
sudo systemctl reload nginx
```

**✓ Expected output:** No output (silent success).

---

### Step 9.5: Configure Router Port Forwarding

Now configure your router to forward internet traffic to your server.

**⚠ This is the most hardware-specific step.** Every router brand has different menus. We'll provide general guidance.

---

**Find your router's admin page:**

**Common router IPs:**
- `192.168.1.1` (most common)
- `192.168.0.1`
- `192.168.2.1`
- `10.0.0.1`

**How to find it:**

```bash
ip route | grep default
```

**✓ Expected output:**

```
default via 192.168.1.1 dev enp0s3 proto dhcp
```

The IP after "via" is your router.

---

**Log in to router:**

1. **Open a web browser**
2. **Go to:** `http://192.168.1.1` (your router's IP)
3. **Enter username and password**

**Don't know the password?**
- Check the label on your router (often on the bottom)
- Common defaults: `admin`/`admin`, `admin`/`password`, `admin`/`(blank)`
- If you changed it and forgot: you may need to factory reset the router

---

**Find port forwarding settings:**

Router menus vary, but look for sections named:
- "Port Forwarding"
- "NAT Forwarding"
- "Virtual Servers"
- "Applications & Gaming"
- "Firewall" → "Port Forwarding"
- "Advanced" → "Port Forwarding"

**💡 Can't find it?**

Search online: `[your router model] port forwarding`

Example: "netgear nighthawk port forwarding"

---

**Create port forwarding rules:**

You need to create **two** rules (one for HTTP, one for HTTPS).

**Common router interface examples:**

**Example 1: Simple form**

```
Service Name:    Immich-HTTP
External Port:   80
Internal IP:     192.168.1.50
Internal Port:   80
Protocol:        TCP
```

**Example 2: Range-based**

```
Application:     Immich-HTTP
Start Port:      80
End Port:        80
Server IP:       192.168.1.50
Protocol:        TCP
```

**Example 3: Separate fields**

```
Name:            Immich-HTTP
Protocol:        TCP
External Port:   80
Internal IP:     192.168.1.50
Internal Port:   80
```

---

**Rule 1: HTTP (Port 80)**

Fill in these values (adapt to your router's interface):

| Setting | Value |
|---------|-------|
| **Service/Name** | `Immich-HTTP` (or any name) |
| **External Port** | `80` |
| **Internal IP** | `192.168.1.50` (your server's static IP) |
| **Internal Port** | `80` |
| **Protocol** | `TCP` (not UDP, not Both) |

**Why port 80?**
- Standard HTTP port
- Used by Let's Encrypt for certificate validation (Section 10)

---

**Rule 2: HTTPS (Port 443)**

| Setting | Value |
|---------|-------|
| **Service/Name** | `Immich-HTTPS` (or any name) |
| **External Port** | `443` |
| **Internal IP** | `192.168.1.50` (your server's static IP) |
| **Internal Port** | `443` |
| **Protocol** | `TCP` |

**Why port 443?**
- Standard HTTPS port
- Encrypted traffic for remote access

---

**Save and apply:**

- Look for "Save", "Apply", or "OK" button
- Some routers require a reboot (will tell you)
- Wait 30 seconds for changes to take effect

---

**Verify port forwarding rules:**

In your router, you should see both rules listed:

```
Name           External Port    Internal IP      Internal Port    Protocol
Immich-HTTP    80              192.168.1.50     80               TCP
Immich-HTTPS   443             192.168.1.50     443              TCP
```

**✓ Success Criteria:**
* Two rules created
* Both use TCP protocol
* Both point to your server's IP (192.168.1.50)
* Changes are saved/applied

---

**📝 Common router brands - where to find port forwarding:**

| Brand | Menu Location |
|-------|---------------|
| **TP-Link** | Advanced → NAT Forwarding → Virtual Servers |
| **Netgear** | Advanced → Advanced Setup → Port Forwarding |
| **Linksys** | Security → Apps & Gaming → Single Port Forwarding |
| **Asus** | WAN → Virtual Server / Port Forwarding |
| **D-Link** | Advanced → Port Forwarding |
| **Google WiFi** | Google Home app → WiFi → Settings → Advanced → Port management |
| **Ubiquiti** | Settings → Routing & Firewall → Port Forwarding |

---

### Step 9.6: Update Firewall for HTTPS

Allow HTTPS traffic through your server's firewall.

**Add firewall rule:**

```bash
sudo ufw allow 443/tcp
```

**✓ Expected output:**

```
Rule added
Rule added (v6)
```

---

**Verify all required ports are open:**

```bash
sudo ufw status
```

**✓ Expected output:**

```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
2283/tcp                   ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)
2283/tcp (v6)              ALLOW       Anywhere (v6)
```

**✓ Success Criteria:**
* You see rules for 22, 80, 443, and 2283
* All show "ALLOW Anywhere"

---

### Step 9.7: Test DNS Resolution

Wait for DNS to propagate, then test that your domain resolves to your public IP.

**⏱ Wait 5-10 minutes** after creating the DNS record.

**Test DNS lookup:**

```bash
nslookup photos.yourname-photos.com
```

(Use your actual domain)

**✓ Expected output:**

```
Server:  8.8.8.8
Address: 8.8.8.8#53

Non-authoritative answer:
Name:    photos.yourname-photos.com
Address: 203.0.113.45
```

**📝 Key indicator:** "Address" shows your public IP (from Step 9.1).

**✓ Success Criteria:**
* DNS query succeeds
* Returns your public IP
* No "can't find" error

**⚠ If DNS doesn't resolve:**

**"server can't find photos.yourname-photos.com"**
- DNS hasn't propagated yet - wait longer (up to 24 hours in rare cases)
- Typo in domain name - check spelling
- DNS record not created - verify in Cloudflare dashboard

**Returns wrong IP:**
- Old cached data - try `nslookup photos.yourname-photos.com 1.1.1.1`
- Wrong IP in DNS record - check Cloudflare dashboard

---

### Step 9.8: Test External Access (The Moment of Truth!)

Now test that you can access Immich from outside your network.

**⚠ Important:** You must test from **outside** your home network.

**Option 1: Use your phone on cellular data**

1. **Turn off WiFi** on your phone (use cellular/mobile data)
2. **Open a web browser**
3. **Go to:** `http://photos.yourname-photos.com`

**Option 2: Use a mobile hotspot**

1. **Enable mobile hotspot** on your phone
2. **Connect your computer** to the hotspot
3. **Open a browser** and go to your domain

**Option 3: Ask a friend**

Have someone outside your home try accessing your domain.

---

**✓ Expected result:**

You should see:
- **Immich login page**
- URL shows `http://photos.yourname-photos.com`
- You can log in with your credentials

**🎉 If you see Immich - SUCCESS!** Your domain is working and port forwarding is configured correctly!

---

**⚠ Common issues:**

**"This site can't be reached" or "Connection timed out"**

Possible causes:
1. **Port forwarding not configured:**
   - Double-check router rules (ports 80 and 443)
   - Verify Internal IP matches your server (192.168.1.50)
   - Try rebooting router

2. **ISP blocks port 80:**
   - Some ISPs block incoming port 80 for residential customers
   - Solution: Use only port 443 (HTTPS) - covered in Section 10
   - Or: Call ISP and ask them to unblock port 80

3. **Router firewall:**
   - Some routers have separate firewall rules
   - Check router for "SPI Firewall" or "Firewall" settings
   - May need to allow incoming connections

4. **Testing from home WiFi:**
   - Many routers don't support "NAT loopback" (accessing from inside)
   - Must test from cellular or external network

---

**"502 Bad Gateway"**

This means:
- ✅ Domain DNS is working
- ✅ Port forwarding is working
- ❌ Nginx or Immich isn't running

**Fix:**
```bash
# Check Nginx
sudo systemctl status nginx

# Check Immich
cd ~/immich
docker compose ps
```

---

**"Connection refused"**

This means:
- ✅ Domain DNS is working
- ❌ Port forwarding might not be working
- ❌ Or Nginx isn't listening on port 80

**Fix:**
```bash
# Verify Nginx is listening on port 80
sudo netstat -tlnp | grep :80

# Should show:
# tcp  0  0 0.0.0.0:80  0.0.0.0:*  LISTEN  12345/nginx
```

---

### Step 9.9: Test From Multiple Networks

For confidence, test from several sources:

**Test checklist:**

| Location | URL | Result |
|----------|-----|--------|
| Home WiFi (local) | `http://192.168.1.50` | ☐ Works |
| Home WiFi (domain) | `http://photos.yourname-photos.com` | ☐ Works or NAT loopback issue |
| Cellular data | `http://photos.yourname-photos.com` | ☐ Works |
| Friend's internet | `http://photos.yourname-photos.com` | ☐ Works |

**💡 If domain doesn't work from home WiFi:**

This is called "NAT loopback" or "hairpin NAT" - some routers don't support it.

**Not a problem!** It works from outside, which is what matters.

**Workaround:** Use local IP at home, domain when away.

---

### Step 9.10: Document Your Configuration

Record the details for future reference.

**✍️ Network configuration:**

| Setting | Value |
|---------|-------|
| Public IP | `_________________________` |
| Domain name | `_________________________` |
| DNS record | `A photos → [public IP]` |
| Port forwarding | `80, 443 → 192.168.1.50` |
| Nginx server_name | `_________________________` |

**💡 Save this information** - you'll need your public IP if you need to update DNS later.

---

### ✅ Checkpoint: Section 9 Complete

**You should now have:**

* ✓ Public IP address identified
* ✓ DNS A record created in Cloudflare
* ✓ Cloudflare proxy disabled (gray cloud)
* ✓ Nginx configured with domain name
* ✓ Router port forwarding configured (ports 80 and 443)
* ✓ Firewall updated for HTTPS (port 443)
* ✓ DNS resolves to your public IP
* ✓ Immich accessible via domain from external network
* ✓ Understanding of NAT loopback limitations

**What you learned:**

* Difference between public and private IPs
* How DNS A records work (domain → IP mapping)
* What port forwarding does (router → server routing)
* Why Cloudflare proxy needs to be disabled for home servers
* How to test connectivity from outside your network
* What NAT loopback is and why it might not work
* How to troubleshoot connection issues systematically

**Access methods summary:**

| Method | When to use | URL |
|--------|-------------|-----|
| Local IP | At home, always works | `http://192.168.1.50` |
| Domain (internal) | At home, if NAT loopback works | `http://photos.yourname-photos.com` |
| Domain (external) | Away from home | `http://photos.yourname-photos.com` |
| With HTTPS (Section 10) | Everywhere (recommended) | `https://photos.yourname-photos.com` |

**Port forwarding configuration:**

| External Port | Internal IP | Internal Port | Purpose |
|---------------|-------------|---------------|---------|
| 80 | 192.168.1.50 | 80 | HTTP, Let's Encrypt validation |
| 443 | 192.168.1.50 | 443 | HTTPS (secure access) |

**Troubleshooting quick reference:**

| Problem | Likely Cause | Solution |
|---------|--------------|----------|
| DNS doesn't resolve | Propagation delay | Wait up to 24 hours |
| Can't reach domain | Port forwarding issue | Check router config |
| 502 Bad Gateway | Immich/Nginx not running | Restart services |
| Works externally, not at home | No NAT loopback | Use local IP at home |

**Next steps:**

In Section 10, you'll set up HTTPS with Let's Encrypt. This will:
- Encrypt all traffic (security and privacy)
- Enable the green padlock in browsers
- Make mobile apps work better
- Give you a professional `https://` URL

**⚠ Important:** Right now your connection is **not encrypted** (HTTP). Don't upload sensitive photos until you complete Section 10 (HTTPS).

**🔖 Dynamic IP reminder:** If your public IP changes in the future, just update the DNS A record in Cloudflare (takes 5 minutes). You can check your current IP anytime with `curl ifconfig.me`.

---

## 10. Set Up HTTPS with Let's Encrypt

**📋 What You'll Accomplish:**
* Install Certbot (Let's Encrypt client)
* Obtain a free SSL/TLS certificate
* Configure Nginx for HTTPS encryption
* Set up automatic certificate renewal
* Add security headers for enhanced protection
* Verify HTTPS is working correctly

**⏱ Estimated Time:** 10-15 minutes

**💰 Cost:** Free (Let's Encrypt is free forever)

**📦 Prerequisites:**
* ✓ Section 9 complete (Domain working, port 80/443 forwarded)
* ✓ Domain accessible from internet via HTTP
* ✓ Nginx configured with your domain name

**💡 Why HTTPS is critical:**

**Without HTTPS (HTTP):**
- ❌ All traffic is **unencrypted** (anyone can intercept)
- ❌ Passwords sent in plain text
- ❌ Photos visible to anyone monitoring your network
- ❌ Modern browsers show "Not Secure" warning
- ❌ Mobile apps may refuse to connect

**With HTTPS:**
- ✅ All traffic is **encrypted**
- ✅ Passwords and photos are protected
- ✅ Green padlock in browsers
- ✅ Mobile apps work properly
- ✅ Required for modern web standards

**💡 HTTPS is not optional.** This section is essential for security.

---

### Understanding Let's Encrypt and Certbot

**What is Let's Encrypt?**
- Free, automated certificate authority (CA)
- Provides SSL/TLS certificates at no cost
- Trusted by all modern browsers
- Certificates valid for 90 days (auto-renewable)

**What is Certbot?**
- Official Let's Encrypt client software
- Automates certificate obtaining and renewal
- Integrates with Nginx automatically
- Free and open source

**How it works:**
1. Certbot requests certificate from Let's Encrypt
2. Let's Encrypt verifies you own the domain (via port 80)
3. Certificate is issued and installed
4. Nginx is automatically reconfigured for HTTPS
5. Certbot sets up auto-renewal (runs twice daily)

---

### Step 10.1: Verify Prerequisites

Before starting, confirm everything is ready.

**Test that port 80 is accessible from internet:**

From your server:

```bash
curl -I http://photos.yourdomain.com
```

(Use your actual domain)

**✓ Expected output:**

```
HTTP/1.1 200 OK
Server: nginx/1.24.0
...
```

**📝 Key indicator:** You get a response (doesn't have to be 200 OK, just any HTTP response).

**⚠ If connection fails:**
- Port 80 isn't forwarded correctly
- Go back to Section 9 and verify port forwarding
- Let's Encrypt needs port 80 to verify domain ownership

---

**Verify Nginx server_name is set:**

```bash
grep server_name /etc/nginx/sites-available/immich
```

**✓ Expected output:**

```
    server_name photos.yourdomain.com;
```

**✓ Success Criteria:**
* Shows your actual domain (not `_`)

**⚠ If it shows `server_name _;`:**
- Go back to Section 9, Step 9.4
- Update the server_name to your domain

---

### Step 10.2: Install Certbot

Install the Certbot software and Nginx plugin.

```bash
sudo apt update
sudo apt install -y certbot python3-certbot-nginx
```

**What this does:** Installs Certbot and the plugin that integrates with Nginx.

**✓ Expected output:**

```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  python3-acme python3-certbot python3-josepy python3-parsedatetime
  python3-requests-toolbelt python3-zope.component python3-zope.hookable
  ...
The following NEW packages will be installed:
  certbot python3-acme python3-certbot python3-certbot-nginx ...
0 upgraded, 15 newly installed, 0 to remove and 0 not upgraded.
Need to get 987 kB of archives.
After this operation, 4,567 kB of additional disk space will be used.
...
Setting up certbot (2.7.4-1) ...
Created symlink /etc/systemd/system/timers.target.wants/certbot.timer
```

**📝 Key indicator:** "Created symlink" for `certbot.timer` - this is the auto-renewal service.

**⏱ This takes 1-2 minutes.**

**✓ Success Criteria:**
* Certbot and plugins installed
* No error messages
* Auto-renewal timer created

---

**Verify Certbot installation:**

```bash
certbot --version
```

**✓ Expected output:**

```
certbot 2.7.4
```

(Version number may differ - as long as it's 1.0+ you're good)

---

### Step 10.3: Obtain SSL Certificate

Now request a certificate from Let's Encrypt.

**⚠ Before running this command:**

1. **Make sure your domain resolves to your public IP** (test with `nslookup`)
2. **Port 80 must be open and forwarded** to your server
3. **Nginx must be running** (`sudo systemctl status nginx`)

---

**Run Certbot:**

```bash
sudo certbot --nginx -d photos.yourdomain.com
```

(Use your actual domain)

**What this does:**
- Requests certificate for your domain
- Verifies you control the domain (via HTTP challenge on port 80)
- Automatically configures Nginx for HTTPS
- Sets up auto-renewal

**📝 Understanding the options:**
- `--nginx` - Use Nginx plugin (auto-configures Nginx)
- `-d photos.yourdomain.com` - Domain to get certificate for

---

**Follow the interactive prompts:**

**Prompt 1: Email address**

```
Enter email address (used for urgent renewal and security notices)
(Enter 'c' to cancel): your@email.com
```

**Enter your email address** (you'll receive renewal reminders if auto-renewal fails).

**✍️ Use a valid email you check regularly.**

---

**Prompt 2: Terms of Service**

```
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.3-September-21-2022.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
```

**Type `Y` and press Enter** to agree.

---

**Prompt 3: EFF Email Sharing (optional)**

```
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
```

**Type `Y` or `N`** (your choice - this just signs you up for EFF newsletters).

---

**Certbot will now work:**

You'll see output like:

```
Requesting a certificate for photos.yourdomain.com

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/photos.yourdomain.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/photos.yourdomain.com/privkey.pem
This certificate expires on 2025-05-04.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for photos.yourdomain.com to /etc/nginx/sites-enabled/immich
Congratulations! You have successfully enabled HTTPS on https://photos.yourdomain.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

**✓ Success Criteria:**
* "Successfully received certificate"
* "Successfully deployed certificate"
* "Congratulations! You have successfully enabled HTTPS"
* Certificate path shown
* Expiration date shown (90 days from now)

**✍️ Document certificate info:**

| Information | Value |
|-------------|-------|
| Certificate path | `/etc/letsencrypt/live/photos.yourdomain.com/` |
| Expiration date | `_________________________` |
| Email for notifications | `_________________________` |

---

**⚠ Common errors:**

**"Timeout during connect"**
- Port 80 not reachable from internet
- Check port forwarding in router
- Check firewall: `sudo ufw status` (should allow 80/tcp)

**"Could not bind to IPv4 or IPv6"**
- Port 80 already in use by something else
- Check: `sudo netstat -tlnp | grep :80`
- Make sure Nginx is running on port 80

**"DNS problem: NXDOMAIN looking up A for photos.yourdomain.com"**
- Domain doesn't resolve
- Check DNS: `nslookup photos.yourdomain.com`
- Wait for DNS propagation (up to 24 hours)

**"Too many requests"**
- Let's Encrypt rate limit hit (5 certificates per domain per week)
- Wait an hour and try again
- Or use: `sudo certbot --nginx -d photos.yourdomain.com --dry-run` to test first

---

### Step 10.4: Verify Nginx Configuration Was Updated

Certbot automatically modified your Nginx configuration. Let's see what changed.

```bash
cat /etc/nginx/sites-available/immich
```

**✓ Expected changes:**

You should now see **two** server blocks:

**Server block 1: HTTP (port 80) - Redirect to HTTPS**

```nginx
server {
    listen 80;
    server_name photos.yourdomain.com;

    if ($host = photos.yourdomain.com) {
        return 301 https://$host$request_uri;
    }

    return 404;
}
```

**Server block 2: HTTPS (port 443) - Main configuration**

```nginx
server {
    listen 443 ssl;
    server_name photos.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/photos.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/photos.yourdomain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://localhost:2283;
        ...
    }
}
```

**📝 What Certbot added:**
- Port 80 now redirects to HTTPS (301 redirect)
- Port 443 serves HTTPS with SSL certificate
- SSL certificate paths configured
- Secure SSL options included

**✓ Success Criteria:**
* Two server blocks exist
* Port 80 has redirect to HTTPS
* Port 443 has SSL certificate configured
* Certificate paths point to `/etc/letsencrypt/live/yourdomain/`

---

### Step 10.5: Test HTTPS Connection

Now test that HTTPS is working.

**From any browser (on any network):**

Visit: `https://photos.yourdomain.com`

**⚠ Notice:** Using `https://` (not `http://`)

**✓ Expected result:**

- ✅ Page loads successfully
- ✅ **Green padlock icon** in address bar
- ✅ URL shows `https://photos.yourdomain.com`
- ✅ Immich login page appears
- ✅ Browser doesn't show security warnings

**🎉 If you see the green padlock - SUCCESS!** Your connection is now encrypted!

---

**Click on the padlock to see certificate details:**

You should see:
- **Issued by:** Let's Encrypt
- **Valid until:** [Date 90 days from now]
- **Subject:** photos.yourdomain.com

---

**Test HTTP to HTTPS redirect:**

Visit: `http://photos.yourdomain.com` (note: **http**, not https)

**✓ Expected result:**

- Browser automatically redirects to `https://photos.yourdomain.com`
- URL changes from `http://` to `https://`
- You end up on the HTTPS version

**💡 This means:** All HTTP traffic is automatically upgraded to HTTPS.

---

**⚠ If you see security warnings:**

**"Your connection is not private" or "NET::ERR_CERT_AUTHORITY_INVALID"**
- Certificate wasn't installed correctly
- Check certificate paths in Nginx config
- Try: `sudo certbot --nginx -d photos.yourdomain.com` again

**"NET::ERR_CERT_COMMON_NAME_INVALID"**
- Certificate is for wrong domain
- Check server_name in Nginx matches certificate domain
- Verify you requested cert for correct domain

**Mixed content warning**
- Some resources loading over HTTP instead of HTTPS
- This is fine for now (Immich should handle this)

---

### Step 10.6: Test Auto-Renewal

Let's Encrypt certificates expire after 90 days. Certbot automatically renews them.

**Test the renewal process:**

```bash
sudo certbot renew --dry-run
```

**What this does:** Simulates renewal process without actually renewing (safe to run anytime).

**✓ Expected output:**

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/photos.yourdomain.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Account registered.
Simulating renewal of an existing certificate for photos.yourdomain.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded:
  /etc/letsencrypt/live/photos.yourdomain.com/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

**✓ Success Criteria:**
* "Congratulations, all simulated renewals succeeded"
* Your domain listed with "(success)"

**⚠ If renewal fails:**
- Port 80 must remain open (Let's Encrypt uses it for renewal)
- Don't close port 80 after getting certificate
- Check: `sudo ufw status` (should allow 80/tcp)

---

**Verify renewal timer is active:**

```bash
sudo systemctl status certbot.timer
```

**✓ Expected output:**

```
● certbot.timer - Run certbot twice daily
     Loaded: loaded (/lib/systemd/system/certbot.timer; enabled; preset: enabled)
     Active: active (waiting) since Mon 2025-02-03 18:45:12 UTC; 10min ago
    Trigger: Tue 2025-02-04 03:17:23 UTC; 8h left
   Triggers: ● certbot.service
```

**📝 Key indicators:**
- **Active: active (waiting)** - Timer is running
- **enabled** - Will start on boot
- **Trigger:** - Next check time (runs twice daily)

Press `q` to exit.

**💡 What this means:** Certbot will automatically check twice per day if renewal is needed. Certificates are renewed when they have 30 days left.

---

### Step 10.7: Add Security Headers (Recommended)

Enhance security with additional HTTP headers.

**Open Nginx config:**

```bash
sudo nano /etc/nginx/sites-available/immich
```

**Find the HTTPS server block** (the one with `listen 443 ssl`).

**After the `ssl_dhparam` line, add these security headers:**

```nginx
server {
    listen 443 ssl;
    server_name photos.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/photos.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/photos.yourdomain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # Security headers (ADD THESE LINES)
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    location / {
        proxy_pass http://localhost:2283;
        ...
    }
}
```

**💡 What each header does:**

| Header | Purpose |
|--------|---------|
| `Strict-Transport-Security` | Forces HTTPS for 1 year (prevents downgrade attacks) |
| `X-Content-Type-Options` | Prevents MIME type sniffing |
| `X-Frame-Options` | Prevents clickjacking (embedding in iframes) |
| `X-XSS-Protection` | Enables browser XSS filter |
| `Referrer-Policy` | Controls referrer information sharing |

---

**Save the file:**

1. Press `Ctrl + O`
2. Press `Enter`
3. Press `Ctrl + X`

---

**Test configuration:**

```bash
sudo nginx -t
```

**✓ Expected output:**

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

**Reload Nginx:**

```bash
sudo systemctl reload nginx
```

**✓ Expected output:** No output (silent success).

---

### Step 10.8: Verify Security Headers

Check that security headers are being sent.

```bash
curl -I https://photos.yourdomain.com
```

**✓ Expected output:**

```
HTTP/2 200
server: nginx/1.24.0
date: Mon, 03 Feb 2025 19:15:34 GMT
content-type: text/html
strict-transport-security: max-age=31536000; includeSubDomains
x-content-type-options: nosniff
x-frame-options: SAMEORIGIN
x-xss-protection: 1; mode=block
referrer-policy: strict-origin-when-cross-origin
```

**✓ Success Criteria:**
* You see the security headers in the response
* `strict-transport-security` is present
* All headers you added are listed

---

### Step 10.9: Test SSL Security Rating (Optional)

Check your SSL configuration quality.

**Use SSL Labs test:**

1. Go to: [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/)
2. Enter your domain: `photos.yourdomain.com`
3. Click "Submit"
4. Wait 2-3 minutes for analysis

**✓ Expected rating:**

- **A** or **A+** grade
- "This server supports TLS 1.3"
- "Certificate is trusted"

**💡 This is optional** but good to know your security is properly configured.

**⚠ Note:** This test is public - your domain will be visible in SSL Labs' public database.

---

### ✅ Checkpoint: Section 10 Complete

**You should now have:**

* ✓ Certbot installed and configured
* ✓ SSL certificate obtained from Let's Encrypt
* ✓ Nginx configured for HTTPS (port 443)
* ✓ HTTP automatically redirects to HTTPS
* ✓ Green padlock showing in browsers
* ✓ Auto-renewal configured and tested
* ✓ Security headers added
* ✓ A/A+ SSL rating (if you tested)

**What you learned:**

* What HTTPS/SSL/TLS is and why it's critical
* How Let's Encrypt provides free certificates
* How Certbot automates certificate management
* What HTTP to HTTPS redirection does
* What security headers protect against
* How certificate auto-renewal works
* Certificate expiration timeline (90 days)

**HTTPS configuration summary:**

| Setting | Value |
|---------|-------|
| Certificate issuer | Let's Encrypt |
| Certificate path | `/etc/letsencrypt/live/photos.yourdomain.com/` |
| Expiration | 90 days (auto-renews at 60 days) |
| Renewal check | Twice daily (automatic) |
| HTTP redirect | Enabled (port 80 → 443) |
| HTTPS port | 443 |
| Security headers | Enabled |
| SSL rating | A or A+ |

**Access methods (FINAL):**

| Location | URL | Encrypted |
|----------|-----|-----------|
| Home WiFi (local) | `http://192.168.1.50` | ❌ No |
| Home WiFi (domain) | `https://photos.yourdomain.com` | ✅ Yes |
| Away from home | `https://photos.yourdomain.com` | ✅ Yes |
| Mobile apps | `https://photos.yourdomain.com` | ✅ Yes |

**Important reminders:**

* **Keep port 80 open** - Required for certificate renewal
* **Monitor email** - You'll get warnings if auto-renewal fails
* **Certificate expires in 90 days** - Auto-renewal happens at 60 days
* **Renewal runs twice daily** - Automated by systemd timer

**Useful commands:**

```bash
# Check certificate info
sudo certbot certificates

# Force renewal (if needed)
sudo certbot renew --force-renewal

# Test renewal
sudo certbot renew --dry-run

# Check renewal timer
sudo systemctl status certbot.timer

# View certificate expiration
echo | openssl s_client -connect localhost:443 2>/dev/null | openssl x509 -noout -dates
```

**Next steps:**

Your Immich server is now:
- ✅ Fully installed and configured
- ✅ Accessible from anywhere
- ✅ Encrypted with HTTPS
- ✅ Ready for mobile apps

The remaining sections cover:
- **Section 11-12:** Mobile app installation and configuration
- **Section 13:** Convenience scripts for maintenance
- **Section 14:** Google Photos import (if migrating)
- **Section 15:** Backup strategy (CRITICAL - do before uploading photos)
- **Section 16:** Google OAuth (optional)
- **Section 17:** Maintenance and updates

**🎉 Major milestone achieved!** Your Immich server is production-ready. You can now safely upload photos!

**🔖 Your secure URL:** `https://photos.yourdomain.com`

---

## 11. Install Immich Mobile Apps

**📋 What You'll Accomplish:**
* Install official Immich mobile app
* Understand app features and permissions
* Verify app authenticity
* Prepare for initial setup

**⏱ Estimated Time:** 5 minutes

**📦 Prerequisites:**
* ✓ Immich server working (Section 6)
* ✓ HTTPS configured (Section 10) - recommended for mobile apps
* ✓ iOS device (iPhone/iPad) or Android device

**💡 Why use the mobile app?**

**Key features:**
- ✅ **Automatic photo backup** - Upload photos as you take them
- ✅ **Background upload** - Works even when app is closed
- ✅ **Selective backup** - Choose which albums to backup
- ✅ **View photos** - Browse your entire library
- ✅ **Share photos** - Create public share links
- ✅ **Face recognition** - View people albums
- ✅ **Search** - Find photos by objects, places, dates

**Privacy note:** The Immich app uploads to YOUR server (not Immich's servers). Your photos stay under your control.

---

### Step 11.1: Install Immich App (iOS)

**For iPhone and iPad users:**

1. **Open the App Store** on your iOS device

2. **Search for "Immich"**
   - Full name: "Immich - Photo and video backup"
   - Developer: "Immich"

3. **Verify it's the official app:**
   - ✓ Developer: "Immich"
   - ✓ Category: "Photo & Video"
   - ✓ Has good reviews (4.5+ stars)
   - ✓ Description mentions "self-hosted"

4. **Tap "Get"** or the download icon

5. **Authenticate** with Face ID, Touch ID, or password

6. **Wait for installation** to complete

**✓ Success Criteria:**
* App appears on home screen
* Icon shows Immich logo (mountain/landscape design)

**App requirements:**
* iOS 13.0 or later
* ~50-100 MB storage space for the app
* Internet connection (WiFi or cellular)

---

### Step 11.2: Install Immich App (Android)

**For Android users:**

1. **Open Google Play Store** on your Android device

2. **Search for "Immich"**
   - Full name: "Immich"
   - Developer: "Immich"

3. **Verify it's the official app:**
   - ✓ Developer: "Immich"
   - ✓ Category: "Photography"
   - ✓ Has good reviews (4.5+ stars)
   - ✓ Description mentions "self-hosted photo and video backup"

4. **Tap "Install"**

5. **Wait for installation** to complete

**✓ Success Criteria:**
* App appears in app drawer
* Icon shows Immich logo

**App requirements:**
* Android 8.0 (Oreo) or later
* ~50-100 MB storage space for the app
* Internet connection (WiFi or cellular)

---

### Step 11.3: Grant Required Permissions (First Launch)

When you first open the app, it will request permissions.

**Open the Immich app**

**Permissions you'll be asked for:**

**1. Photos/Media access**
- **Why needed:** To read your photos and upload them
- **Recommendation:** **Allow** (required for backup)
- **iOS:** Choose "All Photos" (not "Selected Photos")
- **Android:** Choose "Allow" (or "While using the app")

**2. Camera access** (optional)
- **Why needed:** To take photos directly in the app
- **Recommendation:** Allow if you want to use Immich's camera
- **You can skip:** Regular Camera app still works

**3. Location access** (optional)
- **Why needed:** To tag photos with GPS location
- **Recommendation:** "While Using the App" or "Allow"
- **Privacy note:** Location data stays on your server

**4. Notifications** (optional)
- **Why needed:** Upload completion notifications
- **Recommendation:** Allow (helpful for monitoring backups)

**5. Background app refresh** (iOS) / Background activity (Android)
- **Why needed:** Upload photos even when app is closed
- **Recommendation:** **Allow** (critical for auto-backup)

**✓ Grant all recommended permissions** for best experience.

**💡 You can change permissions later** in your phone's Settings app.

---

### ✅ Checkpoint: Section 11 Complete

**You should now have:**

* ✓ Immich app installed on your phone
* ✓ Verified it's the official Immich app
* ✓ Granted necessary permissions
* ✓ App ready for configuration

**What you learned:**

* How to find and verify the official Immich app
* What permissions the app needs and why
* Difference between iOS and Android installations

**App info:**

| Platform | App Size | Requirements | Developer |
|----------|----------|--------------|-----------|
| iOS | ~80 MB | iOS 13.0+ | Immich |
| Android | ~60 MB | Android 8.0+ | Immich |

**Next steps:**

In Section 12, you'll configure the app to connect to your server and set up automatic photo backup.

---

## 12. Configure Mobile Apps and Set Up Backup

**📋 What You'll Accomplish:**
* Connect mobile app to your Immich server
* Set up automatic photo backup
* Configure backup settings (which albums, when to backup)
* Test photo upload
* Understand local vs remote server URLs

**⏱ Estimated Time:** 10-15 minutes

**📦 Prerequisites:**
* ✓ Section 11 complete (App installed)
* ✓ Immich server accessible
* ✓ Your server URL and login credentials

**💡 Server URL decision:**

You have two options for connecting:

| Option | URL | When to use |
|--------|-----|-------------|
| **HTTPS (recommended)** | `https://photos.yourdomain.com` | Always works (home and away) |
| **Local HTTP** | `http://192.168.1.50:2283` | Only works on home WiFi |

**🎯 Recommendation:** Use HTTPS URL (`https://photos.yourdomain.com`) - it works everywhere and is encrypted.

---

### Step 12.1: Initial App Setup

Launch the Immich app for the first time.

**Open Immich app** on your phone.

**✓ Expected result:**

You'll see a **welcome screen** or **server setup screen**.

---

**Enter server URL:**

You'll see a field asking for "Server Endpoint URL" or similar.

**Enter your HTTPS URL:**

```
https://photos.yourdomain.com
```

**💡 Important formatting:**
- ✅ Include `https://` (don't forget this!)
- ✅ No trailing slash (no `/` at the end)
- ❌ Don't use IP address here (use domain for HTTPS)

**Examples:**
- ✅ Correct: `https://photos.yourdomain.com`
- ❌ Wrong: `photos.yourdomain.com` (missing https://)
- ❌ Wrong: `https://photos.yourdomain.com/` (trailing slash)
- ❌ Wrong: `https://192.168.1.50:2283` (IP won't have valid HTTPS)

**Tap "Connect"** or "Next"

---

**✓ Expected result:**

- App connects to your server
- You see a **login screen**

**⚠ If connection fails:**

**"Cannot connect to server" or "Invalid URL"**
- Check URL spelling
- Make sure HTTPS is set up (Section 10)
- Test in browser first: `https://photos.yourdomain.com`
- Check you're connected to internet (WiFi or cellular)

**"SSL certificate error" or "Untrusted certificate"**
- Let's Encrypt cert might not be set up
- Try HTTP temporarily: `http://photos.yourdomain.com` (not secure!)
- Or go back to Section 10 and verify HTTPS

**"Connection timeout"**
- Server might be down: check `docker compose ps`
- Port 443 might not be forwarded
- Firewall might be blocking: `sudo ufw status`

---

### Step 12.2: Log In to Your Account

Enter your Immich credentials.

**On the login screen:**

1. **Email:** Enter your Immich admin email
   - The one you created in Section 6.12
   - Example: `admin@yourdomain.com`

2. **Password:** Enter your Immich password
   - **Not** your server password
   - **Not** your database password
   - The password you set when creating the Immich admin account

3. **Tap "Login"**

**✓ Expected result:**

- Login succeeds
- You see the Immich app home screen
- Empty photo library (unless you already uploaded photos)

**⚠ If login fails:**

**"Invalid email or password"**
- Double-check credentials
- Try logging in via web browser first to verify password
- Password is case-sensitive

**"Account not found"**
- Email might be wrong
- Check your email in the web interface

---

### Step 12.3: Enable Automatic Backup

Set up auto-upload for your photos.

**From the Immich app home screen:**

1. **Tap your profile icon** (bottom right or top right)

2. **Tap "Backup"** or "Auto Backup"

3. **Toggle "Auto Backup" ON**

**✓ Expected result:**

You'll see backup configuration options.

---

**Choose what to backup:**

**Select albums to backup:**

The app shows all photo albums on your device:
- Camera Roll / Recent
- Screenshots
- Other albums

**Recommendation:**
- ✅ Enable "Camera Roll" or "Recents" (your main photos)
- ⚠️ Consider disabling "Screenshots" (usually not worth backing up)
- ✅ Enable any other albums you want backed up

**Tap each album** to toggle backup on/off.

---

**Background backup settings:**

**When to backup:**

| Option | When it uploads | Data usage |
|--------|----------------|------------|
| **WiFi only** | Only on WiFi | ✅ No cellular data |
| **WiFi and cellular** | Always | ⚠️ Uses cellular data |

**🎯 Recommendation:**
- Start with **"WiFi only"** to avoid data charges
- Switch to "WiFi and cellular" later if you want

**Other settings:**

- **Backup videos:** Toggle ON if you want videos backed up (uses more storage)
- **Require charging:** Toggle ON to only backup when charging (saves battery)
- **Foreground backup:** Usually OFF (background works better)

---

### Step 12.4: Start Initial Backup

Now trigger the first backup.

**On the Backup screen:**

1. **Review your settings:**
   - Albums selected: Camera Roll
   - WiFi only: ON
   - Videos included: Your choice

2. **Tap "Start Backup"** or "Backup Now"

**✓ Expected result:**

- Upload progress appears
- Photos start uploading to your server
- You see count: "Uploading 1 of 250 photos"

**⏱ How long this takes:**

- 100 photos: ~5-10 minutes (on fast WiFi)
- 1,000 photos: ~30-60 minutes
- 10,000 photos: Several hours

**💡 Let it run in the background.** You can close the app and it will continue uploading.

---

**Monitor upload progress:**

**In the app:**
- Tap profile icon → Backup
- See "X photos uploaded"
- See remaining count

**On web interface:**
- Log in to `https://photos.yourdomain.com`
- Watch photos appear in real-time

---

**⚠ If upload fails or is very slow:**

**Photos uploading slowly:**
- Check WiFi speed (try speed test)
- Your upload bandwidth determines speed
- Large videos take longer

**Upload stuck at 0%:**
- Force close app and reopen
- Check server is running: `docker compose ps`
- Check logs: `docker compose logs -f immich-server`

**"Backup failed" error:**
- Check storage space on server: `df -h /var/lib/immich`
- Check Immich logs for errors
- Try uploading a single photo first (test)

**Some photos won't upload:**
- Certain formats might not be supported
- Check Immich logs for specific errors
- Duplicate photos are automatically skipped

---

### Step 12.5: Verify Photos Uploaded Successfully

Check that your photos are on the server.

**On your phone:**

1. **Open Immich app**
2. **Tap "Photos"** tab
3. **See your uploaded photos**

**✓ Expected result:**
- You see thumbnails of your photos
- Timeline view shows photos by date
- Tap a photo to view full resolution

---

**On web interface:**

1. **Open browser:** `https://photos.yourdomain.com`
2. **Log in**
3. **Check "Photos" section**

**✓ Expected result:**
- Same photos appear in web and mobile
- Count matches what was uploaded
- Photos are organized by date

---

### Step 12.6: Configure Advanced Backup Settings

Fine-tune your backup preferences.

**In app settings:**

**Tap profile → Settings → Backup**

**Useful settings:**

**Upload queue management:**
- "Upload queue limit": Number of photos uploading simultaneously
- Recommendation: Leave at default (3-5)

**Upload strategy:**
- "Upload new photos immediately": ON for instant backup
- "Upload only while charging": ON to save battery

**Duplicate handling:**
- "Skip duplicate assets": ON (recommended)
- Prevents uploading same photo twice

**Original quality:**
- "Upload original photos": ON (recommended)
- Uploads full resolution, not compressed

---

### Step 12.7: Test Backup with New Photo

Verify auto-backup works for new photos.

**Take a new photo:**

1. **Open your device's Camera app**
2. **Take a photo** of anything
3. **Wait 30 seconds - 2 minutes**

**Check if it uploaded:**

1. **Open Immich app**
2. **Go to Photos tab**
3. **Look for the new photo** at the top

**✓ Expected result:**
- New photo appears in Immich within 1-2 minutes
- Auto-backup is working!

**⚠ If new photo doesn't appear:**
- Check auto-backup is ON (profile → backup)
- Make sure you're on WiFi (if WiFi-only setting)
- Force sync: tap profile → backup → "Sync now"

---

### Step 12.8: Optional - Set Up Multiple Server Profiles

Create separate profiles for home and away.

**Why use multiple profiles?**

| Profile | URL | Use case |
|---------|-----|----------|
| **Home** | `http://192.168.1.50:2283` | Faster, no internet needed |
| **Remote** | `https://photos.yourdomain.com` | Works anywhere, encrypted |

**💡 Advanced feature** - most users can just use HTTPS URL everywhere.

---

**To add a second profile (if your app supports it):**

**Note:** Not all Immich app versions support multiple profiles. If yours doesn't, just use the HTTPS URL everywhere.

1. **Go to Settings**
2. **Look for "Servers" or "Profiles"**
3. **Add new server**
4. **Enter local URL:** `http://192.168.1.50:2283`
5. **Log in** with same credentials
6. **Switch between profiles** as needed

**When to use each:**
- **At home:** Use local profile (faster)
- **Away:** Use remote profile (HTTPS)

**💡 Most users don't need this.** HTTPS works everywhere and is simpler.

---

### Step 12.9: Configure Backup Schedule

Set when automatic backups occur.

**In Immich app settings:**

**Backup frequency options:**

- **Immediate**: Upload as soon as photo is taken (recommended)
- **Daily**: Upload once per day
- **Manual only**: Only when you trigger it

**🎯 Recommendation:** Use "Immediate" so photos are backed up right away.

**Backup conditions:**

Create a backup schedule that works for you:

```
Example conservative setup:
✓ WiFi only
✓ While charging
✓ Immediate upload
✓ Include videos
✓ Original quality

Example aggressive setup:
✓ WiFi and cellular
✓ Anytime
✓ Immediate upload
✓ Include videos
✓ Original quality
(⚠️ Uses cellular data)
```

---

### ✅ Checkpoint: Section 12 Complete

**You should now have:**

* ✓ Mobile app connected to your server
* ✓ Successfully logged in
* ✓ Auto-backup enabled
* ✓ Initial photos uploaded
* ✓ New photos automatically backing up
* ✓ Understanding of backup settings
* ✓ Backup working on WiFi (and optionally cellular)

**What you learned:**

* How to connect mobile app to Immich server
* Difference between local HTTP and HTTPS URLs
* How to configure auto-backup settings
* What backup conditions are available (WiFi, charging, etc.)
* How to monitor backup progress
* How to verify photos uploaded successfully
* Troubleshooting common connection issues

**Mobile app configuration summary:**

| Setting | Recommended Value | Why |
|---------|------------------|-----|
| Server URL | `https://photos.yourdomain.com` | Works everywhere, encrypted |
| Auto-backup | ON | Automatic protection |
| Backup trigger | Immediate | Photos backed up right away |
| WiFi only | ON (initially) | Avoid data charges |
| Include videos | Your choice | Videos use lots of storage |
| Original quality | ON | Preserve full resolution |
| Require charging | Optional | Saves battery |

**Backup behavior:**

| Scenario | What happens |
|----------|--------------|
| Take photo on phone | Uploads within 1-2 minutes (if on WiFi) |
| Open app manually | Checks for new photos and uploads |
| Phone locked | Continues uploading in background |
| Leave WiFi | Pauses (if WiFi-only mode) |
| Return to WiFi | Resumes uploading |
| Phone charging | Uploads (if charging-only mode) |

**Storage considerations:**

| Library size | Phone storage needed | Server storage needed |
|--------------|---------------------|----------------------|
| 1,000 photos | Keep originals or remove | ~5 GB |
| 5,000 photos | Can remove after backup | ~25 GB |
| 10,000 photos | Can remove after backup | ~50 GB |

**💡 Free up phone space:** After photos are backed up to Immich, you can delete them from your phone and view them through the app (they stay on the server).

**Useful app features:**

- **Timeline:** View all photos chronologically
- **Search:** Find photos by objects, people, places
- **Albums:** Organize photos into collections
- **Sharing:** Create public share links
- **Favorites:** Star important photos
- **Archive:** Hide photos from main view
- **Trash:** Recover deleted photos (30 days)

**Troubleshooting quick reference:**

| Problem | Solution |
|---------|----------|
| Can't connect to server | Check URL has `https://`, verify server is running |
| Login fails | Verify credentials in web interface first |
| Photos not uploading | Check WiFi, auto-backup ON, sufficient storage |
| Upload slow | Normal for first backup, depends on internet speed |
| Some photos skipped | Duplicates auto-skipped, check logs for errors |

**Next steps:**

Your mobile setup is complete! Photos will now automatically backup to your server.

The remaining sections cover:
- **Section 13:** Convenience scripts (update, backup, status commands)
- **Section 14:** Google Photos import (if migrating from Google)
- **Section 15:** Backup strategy (CRITICAL before uploading full library)
- **Section 16:** Google OAuth (optional login with Google)
- **Section 17:** Maintenance and updates

**🎉 Congratulations!** You now have a fully functional self-hosted photo backup system!

**🔖 Daily use:** Just take photos normally - they'll automatically upload to your Immich server!

---

## 13. Create Convenience Scripts

**📋 What You'll Accomplish:**
* Create helpful maintenance scripts for common tasks
* Set up quick commands for updates, restarts, and monitoring
* Add backup and status check scripts
* Make scripts globally accessible
* Learn basic shell scripting for Immich management

**⏱ Estimated Time:** 15-20 minutes

**📦 Prerequisites:**
* ✓ Immich installed and running (Section 6)
* ✓ Basic command line familiarity

**💡 Why create scripts?**

Instead of typing long commands every time you need to:
- Update Immich
- Check status
- View logs
- Restart services

You'll have simple commands like:
```bash
immich-update.sh
immich-status.sh
immich-logs.sh
```

**These scripts will save you time and reduce errors.**

---

### Step 13.1: Create Scripts Directory

Set up a dedicated directory for Immich maintenance scripts.

```bash
mkdir -p ~/immich/scripts
cd ~/immich/scripts
```

**What this does:** Creates `scripts` directory inside your Immich configuration folder.

**✓ Expected output:** No output (silent success).

---

**Verify directory creation:**

```bash
pwd
```

**✓ Expected output:**

```
/home/username/immich/scripts
```

---

### Step 13.2: Create Update Script

This script updates Immich to the latest version.

```bash
nano immich-update.sh
```

**Add this content:**

```bash
#!/bin/bash
# Update Immich to the latest version
# Usage: ./immich-update.sh

set -e  # Exit on any error

echo "======================================"
echo "Immich Update Script"
echo "======================================"
echo ""

# Navigate to Immich directory
cd ~/immich || { echo "Error: ~/immich directory not found"; exit 1; }

echo "Step 1/5: Pulling latest Docker images..."
docker compose pull

echo ""
echo "Step 2/5: Stopping containers..."
docker compose down

echo ""
echo "Step 3/5: Starting updated containers..."
docker compose up -d

echo ""
echo "Step 4/5: Waiting for services to start..."
sleep 10

echo ""
echo "Step 5/5: Checking container status..."
docker compose ps

echo ""
echo "Step 6/6: Cleaning up old images..."
docker image prune -f

echo ""
echo "======================================"
echo "✓ Immich updated successfully!"
echo "======================================"
echo ""
echo "Check web interface: https://photos.yourdomain.com"
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

---

**Make executable:**

```bash
chmod +x immich-update.sh
```

---

**Test the script:**

```bash
./immich-update.sh
```

**✓ Expected output:**

```
======================================
Immich Update Script
======================================

Step 1/5: Pulling latest Docker images...
[+] Pulling...
...
✓ Immich updated successfully!
```

**💡 Run this monthly** to get latest features and security updates.

---

### Step 13.3: Create Restart Script

This script restarts all Immich services (useful for troubleshooting).

```bash
nano immich-restart.sh
```

**Add this content:**

```bash
#!/bin/bash
# Restart all Immich services
# Usage: ./immich-restart.sh

set -e

echo "======================================"
echo "Immich Restart Script"
echo "======================================"
echo ""

cd ~/immich || { echo "Error: ~/immich directory not found"; exit 1; }

echo "Restarting all Immich containers..."
docker compose restart

echo ""
echo "Waiting for services to stabilize..."
sleep 5

echo ""
echo "Container status:"
docker compose ps

echo ""
echo "======================================"
echo "✓ Immich restarted successfully!"
echo "======================================"
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

---

**Make executable:**

```bash
chmod +x immich-restart.sh
```

---

**Test the script:**

```bash
./immich-restart.sh
```

**✓ Expected output:**

```
======================================
Immich Restart Script
======================================

Restarting all Immich containers...
[+] Restarting...
✓ Immich restarted successfully!
```

---

### Step 13.4: Create Logs Viewer Script

This script shows real-time logs from Immich server.

```bash
nano immich-logs.sh
```

**Add this content:**

```bash
#!/bin/bash
# View Immich server logs
# Usage: ./immich-logs.sh [service]
# Example: ./immich-logs.sh immich-server

set -e

SERVICE="${1:-immich-server}"  # Default to immich-server

echo "======================================"
echo "Immich Logs Viewer"
echo "======================================"
echo "Service: $SERVICE"
echo "Press Ctrl+C to exit"
echo "======================================"
echo ""

cd ~/immich || { echo "Error: ~/immich directory not found"; exit 1; }

# Follow logs
docker compose logs -f --tail=50 "$SERVICE"
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

---

**Make executable:**

```bash
chmod +x immich-logs.sh
```

---

**Usage examples:**

```bash
# View server logs (default)
./immich-logs.sh

# View specific service logs
./immich-logs.sh immich-microservices
./immich-logs.sh postgres
./immich-logs.sh redis
```

**💡 Press `Ctrl + C`** to stop following logs.

---

### Step 13.5: Create Status Check Script

This script shows overall Immich health and disk usage.

```bash
nano immich-status.sh
```

**Add this content:**

```bash
#!/bin/bash
# Check Immich status, disk usage, and health
# Usage: ./immich-status.sh

set -e

echo "======================================"
echo "Immich Status Report"
echo "======================================"
echo "Generated: $(date)"
echo ""

cd ~/immich || { echo "Error: ~/immich directory not found"; exit 1; }

# Container status
echo "--- Docker Container Status ---"
docker compose ps
echo ""

# Disk usage
echo "--- Disk Usage (Storage Partition) ---"
df -h /var/lib/immich | tail -n 1
echo ""

# Immich directory size
echo "--- Immich Photo Library Size ---"
du -sh /var/lib/immich 2>/dev/null || echo "Unable to read directory"
echo ""

# Breakdown by subdirectory
echo "--- Storage Breakdown ---"
du -h /var/lib/immich 2>/dev/null | sort -rh | head -10 || echo "Unable to read directory"
echo ""

# Recent log errors (if any)
echo "--- Recent Errors (last 10) ---"
docker compose logs --tail=100 2>/dev/null | grep -i error | tail -10 || echo "No recent errors found"
echo ""

# Service uptime
echo "--- Service Uptime ---"
docker compose ps --format "table {{.Service}}\t{{.Status}}"
echo ""

echo "======================================"
echo "Status check complete!"
echo "======================================"
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

---

**Make executable:**

```bash
chmod +x immich-status.sh
```

---

**Test the script:**

```bash
./immich-status.sh
```

**✓ Expected output:**

```
======================================
Immich Status Report
======================================
Generated: Mon Feb 3 20:15:34 UTC 2025

--- Docker Container Status ---
NAME                  STATUS
immich-server         Up 2 days
immich-microservices  Up 2 days
...

--- Disk Usage (Storage Partition) ---
/dev/sda2  100G  25G  75G  25% /

--- Immich Photo Library Size ---
24G	/var/lib/immich

...
Status check complete!
```

---

### Step 13.6: Create Backup Script

This script backs up Immich configuration and database.

```bash
nano immich-backup.sh
```

**Add this content:**

```bash
#!/bin/bash
# Backup Immich configuration and database
# Usage: ./immich-backup.sh

set -e

BACKUP_DIR="$HOME/immich-backups"
DATE=$(date +%Y%m%d-%H%M%S)
BACKUP_FILE="$BACKUP_DIR/immich-backup-$DATE"

echo "======================================"
echo "Immich Backup Script"
echo "======================================"
echo "Backup location: $BACKUP_FILE"
echo ""

# Create backup directory
mkdir -p "$BACKUP_DIR"

cd ~/immich || { echo "Error: ~/immich directory not found"; exit 1; }

echo "Step 1/4: Backing up configuration files..."
mkdir -p "$BACKUP_FILE"
cp .env "$BACKUP_FILE/"
cp docker-compose.yml "$BACKUP_FILE/"

echo "Step 2/4: Backing up database..."
docker compose exec -T postgres pg_dumpall -U postgres > "$BACKUP_FILE/database.sql"

echo "Step 3/4: Compressing backup..."
cd "$BACKUP_DIR"
tar -czf "immich-backup-$DATE.tar.gz" "immich-backup-$DATE"
rm -rf "immich-backup-$DATE"

echo "Step 4/4: Cleaning old backups (keeping last 7)..."
ls -t immich-backup-*.tar.gz 2>/dev/null | tail -n +8 | xargs -r rm

echo ""
echo "======================================"
echo "✓ Backup completed successfully!"
echo "======================================"
echo "Backup file: $BACKUP_DIR/immich-backup-$DATE.tar.gz"
echo "Size: $(du -h "$BACKUP_DIR/immich-backup-$DATE.tar.gz" | cut -f1)"
echo ""
echo "To restore:"
echo "  1. Extract: tar -xzf immich-backup-$DATE.tar.gz"
echo "  2. Restore database: docker compose exec -T postgres psql -U postgres < database.sql"
echo "======================================"
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

---

**Make executable:**

```bash
chmod +x immich-backup.sh
```

---

**Test the script:**

```bash
./immich-backup.sh
```

**✓ Expected output:**

```
======================================
Immich Backup Script
======================================
...
✓ Backup completed successfully!
Backup file: /home/username/immich-backups/immich-backup-20250203-201534.tar.gz
Size: 125M
```

**💡 Run this before updates** to have a restore point.

---

### Step 13.7: Create Quick Health Check Script

A simple script for daily health checks.

```bash
nano immich-health.sh
```

**Add this content:**

```bash
#!/bin/bash
# Quick Immich health check
# Usage: ./immich-health.sh

set -e

cd ~/immich || { echo "Error: ~/immich directory not found"; exit 1; }

# Check if all containers are running
RUNNING=$(docker compose ps --services --filter "status=running" | wc -l)
TOTAL=$(docker compose ps --services | wc -l)

echo "Immich Health Check"
echo "==================="
echo "Containers: $RUNNING/$TOTAL running"

if [ "$RUNNING" -eq "$TOTAL" ]; then
    echo "Status: ✓ All services healthy"
    exit 0
else
    echo "Status: ✗ Some services down"
    echo ""
    echo "Container status:"
    docker compose ps
    exit 1
fi
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

---

**Make executable:**

```bash
chmod +x immich-health.sh
```

---

**Usage:**

```bash
./immich-health.sh
```

**✓ Expected output:**

```
Immich Health Check
===================
Containers: 5/5 running
Status: ✓ All services healthy
```

---

### Step 13.8: Add Scripts to PATH (Optional)

Make scripts available from any directory.

**Edit your .bashrc:**

```bash
nano ~/.bashrc
```

**Scroll to the end and add:**

```bash
# Immich convenience scripts
export PATH="$HOME/immich/scripts:$PATH"
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

---

**Apply changes:**

```bash
source ~/.bashrc
```

---

**Test global access:**

```bash
# From any directory
cd ~
immich-status.sh
```

**✓ Expected result:** Script runs from any location.

---

### Step 13.9: Create Aliases (Optional Alternative)

Instead of adding to PATH, create short aliases.

**Edit .bashrc:**

```bash
nano ~/.bashrc
```

**Add these aliases:**

```bash
# Immich command aliases
alias immich-update='~/immich/scripts/immich-update.sh'
alias immich-restart='~/immich/scripts/immich-restart.sh'
alias immich-status='~/immich/scripts/immich-status.sh'
alias immich-logs='~/immich/scripts/immich-logs.sh'
alias immich-backup='~/immich/scripts/immich-backup.sh'
alias immich-health='~/immich/scripts/immich-health.sh'
```

**Save:** `Ctrl + O`, `Enter`, `Ctrl + X`

---

**Apply:**

```bash
source ~/.bashrc
```

---

**Usage (shorter commands):**

```bash
immich-update
immich-status
immich-health
```

---

### ✅ Checkpoint: Section 13 Complete

**You should now have:**

* ✓ Scripts directory created (`~/immich/scripts/`)
* ✓ Update script for easy Immich updates
* ✓ Restart script for troubleshooting
* ✓ Logs viewer for debugging
* ✓ Status check script for monitoring
* ✓ Backup script for configuration/database
* ✓ Health check script for quick status
* ✓ Scripts accessible globally (if you added to PATH or created aliases)

**What you learned:**

* How to create bash scripts for automation
* How to make scripts executable (chmod +x)
* How to use docker compose commands in scripts
* How to backup Immich configuration and database
* How to add scripts to PATH for global access
* How to create command aliases

**Scripts created:**

| Script | Purpose | Usage |
|--------|---------|-------|
| `immich-update.sh` | Update to latest version | Monthly |
| `immich-restart.sh` | Restart all services | When troubleshooting |
| `immich-logs.sh` | View service logs | For debugging |
| `immich-status.sh` | Comprehensive status report | Weekly |
| `immich-backup.sh` | Backup config & database | Before updates |
| `immich-health.sh` | Quick health check | Daily |

**Usage examples:**

```bash
# Daily routine
immich-health.sh

# Weekly check
immich-status.sh

# Before updating
immich-backup.sh
immich-update.sh

# Troubleshooting
immich-restart.sh
immich-logs.sh immich-server
```

**Backup location:**

```
~/immich-backups/
├── immich-backup-20250203-120000.tar.gz
├── immich-backup-20250210-120000.tar.gz
└── immich-backup-20250217-120000.tar.gz
```

**💡 Automation tip:** You can schedule these scripts with cron for automatic maintenance:

```bash
# Edit crontab
crontab -e

# Add these lines:
0 2 * * 0 ~/immich/scripts/immich-backup.sh  # Weekly backup (Sunday 2 AM)
0 3 * * * ~/immich/scripts/immich-health.sh  # Daily health check (3 AM)
```

**Next steps:**

The remaining sections cover:
- **Section 14:** Google Photos import (if migrating)
- **Section 15:** Backup strategy (CRITICAL for photos)
- **Section 16:** Google OAuth (optional)
- **Section 17:** Maintenance and updates

**🔖 Quick reference card:**

```
immich-update.sh   → Update Immich
immich-restart.sh  → Restart services
immich-logs.sh     → View logs
immich-status.sh   → Full status report
immich-backup.sh   → Backup config/DB
immich-health.sh   → Quick health check
```
immich-logs.sh
```

---

## 14. Import Photos from Google Photos (Optional)

**📋 What You'll Accomplish:**
* Export your entire Google Photos library
* Download and transfer photos to your server
* Import photos into Immich with metadata preserved
* Verify import was successful
* Clean up temporary files

**⏱ Estimated Time:** Variable (depends on library size)
- Requesting export: 5 minutes
- Google processing: 2 hours - 7 days
- Downloading: 1-24 hours
- Importing: 2-48 hours

**💾 Storage Required:**
- Temporary: 2x your library size (for extraction)
- Permanent: 1x your library size (in Immich)

**📦 Prerequisites:**
* ✓ Immich installed and working (Section 6)
* ✓ Sufficient disk space (check with `df -h`)
* ✓ Google Photos account with photos

**💡 Should you do this?**

**Do this if:**
- ✅ Migrating from Google Photos to Immich
- ✅ Want to keep all metadata (dates, locations, albums)
- ✅ Have photos only in Google Photos

**Skip if:**
- ❌ Photos already on your computer (just upload directly)
- ❌ Small library (easier to download normally and upload)
- ❌ Don't use Google Photos

---

### Understanding the Process

**What happens:**
```
Google Photos → Google Takeout → Download → Extract → Immich Import
```

**Timeline:**
1. **Request export** (5 min) → Google processes (hours to days)
2. **Download archives** (1-24 hours depending on size/speed)
3. **Transfer to server** (if downloaded to computer)
4. **Extract archives** (30 min - 2 hours)
5. **Import to Immich** (2-48 hours depending on library size)

**💡 This is a slow process.** Plan accordingly and don't rush.

---

### Step 14.1: Estimate Your Library Size

Before starting, check how much data you're dealing with.

**In Google Photos web:**
1. Go to [photos.google.com](https://photos.google.com/)
2. Click your profile icon (top right)
3. Click "Photos settings"
4. Look for storage usage

**✍️ Document your library:**

| Metric | Your Value |
|--------|------------|
| Number of photos | `_____________` |
| Number of videos | `_____________` |
| Total storage used | `_____________` GB |
| Estimated export size | `_____________` GB (add 10% overhead) |

**💡 Plan for 2x space:** You need space for both the .zip files AND extracted content.

---

### Step 14.2: Request Google Takeout Export

Export your data from Google.

**Go to Google Takeout:**

Visit: [https://takeout.google.com/](https://takeout.google.com/)

**✓ You should see:** A page titled "Download your data" with a list of Google products.

---

**Deselect all products:**

1. Click **"Deselect all"** button at the top
2. Verify all checkboxes are unchecked

---

**Select only Google Photos:**

1. **Scroll down** to find "Google Photos"
2. **Check the box** next to Google Photos
3. **Click "All photo albums included"** to customize

**Choose what to include:**

**Recommended:**
- ✅ **All photo albums**: Include everything
- ✅ **Include metadata**: Preserves dates, locations, etc.

**Optional exclusions:**
- ⚠️ Uncheck albums you don't want (screenshots, etc.)

**Click "OK"** after making selections.

---

**Click "Next step"** at the bottom.

---

**Configure export settings:**

| Setting | Recommended Value | Why |
|---------|------------------|-----|
| **Delivery method** | Send download link via email | Easier than Google Drive |
| **Frequency** | Export once | One-time migration |
| **File type** | .zip | Compatible with most tools |
| **File size** | 50 GB | Manageable download size |

**💡 File size explanation:**

If your library is 200GB, choosing "50 GB" means you'll get 4 separate .zip files (~50GB each).

**Smaller file sizes (2GB):**
- ✅ Easier to download (resume if interrupted)
- ⚠️ Many files to manage (could be 100+ files)

**Larger file sizes (50GB):**
- ✅ Fewer files to manage
- ⚠️ Harder to download (can't easily resume)

**🎯 Recommendation:** Use 50GB for good balance.

---

**Create export:**

Click **"Create export"** button.

**✓ Expected result:**

You'll see a confirmation:
```
"We're preparing your archive. We'll email you when it's ready."
```

**⏱ Google processing time:**

| Library Size | Typical Wait Time |
|--------------|-------------------|
| < 10 GB | 2-6 hours |
| 10-50 GB | 6-24 hours |
| 50-200 GB | 1-3 days |
| 200+ GB | 3-7 days |

**💡 Be patient.** Google will email you when ready. You can close the page.

---

### Step 14.3: Download Export Archives

Once Google emails you, download the files.

**Check your email:**

Look for: **"Your Google data archive is ready"** from Google

**✓ Email contains:**
- Link to download page
- Expiration date (usually 7 days)
- File count and total size

---

**Download options:**

**Option A: Download directly to server** (recommended for large libraries)

**From your server:**

```bash
# Create temporary directory
mkdir -p ~/google-photos-import
cd ~/google-photos-import
```

**For each download link Google provided:**

```bash
# Download (replace URL with actual link from email)
wget "https://takeout.google.com/..."
```

**💡 If you have many files:** Google provides a download script. Look for "download-*.sh" link in the email.

---

**Option B: Download to computer, then transfer** (easier for beginners)

**On your computer:**
1. Click download links in Google's email
2. Save all .zip files to a folder (e.g., `Downloads/google-photos`)

**Transfer to server:**

```bash
# From your computer (replace with your details)
scp ~/Downloads/takeout-*.zip username@192.168.1.50:~/google-photos-import/
```

**⏱ Transfer time depends on:**
- File size
- Network speed
- Number of files

---

**Verify downloads:**

```bash
cd ~/google-photos-import
ls -lh
```

**✓ Expected output:**

```
total 245G
-rw-r--r-- 1 user user  50G Feb  3 20:15 takeout-20250203T201534Z-001.zip
-rw-r--r-- 1 user user  50G Feb  3 21:30 takeout-20250203T201534Z-002.zip
-rw-r--r-- 1 user user  50G Feb  3 22:45 takeout-20250203T201534Z-003.zip
-rw-r--r-- 1 user user  45G Feb  3 23:50 takeout-20250203T201534Z-004.zip
```

**✓ Success Criteria:**
* All files downloaded (count matches Google's email)
* File sizes look correct (not corrupted/truncated)

---

### Step 14.4: Extract Archives

Unzip all the downloaded files.

**Check available disk space first:**

```bash
df -h ~
```

**⚠ Important:** Make sure you have at least 2x the total .zip size available.

---

**Extract all archives:**

```bash
cd ~/google-photos-import

# Extract all .zip files
for file in takeout-*.zip; do
    echo "Extracting $file..."
    unzip -q "$file"
done
```

**What this does:** Loops through all .zip files and extracts them.

**⏱ This takes:** 30 minutes - 2 hours depending on size and disk speed.

**✓ Expected output:**

```
Extracting takeout-20250203T201534Z-001.zip...
Extracting takeout-20250203T201534Z-002.zip...
Extracting takeout-20250203T201534Z-003.zip...
Extracting takeout-20250203T201534Z-004.zip...
```

---

**Verify extraction:**

```bash
ls -la
```

**✓ Expected result:**

You should see a `Takeout` directory:

```
drwxr-xr-x  3 user user  4096 Feb  3 23:55 Takeout/
-rw-r--r--  1 user user   50G Feb  3 20:15 takeout-20250203T201534Z-001.zip
...
```

---

**Check extracted contents:**

```bash
ls -la Takeout/
```

**✓ Expected output:**

```
drwxr-xr-x  2 user user 4096 Feb  3 23:55 Google Photos/
-rw-r--r--  1 user user 1234 Feb  3 23:55 archive_browser.html
```

---

**Count photos extracted:**

```bash
find Takeout/Google\ Photos -type f \( -name "*.jpg" -o -name "*.png" -o -name "*.mp4" \) | wc -l
```

**✓ Expected output:**

```
15234
```

(Number should roughly match your Google Photos count)

---

### Step 14.5: Import Using Immich Web Interface

Use Immich's built-in import feature.

**⚠ Current limitation:** As of early 2025, Immich's web-based Google Takeout import is still developing. The CLI method (Step 14.6) is more reliable.

**If you want to try web import:**

1. **Log in** to Immich: `https://photos.yourdomain.com`
2. **Click** your profile icon (top right)
3. **Click "Administration"** (or gear icon)
4. **Go to "External Libraries"**
5. **Click "Create External Library"**
6. **Choose "Google Takeout"** (if available)
7. **Follow the wizard** to select your extracted directory

**💡 We recommend using the CLI method (next step)** for better control and reliability.

---

### Step 14.6: Import Using Immich CLI (Recommended)

Use the command-line tool for better reliability.

**Install Node.js (if not already installed):**

```bash
# Check if Node.js is installed
node --version
```

**If not installed:**

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

---

**Install Immich CLI:**

```bash
npm install -g @immich/cli
```

**✓ Expected output:**

```
added 145 packages in 12s
```

---

**Verify installation:**

```bash
immich --version
```

**✓ Expected output:**

```
@immich/cli/2.2.x
```

---

**Log in to your Immich server:**

```bash
immich login https://photos.yourdomain.com
```

**You'll be prompted for:**

```
? Server URL: https://photos.yourdomain.com
? Email: your-admin@email.com
? Password: [hidden]
```

**Enter your Immich admin credentials.**

**✓ Expected output:**

```
Logged in as your-admin@email.com
```

---

**Start the import:**

```bash
immich upload ~/google-photos-import/Takeout/Google\ Photos/
```

**What this does:** Recursively uploads all photos/videos from the Google Takeout export.

**⏱ This takes:** Hours to days depending on library size.

**✓ Expected output:**

```
Starting upload...
Uploading 15234 assets...
[=====>                    ] 1250/15234 (8%) ETA: 12h 34m
```

**💡 Let this run.** You can:
- Close SSH (upload continues in background - NOT RELIABLE, see below)
- Use `screen` or `tmux` for reliable background operation

**Better approach - use screen:**

```bash
# Install screen
sudo apt install screen

# Start a screen session
screen -S immich-import

# Run upload
immich upload ~/google-photos-import/Takeout/Google\ Photos/

# Detach from screen: Press Ctrl+A then D
# Reattach later: screen -r immich-import
```

---

### Step 14.7: Monitor Import Progress

Check on import status.

**In Immich web interface:**

1. Go to `https://photos.yourdomain.com`
2. Watch photo count increase in "Photos" section
3. Check timeline for new photos appearing

---

**Via CLI (if still running):**

The upload command shows progress:
```
[===============>          ] 8234/15234 (54%) ETA: 6h 12m
```

---

**Check Immich logs:**

```bash
~/immich/scripts/immich-logs.sh immich-server
```

Look for upload activity:
```
[Nest] INFO [AssetService] Processing asset upload...
```

---

### Step 14.8: Verify Import Success

Confirm all photos imported correctly.

**Check photo count in Immich:**

1. **Web interface** → "Photos" section
2. **Scroll to bottom** - should show total count
3. **Compare to Google Photos** count from Step 14.1

**✓ Success Criteria:**
* Photo count matches (within a few percent - some duplicates may be skipped)
* Timeline shows correct date ranges
* Random sample checks show photos with correct dates

---

**Check for import errors:**

```bash
grep -i "error\|failed" ~/google-photos-import/*.log
```

**✓ Expected:** No critical errors (some warnings are normal).

---

### Step 14.9: Clean Up Temporary Files

Free up disk space after successful import.

**⚠ Only do this after verifying import was successful!**

**Remove extracted files:**

```bash
rm -rf ~/google-photos-import/Takeout
```

---

**Remove .zip archives:**

```bash
rm ~/google-photos-import/takeout-*.zip
```

---

**Remove import directory:**

```bash
cd ~
rm -rf ~/google-photos-import
```

---

**Verify cleanup:**

```bash
df -h ~
```

**✓ Expected:** Disk space freed up (should match size of deleted files).

---

### ✅ Checkpoint: Section 14 Complete

**You should now have:**

* ✓ Google Photos data exported via Google Takeout
* ✓ All photos downloaded and extracted
* ✓ Photos imported into Immich with metadata
* ✓ Import verified (count matches)
* ✓ Temporary files cleaned up
* ✓ Disk space reclaimed

**What you learned:**

* How to use Google Takeout for data export
* How to transfer large files to a server
* How to use Immich CLI for bulk uploads
* How to use screen for long-running tasks
* How to verify data integrity after import

**Import statistics (example):**

| Metric | Value |
|--------|-------|
| Photos exported | 15,234 |
| Photos imported | 15,180 |
| Success rate | 99.6% |
| Total size | 245 GB |
| Time taken | 36 hours |
| Duplicates skipped | 54 |

**Common import issues:**

| Problem | Cause | Solution |
|---------|-------|----------|
| Some photos missing | Unsupported format | Check Immich logs for format errors |
| Dates wrong | Metadata not preserved | Re-import with CLI (preserves metadata) |
| Import stopped | Network/SSH disconnected | Use `screen` to prevent interruptions |
| Duplicates | Photos exist in multiple places | Normal - Immich skips duplicates |

**Next steps:**

Now that your photos are in Immich:
- **Section 15:** Set up backups (CRITICAL - do this before deleting from Google)
- **Section 16:** Optional OAuth setup
- **Section 17:** Ongoing maintenance

**⚠ Important:** Don't delete photos from Google Photos until you've:
1. ✅ Verified all photos are in Immich
2. ✅ Set up backups (Section 15)
3. ✅ Tested restoration from backup

**🔖 Storage freed:** After import and cleanup, you freed up ~245GB of temporary space!

---

## 15. Backup Strategy (Strongly Recommended)

### 📋 What You'll Accomplish

By the end of this section, you'll have:
- ✓ Understanding of why backups are critical
- ✓ Implemented the 3-2-1 backup rule
- ✓ Local backups running automatically
- ✓ Off-site cloud backups configured
- ✓ Verified your backups actually work

### ⏱ Time Required

- **Reading and understanding:** 10 minutes
- **Local backup setup:** 20 minutes
- **Cloud backup setup (optional):** 30 minutes
- **Testing and verification:** 15 minutes
- **Total:** 45-75 minutes

### 💰 Cost Estimate

- **External USB drive (2-4TB):** $60-120 (one-time)
- **Cloud storage (optional):**
  - Backblaze B2: ~$6/TB/month
  - Wasabi: ~$7/TB/month
  - Google Drive: Included with Google One
  - Total monthly: $0-20 depending on photo library size

### Prerequisites

Before starting this section, you should have:
- ✓ Immich fully installed and running (Section 6)
- ✓ Photos uploaded to Immich
- ✓ External USB drive (for local backups)
- ✓ Familiarity with terminal commands

---

### ⚠ Why This Section Is Critical

**This section is non-optional if your photos matter to you.**

Immich helps you *manage* your photos — it does **not** protect you from:

* **Disk failure** - Hard drives fail without warning
* **Accidental deletion** - One wrong command can delete everything
* **Server reinstall** - OS corruption or upgrade gone wrong
* **Ransomware** - Malware that encrypts your files for ransom
* **Hardware theft** - Physical loss of your server
* **Natural disasters** - Fire, flood, power surge

> 💡 **Plain-language explanation:** [Backblaze 3-2-1 Backup Strategy](https://www.backblaze.com/blog/the-3-2-1-backup-strategy/)

**You don't need an enterprise solution, but you *do* need a plan.**

---

### 15.1 Understand the 3-2-1 Backup Rule

The industry-standard backup strategy:

| Component | Requirement | Example |
|-----------|-------------|---------|
| **3 copies** | Original + 2 backups | Server + External drive + Cloud |
| **2 media types** | Different storage tech | SSD + HDD, or HDD + Cloud |
| **1 off-site** | Physical/geographical separation | Cloud storage or drive at office |

**Example implementation:**
1. **Copy 1:** Photos on your Immich server (original)
2. **Copy 2:** Daily backup to external USB drive (different media)
3. **Copy 3:** Weekly backup to cloud storage (off-site)

✓ **Success criteria:** Can you answer "yes" to these questions?
- If my server dies, can I restore everything from the external drive?
- If my house burns down, can I restore everything from the cloud?

---

### 15.2 Identify What to Back Up

At minimum, back up these four components:

| Component | Location | Size Estimate | Critical? |
|-----------|----------|---------------|-----------|
| **Photos & videos** | `/var/lib/immich` | Varies (largest) | ✓✓✓ |
| **Immich config** | `~/immich/.env` | <1 KB | ✓✓ |
| **Docker config** | `~/immich/docker-compose.yml` | <10 KB | ✓✓ |
| **Database** | PostgreSQL container | 10-100 MB | ✓✓✓ |

**Check your current data size:**

```bash
du -sh /var/lib/immich
```

**Expected output:**
```
125G    /var/lib/immich
```

This tells you how big your external drive and cloud storage need to be.

💡 **Planning tip:** Buy an external drive at least 2x the size of your current photo library to allow for growth.

---

### 15.3 Set Up Local External Drive Backup

**What you need:**
- External USB hard drive (2-4TB recommended)
- USB cable to connect to your server

#### Step 1: Identify and Mount Your External Drive

**Connect the drive to your server, then identify it:**

```bash
lsblk
```

**Expected output:**
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 238.5G  0 disk
├─sda1   8:1    0   512M  0 part /boot/efi
├─sda2   8:2    0     2G  0 part [SWAP]
└─sda3   8:3    0   236G  0 part /
sdb      8:16   0   3.6T  0 disk            ← Your external drive
└─sdb1   8:17   0   3.6T  0 part
```

💡 Look for a drive with the size matching your external drive (e.g., 3.6T). It's usually `sdb` or `sdc`.

⚠ **Warning:** Make sure you identify the correct drive. Using the wrong device name will overwrite the wrong disk!

**Create mount point and mount the drive:**

```bash
sudo mkdir -p /mnt/backup
sudo mount /dev/sdb1 /mnt/backup
```

Replace `sdb1` with your actual drive partition.

**Verify it's mounted:**

```bash
df -h | grep backup
```

**Expected output:**
```
/dev/sdb1       3.6T   89M  3.4T   1% /mnt/backup
```

✓ **Success:** You see your backup drive listed with available space.

**Troubleshooting:**

| Problem | Cause | Solution |
|---------|-------|----------|
| `mount: wrong fs type` | Drive not formatted | Format: `sudo mkfs.ext4 /dev/sdb1` |
| `mount: already mounted` | Drive already mounted | Check `df -h` to see where |
| `Permission denied` | Need sudo | Add `sudo` before mount command |

#### Step 2: Create Backup Script

**Create the scripts directory if it doesn't exist:**

```bash
mkdir -p ~/immich/scripts
```

**Create the backup script:**

```bash
nano ~/immich/scripts/backup-immich.sh
```

**Paste this complete script:**

```bash
#!/bin/bash
# Immich Local Backup Script
# Backs up photos, config, and database to external drive

set -e  # Exit on error

BACKUP_DIR="/mnt/backup/immich-backups"
DATE=$(date +%Y-%m-%d_%H-%M-%S)
LOG_FILE="$HOME/immich/backup.log"

# Function to log messages
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Check if backup drive is mounted
if ! mountpoint -q /mnt/backup; then
    log "ERROR: Backup drive not mounted at /mnt/backup"
    exit 1
fi

log "=== Starting Immich backup ==="

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Backup photos and videos
log "Backing up photos and videos..."
rsync -av --delete \
    --exclude="thumbs/" \
    --exclude="encoded-video/" \
    /var/lib/immich/ "$BACKUP_DIR/photos/" 2>&1 | tee -a "$LOG_FILE"

# Backup configuration files
log "Backing up configuration..."
mkdir -p "$BACKUP_DIR/config"
cp ~/immich/.env "$BACKUP_DIR/config/.env"
cp ~/immich/docker-compose.yml "$BACKUP_DIR/config/docker-compose.yml"

# Backup database
log "Backing up database..."
cd ~/immich
docker compose exec -T postgres pg_dumpall -U postgres > "$BACKUP_DIR/database-$DATE.sql"

# Keep only last 7 database backups (saves space)
log "Cleaning old database backups..."
find "$BACKUP_DIR" -name "database-*.sql" -mtime +7 -delete

# Create backup success marker
echo "Last successful backup: $(date)" > "$BACKUP_DIR/last-backup.txt"

# Show backup summary
log "=== Backup completed successfully! ==="
log "Total backup size: $(du -sh $BACKUP_DIR | cut -f1)"
log "Available space: $(df -h /mnt/backup | tail -1 | awk '{print $4}')"
log "Database backups: $(ls -1 $BACKUP_DIR/database-*.sql 2>/dev/null | wc -l)"
```

**Save and exit:** Press `Ctrl+O`, `Enter`, then `Ctrl+X`

**Make the script executable:**

```bash
chmod +x ~/immich/scripts/backup-immich.sh
```

**Verify the script is executable:**

```bash
ls -lah ~/immich/scripts/backup-immich.sh
```

**Expected output:**
```
-rwxr-xr-x 1 youruser youruser 1.5K Jan 15 10:30 /home/youruser/immich/scripts/backup-immich.sh
```

✓ **Success:** The `x` in the permissions means it's executable.

#### Step 3: Test the Backup Script

**Run the backup manually:**

```bash
~/immich/scripts/backup-immich.sh
```

**Expected output:**
```
[2026-02-03 14:23:10] === Starting Immich backup ===
[2026-02-03 14:23:10] Backing up photos and videos...
sending incremental file list
./
library/
library/user1/
library/user1/2024/
library/user1/2024/IMG_1234.jpg
...
[2026-02-03 14:28:42] Backing up configuration...
[2026-02-03 14:28:42] Backing up database...
[2026-02-03 14:28:45] Cleaning old database backups...
[2026-02-03 14:28:45] === Backup completed successfully! ===
[2026-02-03 14:28:45] Total backup size: 127G
[2026-02-03 14:28:45] Available space: 3.3T
[2026-02-03 14:28:45] Database backups: 1
```

**Verify backup contents:**

```bash
ls -lh /mnt/backup/immich-backups/
```

**Expected output:**
```
total 128G
drwxr-xr-x 3 youruser youruser 4.0K Jan 15 14:28 photos
drwxr-xr-x 2 youruser youruser 4.0K Jan 15 14:28 config
-rw-r--r-- 1 youruser youruser 45M Jan 15 14:28 database-2026-02-03_14-28-42.sql
-rw-r--r-- 1 youruser youruser  58 Jan 15 14:28 last-backup.txt
```

✓ **Success criteria:**
- `photos/` directory exists and contains your photo library
- `config/` directory contains `.env` and `docker-compose.yml`
- `database-*.sql` file exists and is larger than 1MB
- No error messages in the output

**Check the log file:**

```bash
tail -20 ~/immich/backup.log
```

This shows the last 20 lines of your backup log.

#### Step 4: Automate Daily Backups with Cron

**Open your crontab:**

```bash
crontab -e
```

If prompted to choose an editor, select `nano` (usually option 1).

**Add this line at the end of the file:**

```bash
0 2 * * * /home/youruser/immich/scripts/backup-immich.sh >> /home/youruser/immich/backup.log 2>&1
```

**⚠ Important:** Replace `youruser` with your actual username.

**To find your username:**

```bash
whoami
```

**Cron schedule explanation:**
```
0 2 * * *  = Run at 2:00 AM every day
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7, both 0 and 7 = Sunday)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

**Save and exit:** Press `Ctrl+O`, `Enter`, then `Ctrl+X`

**Verify cron job was added:**

```bash
crontab -l
```

**Expected output:**
```
0 2 * * * /home/youruser/immich/scripts/backup-immich.sh >> /home/youruser/immich/backup.log 2>&1
```

✓ **Success:** Your backup will now run automatically at 2 AM every day.

💡 **Why 2 AM?** Most people aren't uploading photos at 2 AM, so backups run during low activity.

**Troubleshooting cron:**

| Problem | Solution |
|---------|----------|
| Backup didn't run | Check log: `tail ~/immich/backup.log` |
| Permission errors | Ensure drive is mounted before 2 AM |
| Drive not mounted | Set up auto-mount (see Step 5) |

#### Step 5: Auto-Mount External Drive on Boot

**Find the drive's UUID:**

```bash
sudo blkid | grep /dev/sdb1
```

**Expected output:**
```
/dev/sdb1: UUID="a1b2c3d4-e5f6-7890-abcd-ef1234567890" TYPE="ext4" PARTUUID="12345678-01"
```

Copy the UUID (the long string of numbers and letters).

**Edit fstab to auto-mount:**

```bash
sudo nano /etc/fstab
```

**Add this line at the end:**

```bash
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890 /mnt/backup ext4 defaults 0 2
```

Replace the UUID with your actual UUID from the blkid command.

**Save and exit:** Press `Ctrl+O`, `Enter`, then `Ctrl+X`

**Test the auto-mount configuration:**

```bash
sudo umount /mnt/backup
sudo mount -a
df -h | grep backup
```

**Expected output:**
```
/dev/sdb1       3.6T   127G  3.3T   4% /mnt/backup
```

✓ **Success:** Drive mounts automatically without errors.

---

### 15.4 Set Up Off-Site Cloud Backup (Recommended)

**Why cloud backup matters:**

Local backups protect against disk failure and accidental deletion. Cloud backups protect against:
- Fire, flood, or theft of your entire server
- Catastrophic local disaster
- Ransomware that encrypts both your server and local backup

**Cost comparison for 1TB of photos:**

| Provider | Monthly Cost | Notes |
|----------|--------------|-------|
| Backblaze B2 | ~$6 | Pay for storage only, cheap egress |
| Wasabi | ~$7 | Minimum 1TB, no egress fees |
| Google Drive | Included | With Google One 2TB plan ($10/month) |
| AWS S3 Glacier | ~$4 | Slow retrieval, complex pricing |

💡 **Recommendation:** Start with Backblaze B2 (easiest setup, good pricing).

#### Step 1: Install Rclone

```bash
sudo apt update
sudo apt install -y rclone
```

**Verify installation:**

```bash
rclone version
```

**Expected output:**
```
rclone v1.64.0
- os/version: ubuntu 25.10 (64 bit)
- os/kernel: 6.11.0-9-generic (x86_64)
- os/type: linux
```

✓ **Success:** Rclone is installed and shows version information.

#### Step 2: Create Backblaze B2 Account and Bucket

**Sign up for Backblaze B2:**
1. Go to https://www.backblaze.com/b2/sign-up.html
2. Create a free account (10 GB free, then pay-as-you-go)
3. Log in to your account

**Create a bucket:**
1. Click "Buckets" in left sidebar
2. Click "Create a Bucket"
3. Bucket name: `immich-backup` (must be globally unique, add numbers if taken)
4. Files in bucket: **Private**
5. Default encryption: **Enabled**
6. Object lock: **Disabled**
7. Click "Create a Bucket"

**Create application key:**
1. Go to "App Keys" in left sidebar
2. Click "Add a New Application Key"
3. Name: `immich-rclone`
4. Allow access to bucket: Select `immich-backup`
5. Type of access: **Read and Write**
6. Click "Create New Key"
7. **⚠ Important:** Copy the `keyID` and `applicationKey` immediately - you can't view them again!

**Document your credentials:**

| Field | Value |
|-------|-------|
| Bucket name | `immich-backup` (or your chosen name) |
| Key ID | `002abc...` (copy from Backblaze) |
| Application Key | `K002def...` (copy from Backblaze) |

#### Step 3: Configure Rclone for Backblaze B2

**Run rclone config:**

```bash
rclone config
```

**Follow these prompts:**

```
n) New remote
name> b2backup
Storage> 5
  (Choose "Backblaze B2")
account> [Paste your Key ID]
key> [Paste your Application Key]
endpoint> [Press Enter for default]
Edit advanced config? n
Keep this "b2backup" remote? y
q) Quit config
```

**Test the connection:**

```bash
rclone lsd b2backup:
```

**Expected output:**
```
     -1 2026-02-03 14:45:00        -1 immich-backup
```

✓ **Success:** You can see your bucket listed.

#### Step 4: Create Cloud Backup Script

```bash
nano ~/immich/scripts/backup-immich-cloud.sh
```

**Paste this script:**

```bash
#!/bin/bash
# Immich Cloud Backup Script
# Syncs photos to Backblaze B2 (or other rclone remote)

set -e

BACKUP_DIR="/var/lib/immich"
CLOUD_REMOTE="b2backup:immich-backup"
LOG_FILE="$HOME/immich/cloud-backup.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

log "=== Starting cloud backup ==="

# Sync photos to cloud (excluding thumbnails and transcoded videos to save space)
log "Syncing photos to $CLOUD_REMOTE..."
rclone sync "$BACKUP_DIR" "$CLOUD_REMOTE/photos" \
    --progress \
    --exclude "thumbs/**" \
    --exclude "encoded-video/**" \
    --transfers 8 \
    --checkers 8 \
    --log-file "$LOG_FILE" \
    --log-level INFO

# Backup config files
log "Backing up config files..."
rclone copy ~/immich/.env "$CLOUD_REMOTE/config/" --log-file "$LOG_FILE"
rclone copy ~/immich/docker-compose.yml "$CLOUD_REMOTE/config/" --log-file "$LOG_FILE"

# Backup latest database dump from local backup
LATEST_DB=$(ls -t /mnt/backup/immich-backups/database-*.sql 2>/dev/null | head -1)
if [ -n "$LATEST_DB" ]; then
    log "Backing up database: $LATEST_DB"
    rclone copy "$LATEST_DB" "$CLOUD_REMOTE/database/" --log-file "$LOG_FILE"
fi

log "=== Cloud backup completed ==="
log "Remote size: $(rclone size $CLOUD_REMOTE 2>&1 | grep 'Total size')"
```

**Make executable:**

```bash
chmod +x ~/immich/scripts/backup-immich-cloud.sh
```

#### Step 5: Test Cloud Backup

**Run a test backup:**

```bash
~/immich/scripts/backup-immich-cloud.sh
```

**Expected output:**
```
[2026-02-03 15:10:00] === Starting cloud backup ===
[2026-02-03 15:10:00] Syncing photos to b2backup:immich-backup...
Transferred:   	  5.123 GiB / 125.456 GiB, 4%, 12.5 MiB/s, ETA 3h15m
...
[2026-02-03 18:25:00] Backing up config files...
[2026-02-03 18:25:05] Backing up database...
[2026-02-03 18:25:10] === Cloud backup completed ===
```

⚠ **Note:** First cloud backup takes hours depending on your library size and upload speed. Subsequent backups are much faster (only changed files).

**Verify files in cloud:**

```bash
rclone ls b2backup:immich-backup/photos | head -10
```

**Expected output:**
```
 2456789 library/user1/2024/IMG_1234.jpg
 3567890 library/user1/2024/IMG_1235.jpg
...
```

✓ **Success:** Your photos are visible in cloud storage.

#### Step 6: Automate Weekly Cloud Backups

**Add to crontab:**

```bash
crontab -e
```

**Add this line:**

```bash
0 3 * * 0 /home/youruser/immich/scripts/backup-immich-cloud.sh >> /home/youruser/immich/cloud-backup.log 2>&1
```

Replace `youruser` with your username.

**Schedule explanation:**
```
0 3 * * 0 = Run at 3:00 AM every Sunday
        │
        └─── 0 = Sunday (run once per week)
```

💡 **Why weekly?** Cloud backups are slower and cost bandwidth. Weekly is sufficient for most users.

**Save and verify:**

```bash
crontab -l
```

**Expected output:**
```
0 2 * * * /home/youruser/immich/scripts/backup-immich.sh >> /home/youruser/immich/backup.log 2>&1
0 3 * * 0 /home/youruser/immich/scripts/backup-immich-cloud.sh >> /home/youruser/immich/cloud-backup.log 2>&1
```

✓ **Success:** Both backup jobs are scheduled.

---

### 15.5 Test Backup Restoration

**⚠ Critical:** A backup you haven't tested is not a real backup.

Test your backups quarterly (every 3 months) to ensure they actually work.

#### Test 1: Verify Database Backup

**Check database file exists and has content:**

```bash
ls -lh /mnt/backup/immich-backups/database-*.sql | tail -1
```

**Expected output:**
```
-rw-r--r-- 1 youruser youruser 45M Feb  3 02:15 /mnt/backup/immich-backups/database-2026-02-03_02-15-30.sql
```

✓ File should be at least 1 MB (larger if you have many photos).

**Preview database contents (first 20 lines):**

```bash
head -20 /mnt/backup/immich-backups/database-$(ls /mnt/backup/immich-backups/database-*.sql | tail -1 | xargs basename)
```

**Expected output:**
```
--
-- PostgreSQL database cluster dump
--

SET default_transaction_read_only = off;

SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
...
```

✓ **Success:** Database dump contains SQL commands.

#### Test 2: Dry-Run Photo Restoration

**Simulate restoring photos (doesn't actually copy):**

```bash
rsync -avn --delete /mnt/backup/immich-backups/photos/ /var/lib/immich/
```

**Expected output:**
```
sending incremental file list
./

sent 1,234,567 bytes  received 890 bytes  246,891.40 bytes/sec
total size is 134,567,890,123  speedup is 108,901.23 (DRY RUN)
```

✓ **Success:** If it shows "DRY RUN" and completes without errors, your files are restorable.

💡 **The `-n` flag** means "dry run" - it doesn't actually copy anything, just shows what would happen.

#### Test 3: Verify Cloud Backup Integrity

**Check cloud backup size matches local:**

```bash
rclone size b2backup:immich-backup/photos
```

**Expected output:**
```
Total objects: 12,345
Total size: 125.456 GiB (134,567,890,123 bytes)
```

**Compare with local size:**

```bash
du -sb /var/lib/immich | awk '{print $1 " bytes"}'
```

**Expected output:**
```
134567890123 bytes
```

✓ **Success:** Cloud and local sizes should be within 1% of each other (small differences are normal due to excluded files).

---

### 15.6 Document Your Backup System

**Create a backup documentation file:**

```bash
nano ~/immich/BACKUP-INFO.md
```

**Paste this template and fill in your details:**

```markdown
# Immich Backup Documentation

## Backup Schedule

| Backup Type | Frequency | Time | Destination |
|-------------|-----------|------|-------------|
| Local | Daily | 2:00 AM | `/mnt/backup/immich-backups` |
| Cloud | Weekly | 3:00 AM Sunday | Backblaze B2 bucket: `immich-backup` |

## Backup Contents

- ✓ Photos and videos: `/var/lib/immich`
- ✓ Configuration: `~/immich/.env` and `docker-compose.yml`
- ✓ Database: PostgreSQL dumps (last 7 days retained)

## External Drive Details

- **Make/Model:** [e.g., Seagate Backup Plus 4TB]
- **Capacity:** [e.g., 4TB]
- **Mount point:** `/mnt/backup`
- **Device:** `/dev/sdb1`
- **UUID:** [paste from `sudo blkid`]
- **Purchase date:** [e.g., Feb 2026]

## Cloud Backup Details

- **Provider:** Backblaze B2
- **Bucket name:** immich-backup
- **Rclone remote:** b2backup
- **Account email:** [your email]
- **Monthly cost (approx):** $6-12

## Restoration Commands

### Restore from local backup:

```bash
# Stop Immich
cd ~/immich
docker compose down

# Restore photos
sudo rsync -av /mnt/backup/immich-backups/photos/ /var/lib/immich/

# Restore config
cp /mnt/backup/immich-backups/config/.env ~/immich/
cp /mnt/backup/immich-backups/config/docker-compose.yml ~/immich/

# Restore database
docker compose up -d postgres
docker compose exec -T postgres psql -U postgres < /mnt/backup/immich-backups/database-[latest].sql

# Start Immich
docker compose up -d
```

### Restore from cloud:

```bash
# Restore photos from cloud
rclone sync b2backup:immich-backup/photos /var/lib/immich

# Restore config
rclone copy b2backup:immich-backup/config/.env ~/immich/
rclone copy b2backup:immich-backup/config/docker-compose.yml ~/immich/
```

## Last Tested

- **Local backup:** [date you tested]
- **Cloud backup:** [date you tested]
- **Restoration:** [date you tested]

## Emergency Contacts

If I'm incapacitated, this information helps family recover photos:

- Server location: [e.g., "Office closet, bottom shelf"]
- External drive location: [e.g., "Connected to server"]
- Cloud login: [e.g., "Backblaze account under my email"]
- Password manager: [e.g., "1Password family account"]
```

**Save:** Press `Ctrl+O`, `Enter`, `Ctrl+X`

💡 **Why document this?** Six months from now, you'll forget the details. This file helps you (and family) recover from disasters.

---

### ✓ Checkpoint: Backup Strategy Complete

At this point, you should have:

**Local backups:**
- ✓ External drive mounted at `/mnt/backup`
- ✓ Backup script at `~/immich/scripts/backup-immich.sh`
- ✓ Daily automatic backups at 2 AM
- ✓ Drive auto-mounts on server reboot

**Cloud backups (if configured):**
- ✓ Rclone installed and configured
- ✓ Backblaze B2 account and bucket created
- ✓ Cloud backup script at `~/immich/scripts/backup-immich-cloud.sh`
- ✓ Weekly automatic cloud backups

**Verification:**
- ✓ Tested backup scripts manually
- ✓ Verified backup contents exist
- ✓ Documented backup system
- ✓ Scheduled quarterly restoration tests

**Quick test commands:**

```bash
# Check last local backup
ls -lh /mnt/backup/immich-backups/last-backup.txt
cat /mnt/backup/immich-backups/last-backup.txt

# Check backup logs
tail -20 ~/immich/backup.log

# Check cron schedule
crontab -l

# Check disk space
df -h /mnt/backup
```

### 💡 What You Learned

- **3-2-1 backup rule:** Industry standard for data protection
- **rsync:** Efficient file synchronization tool
- **cron:** Linux task scheduler for automation
- **rclone:** Universal cloud storage tool
- **PostgreSQL dumps:** Database backup method
- **/etc/fstab:** Auto-mount configuration
- **Dry-run testing:** Safe way to test restoration

### ⚠ Important Reminders

1. **Test your backups quarterly** - Set a calendar reminder
2. **Monitor backup logs** - Check for errors occasionally
3. **Update documentation** - When you change backup settings
4. **Keep external drive connected** - Or backups can't run
5. **Monitor cloud costs** - Check Backblaze billing monthly

**Next:** Section 16 will cover optional Google OAuth integration for easier login.

---

## 16. Optional: Google OAuth Integration

### 📋 What You'll Accomplish

By the end of this section, you'll have:
- ✓ Google OAuth configured for Immich login
- ✓ Password-free sign-in for family members
- ✓ Secure authentication via Google accounts
- ✓ Automatic user account creation (optional)

### ⏱ Time Required

- **Creating Google Cloud project:** 10 minutes
- **Configuring OAuth:** 15 minutes
- **Testing and troubleshooting:** 10 minutes
- **Total:** 35 minutes

### 💰 Cost Estimate

- **Free** (Google Cloud OAuth is free for personal use)
- Testing mode supports up to 100 users at no cost

### Prerequisites

Before starting this section, you should have:
- ✓ Immich accessible via HTTPS with a domain name (Section 10)
- ✓ Admin access to your Immich instance
- ✓ A Google account
- ✓ Your Immich domain name (e.g., `photos.yourdomain.com`)

---

### Should You Use OAuth?

**Benefits:**
- ✓ No password management for family members
- ✓ More secure than traditional passwords
- ✓ Easy "Sign in with Google" button
- ✓ Familiar login experience
- ✓ Automatic password resets handled by Google

**Limitations:**
- ⚠ Requires domain with HTTPS (can't use IP address)
- ⚠ "Testing mode" limited to 100 users (plenty for families)
- ⚠ "Production mode" requires Google verification process (complex, not needed for home use)
- ⚠ Users need Google accounts
- ⚠ Dependency on Google's services

**Decision guide:**

| Your situation | Recommendation |
|----------------|----------------|
| Just me using Immich | **Skip** - regular password is fine |
| Sharing with 2-10 family members | **Use OAuth** - easier for everyone |
| Family members aren't tech-savvy | **Use OAuth** - less confusion |
| Don't have HTTPS set up | **Skip** - required for OAuth |
| Privacy-focused, avoid Google | **Skip** - keep everything self-hosted |

---

### 16.1 Create Google Cloud Project

#### Step 1: Access Google Cloud Console

**Navigate to Google Cloud Console:**

Open your browser and go to: https://console.cloud.google.com/

**Sign in with your Google account.**

#### Step 2: Create New Project

**Click the project dropdown** (top left, next to "Google Cloud").

**Click "New Project".**

**Fill in project details:**

| Field | Value |
|-------|-------|
| Project name | `Immich Photos` |
| Organization | `No organization` |
| Location | `No organization` |

**Click "Create".**

**Wait for project creation** (takes 10-30 seconds).

**Expected notification:**
```
Project "Immich Photos" has been created
```

✓ **Success:** Your project appears in the project dropdown.

**Select your new project** from the dropdown to make it active.

---

### 16.2 Configure OAuth Consent Screen

The consent screen is what users see when they click "Sign in with Google."

#### Step 1: Navigate to OAuth Consent Screen

**In the left sidebar:**
1. Click "APIs & Services"
2. Click "OAuth consent screen"

**Or use direct link:** https://console.cloud.google.com/apis/credentials/consent

#### Step 2: Choose User Type

**Select "External"** (this allows any Google account to sign in).

**Click "Create".**

💡 **Why External?** "Internal" requires a Google Workspace organization (paid). External works for personal Gmail accounts.

#### Step 3: Fill in App Information

**On the "OAuth consent screen" page, fill in:**

| Field | Your Value | Example |
|-------|------------|---------|
| App name | Immich Photos | `Immich Photos` |
| User support email | Your email | `yourname@gmail.com` |
| App logo | (optional, skip) | - |
| Application home page | (optional) | `https://photos.yourdomain.com` |
| Application privacy policy | (optional, skip) | - |
| Application terms of service | (optional, skip) | - |
| Authorized domains | Your domain | `yourdomain.com` |
| Developer contact email | Your email | `yourname@gmail.com` |

**Click "Save and Continue".**

#### Step 4: Configure Scopes

**On the "Scopes" page:**

**Click "Save and Continue"** (no need to add scopes - Immich handles this).

💡 **Note:** Immich will request `openid`, `email`, and `profile` scopes automatically.

#### Step 5: Add Test Users

**On the "Test users" page:**

**Click "+ Add Users".**

**Add email addresses** of people who will use Immich (including yourself):

```
yourname@gmail.com
spouse@gmail.com
kid1@gmail.com
```

**Click "Add".**

**Click "Save and Continue".**

⚠ **Important:** In Testing mode, ONLY these email addresses can log in. Add everyone who needs access.

#### Step 6: Review and Finish

**Review the summary page.**

**Click "Back to Dashboard".**

**Verify the consent screen status:**

Expected status:
```
Publishing status: Testing
User type: External
```

✓ **Success:** OAuth consent screen is configured.

💡 **Testing vs Production:** Testing mode is perfect for home use. Production mode requires Google verification (weeks of waiting, detailed review). Stick with Testing mode.

---

### 16.3 Create OAuth Credentials

#### Step 1: Navigate to Credentials

**In the left sidebar:**
1. Click "APIs & Services"
2. Click "Credentials"

**Or direct link:** https://console.cloud.google.com/apis/credentials

#### Step 2: Create OAuth Client ID

**Click "+ Create Credentials"** (top of page).

**Select "OAuth client ID".**

**If prompted about consent screen configuration:**
- This means you haven't completed section 16.2
- Go back and finish the consent screen setup first

#### Step 3: Configure Application Type

**Choose application type:** `Web application`

**Name:** `Immich Web Client`

#### Step 4: Set Authorized Redirect URIs

This is the most critical step. Get this wrong and OAuth won't work.

**Under "Authorized redirect URIs":**

**Click "+ Add URI".**

**Enter your callback URL exactly:**

```
https://photos.yourdomain.com/api/oauth/callback/google
```

**⚠ Important formatting rules:**
- Must start with `https://` (not `http://`)
- Must end with `/api/oauth/callback/google`
- No trailing slash
- Use your actual domain name
- Must match exactly how you access Immich

**Examples:**

| Your Immich URL | Redirect URI |
|-----------------|--------------|
| `https://photos.example.com` | `https://photos.example.com/api/oauth/callback/google` |
| `https://immich.mydomain.net` | `https://immich.mydomain.net/api/oauth/callback/google` |
| `https://192.168.1.50` | ❌ **Won't work** (OAuth requires domain) |

**Click "Create".**

#### Step 5: Save Your Credentials

**A popup appears with your credentials:**

| Credential | Example | What to do |
|------------|---------|------------|
| Client ID | `123456789-abc...googleusercontent.com` | Copy and save |
| Client Secret | `GOCSPX-abc123...` | Copy and save |

**⚠ Critical:** Copy these NOW. You can view Client ID later, but Client Secret is only shown once.

**Save credentials to a temporary file:**

```bash
nano ~/oauth-credentials.txt
```

Paste:
```
Client ID: [paste here]
Client Secret: [paste here]
```

Save with `Ctrl+O`, `Enter`, `Ctrl+X`.

**Click "OK"** to close the popup.

✓ **Success:** Your credentials are created and saved.

---

### 16.4 Configure Immich for OAuth

#### Step 1: Log in to Immich Admin

**Open Immich in your browser:**

```
https://photos.yourdomain.com
```

**Log in with your admin account** (the one you created during initial setup).

#### Step 2: Navigate to Authentication Settings

**Click the gear icon** (top right) to open Administration.

**In the left sidebar, click "Settings".**

**Scroll down to "OAuth Authentication"** section.

#### Step 3: Enable and Configure OAuth

**Toggle "Enable OAuth" to ON.**

**Select "Google" as the OAuth provider** (if there's a dropdown).

**Fill in the configuration:**

| Field | Value | Where to find it |
|-------|-------|------------------|
| **Issuer URL** | `https://accounts.google.com` | (exact text) |
| **Client ID** | `123456...googleusercontent.com` | From step 16.3.5 |
| **Client Secret** | `GOCSPX-abc123...` | From step 16.3.5 |
| **Scope** | `openid email profile` | (exact text) |
| **Button Text** | `Sign in with Google` | (optional, default is fine) |
| **Auto Register** | Toggle as desired | See decision below |

**Auto Register decision:**

| Setting | Behavior | Use when |
|---------|----------|----------|
| **ON** | Any Google user in your test users list is automatically created | Family members can self-onboard |
| **OFF** | You must manually create user accounts in Immich first | You want tight control over users |

💡 **Recommendation:** Turn ON for family use (less admin work).

**Click "Save"** (bottom of settings page).

**Expected confirmation:**
```
Settings saved successfully
```

✓ **Success:** OAuth is now configured.

#### Step 4: Verify OAuth Configuration

**Log out of Immich:**
- Click your profile icon (top right)
- Click "Sign Out"

**On the login page, you should now see:**
```
[Sign in with Google button]
```

✓ **Success:** The OAuth button appears.

**Troubleshooting:**

| Problem | Cause | Solution |
|---------|-------|----------|
| No OAuth button | OAuth not enabled | Check Settings → OAuth → Enable toggle |
| Button shows but doesn't work | Wrong redirect URI | Double-check redirect URI in Google Cloud Console |
| "Redirect URI mismatch" error | Domain doesn't match | Ensure domain in redirect URI exactly matches how you access Immich |

---

### 16.5 Test OAuth Login

#### Step 1: Sign In with Google

**On the Immich login page:**

**Click "Sign in with Google".**

**Google's OAuth consent screen appears:**

**Expected screen:**
```
Sign in with Google

Choose an account to continue to Immich Photos

[your email]
[other email]
```

**Select your Google account** (must be one you added as a test user).

#### Step 2: Grant Permissions

**If this is your first time, Google asks for permissions:**

```
Immich Photos wants to:
- Know who you are on Google
- See your email address
- See your personal info, including any personal info you've made publicly available

[Cancel] [Continue]
```

**Click "Continue".**

💡 **What these permissions mean:**
- **Know who you are:** Get your Google user ID
- **Email address:** Used as username in Immich
- **Personal info:** Gets your name and profile picture

These are minimal permissions - Immich cannot access your Gmail, Drive, or other Google services.

#### Step 3: Verify Login Success

**After granting permissions, you should:**
1. Be redirected back to Immich
2. Automatically logged in
3. See your Immich library

**Check your user info:**
- Click profile icon (top right)
- Your email should match your Google account
- Your name should match your Google profile

✓ **Success criteria:**
- Logged in without entering password
- Email matches Google account
- Can access Immich features normally

**Troubleshooting:**

| Error Message | Cause | Solution |
|---------------|-------|----------|
| "Error 400: redirect_uri_mismatch" | Redirect URI doesn't match | Fix redirect URI in Google Cloud Console |
| "Error 403: access_denied" | User not in test users list | Add user email to test users (Section 16.2.5) |
| "Invalid credentials" | Wrong Client ID/Secret | Double-check credentials in Immich settings |
| "User not found" | Auto Register is OFF | Enable Auto Register or manually create user |

#### Step 4: Test with Another User (Optional)

**Add a family member's email to test users:**
1. Go back to Google Cloud Console
2. APIs & Services → OAuth consent screen
3. Scroll to "Test users"
4. Click "Add Users"
5. Add their email
6. Click "Add"

**Have them test login:**
1. They go to `https://photos.yourdomain.com`
2. Click "Sign in with Google"
3. Choose their Google account
4. Grant permissions
5. Should be logged in

If Auto Register is ON, their account is created automatically.

---

### 16.6 Manage OAuth Users

#### View OAuth Users

**In Immich Admin:**
1. Click gear icon → "Users"
2. Look for users with "OAuth" indicator

**OAuth users show:**
- Email from Google account
- Name from Google profile
- Profile picture from Google (optional)

#### Manually Create Users for OAuth (if Auto Register is OFF)

**To allow a specific Google user to log in:**

1. Go to Administration → Users
2. Click "+ Create User"
3. Fill in:
   - **Email:** Must exactly match their Google account email
   - **Name:** Their display name
   - **Password:** (leave blank - not used for OAuth)
4. Click "Create"

Now when they sign in with Google, they'll link to this user account.

#### Remove OAuth Access

**To revoke someone's access:**

1. Administration → Users
2. Find the user
3. Click "Delete" or "Disable"

**They'll also need to remove access in their Google account:**
1. Go to https://myaccount.google.com/permissions
2. Find "Immich Photos"
3. Click "Remove access"

---

### 16.7 Optional: Configure Additional OAuth Settings

#### Change Button Text

**In Immich Administration → Settings → OAuth:**

**Button Text field:** Change "Sign in with Google" to:
- "Family Photos Login"
- "Use Google Account"
- Whatever text makes sense for your users

#### Enable Auto-Launch

**Auto Launch setting:** When enabled, clicking login automatically redirects to Google (skips the login page).

**Use when:**
- You ONLY use OAuth (no password logins)
- Family members find the extra click confusing

**Don't use when:**
- You still use password login for admin
- You want flexibility in login methods

#### Configure Profile Picture Sync

**In Immich settings:**

**Sync Profile Picture:** When enabled, Immich fetches profile pictures from Google accounts.

**Benefit:** Users see familiar profile pictures.

---

### ✓ Checkpoint: Google OAuth Integration Complete

At this point, you should have:

**Google Cloud Platform:**
- ✓ Project created (`Immich Photos`)
- ✓ OAuth consent screen configured
- ✓ Test users added
- ✓ OAuth credentials created (Client ID and Secret)

**Immich Configuration:**
- ✓ OAuth enabled in Immich settings
- ✓ Google OAuth configured with credentials
- ✓ Auto Register enabled/disabled as desired
- ✓ OAuth button visible on login page

**Testing:**
- ✓ Successfully logged in with Google account
- ✓ User account created or linked
- ✓ Additional family members can log in

**Quick verification commands:**

```bash
# View your OAuth credentials (saved earlier)
cat ~/oauth-credentials.txt

# Check Immich logs for OAuth activity
cd ~/immich
docker compose logs immich-server | grep -i oauth

# Clean up credentials file (optional, after setup complete)
rm ~/oauth-credentials.txt
```

### 💡 What You Learned

- **OAuth 2.0:** Modern authentication protocol
- **Google Cloud Console:** Managing Google developer projects
- **Consent screens:** What users see during OAuth login
- **Redirect URIs:** How OAuth securely returns users to your app
- **Test mode vs Production:** Google OAuth publication statuses
- **Auto registration:** Automatic user provisioning

### ⚠ Important Reminders

1. **Test users limit:** Only 100 users in Testing mode (plenty for families)
2. **Add new users:** Remember to add their email to test users list
3. **HTTPS required:** OAuth won't work without valid SSL certificate
4. **Backup credentials:** Save Client ID and Secret somewhere safe
5. **Domain changes:** If you change domains, update redirect URI in Google Cloud Console

### 🔒 Security Notes

- OAuth is generally MORE secure than passwords (no password to steal)
- Google handles password security, 2FA, breach detection
- Revoke access anytime from Google account settings
- Immich never sees your Google password
- Users can't accidentally use weak passwords

**Next:** Section 17 will cover ongoing maintenance and updates to keep your Immich instance running smoothly.

---

## 17. Maintenance and Updates

### 📋 What You'll Accomplish

By the end of this section, you'll have:
- ✓ Regular update schedule established
- ✓ Automatic security updates configured
- ✓ Monitoring tools and commands ready
- ✓ Troubleshooting skills for common issues
- ✓ Maintenance checklist to follow

### ⏱ Time Required

- **Initial setup:** 20 minutes
- **Monthly maintenance:** 15 minutes
- **Quarterly review:** 30 minutes

### Prerequisites

Before starting this section, you should have:
- ✓ Fully functional Immich installation
- ✓ Convenience scripts from Section 13 (optional but helpful)
- ✓ Backup system from Section 15 (recommended)

---

### 17.1 Update Immich (Monthly)

Immich is actively developed with new features, bug fixes, and security patches. Update monthly to stay current.

#### Check Current Version

**View your current Immich version:**

```bash
cd ~/immich
docker compose exec immich-server cat /usr/src/app/package.json | grep version
```

**Expected output:**
```
  "version": "1.95.1",
```

**Or check in the web interface:**
1. Click gear icon → "Settings"
2. Scroll to bottom
3. Look for "Server Version"

#### Manual Update Process

**Step 1: Pull latest Docker images:**

```bash
cd ~/immich
docker compose pull
```

**Expected output:**
```
[+] Pulling 8/8
 ✔ immich-server Pulled
 ✔ immich-machine-learning Pulled
 ✔ immich-microservices Pulled
 ✔ redis Pulled
 ✔ database Already exists
```

This downloads the latest Immich version but doesn't apply it yet.

**Step 2: Stop and restart containers:**

```bash
docker compose down
docker compose up -d
```

**Expected output:**
```
[+] Running 8/8
 ✔ Container immich-redis      Started
 ✔ Container immich-postgres   Started
 ✔ Container immich-server     Started
 ✔ Container immich-microservices  Started
 ✔ Container immich-machine-learning Started
```

**Step 3: Clean up old images:**

```bash
docker image prune -f
```

**Expected output:**
```
Deleted Images:
untagged: ghcr.io/immich-app/immich-server@sha256:abc123...
Total reclaimed space: 1.2GB
```

This removes old Docker images to free disk space.

✓ **Success criteria:**
- No error messages during pull or up
- Can access Immich web interface
- New version shows in Settings

#### Using the Convenience Script

**If you created the update script in Section 13:**

```bash
~/immich/scripts/immich-update.sh
```

This runs all three commands above automatically.

**Troubleshooting updates:**

| Problem | Cause | Solution |
|---------|-------|----------|
| `pull` fails with network error | Internet connection issue | Check connection: `ping 8.8.8.8` |
| `up` fails with port conflict | Another service using port 2283 | Check: `sudo lsof -i :2283` |
| Can't access Immich after update | Container failed to start | Check logs: `docker compose logs immich-server` |
| Database errors after update | Database migration failed | Restore from backup, investigate logs |

---

### 17.2 Update Ubuntu System (Monthly)

Security patches are released regularly. Keep your server up-to-date.

#### Manual System Update

**Update package lists and upgrade:**

```bash
sudo apt update
sudo apt upgrade -y
```

**Expected output:**
```
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
...
Reading package lists... Done
Building dependency tree... Done
The following packages will be upgraded:
  linux-headers-generic linux-image-generic openssh-server
3 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
...
```

**Remove old packages:**

```bash
sudo apt autoremove -y
```

**If kernel was updated, reboot:**

```bash
sudo reboot
```

⚠ **Note:** Reboot is only needed if kernel (`linux-image-*`) or systemd packages were updated.

**Check if reboot is needed:**

```bash
[ -f /var/run/reboot-required ] && echo "Reboot required" || echo "No reboot needed"
```

**Expected output:**
```
No reboot needed
```

Or:

```
Reboot required
```

#### Set Up Automatic Security Updates

**Install unattended-upgrades package:**

```bash
sudo apt install -y unattended-upgrades
```

**Configure automatic updates:**

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

**A dialog appears:**
```
Automatically download and install stable updates?
<Yes>  <No>
```

**Press `Tab` to select "Yes", then `Enter`.**

**Verify configuration:**

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

**Expected output:**
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

✓ **Success:** Security updates will install automatically daily.

**What gets auto-updated:**
- ✓ Security patches
- ✓ Critical bug fixes
- ✗ Major version upgrades (requires manual update)

💡 **Benefit:** Your server stays patched even if you forget to update manually.

**Check auto-update logs:**

```bash
sudo tail -50 /var/log/unattended-upgrades/unattended-upgrades.log
```

---

### 17.3 Monitor Disk Space

Running out of disk space can crash Immich. Monitor regularly.

#### Check Overall Disk Usage

```bash
df -h
```

**Expected output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3       236G  145G   79G  65% /
/dev/sdb1       3.6T  127G  3.3T   4% /mnt/backup
tmpfs           7.8G     0  7.8G   0% /dev/shm
```

**What to watch:**
- **Root filesystem (/):** Should stay below 80%
- **Backup drive (/mnt/backup):** Should have 2x your photo library size free

⚠ **Warning levels:**
- **80-90% full:** Start investigating
- **90-95% full:** Clean up soon
- **95%+ full:** Critical, clean up immediately

#### Check Immich Storage Usage

```bash
sudo du -sh /var/lib/immich
```

**Expected output:**
```
127G    /var/lib/immich
```

**Break down by subdirectory:**

```bash
sudo du -h /var/lib/immich | sort -rh | head -10
```

**Expected output:**
```
127G    /var/lib/immich
89G     /var/lib/immich/library
24G     /var/lib/immich/upload
8.2G    /var/lib/immich/thumbs
4.1G    /var/lib/immich/encoded-video
...
```

**What these directories are:**
- `library/` - Original uploaded photos and videos (largest)
- `upload/` - Temporary upload staging area
- `thumbs/` - Generated thumbnail images
- `encoded-video/` - Transcoded video files
- `profile/` - User profile pictures

#### Clean Up Docker Resources

**Remove unused Docker images:**

```bash
docker image prune -a -f
```

**Remove unused Docker volumes (⚠ careful):**

```bash
docker volume prune -f
```

**Remove all stopped containers:**

```bash
docker container prune -f
```

**Check Docker disk usage:**

```bash
docker system df
```

**Expected output:**
```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          8         5         4.2GB     1.8GB (42%)
Containers      5         5         125MB     0B (0%)
Local Volumes   3         3         127GB     0B (0%)
Build Cache     0         0         0B        0B
```

**Reclaim space from all unused Docker resources:**

```bash
docker system prune -a -f
```

⚠ **Warning:** This removes ALL unused images. Harmless but you'll re-download them on next `docker compose pull`.

---

### 17.4 Monitor Immich Health

#### Check Container Status

**View running containers:**

```bash
cd ~/immich
docker compose ps
```

**Expected output:**
```
NAME                        STATUS          PORTS
immich-machine-learning     Up 5 days       0.0.0.0:3003->3003/tcp
immich-microservices        Up 5 days
immich-postgres             Up 5 days       0.0.0.0:5432->5432/tcp
immich-redis                Up 5 days       0.0.0.0:6379->6379/tcp
immich-server               Up 5 days       0.0.0.0:2283->3001/tcp
```

✓ **All containers should show "Up"** (with varying uptimes).

**Troubleshooting:**

| Status | Meaning | Action |
|--------|---------|--------|
| `Up 5 days` | Running normally | ✓ Good |
| `Up 3 seconds (health: starting)` | Just started | Wait 30 seconds, check again |
| `Restarting` | Crash loop | Check logs: `docker compose logs` |
| `Exited (1)` | Crashed | Check logs, restart: `docker compose up -d` |

#### Check Immich Logs

**View live logs from all containers:**

```bash
cd ~/immich
docker compose logs -f
```

Press `Ctrl+C` to stop following.

**View logs from specific container:**

```bash
docker compose logs -f immich-server
```

**View last 100 lines:**

```bash
docker compose logs --tail=100 immich-server
```

**Search logs for errors:**

```bash
docker compose logs immich-server | grep -i error
```

**Expected normal output:**
```
immich-server | [Nest] 1  - 02/03/2026, 2:30:15 PM     LOG [Api] Immich Server is listening on http://0.0.0.0:3001
immich-server | [Nest] 1  - 02/03/2026, 2:30:15 PM     LOG [Microservices] Immich Microservices is listening
```

⚠ **Red flags in logs:**
- `ERROR` messages (indicate problems)
- `FATAL` messages (critical issues)
- Stack traces (code crashes)
- Database connection errors
- Out of memory errors

**Using the convenience script:**

```bash
~/immich/scripts/immich-logs.sh
```

#### Check Immich Web Interface

**Health check via web:**

```bash
curl -I http://localhost:2283/api/server-info/ping
```

**Expected output:**
```
HTTP/1.1 200 OK
Content-Type: application/json
```

✓ **200 OK** means Immich is responding.

**Check from external access (if configured):**

```bash
curl -I https://photos.yourdomain.com/api/server-info/ping
```

Should also return `200 OK`.

---

### 17.5 Common Troubleshooting

#### Problem: Immich Web Interface Not Loading

**Symptoms:**
- Browser shows "Site can't be reached"
- Connection timeout
- Blank page

**Diagnosis steps:**

1. **Check if containers are running:**
   ```bash
   cd ~/immich
   docker compose ps
   ```

2. **Check if Immich port is listening:**
   ```bash
   sudo lsof -i :2283
   ```
   Expected: Shows `docker-pr` process.

3. **Check Nginx (if configured):**
   ```bash
   sudo systemctl status nginx
   ```
   Expected: `active (running)`

4. **Check firewall:**
   ```bash
   sudo ufw status
   ```
   Expected: Port 80 and 443 ALLOW.

**Solutions:**

| Cause | Fix |
|-------|-----|
| Containers stopped | `cd ~/immich && docker compose up -d` |
| Nginx not running | `sudo systemctl start nginx` |
| Firewall blocking | `sudo ufw allow 80 && sudo ufw allow 443` |
| Server rebooted, forgot auto-start | Check Section 6 - enable Docker auto-start |

#### Problem: Photos Not Uploading

**Symptoms:**
- Upload progress bar stuck
- "Upload failed" error
- Photos disappear after upload

**Diagnosis:**

1. **Check disk space:**
   ```bash
   df -h /
   ```
   If >95% full, free up space.

2. **Check Immich logs:**
   ```bash
   cd ~/immich
   docker compose logs immich-server | grep -i upload
   ```

3. **Check file permissions:**
   ```bash
   ls -lah /var/lib/immich/upload/
   ```
   Should be owned by user 1000 (or PUID from .env).

**Solutions:**

| Cause | Fix |
|-------|-----|
| Disk full | Free up space, move photos |
| Permission denied | `sudo chown -R 1000:1000 /var/lib/immich` |
| Browser issue | Try different browser, clear cache |
| Large video timeout | Increase nginx `client_max_body_size` |

#### Problem: Face Recognition Not Working

**Symptoms:**
- No faces detected
- "Processing" jobs stuck

**Check machine learning container:**

```bash
docker compose logs immich-machine-learning
```

**Restart ML container:**

```bash
docker compose restart immich-machine-learning
```

**Trigger manual face detection:**
1. Go to Administration → Jobs
2. Find "Face Detection"
3. Click "Run All"

#### Problem: High CPU/Memory Usage

**Check resource usage:**

```bash
docker stats
```

**Expected output:**
```
CONTAINER           CPU %     MEM USAGE / LIMIT     MEM %
immich-server       2.5%      450MiB / 7.8GiB      5.6%
immich-ml           15.3%     1.2GiB / 7.8GiB      15.4%
immich-postgres     1.2%      180MiB / 7.8GiB      2.3%
```

**Normal patterns:**
- Machine learning uses most CPU during face detection/object recognition
- Server uses more memory when uploading
- Redis uses minimal resources

**If persistently high:**
1. Check Jobs page - cancel unnecessary jobs
2. Reduce concurrent jobs in Settings
3. Restart containers: `docker compose restart`

---

### 17.6 Restart Services

#### Restart Immich Containers

**Graceful restart (recommended):**

```bash
cd ~/immich
docker compose restart
```

**Expected output:**
```
[+] Restarting 5/5
 ✔ Container immich-redis Restarted
 ✔ Container immich-postgres Restarted
 ✔ Container immich-server Restarted
 ✔ Container immich-microservices Restarted
 ✔ Container immich-machine-learning Restarted
```

Containers restart in 5-10 seconds. No data loss.

**Full restart (if issues persist):**

```bash
cd ~/immich
docker compose down
docker compose up -d
```

**Using convenience script:**

```bash
~/immich/scripts/immich-restart.sh
```

#### Restart Nginx

```bash
sudo systemctl restart nginx
```

**Check status:**

```bash
sudo systemctl status nginx
```

#### Restart Entire Server

**When to reboot:**
- After kernel updates
- Persistent weird issues
- Memory leaks (rare)
- System feels slow

```bash
sudo reboot
```

Server will be down for 1-2 minutes. Immich containers auto-start if Docker is enabled on boot.

---

### 17.7 Maintenance Checklist

#### Weekly Tasks (5 minutes)

- ☐ Check disk space: `df -h`
- ☐ Verify backups ran: `cat /mnt/backup/immich-backups/last-backup.txt`
- ☐ Check Immich is accessible
- ☐ Review backup logs: `tail ~/immich/backup.log`

**Quick weekly check script:**

```bash
nano ~/immich/scripts/weekly-check.sh
```

```bash
#!/bin/bash
echo "=== Weekly Immich Health Check ==="
echo ""
echo "Disk Space:"
df -h / | grep -v Filesystem
echo ""
echo "Immich Storage:"
sudo du -sh /var/lib/immich
echo ""
echo "Container Status:"
cd ~/immich && docker compose ps
echo ""
echo "Last Backup:"
cat /mnt/backup/immich-backups/last-backup.txt 2>/dev/null || echo "No backup found"
echo ""
echo "=== Check Complete ==="
```

```bash
chmod +x ~/immich/scripts/weekly-check.sh
~/immich/scripts/weekly-check.sh
```

#### Monthly Tasks (15 minutes)

- ☐ Update Immich: `~/immich/scripts/immich-update.sh`
- ☐ Update Ubuntu: `sudo apt update && sudo apt upgrade -y`
- ☐ Review logs for errors: `docker compose logs | grep -i error`
- ☐ Check SSL certificate expiry: `sudo certbot certificates`
- ☐ Verify external backup drive is connected and mounted

#### Quarterly Tasks (30 minutes)

- ☐ Test backup restoration (Section 15.5)
- ☐ Review user accounts (remove inactive users)
- ☐ Check for Immich release notes (new features)
- ☐ Review disk space projections (will you need more storage?)
- ☐ Update documentation if configuration changed

#### Annual Tasks (1 hour)

- ☐ Review entire setup (is it still meeting your needs?)
- ☐ Consider hardware upgrades if needed
- ☐ Review and update this guide
- ☐ Verify disaster recovery plan
- ☐ Check backup drive health: `sudo smartctl -a /dev/sdb`

---

### ✓ Checkpoint: Maintenance Strategy Complete

At this point, you should have:

**Update procedures:**
- ✓ Know how to update Immich manually
- ✓ Know how to update Ubuntu system
- ✓ Automatic security updates enabled
- ✓ Update scripts ready to use

**Monitoring:**
- ✓ Commands to check disk space
- ✓ Commands to view logs
- ✓ Container health check process
- ✓ Weekly health check script

**Troubleshooting:**
- ✓ Common issue diagnostic steps
- ✓ Restart procedures
- ✓ Log investigation skills

**Maintenance schedule:**
- ✓ Weekly checklist (5 min)
- ✓ Monthly checklist (15 min)
- ✓ Quarterly checklist (30 min)

**Quick health check:**

```bash
cd ~/immich
echo "Containers:" && docker compose ps
echo "" && echo "Disk:" && df -h / | grep -v Filesystem
echo "" && echo "Immich Size:" && sudo du -sh /var/lib/immich
echo "" && echo "Last Backup:" && cat /mnt/backup/immich-backups/last-backup.txt
```

### 💡 What You Learned

- **Docker container lifecycle:** Pull, down, up, restart, prune
- **System package management:** apt update, upgrade, autoremove
- **Unattended upgrades:** Automatic security patching
- **Log investigation:** docker compose logs, grep for errors
- **Resource monitoring:** df, du, docker stats
- **Preventive maintenance:** Regular checks prevent big problems

### ⚠ Important Reminders

1. **Update regularly** - Monthly updates prevent security issues
2. **Monitor disk space** - Running out of space crashes Immich
3. **Test backups** - Quarterly restoration tests ensure they work
4. **Read release notes** - Know what changed in updates
5. **Don't skip reboots** - Kernel updates require reboot to apply
6. **Keep documentation updated** - Future you will thank present you

### 🎉 Congratulations!

**You've completed the entire Immich installation and maintenance guide!**

You now have:
- ✅ Fully functional Immich server
- ✅ Secure remote access with HTTPS
- ✅ Automated backup system
- ✅ Maintenance procedures
- ✅ Troubleshooting skills

**Your photos are:**
- 🔒 Backed up (3-2-1 rule)
- 🌐 Accessible from anywhere
- 🔐 Secure (HTTPS, firewall)
- 🤖 Organized (face recognition, ML features)
- 👨‍👩‍👧‍👦 Shareable (albums, user accounts)

**What's next?**

- **Explore Immich features:** Albums, sharing, mobile auto-upload
- **Migrate from Google Photos:** Use the import process (Section 14)
- **Invite family members:** Create user accounts or use OAuth
- **Customize:** Explore Immich settings for your preferences
- **Contribute:** Report bugs, suggest features on Immich GitHub

**Getting help:**

- 📖 **Official docs:** https://immich.app/docs
- 💬 **Discord community:** https://discord.gg/immich
- 🐛 **Report issues:** https://github.com/immich-app/immich/issues
- 🙋 **Discussions:** https://github.com/immich-app/immich/discussions

**Share your success:** Consider sharing your experience to help others on the journey to photo ownership!

---

## Beginner Tips: How to Think While Using Linux

If this is your first time using Linux, a few mindset tips will save you a lot of stress.

### Commands Either Work — or Tell You Why They Didn't

Linux error messages are usually literal.

If something fails:

1. Read the error message carefully
2. Copy it *exactly* (the whole thing)
3. Paste it into a search engine

Chances are someone has hit the same issue before and documented the solution.

### It's Okay to Go Slowly

Nothing in this guide is time-sensitive.

* You can stop after any step
* Rebooting is safe (and often helpful)
* Taking breaks is normal
* You don't need to finish everything in one session

### Editing Files with Nano

When you see:

```bash
nano somefile
```

You're opening a text editor called **nano**. It's simple but powerful:

* Use arrow keys to navigate
* Type normally to edit
* **Ctrl + O** to save ("Write Out")
* **Ctrl + X** to exit
* **Ctrl + K** to cut a line
* **Ctrl + U** to paste

Mistakes are reversible — files can be edited again. If you mess up, just edit the file again.

### If You Get Lost

These commands are safe and helpful:

```bash
pwd       # Print Working Directory - where am I?
ls        # List files - what's here?
ls -lah   # Detailed list with hidden files
cd ..     # Go up one directory
cd ~      # Go to home directory
```

### Copy-Paste in Terminal

* **In SSH:** Right-click to paste (or Shift+Insert)
* **Selecting text:** Click and drag, automatically copies
* **Be careful:** Commands are case-sensitive

---

## Common Beginner Mistakes (And How to Avoid Them)

### Running Out of Disk Space

**Problem:** Filling `/` (root partition) breaks the system.

**Solution:**
* Store photos in a dedicated location (`/var/lib/immich`)
* Monitor disk usage regularly with `df -h`
* Set up alerts when disk usage exceeds 80%

### Forgetting Backups

**Problem:** Hardware fails, data is lost.

**Solution:**
* Set up backups *before* you upload all your photos
* Test restoration at least once
* Automate backups with cron

### Breaking Things by Experimenting

**Problem:** Changing random settings because "it might help."

**Solution:**
* If unsure, stop and search first
* Read documentation before changing configuration
* Ask in Immich Discord or forums before making big changes

### Copy-Paste Errors

**Problem:** Smart quotes (`"` instead of `"`), missing characters, or extra spaces break commands.

**Solution:**
* Copy commands directly from this guide
* Watch for quote marks that look different
* If a command fails, retype it manually

### Not Reading Error Messages

**Problem:** Skipping error messages and trying random fixes.

**Solution:**
* Errors tell you exactly what's wrong
* Read the whole message
* Search for the specific error text

### Using `sudo` Unnecessarily

**Problem:** Running everything as root creates permission issues later.

**Solution:**
* Only use `sudo` when the command requires it
* If a command fails without `sudo`, read the error first
* Docker commands should NOT need `sudo` after adding your user to the docker group

---

## Advanced Storage Setup (Optional)

If you have a separate drive for photos, here's how to set it up.

### Identify Your Drive

```bash
lsblk
```

Look for your drive (e.g., `/dev/sdb`).

### Format the Drive (if new)

**Warning:** This erases all data on the drive.

```bash
# Format as ext4
sudo mkfs.ext4 /dev/sdb1

# Label it
sudo e2label /dev/sdb1 immich-storage
```

### Mount the Drive

```bash
# Create mount point
sudo mkdir -p /mnt/immich

# Mount
sudo mount /dev/sdb1 /mnt/immich

# Set ownership
sudo chown -R $USER:$USER /mnt/immich
```

### Auto-Mount on Boot

```bash
# Get UUID
sudo blkid /dev/sdb1
```

Copy the UUID, then edit fstab:

```bash
sudo nano /etc/fstab
```

Add this line (replace UUID):

```
UUID=your-uuid-here  /mnt/immich  ext4  defaults  0  2
```

Save and test:

```bash
sudo mount -a
df -h /mnt/immich
```

### Update Immich Configuration

Edit `.env`:

```bash
nano ~/immich/.env
```

Change upload location:

```bash
UPLOAD_LOCATION=/mnt/immich
```

Restart Immich:

```bash
cd ~/immich
docker compose down
docker compose up -d
```

---

## Troubleshooting

### Immich Won't Start

**Check logs:**

```bash
cd ~/immich
docker compose logs immich-server
```

**Common issues:**
* Database password mismatch in `.env`
* Port 2283 already in use
* Permission issues on upload directory

**Fix permissions:**

```bash
sudo chown -R $USER:$USER /var/lib/immich
```

### Can't Access Remotely

**Check DNS:**

```bash
nslookup photos.yourdomain.com
```

Should return your public IP.

**Check port forwarding:**

Test from outside your network:

```bash
curl -I http://your-public-ip
```

Should return HTTP headers from Nginx.

**Check firewall:**

```bash
sudo ufw status
```

Ports 80 and 443 should be allowed.

### Mobile App Won't Upload

**Check server URL:**
* Must be exact (including `https://` or `http://`)
* No trailing slash

**Check network:**
* Are you on the same network as the server?
* Is remote access configured?

**Check logs:**

```bash
docker compose logs -f immich-server
```

### SSL Certificate Errors

**Renew manually:**

```bash
sudo certbot renew
```

**Check expiration:**

```bash
sudo certbot certificates
```

**Force renewal (if needed):**

```bash
sudo certbot renew --force-renewal
```

---

## Additional Resources

### Official Immich Documentation

* [Immich Docs](https://immich.app/docs)
* [Immich GitHub](https://github.com/immich-app/immich)
* [Immich Discord](https://discord.gg/immich)

### Learning Linux

* [Ubuntu Server Guide](https://ubuntu.com/server/docs)
* [Linux Journey](https://linuxjourney.com/)
* [Command Line Basics](https://ubuntu.com/tutorials/command-line-for-beginners)

### Docker

* [Docker Getting Started](https://docs.docker.com/get-started/)
* [Docker Compose Tutorial](https://docs.docker.com/compose/gettingstarted/)

### Nginx

* [Nginx Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)

### Backup Tools

* [Rclone Documentation](https://rclone.org/docs/)
* [Restic Backup](https://restic.net/)

---

## Conclusion

You now have a fully functional Immich server with:

* Local and remote access
* HTTPS encryption
* Automated backups
* Mobile app integration
* Optional Google OAuth

### What's Next?

* Upload your photos
* Set up auto-backup on your phone
* Test restoration from backup
* Invite family members (create accounts in Immich admin)

### Getting Help

If you run into issues:

1. Check logs: `docker compose logs -f immich-server`
2. Search [Immich Discord](https://discord.gg/immich)
3. Check [GitHub Issues](https://github.com/immich-app/immich/issues)
4. Ask in Linux forums (include error messages)

### Stay Updated

Join the Immich community to learn about new features:

* [Immich Discord](https://discord.gg/immich)
* [GitHub Releases](https://github.com/immich-app/immich/releases)

---

**Congratulations on completing your Immich setup!** Your photos are now backed up to a server you control, with all the convenience of cloud services and none of the privacy concerns.
