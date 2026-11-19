---
layout: post
title: Lau Lab NAS Standard Operating Procedure
date: 2026-11-19 12:00:00
description: how to access and use the lab's Polaris, Rosalind and Synology NAS storage
tags: computational
categories: sops
toc:
  sidebar: left
---

## Overview

| NAS | Capacity | Notes |
|-----|----------|-------|
| **Polaris LauLab** | 150 TB total | 50 TB reserved for Monica; 100 TB open to everyone. Primary storage for all lab data. Mounted on Rosalind at `/Polaris/LauLab` |
| **Rosalind NAS** | 100 TB (⚠️ 70% full) | Mounted at `/home/NAShome`. Every user's home directory is at `/home/NAS/VUNetID/` |
| **Synology NAS (LauLabNAS1)** | 62.8 TB usable / **12.7 TB free** | Access via browser at `http://10.18.26.54:5000/`. Open to everyone. Only `level_1_data` is mounted on Rosalind at `/mnt/launas` |

---

## Synology NAS (LauLabNAS1)

> ⚠️ **Note:** Synology NAS is a consumer/prosumer-grade system and is **slower than Polaris**. It is not optimized for heavy computation or large dataset I/O. Use it for storing documents, scripts, and `level1_data` intake only. For intensive workloads, use Polaris/Rosalind instead.

### Storage Info

| | Details |
|---|---|
| **RAID type** | Synology Hybrid RAID (SHR) — 2-drive fault tolerance |
| **Storage Pool** | Storage Pool 1 — 65.4 TB (6× 16.4 TB HDDs) |
| **SSD Cache** | RAID 0 — 745 GB (2× 372.6 GB M.2 drives) |
| **Volume 1** | 62.8 TB total, **12.7 TB free** (Btrfs) |
| **Status** | Healthy ✅ |

#### How to Check Available Space

1. Log in at `http://10.18.26.54:5000/`
2. Open **Storage Manager** from the main menu
3. In the left sidebar, click **Storage Pool 1** or **Volume 1**
4. The top of the page shows allocated space and free space at a glance

---

### Accessing via Browser

1. Connect to VPN
2. Go to `http://10.18.26.54:5000/`
3. Log in with your credentials

---

### Accessing via Your Computer

#### Windows

1. Make sure you are on VPN (or connected via ethernet)
2. Search for **"Turn Windows Features on or off"**
3. Find **SMB 1.0/CIFS File Sharing Support** and check the box to enable it, then restart your computer
4. Open **File Explorer**, navigate to **Network**, and right-click to get options
5. Select **Map Network Drive**
6. Choose any drive letter and enter the server path:
   ```
   \\10.18.26.54\<folder_to_mount>
   ```
7. Check **"Use different credentials"**
8. Enter your credentials when prompted:
   - **Username:** `LauLabNAS1\<your_username>`
   - **Password:** your NAS account password

---

#### Mac

**Method 1 — via Notes App**

1. Make sure you are on VPN
2. Open the **Notes** application
3. In a new note, type: `smb://username@LauLabNAS1/`
4. A dialog will appear: *"You are attempting to connect to the server 'LauLabNAS1'"* — click **Continue**
5. In the next pop-up, make sure **Registered User** is selected
6. Fill in your NAS username and password, then click **Connect**
7. Select the volume you want in the next window; all volumes will be accessible through the LauLabNAS1 connected drive in Finder

**Method 2 — via Finder (Connect to Server)**

1. Make sure you are on VPN
2. Open **Finder** and press `⌘ + K`
3. In the **Connect to Server** window, enter:
   - **If on VPN:** `smb://10.18.26.54/<folder>`
   - **If on VU ethernet:** `smb://LauLabNAS1/<folder>`
4. Make sure **Registered User** is selected
5. Fill in your NAS username and password, then click **Connect**
6. Select the desired volume; all volumes will be accessible in Finder

**Method 3 — via Finder Network Tab (temporary access on VU ethernet)**

1. Open **Finder**
2. In the left sidebar, navigate to **Locations → Network**
3. Find **LauLabNAS1** among the network drives
4. Click the **Connect As** button (top-right of Finder)
5. A dialog will appear: *"You are attempting to connect to the server 'LauLabNAS1'"* — click **Continue**
6. Make sure **Registered User** is selected
7. Fill in your NAS username and password, then click **Connect**
8. All volumes on the NAS will be shown in Finder

---

#### Linux

**Via NFS**

1. **Create an NFS Share on the NAS (Admin only)**
   - Log in at `http://10.18.26.54:5000/`
   - Go to **Control Panel → Shared Folder**
   - Select the folder you want to share and click **Edit**
   - Go to the **NFS Permissions** tab and click **Create**
   - Fill in the dialog as follows:

     | Field | Value |
     |---|---|
     | **Hostname or IP** | IP address of the Linux machine (e.g. `10.94.26.35`) |
     | **Privilege** | `Read/Write` |
     | **Squash** | `No mapping` |
     | **Security** | `sys` |
     | **Enable asynchronous** | ✅ Checked |
     | Allow connections from non-privileged ports | ☐ Unchecked |
     | Allow users to access mounted subfolders | ☐ Unchecked |

   - Click **Save**
   - Note the **Mount path** shown at the bottom of the window (e.g. `/volume1/homes`) — you will need this for the mount command below
   - > ⚠️ *Squash setting: currently using "Map all users to admin" on some shares — a safer option is under investigation*

2. **Mount on Linux terminal:**
   ```bash
   sudo mount nas_ip:<shared_path> <where_to_mount_path>
   ```

3. **[Optional] Permanent mount via `/etc/fstab`:**

   Add the following line to `/etc/fstab` using `vim`:
   ```
   10.94.27.26:/path_to_share /mnt_path nfs defaults 0 0
   ```
   Then run:
   ```bash
   sudo systemctl daemon-reload
   sudo mount -a
   ```
   > ⚠️ **Warning:** Mount shared folders to *different* destination directories on your computer. If multiple shared folders are mounted to the same directory, NAS may create random folders unexpectedly.

**Via GUI**

Use the Synology web interface at `http://10.18.26.54:5000/` to manage files through **File Station**.

---

## Note

> Simran will be only using Synology NAS.
