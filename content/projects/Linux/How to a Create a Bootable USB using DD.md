---
title: How to Create a Bootable USB from an ISO Using the dd Command
date: 2026-02-13
draft: false
tags: [linux, usb, bootable, dd, tutorial, command-line]
categories: [Linux Tutorials, System Administration]
---
Creating a bootable USB drive is one of the most common tasks for Linux users — whether you're installing a new distribution, running a live system, testing rescue tools, or setting up a recovery drive.

One of the most reliable, lightweight, and universal ways to do this on Linux is with the **`dd`** command. It's built into every Linux distribution, requires no extra software, and gives you precise control.

**But first: a very important warning**  
`dd` is incredibly powerful — and famously dangerous. A single typo (like writing to your hard drive instead of the USB) can **permanently erase everything** with no warning and no recycle bin. In the Linux community, people half-jokingly call it the **"disk destroyer"** or **"delete data"** command.  

We'll show you exactly how to use it **safely** in this guide, but always triple-check every command before pressing Enter.

## What is `dd` and what does it stand for?

`dd` is a standard Unix/Linux shell command designed for **low-level copying and conversion** of data. It reads from an input source (`if=`) and writes to an output destination (`of=`), block by block, while optionally converting the data (e.g., changing character sets, block sizes, or byte order).

### The name origin
Officially, `dd` stands for **"Data Definition"** (or sometimes "Dataset Definition").  

This is an allusion to the **`DD`** statement in IBM's Job Control Language (JCL), used on mainframe systems for defining datasets (files) and their attributes. When Unix was being developed in the early 1970s, the creators borrowed the name and syntax style for this utility. (Source: Dennis Ritchie himself confirmed this in old Usenet posts, and it's documented on Wikipedia and in the GNU Coreutils manual.)

**Common nicknames you’ll hear:**
- **Disk Destroyer** or **Delete Data** — because mistakes are catastrophic
- **Data Duplicator** or **Disk Dumper** — more positive spins on its actual purpose

Despite the jokes, when used carefully, `dd` is one of the most trustworthy tools for tasks like:
- Writing ISO files to USB drives
- Creating exact disk/partition images (backups)
- Cloning drives
- Recovering data from failing media

In the next sections, we'll walk through the **safe, step-by-step process** to create a bootable USB from an ISO file using `dd` — including how to identify your USB correctly, modern best-practice flags, and verification steps.

## Step 1: Identify Your USB Device (The Most Critical Step!)

Before running `dd`, you **must** know exactly which device is your USB stick. Getting this wrong is the #1 cause of data loss.

1. Insert your USB drive (at least 8 GB recommended; 16–32 GB ideal).
2. Open a terminal and run:

```bash
   lsblk -o NAME,SIZE,TYPE,RM,MODEL,MOUNTPOINT
```
   - RM=1 indicates removable media (your USB).
   - Match the size to your drive.
Example:

```bash
NAME   SIZE TYPE RM MODEL              MOUNTPOINT
sda    500G disk  0 Samsung_SSD       /
sdb     32G disk  1 Kingston_DataTraveler
└─sdb1  32G part  1                     /run/media/user/USB
```
→ Target the whole device (/dev/sdb), not a partition (/dev/sdb1).


3. For extra safety, check persistent names (these survive reboots):
```bash
ls -l /dev/disk/by-id/ | grep -i usb
```
Example output:
```bash
lrwxrwxrwx 1 root root 9 Feb 13 20:30 usb-Kingston_DataTraveler_3.0_ABC123DEF-0:0 -> ../../sdb
```
→ Use /dev/disk/by-id/usb-... in later commands — it's harder to mistype.

Pro tip: Run lsblk before and after inserting the USB. The new device is your target.

## Step 2: Unmount the USB Device

If the USB auto-mounts (appears in your file manager), unmount it first — writing to a mounted device can fail or corrupt data.
```bash
sudo umount /dev/sdb*  2>/dev/null || true
```
(Replace sdb with your device. * handles all partitions like sdb1/sdb2. Ignore "not mounted" errors.)
## Step 3: Write the ISO to the USB with dd
Now the main command. Use these modern, safe flags:

bs=4M — good speed vs. reliability balance
status=progress — shows real-time bytes transferred & speed
conv=fsync (or oflag=direct,sync) — forces data flush to hardware (prevents early removal corruption)

Recommended command:
```bash
sudo dd if=/path/to/your.iso of=/dev/sdX bs=4M status=progress conv=fsync
```
Even safer variant (using persistent ID + direct I/O for reliability)
```bash
sudo dd if=~/Downloads/ubuntu-24.04.3-desktop-amd64.iso \
    of=/dev/disk/by-id/usb-Kingston_DataTraveler_3.0_ABC123DEF-0:0 \
    bs=4M status=progress conv=fsync oflag=direct
```
- Replace paths/device names!
- It may take 5-20 minutes (USB 3.0+ is faster).
- You'll see progress like:
```text
2,345,678,901 bytes (2.3 GB, 2.2 GiB) copied, 142 s, 16.5 MB/s
```
Important: Do not interrupt this (no Ctrl+c unless emergency). Let id finish.

## Step 4: Flush & Safely Remove the USB
After `dd` completes:
```bash
sync
sudo sync   # extra flush just in case
sudo eject /dev/sdb   # or your device
```
Now safely unplug it.

## Step 5: Verify & Test the Bootable USB
1. Re-insert the USB → run lsblk again. You should see a new partition layout (often 1–3 small ones).

2. Boot from it
    - Restart your computer.
    - Enter BIOS/UEFI (usually F2, Del, Esc, F12).
    - Set USB as first boot device (or use one-time boot menu: often F12).
    - Disable Secure Boot if needed (for some distros).
3. If it boots to the live/installer → success!

If not: Re-check ISO integrity (checksums), try another USB, or test in a VM first.

## Common Mistakes & How to Avoid Them

| Mistake                          | Consequence                     | How to Avoid                          |
|----------------------------------|---------------------------------|---------------------------------------|
| Wrong `of=` (e.g. `/dev/sda`)    | Wipes main drive                | Triple-check with `lsblk`             |
| Write to partition (e.g. `/dev/sdb1`) | Non-bootable / corrupted USB | Always use whole device (`/dev/sdb`) |
| Forget to unmount                | "Device busy" or corruption     | Run `umount` first                    |
| Pull USB before `sync`/eject     | Incomplete write → boot fail    | Always `sync` + `eject`               |
| No `status=progress`             | Looks frozen                    | Add it every time                     |
| Tiny `bs=` (e.g. 512)            | Very slow write                 | Use `bs=4M` or `bs=1M`                |


## Alternatives If dd Feels Too Risky
- Ventoy — Copy multiple ISOs to one USB (no rewriting needed each time).
- balenaEtcher or Fedora Media Writer — GUI tools with verification.
- mkusb — dd wrapper with extra safety checks (great for persistence).

`dd` remains the gold standard for control and universality — just respect its power.
Happy booting! If you run into issues or have horror stories, drop them in the comments. 😄