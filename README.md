# Pi-hole Installation and Configuration Guide

## Overview

This document outlines the step-by-step process for setting up Pi-hole on a NanoPi R4S with Debian Bookworm. It also includes additional configurations for blocking YouTube ads and maintaining the setup via GitHub.

---

## 1. Preparing the SD Card

1. **List Available Disks:**
   ```sh
   diskutil list
   ```
2. **Identify and Unmount the SD Card:**
   ```sh
   diskutil unmountDisk /dev/diskX  # Replace X with the correct disk number
   ```
3. **Flash Debian Image to SD Card:**
   ```sh
   sudo dd if=rk3399-sd-debian-bookworm-core-4.19-arm64-20241112.img of=/dev/rdiskX bs=4M status=progress
   ```
4. **Eject the SD Card:**
   ```sh
   diskutil eject /dev/diskX
   ```
5. Insert the SD card into the NanoPi and boot up the system.

---

## 2. Connecting to the NanoPi R4S via SSH

1. **Find the IP Address:** Check your router’s DHCP lease table to locate the IP assigned to NanoPi.
2. **Connect via SSH:**
   ```sh
   ssh root@192.168.3.247  # Replace with your NanoPi's actual IP
   ```

---

## 3. Installing Pi-hole

1. **Update Package List:**
   ```sh
   apt-get update && apt-get upgrade -y
   ```
2. **Install Required Dependencies:**
   ```sh
   apt-get install -y curl
   ```
3. **Run Pi-hole Installer:**
   ```sh
   curl -sSL https://install.pi-hole.net | bash
   ```
4. **During installation, configure:**
   - Upstream DNS: Choose a provider like Cloudflare (1.1.1.1) or Google (8.8.8.8).
   - Network Interface: Select the correct interface (eth0 or wlan0).
   - Set a static IP for the NanoPi.

---

## 4. Setting Up Pi-hole as the Primary DNS

1. **Configure each device manually to use the Pi-hole IP (e.g., 192.168.3.247) as the primary DNS.**
2. **To force DNS redirection via iptables:**
   ```sh
   sudo iptables -t nat -A PREROUTING -p udp --dport 53 -j DNAT --to-destination 192.168.3.247:53
   sudo iptables -t nat -A PREROUTING -p tcp --dport 53 -j DNAT --to-destination 192.168.3.247:53
   ```
   *Make sure to persist these rules by installing iptables-persistent.*
   ```sh
   apt-get install iptables-persistent
   ```

---

## 5. Blocking YouTube Ads with Pi-hole

1. **Add YouTube Ad Block List:**
   ```sh
   curl -sSL https://oisd.nl/downloads | sudo tee -a /etc/pihole/custom.list
   ```
2. **Restart Pi-hole to Apply Changes:**
   ```sh
   pihole restartdns
   ```

---

## 6. Uploading Configuration to GitHub

1. **Install Git:**
   ```sh
   apt-get install git -y
   ```
2. **Initialize a Git Repository:**
   ```sh
   mkdir ~/pihole-config && cd ~/pihole-config
   git init
   ```
3. **Backup Pi-hole Configuration:**
   ```sh
   cp -r /etc/pihole ~/pihole-config
   ```
4. **Push to GitHub:**
   ```sh
   git add .
   git commit -m "Initial Pi-hole configuration"
   git remote add origin https://github.com/your-username/pihole-config.git
   git push -u origin main
   ```

---

## 7. Troubleshooting

### Reset Pi-hole Admin Password

```sh
pihole -a -p
```

### Checking DNS Requests

```sh
pihole -t
```

### Checking Network Traffic

```sh
nmap -sP 192.168.3.0/24  # Adjust for your network range
```

### Debugging Internet Connectivity

```sh
ping -c 4 8.8.8.8  # Check if you have internet access
nslookup google.com  # Ensure DNS resolution is working
```

---

## Conclusion

This guide walks through installing Pi-hole on a NanoPi R4S, configuring it as a network-wide ad blocker, blocking YouTube ads, and maintaining the configuration with GitHub. If needed, you can expand the blacklist further using additional sources.

