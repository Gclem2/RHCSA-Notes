# Overview Of Chapter

This chapter covers disk partitioning, thin provisioning, Logical Volume Manager (LVM), and Virtual Data Optimizer (VDO) in RHEL, focusing on efficient storage allocation and management.

## Disk Partitioning
- Disks are divided into **partitions** that can span part of a disk, an entire disk, or multiple disks.
- Each partition is managed independently and may contain a filesystem or swap.
- Partition metadata is stored in special disk locations used during boot.
- RHEL supports multiple partitioning tools that can coexist.

### Partition Table Types
- **MBR (Master Boot Record)** – Legacy partitioning scheme.
- **GPT (GUID Partition Table)** – Modern standard with improved flexibility and scalability.

## Thin Provisioning
- Allocates storage **on demand** rather than upfront.
- Improves storage efficiency by using only required space.
- Commonly used in modern storage management solutions.

## Logical Volume Manager (LVM)
- Provides an **abstraction layer** between the OS and physical storage.
- Uses virtual storage objects for flexible allocation and resizing.

### LVM Components
- **Physical Volumes (PV)** – Disks or partitions used by LVM.
- **Volume Groups (VG)** – Storage pools created from PVs.
- **Logical Volumes (LV)** – Virtual partitions created from VGs.

### LVM Capabilities
- Create, extend, reduce, rename, and remove LVs
- Extend, reduce, and remove VGs
- Add and remove PVs
- Perform storage changes **non-destructively**

## Virtual Data Optimizer (VDO)
- Builds on thin provisioning.
- Uses **deduplication** and **compression** to conserve space.
- Improves throughput and reduces storage costs.

## RHCSA Objectives Covered
- List, create, and delete MBR and GPT partitions
- Create and remove physical volumes
- Assign PVs to volume groups
- Create and delete logical volumes
- Extend existing logical volumes
- Add partitions, logical volumes, and swap non-destructively
---
# Storage Management Overview

- Disks in RHEL are divided into **partitions**
- Partition data is stored in a small disk region:
  - **MBR** – BIOS systems
  - **GPT** – UEFI systems
- At boot, BIOS/UEFI:
  1. Scans storage devices
  2. Detects MBR/GPT
  3. Identifies boot disk
  4. Loads bootloader
  5. Reads partition table
  6. Locates `/boot`
  7. Loads kernel and hands off control

- **MBR and GPT serve the same purpose**: store partition information and boot code

---
# Master Boot Record (MBR)

- Stored in the **first sector** of the boot disk
- Legacy partition scheme for **BIOS/x86 systems**
- Usage declining in favor of **UEFI/GPT**

## Partition Types
- **Primary** – usable for data
- **Extended** – container only (no data)
- **Logical** – usable, stored inside extended

## Limits
- Max **4 primary** partitions (1–4)
- Logical partitions start at **5**
- Max **14 usable partitions** (3 primary + 11 logical)
- Max disk size: **2 TB**
- **Non-redundant** → corruption can make system unbootable

## When to Use
- Disk ≤ 2 TB
- ≤ 14 partitions required

---
# GUID Partition Table (GPT)

- Modern **64-bit** partitioning standard
- Integrated with **UEFI**
- Designed for disks **> 2 TB**

## Features
- Up to **128 partitions**
- **No primary/extended/logical** distinction
- Supports **4 KB sector size**
- **Redundant partition table** stored at disk end

## Boot Support
- BIOS systems can boot via **protective MBR**
- UEFI supports **Secure Boot** (signed binaries only)

## Advantages over MBR
- Larger disks
- More partitions
- Redundancy and reliability

---
# Disk Partitions

- Disk space is divided into **partitions**
- Avoid **overlapping partitions** and **unused gaps** to prevent data loss/waste

## Device Naming
- First disk: **sda**
- Partitions: **sda1, sda2, …**
- Additional disks: **sdb, sdc, sdd, …**
- `s` = SATA / SAS / SCSI

## Listing Disks
- `lsblk` – shows disks, partitions, mount points, and LVM layout

### Example (server1)
- One disk: **sda (20GB)**
- **sda1** → `/boot`
- **sda2** → LVM PV
  - Logical volumes:
    - `root` (~17GB)
    - `swap` (~2GB)
- `sr0` → mounted ISO (optical media)

## Partition Tools
- `fdisk -l` – detailed partition info
  - Disk size, label type (e.g., `dos`)
  - Partition start/end sectors
  - Boot flag (`*`)
  - Partition types:
    - `83` → Linux
    - `8e` → LVM

---
# Storage Management Tools

- RHEL provides multiple **storage management tools**
- Partitions created by different tools can **coexist on the same disk**

## Tools
- **parted**
  - Supports **MBR and GPT**
  - Simple, general-purpose tool
- **gdisk**
  - **GPT-only**
  - Alternative to `parted`
- **LVM**
  - Feature-rich logical volume manager
  - Enables flexible and dynamic storage management

---
# Thin Provisioning

- Allocates storage **on demand**
- Moves data to **contiguous blocks** → eliminates empty space
- Supported in **LVM**

## Key Concepts
- Create a **thin pool**
- Assign volumes **larger than physical capacity**
- Actual space used only when data is written
- Monitor usage thresholds (e.g., **80%**)
- Expand pool dynamically by adding storage

## Benefits
- Efficient space utilization
- Lower upfront storage costs

---
# MBR/GPT Storage Management with parted

- **parted** – partition editor in RHEL
  - Supports **MBR and GPT**
  - Can create up to **128 partitions** on GPT
  - Run **interactively** or from **command line**

## Common Subcommands
| Subcommand | Description                                                                |
| ---------- | -------------------------------------------------------------------------- |
| `print`    | Shows partition table (geometry, start/end, size, type, filesystem, flags) |
| `mklabel`  | Sets disk label (`gpt` or `msdos`)                                         |
| `mkpart`   | Creates a new partition                                                    |
| `name`     | Assigns a name to a partition                                              |
| `rm`       | Removes a partition                                                        |

- After operations:
  - Check partitions with `print`
  - `/proc/partitions` reflects changes

---
# Create an MBR Partition

1. Execute parted on /dev/sda to view the current partition information:
```bash
sudo parted /dev/sda print
```

2. Assign disk label to the disk with mklabel. This operation is performed only once on a disk.
```bash
sudo parted /dev/sda mklabel msdos
```

3. Create a 100MB primary partition started at 1MB using mkpart:
```bash
sudo parted /dev/sda mkpart primary 1 101m
```
4. Verify with print:
```bash
sudo parted /dev/sda print
```

5. Confirm the new partition with the lsblk command:
```bash
lsblk /dev/sda
```
6. Check the /proc/partitions file also:
```bash
cat /proc/partitions | grep sda
```

---
# Deleting an MBR Partition

1. Execute parted on /dev/sda with the rm subcommand to remove partition number 1:
```bash
sudo parted /dev/sda rm 1
```
2. Confirm the partition deletion with print:
```bash
sudo parted /dev/sda print
```
3. Check the /proc/partitions file:
```bash
cat /proc/partitions | grep sda
```

---
# GPT Storage Management with gdisk

- **gdisk** – text-based, menu-driven GPT partitioning tool
  - Supports **UEFI systems**
  - Can create up to **128 partitions** per disk
  - Operations: show, add, verify, modify, delete partitions

## Usage
- Invoke with disk device: `/dev/sdc`
- Type `help` or `?` at prompt to see subcommands
- Enter `q` to quit and return to command line

- Ideal for **GPT-only disks** as an alternative to `parted`
---
# Create a GPT Partition

1. Execute gdisk on /dev/sdc to view the current partition information:
```bash
sudo gdisk /dev/sdc
```
2. Assign gpt as the partition table type to the disk using the o subcommand
![](../RHCSA_Labs/attachment/Pasted%20image%2020260109214643.png)
3. Run the p subcommand to view disk information and confirm the GUID partition table creation:
![](../RHCSA_Labs/attachment/Pasted%20image%2020260109214810.png)
4. Create the first partition of size 200MB starting at the default sector with default type "Linux filesystem" using n
![](../RHCSA_Labs/attachment/Pasted%20image%2020260109215047.png)
5. Verify the new partition with p:
![](../RHCSA_Labs/attachment/Pasted%20image%2020260109215132.png)

6. Run w to write the partition information to the partition table and exit out of the interface.
![](../RHCSA_Labs/attachment/Pasted%20image%2020260109215351.png)

7. Verifyy the new partition by issuing either of the follow at the command prompt:
	```bash
	grep sdc /proc/partitions
	```

```bash
	lsblk /dev/sdc
	```

---
# Delete a GPT Partition

1. Execute gdisk on /dev/sdc and run d1 at the utility's propmt to delete:
```bash
sudo gdisk /dev/sdc
```
![](../RHCSA_Labs/attachment/Pasted%20image%2020260109215945.png)
2. Confirm the partition deletion with p:
![](../RHCSA_Labs/attachment/Pasted%20image%2020260109220023.png)
3. Write the updated partition information to the disk with w and quit gdisk:
![](../RHCSA_Labs/attachment/Pasted%20image%2020260109220132.png)
4. Verify the partion deletion
```bash
lsblk /dev/sdc
```

---
