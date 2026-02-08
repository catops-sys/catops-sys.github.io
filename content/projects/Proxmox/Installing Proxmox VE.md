---
title: "Setting Up a Two-Node Proxmox Cluster: toothless + superman in My Home Lab"
date: 2026-02-08
draft: false
description: "How I clustered two beefy Proxmox nodes (dual Xeons, tons of RAM and storage) for my homelab – including the quorum fix every two-node setup needs."
tags: ["proxmox", "home-lab", "cluster", "virtualization", "self-hosting", "xeon"]
categories: ["homelab"]
---

I'm only human, and clustering sounded intimidating at first — but once I got both machines running Proxmox, joining them was surprisingly straightforward.  
Here's my real two-node setup: **toothless** and **superman**, both old server beasts repurposed for VMs, containers, and future ZFS/Ceph experiments.

### My Two Nodes – Real Hardware Specs
Pulled straight from `lscpu`, `free -h`, and `lsblk` on each.

**Node 1: toothless** (the "RAM monster")
- **CPU**: Dual Intel Xeon E5-2670 v2 @ 2.50 GHz  
  – 2 sockets × 10 cores × 2 threads = 40 threads total  
- **RAM**: 251 GiB (~256 GB) total — insane headroom for memory-heavy workloads.
- **Storage**: A mix — KINGSTON 240 GB (current boot + LVM thin pool), multiple 1 TB SSDs (T-FORCE, Fanxiang, SanDisk, WD Blue, HP), one 931 GB HDD, and more raw drives ready for ZFS pools.
- **Role so far**: Main workhorse, running most VMs/containers.

**Node 2: superman** (the "storage beast")
- **CPU**: Dual Intel Xeon E5-2690 v2 @ 3.00 GHz  
  – 2 sockets × 10 cores × 2 threads = 40 threads total (slightly faster clocks)
- **RAM**: 125 GiB (~128 GB) total — still plenty.
- **Storage**: Massive! Multiple 12.7 TB drives (OOS14000G, Seagate Exos), 10.9 TB drives, 3.6 TB Seagates, 1.8 TB drives — perfect for bulk/Ceph/ZFS replication.
- **Role so far**: Secondary node for replication, backups, and future HA.

Both on Proxmox VE (latest as of 2026), same version, static IPs on the same LAN.

### Why Cluster Them?
- Centralized management (one web UI for both).
- Live migration of VMs between nodes.
- Shared view of storage/VMs.
- Future HA (high availability) for critical services.
- Easy replication/backups across nodes.

### Step 1: Prep Both Nodes Individually
Before clustering:
- Install Proxmox VE on each (same ISO version!).
- Update fully on both:
- Optional use (proxmox helper scripts)(https://community-scripts.github.io/ProxmoxVE/) This a collection of communtiy scripts.

```bash
  apt update && apt full-upgrade -y
  reboot
```

- Disable enterprise repo (no sub needed for home):

```bash
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
apt update
```
