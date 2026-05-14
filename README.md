# HomeServer Lab: From Legacy Hardware to Modern Docker Environment

A detailed documentation of my journey setting up a lightweight, high-performance home server using a legacy Intel H61 motherboard, 8GB RAM, and a 112GB SSD.

## Project Overview
The goal was to transform older hardware into a functional "Netflix-style" media server (Jellyfin), a private cloud (Nextcloud), and a development environment for my web portfolio, while maintaining a tiny resource footprint.

---

## Hardware Stack
* **Motherboard:** H61 Series (Intel Pentium)
* **RAM:** 8GB DDR3
* **Storage:** 112GB SATA SSD
* **OS:** Debian 13 (Trixie) - Minimal/Netinst (No GUI)

---

## Installation & Configuration Steps

### 1. OS Installation: The "Single Partition" Pivot
Initially, I experimented with **Proxmox**, but found it too RAM-heavy for 8GB. I shifted to **Debian 13**.
* **Key Learning:** My first install used separate partitions (`/var`, `/home`, etc.). This caused storage "walls" where Docker would run out of space while the rest of the disk was empty.
* **The Fix:** Reinstalled Debian using **Guided - All files in one partition**. This created a single ~100GB pool for maximum flexibility.

### 2. System Hardening & Tools
Since the Debian Netinst is minimal, I manually configured the "Sudoer" privileges and essential tools:
```bash
# Switch to root
su -

# Install essentials
apt update && apt install sudo curl -y

# Add user to sudo group
usermod -aG sudo <username>
