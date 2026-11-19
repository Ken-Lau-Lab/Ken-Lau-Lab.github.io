---
layout: post
title: Lau Lab Server Standard Operating Procedure
date: 2026-11-19 12:00:00
description: how to connect to and responsibly use the lab's Threadripper and Matisse analysis servers
tags: sop server
categories: sops
toc:
  sidebar: left
---

## Overview

The Lau lab has two servers, **Threadripper** and **Matisse**, each named after the CPU it houses. Both are physically located in Ken's office, so they depend on the lab's wired ethernet connection and power supply.

Their main job is processing **single-cell RNA-sequencing data**, and their hardware was chosen and tuned for that. They handle data that ordinary computers can't because of memory and time limits. Both are **CPU-heavy machines with weak GPUs**, so don't expect to do much GPU-accelerated image analysis or deep learning on them.

| | **Threadripper** | **Matisse** |
|---|---|---|
| **IP address** | `10.141.26.164` | `10.141.26.101` |
| **CPU** | AMD Threadripper 2990WX, 32-core / 64-thread @ 3.00 GHz | AMD Ryzen 3900X, 12-core / 24-thread @ 3.8 GHz |
| **GPU** | AMD FirePro W2100, 2 GB VRAM | AMD RX570, 4 GB VRAM |
| **Memory (RAM)** | 128 GB DDR4 | 64 GB DDR4 |
| **Storage** | 2 TB SSD, 500 GB SSD, 2× 2 TB HDD | 1 TB SSD |
| **OS** | Ubuntu 20.04 LTS | Ubuntu 20.04 LTS |

> 💡 **Which server should I use?** Matisse has fewer cores and threads, but its base clock is about **26% higher** than Threadripper's. It can't run as many tasks in parallel, but it runs single-threaded tasks faster. Use Threadripper for large, highly parallel or memory-hungry jobs.

---

## Server Etiquette

### Check Usage Before You Start

Every time you connect to a server, check how busy it is with:

```bash
top
```

If someone is running a large job (high CPU **and** high memory usage), use the **#server** Slack channel to coordinate resources and timing, or switch to the other server.

#### Reading the `top` Output

The top section of the display shows system-level resources. The key fields are:

| Field | What it means |
|---|---|
| **%Cpu** | Computational load across all available threads |
| **GiB Mem used** | RAM used by all connected users. ⚠️ **Critical:** going over the limit (e.g. 125.8 GB on Threadripper) may automatically kill processes |
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

If you've determined you truly need more than 30 GB and other users are connected to your server, **post an advisory message in #server on Slack** so others know. Failing to do so will earn you sad emojis 😢 and may push us toward automated resource restrictions, which limit everyone.

---

## Connection and Environment Setup

Using the servers requires setup on your **local machine** (home desktop, laptop, lab desktop, etc.) and on your **server profile**. Once that's done, you can connect and set up your analysis environment.

### 1. Connect from Your Local Machine

1. Connect to the **Vanderbilt VPN** using Pulse Secure (only needed off-campus)
2. Open a terminal:
   - **Windows:** install [Git Bash](https://gitforwindows.org/) and open it
   - **Mac / Linux:** open the built-in **Terminal**
3. Run the SSH command for your server:

   **Threadripper**
   ```bash
   ssh -L 1234:127.0.0.1:XXXX your-username@10.141.26.164
   ```

   **Matisse**
   ```bash
   ssh -L 1234:127.0.0.1:XXXX your-username@10.141.26.101
   ```

   - `XXXX` is a **4-digit port number above 9000** of your choosing. It's the port the server uses to send data back to your machine, so **remember it** for later.
   - This command connects to the server **and** forwards that port to port `1234` on your local machine.
4. Enter your password when prompted. By default it's the **server name followed by your first and last initials, all lowercase** (e.g. `threadripperab`). Change it after your first login with `passwd`.

> 💡 **Tip:** Save the SSH command in a text file (e.g. `connect.sh`) and run it with `bash connect.sh` so you don't have to retype it every time.

---

### 2. Use `tmux` (Highly Recommended)

A dropped connection usually terminates everything you were running. `tmux` creates sessions you can detach from and re-attach to, so your work survives disconnects.

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

1. Download the installer:
   ```bash
   curl -O https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh
   ```
2. Run it:
   ```bash
   bash Anaconda3-2020.02-Linux-x86_64.sh
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

> ⚠️ `XXXX` **must be the same port number** you used in your SSH command.

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

1. In any browser on your local machine, go to: `localhost:1234`
2. The first time, it will ask for a **token**. In your server terminal, find the output line that looks like `localhost:XXXX/?token=...`, then copy everything after `token=` into the browser. Make sure you don't copy any extra spaces.
3. You should now see your JupyterLab environment, hosted on the server.

> 💡 All file paths in Jupyter are relative to the server directory where you ran `jupyter lab`.

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

## Copying Files Between Servers

Log in to the **destination** server first, then pull files from the other one.

**Logged in to Matisse, copying from Threadripper:**
```bash
scp username@10.141.26.164:/home/username/* /home/username
```

**Logged in to Threadripper, copying from Matisse:**
```bash
scp username@10.141.26.101:/home/username/* /home/username
```

To copy whole directories, add the `-r` flag:
```bash
scp -r username@10.141.26.164:/home/username/my_folder /home/username
```
