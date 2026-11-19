---
layout: post
title: Lau Lab Server Standard Operating Procedure
date: 2026-11-19 12:00:00
description: how to connect to and responsibly use the lab's Threadripper, Matisse and Athena analysis servers
tags: sop server
categories: sops
toc:
  sidebar: left
---

*Last updated: October 2026*

## Overview

The Lau lab has three servers, **Threadripper**, **Matisse** and **Athena**, each named after the CPU it houses. All three are physically located in the computational space. Their main job is processing **single-cell RNA-sequencing data**, and their hardware was chosen and tuned for that. They handle data that ordinary computers can't because of memory and time limits.

### Which Resource Should I Use?

> ⚠️ **All computational graduate students and staff are expected to run major computational tasks on Rosalind.** See the [Rosalind User Guide](#rosalind-user-guide) below.

Small tasks can run on the lab servers, divided as follows:

| Server | Best for |
|---|---|
| **Matisse** | Count-based scRNA-seq (analysis that can be done with just `scanpy`), GPU work |
| **Threadripper** | Highly parallelized bioinformatics, matrix factorization, gene network inference |
| **Athena** | Deep learning / AI and image analysis (GPU) |

Athena and Matisse both have NVIDIA RTX 3090 GPUs, which deep learning and AI algorithms need for their very high numbers of parallel calculations. **Image analyses** can be done on the Athena and Matisse GPUs.

> 🎓 **Undergraduates** should use only the lab servers, not Rosalind, unless they have special permission.

### Server Specifications

| | **Threadripper** | **Matisse** | **Athena** |
|---|---|---|---|
| **IP address** | `10.18.26.56` | `10.18.26.63` | `10.18.26.55` |
| **CPU** | AMD Threadripper 2990WX, 32-core / 64-thread | AMD Ryzen 3900X, 12-core / 24-thread | Intel Core i9-13900K, 24-core / 32-thread |
| **GPU** | AMD FirePro W2100, 2 GB VRAM | NVIDIA RTX 3090, 24 GB VRAM | NVIDIA RTX 3090, 24 GB VRAM |
| **Memory (RAM)** | 128 GB DDR4 | 128 GB DDR4 | 128 GB DDR4 |
| **Storage** | 2 TB SSD (system), 5.5 TB SSD RAID (`/mnt/md0`), 512 GB SSD | 1 TB SSD (system), 4 TB SSD (`/mnt/storage`) | 2 TB SSD (system), 8 TB SSD (`/storage`) |
| **OS** | Ubuntu 20.04 LTS | Ubuntu 20.04 LTS | Ubuntu 22.04 LTS |

### Lab Storage: LauLabNAS1

**LauLabNAS1** (Synology DS1621+) is the lab's central storage for raw data, processed datasets and backups. It is **not meant for computation**: process data on Threadripper, Matisse or Athena, then store the results on the NAS. See [Mounting LauLabNAS1](#mounting-laulabnas1) for how to mount it on the servers.

- **Actively used data** → LauLabNAS1
- **Infrequently accessed data** → Rosalind NAS

| | **LauLabNAS1** |
|---|---|
| **Model** | Synology DS1621+ |
| **CPU** | AMD Ryzen Embedded V1500B, 4-core / 8-thread |
| **Memory (RAM)** | 32 GB DDR4 ECC |
| **Storage** | 6× 18 TB HDD (~72 TB usable), 2× 400 GB NVMe SSD cache |
| **Redundancy** | RAID 6 / SHR-2 (survives 2 drive failures) |
| **OS** | Synology DSM 7.4.1 |

### Storage Paths on Each Server

> ⚠️ **Do not use your home directory for main data storage.** Use the server's SSDs for short-term storage, and LauLabNAS1 or Rosalind storage for long-term storage.

#### Threadripper

| Location | Path | Use |
|---|---|---|
| Home directories (2 TB, shared by all users) | `/home` | Not for main data storage |
| 500 GB SSD | `/mnt/500gb_ssd` | Short-term data storage |
| 5.5 TB SSD RAID | `/mnt/md0` | Short-term data storage |
| LauLabNAS1 | `/mnt/laulabnas` and `/mnt/level_1_data` | Long-term storage |

#### Matisse

| Location | Path | Use |
|---|---|---|
| 4 TB SSD | `/mnt/storage` | Short-term data storage |

#### Athena

| Location | Path | Use |
|---|---|---|
| Home directories (2 TB, shared by all users) | `/home` | Not for main data storage |
| 8 TB SSD | `/storage` | Short-term data storage |
| LauLabNAS1 | `/mnt/NAS` and `/mnt/level_1_data` | Long-term storage |

---

## Server Etiquette

### Check Usage Before You Start

Every time you connect to a server, check how busy it is with [`top`](https://manpages.ubuntu.com/manpages/focal/man1/top.1.html):

```bash
top
```

If someone is running a large job (high CPU **and** high memory usage), use the **#computational** Slack channel to coordinate resources and timing, or switch to another server.

#### Reading the `top` Output

The top section of the display shows system-level resources. The key fields are:

| Field | What it means |
|---|---|
| **%Cpu** | Computational load across all available threads |
| **GiB Mem used** | RAM used by all connected users. ⚠️ **Critical:** going over the limit (about 125.8 GB on a 128 GB server) may automatically kill processes |
| **GiB Swap avail Mem** | Remaining headroom. The smaller this number, the higher the risk of running out of memory |

#### Useful `top` Shortcuts

| Key | Action |
|---|---|
| `E` | Cycle display units for per-process memory (e.g. bytes → GB) |
| `U` | Filter processes by username |
| `Shift + E` | Cycle display units for the system-level summary at the top |
| `Shift + M` | Sort processes by memory usage |
| `Shift + P` | Sort processes by CPU usage |
| `q` | Quit `top` |

### Memory Rule of Thumb

> ⚠️ **Do not exceed 30 GB of memory in a single session** for standard, count-based scRNA-seq analysis.

30 GB is roughly what you'd use analyzing **dozens of combined atlas datasets at once**. If your data isn't that large and you're near this limit, take a second look at the notebooks you're running, the datasets loaded into those environments and the variables you're holding in memory.

**Shut down your Jupyter kernel when you're done with a notebook.** Closing the browser tab does *not* free its memory. Use **Kernel → Shut Down Kernel** in the notebook, or shut it down from JupyterLab's **Running** tab.

If you've determined you truly need more than 30 GB and other users are connected to your server, **post an advisory message in #computational on Slack** so others know. Failing to do so will earn you sad emojis 😢 and may push us toward automated resource restrictions, which limit everyone.

### Check Disk Usage per User

To see how much disk space each user's home directory is taking up:

```bash
sudo du -h --max-depth=1 /home
```

---

## Connection and Environment Setup

Using the servers requires setup on your **local machine** (home desktop, laptop, lab desktop, etc.) and on your **server profile**. Once that's done, you can connect and set up your analysis environment.

### 1. Connect from Your Local Machine

1. Connect to the [Vanderbilt VPN using GlobalProtect](https://www.vanderbilt.edu/cybersecurity/services/vpn/) (only needed off-campus)
2. Open a terminal:
   - **Windows:** install [Git Bash](https://gitforwindows.org/) and open it
   - **Mac / Linux:** open the built-in **Terminal**
3. Run the SSH command for your server:

   **Threadripper**
   ```bash
   ssh -L 1234:127.0.0.1:XXXX your-username@10.18.26.56
   ```

   **Matisse**
   ```bash
   ssh -L 1234:127.0.0.1:XXXX your-username@10.18.26.63
   ```

   **Athena**
   ```bash
   ssh -L 8765:127.0.0.1:XXXX your-username@10.18.26.55
   ```

   - `XXXX` is a **4-digit port number above 9000** of your choosing. It's the port the server uses to send data back to your machine, so **remember it** for later.
   - This command connects to the server **and** forwards that port to your local machine (port `1234` for Threadripper and Matisse, `8765` for Athena).
4. Enter your password when prompted. By default it's the **server name followed by your first and last initials, all lowercase**. Change it after your first login with `passwd`.

> 💡 **Tip:** Save the SSH command in a text file (e.g. `connect.sh`) and run it with `bash connect.sh` so you don't have to retype it every time.

---

### 2. Use `tmux` (Highly Recommended)

A dropped connection usually terminates everything you were running. [`tmux`](https://github.com/tmux/tmux/wiki) creates sessions you can detach from and re-attach to, so your work survives disconnects.

| Action | Command |
|---|---|
| Start a new session | `tmux new -s scanpy_session` (a green bar at the bottom means it worked) |
| Detach (send to background) | `Ctrl + b`, release, then `d` |
| Re-attach | `tmux attach-session -t scanpy_session` |
| List running sessions | `tmux ls` |
| Close a session completely | `exit` (while attached) |

Use `ls` at any time to see the files in your current directory.

---

### 3. Install Anaconda

1. Download the installer. Check the [Anaconda archive](https://repo.anaconda.com/archive/) for the newest version and substitute its filename below.
   ```bash
   curl -O https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh
   ```
2. Run it:
   ```bash
   bash Anaconda3-2024.10-1-Linux-x86_64.sh
   ```
3. Respond to the prompts:
   - **Accept** the license terms
   - Keep the **default install location**
   - Say **yes** to initializing conda / modifying your bash profile
4. Disconnect (`exit`) and reconnect to the server. If everything worked, you'll see **`(base)`** at the left of your command prompt.

---

### 4. Create a Conda Environment

Environments keep each project's Python packages separate, since different packages may need conflicting dependency versions.

1. Create an environment (answer **yes** to any prompts):
   ```bash
   conda create --name my_env python=3.7
   ```
2. Activate it:
   ```bash
   conda activate my_env
   ```
3. Make sure the environment is **active before installing anything**. Packages installed with `pip install` or `conda install` are available only while that environment is active.
4. Install JupyterLab:
   ```bash
   pip install jupyterlab
   ```

---

### 5. Launch JupyterLab on the Server

```bash
jupyter lab --no-browser --port=XXXX
```

> ⚠️ `XXXX` **must be the same port number** you used at the end of your SSH command.

This hosts JupyterLab on the server at that port, which is forwarded to your local machine.

#### Managing the Jupyter Process

| Action | Command |
|---|---|
| Suspend the process and get your terminal back | `Ctrl + Z` |
| Resume it in the background (keeps Jupyter running while you use the terminal) | `bg` |
| List background jobs | `jobs` |
| Bring it back to the foreground | `fg` |
| Stop Jupyter (while in the foreground) | `Ctrl + C` twice |

> ⚠️ After `Ctrl + Z`, Jupyter is **paused** until you run `bg` or `fg`. Your browser tab will hang until you do.

---

### 6. Open Jupyter in Your Local Browser

1. In any browser on your local machine, go to the local port from your SSH command (the 4 digits right after `ssh -L`):

   | Server | Address |
   |---|---|
   | Threadripper, Matisse | `localhost:1234` |
   | Athena | `localhost:8765` |

2. The first time, it will ask for a **token**. In your server terminal, find the output line that looks like `localhost:XXXX/?token=...`, then copy everything after `token=` into the browser. Make sure you don't copy any extra spaces.
3. You should now see your JupyterLab environment, hosted on the server.

> 💡 All file paths in Jupyter are relative to the server directory where you ran `jupyter lab`.

---

### 7. RStudio Server (Optional)

RStudio Server runs the RStudio IDE (e.g. R notebooks) on a Linux server, accessed from your web browser. It's useful if you need R packages.

1. **Check whether RStudio Server is installed:**
   ```bash
   rstudio-server version
   ```
   You should see version information such as `2025.05.0+496 (Mariposa Orchid) for Ubuntu Focal`.
2. **If it isn't installed**, follow Posit's [RStudio Server installation instructions](https://posit.co/download/rstudio-server/) (check for updates at least once a year). To find the server's OS version for the installer, run:
   ```bash
   lsb_release -a
   ```
3. **Open RStudio in your browser** at `http://<server-ip>:8787`, for example:

   | Server | Address |
   |---|---|
   | Threadripper | `http://10.18.26.56:8787` |
   | Matisse | `http://10.18.26.63:8787` |
   | Athena | `http://10.18.26.55:8787` |

---

## Mounting LauLabNAS1

See Simran's documentation on Slack: [Mounting LauLabNAS1](https://lau-lab.slack.com/docs/TAVGR2ZPV/F099PSSFRH9)

---

## Exiting Server Operations

**On your local machine:**

1. Save everything (`Ctrl + S` works as usual)
2. Go to the **Running** tab in JupyterLab and **shut down active notebooks**

**On the server terminal:**

1. Make sure the Jupyter process is in the **foreground**. Use `fg` if needed. (If you can't type commands, it's already in the foreground.)
2. Press `Ctrl + C` once to bring up the exit prompt, then again to confirm
3. Type `exit` to disconnect. You can now close the window.

### Resetting Everything

If things get stuck and you need to start over, or you're completely done with the server:

```bash
pkill -u your-username
```

> ⚠️ **Warning:** This kills **all** processes running under your username. Anything not saved to disk will be lost.

---

## Rosalind User Guide

See Hirak's documentation: [Rosalind Guide](https://www.hiraksarkar.com/rosalind_guide/guide/)

> ⚠️ SSH to Rosalind using the domain name `rosalind-login` works **only when you're connected to the Vanderbilt VPN**. From the lab's VU ethernet, SSH works only with Rosalind's IP address.
