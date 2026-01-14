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
# Logical Volume Manager (LVM)

- Linux block storage management solution
- Adds an **abstraction layer** between disks and filesystems
- Enables flexible, dynamic storage usage

## LVM Structure
- **Physical Volume (PV)** – disk or partition
- **Volume Group (VG)** – pool of storage from PVs
- **Logical Volume (LV)** – virtual partition from VG

## Key Features
- Resize filesystems and volumes **online**
- Span volumes across **multiple disks**
- Dynamic storage allocation
- Online data migration
- User-defined naming
- **Mirroring, striping, snapshots**

## Extents
- **Physical Extents (PE)** – chunks of PV space
- **Logical Extents (LE)** – mapped chunks used by LVs

---
# Physical Volume

- LVM-managed **disk or partition**
- Created by initializing a block device for LVM
- Stores LVM **label + metadata**

## PV Metadata
- Label stored on **2nd sector**
- Contains **UUID, size, metadata pointers**
- Metadata is **duplicated at disk end** for redundancy
- Remaining space is usable for data

## PV Commands
- `pvs` – list physical volumes
  - Shows size, VG, format, status (`a` = allocatable), free space
- `pvs -v` – verbose PV details

---
# Volume Group (VG)

- Created by adding **one or more Physical Volumes (PVs)**
- Combines PV space into a **single storage pool**
- PVs may be **different sizes**

## VG Metadata
- Stored on **each PV** (duplicated in two locations)
- Contains:
  - VG name
  - Creation info
  - Extent size
  - PV and LV lists
  - PE ↔ LE mappings

## Naming
- Custom names (e.g., `vg01`, `vgweb`, `vgora`)

## VG Commands
- `vgs` – list volume groups
  - Shows #PV, #LV, #SN
  - Attributes:
    - `w` = writeable
    - `z` = resizable
    - `n` = normal
  - Displays VG size (`VSize`) and free space (`VFree`)
- `vgs -v` – verbose VG details

---
# Physical Extent (PE)

- Smallest **allocatable unit** in LVM
- Created when a PV is added to a VG
- Default size: **4 MB** (configurable at VG creation)

## Key Points
- All PVs in a VG use the **same PE size**
- Example: 20 GB PV ≈ **5,000 PEs**

## Command
- `vgdisplay | grep "PE Size"` – view PE size for a VG

---
# Logical Volume (LV)

- Created from **Volume Group (VG)** space
- Acts like a **virtual partition**
- Can span **multiple Physical Volumes**

## Features
- Create/delete **online**
- Expand/shrink **online**
- Flexible allocation across PVs

## Naming
- Default: `lvol0`, `lvol1`, …
- Custom names (e.g., `root`, `swap`, `webdata1`)

## LV Commands
- `lvs` – list logical volumes
  - Attributes:
    - `w` = writeable
    - `i` = inherited allocation
    - `a` = active
    - `o` = open
- `lvs -v` – verbose LV details

---
# Logical Extent (LE)

- Building block of a **Logical Volume (LV)**
- Each LE maps to a **Physical Extent (PE)**
- Can be **contiguous or non-contiguous**

## Key Facts
- LE size = **PE size**
- Default size: **4 MB**
- Larger LVs contain **more LEs**

## Command
- `lvdisplay` – shows LV details
  - LE size inferred:
    - `LV size (MB) ÷ Current LE count ≈ LE size`

---
# LVM Operations & Commands

- LVM provides commands to **create, remove, extend, reduce, rename, and display** storage objects
- All commands support `-v` for verbose output

## Create / Remove
- `pvcreate` / `pvremove` – initialize/uninitialize PVs
- `vgcreate` / `vgremove` – create/remove VGs
- `lvcreate` / `lvremove` – create/remove LVs

## Extend / Reduce
- `vgextend` / `vgreduce` – add/remove PVs to/from VGs
- `lvextend` / `lvreduce` – grow/shrink LVs
- `lvresize` – resize LV  
  - `-r` also resizes filesystem (`fsadm`)

## Rename
- `vgrename` – rename VG
- `lvrename` – rename LV

## List / Display
- `pvs` / `pvdisplay` – PV info
- `vgs` / `vgdisplay` – VG info
- `lvs` / `lvdisplay` – LV info

## Practice Setup
- Use `lsblk` to confirm disks
- Disks **sdd** and **sde** used for LVM exercises
---
# Create Physical Volume and Volume Group

1. Create a parition of size 90MB on sdd using the parted command and confirm.
```bash
sudo parted /dev/sdd mklabel msdos
sudo parted /dev/sdd mkpart primary 1 91m
sudo parted /dev/sdd print
```

![](../RHCSA_Labs/attachment/Pasted%20image%2020260111141452.png)
2. Initialize the sdd1 parition and the sde disk using the pvcreate command.
```bash
sudo pvcreate /dev/sdd1 /dev/sde -v
```

3. Create vgbook volume group using the vgcreate command and add the two physical volumes. -s option to specify the PE size in MBs
```bash
sudo vgcreate -vs 16 vgbook /dev/sdd1 /dev/sde
```

4. List the volume group information:
```bash
sudo vgs vgbook
```

5. Display detailed information about the volume group and the physical volumes it contains:
```bash
sudo vgdisplay -v vgbook
```
6. List the physical volume information:
```bash
sudo pvs
```
7. Display detailed information about the physical volumes: 
```bash
sudo pvdisplay /dev/sdd1
sudo pvdisplay /dev/sde
```

---
# Create Logical Volumes

1. Create a logical volume with the default name lvol0 using lvcreate
```bash
sudo lvcreate -vL 120 vgbook
```
2. Create lvbook1 of size 192MB using lvcreate command.
```bash
sudo lvcreate -l 12 -n lvbook1 vgbook
```
3. List the logical volume information:
```bash
sudo lvs
```

4. Display detailed information about the volume group including the logical volumes and the physical volumes:
```bash
sudo vgdisplay -v vgbook
sudo lvdisplay /dev/vgbook/lvol0
```

---
# Extend a Volume Group and a Logical Volume 

1. Create a partition of size 158MB on sdd using the parted command.
```bash
sudo parted /dev/sdd mkpart pri 92 250
sudo parted /dev/sdd set 2 lvm on
sudo parted /dev/sdd print
```

2. Initialize sdd2 using the pvcreate command:
```bash
sudo pvcreate /dev/sdd2

```
3. Extend vgbook by adding the new physical volume to it:
```bash
sudo vgextend vgbook /dev/sdd2
```

4. List the volume group:
```bash
sudo vgs
```
5. Extend the size of lvbook1 to 340MB by adding 144MB using the lvextend ocmmand:
```bash
sudo lvextend -L +144 /dev/vgbook/lvbook1
```

6. Issue vgdisplay on vgbook with the -v switch for the updated details:
```bash
 sudo vgdisplay -v vgbook
```
7. View a summary of physical volume
```bash
sudo pvs
```
8. View a summary of the logical volume
```bash
sudo lvs
```

---
# Rename, Reduce, Extend, and Remove Logical Volumes
In this exercise, you will rename lvol0 to lvbook2. You will decrease the size of lvbook2 to 50MB using the lvreduce command and then add 32MB with the lvresize command. You will then remove both logical volumes. You will display the summary for the volume groups, logical volumes, and physical volumes.

1. Rename lvol0 to lvbook2 using the lvrename command and confirm
```bash
sudo lvrename vgbook lvol0 lvbook2
sudo lvs
```
2. Reduce the size of lvbook2 to 50MB with the lvreduce command. Specify the absolute desired size for the logical volume
```bash
sudo lvreduce -L 50 /dev/vgbook/lvbook2
```
3. Add 32MB to lvbook2 with the lvresize command:
```bash
sudo lvresize -L +32 /dev/vgbook/lvbook2
```
4. Use the pvs, lvs, vgs, and vgdisplay commands to view the updated allocation
5. Remvoe both lvbook1 nad lvbook2 with lvremove command

```bash
sudo lvremove /dev/vgbook/lvbook1 -f
sudo lvremove /dev/vgbook/lvbook2 -f
```
6.  Execute the vgdisplay command and grep for "Cur LV" to see the number of logical volumes available

```bash
sudo vgdisplay vgbook | grep 'Cur LV'
```

---
# Reduce and Remove a Volume Group

1. Remove sdd1 and sde physical volumes from vgbook by issuing the vg reduce command:
```bash
sudo vgreduce vgbook /dev/sdd1 /dev/sde
```

2. Remove the volume group using the vgremove command

```bash
sudo vgremove vbook
```

3. Execute the vgs and lvs commands for confirmation:
```bash
sudo vgs
sudo lvs
```

---
# Uninitialize Physical Volumes

1. Remove the LVM structures from sdd1, sdd2, and sde using the pvremove command:
```bash
sudo pvremove /dev/sdd1 /dev/sdd2 /dev/sde
```
2. Confirm the removal using the pvs command:
```bash
sudo pvs
```
3. Remove the partitions from sdd using the parted command:
```bash
sudo parted /dev/sdd rm 1; sudo parted /dev/sdd rm 2
```

4. Verify that all disk return to previous state
```bash
lsblk
```

---
# Storage Optimization with Virtual Data Optimizer (VDO)
- **Device driver layer** between kernel and physical storage
- Goals:
  - Conserve disk space
  - Improve data throughput
  - Reduce storage costs

## Technologies Used
- **Thin provisioning**
- **Deduplication**
- **Compression**

---
# How VDO Conserves Storage

VDO optimizes storage in **three stages**:

## 1. Zero-Block Elimination
- Uses **thin provisioning**
- Removes empty (zero-byte) blocks
- Moves data to **contiguous locations**

## 2. Deduplication
- Detects and avoids writing **duplicate data**
- Uses **UDS (Universal Deduplication Service)** kernel module

## 3. Compression
- Compresses remaining data blocks
- Uses **kvdo** kernel module
- Consolidates data into fewer blocks

## Notes
- Runs **in the background**
- Low **CPU and memory** overhead

---
# VDO Integration with LVM

- RHEL 9 uses **LVM-based VDO**
- No separate VDO tools required
- **LVM utilities** manage VDO volumes

## VDO Components
- **VDO Pool**
  - Deduplicated storage
  - Implemented as an **LVM logical volume**
- **VDO Volume**
  - LVM logical volume provisioned from a pool
  - Must be **formatted** before use

## Required Packages (default)
- `vdo` – VDO management support
- `kmod-kvdo` – virtualization, thin provisioning, compression

---
# Create a LVM VDO Volume

1. Initialize the sde disk using the pvcreate command:
```bash
sudo pvcreate /dev/sde
```
2. Create vgvdo volume group using he vgcreate command:
```bash
sudo vgcreate vgvdo /dev/sde
```
3. Display basic information about the volume group:
```bash
sudo vgdisplay vgvdo
```
4. Create a VDO volume called lvvdo using the lvcreate command
```bash
sudo lvcreate --type vdo -l 1279 -n lvvdo -V 20G vgvdo
```
5. Display 
```bash
sudo vgdisplay -v vgvdo
```

---
# Remove a Volume Group and Uninitialize Physical Volume

1. Remvoe the volume group along with the VDO volumes using the vgremove command:
```bash
sudo vgremove vgvdo -f
```
2. Execute sudo vgs aand sudo lvs commands  for confirmation
3. Remove the LVM structures from sdf using the pvvremove command:

```bash
sudo pvremove /dev/sdf
```

---
# Chapter 14 – Local File Systems & Swap (RHCSA)

## File Systems
- Logical containers for file storage
- Created on **partitions or logical volumes**
- Can be mounted/unmounted **independently**
- Must be attached to the **directory tree** to be accessible

## File System Types
- **ext3 / ext4** – traditional Linux FS
- **XFS** – high-performance, scalable
- **VFAT** – removable media / cross-platform
- **ISO9660** – optical media

## Mounting
- Mount **manually** or **persistently at boot**
- Identify FS using:
  - Device file (`/dev/sda1`)
  - **UUID**
  - **Label**
- Persistent mounts configured in `/etc/fstab`

## File System Management
- Create, mount, unmount file systems
- Resize **ext4** and **XFS** (often via LVM)
- Apply and use **labels**
- Monitor usage:
  - File systems
  - Directories

## LVM + File Systems
- Create and mount FS on **logical volumes**
- Extend logical volumes and resize FS
- Non-destructive storage expansion

## Swap
- Disk space used as **virtual memory**
- Moves idle memory pages between RAM and disk
- Acts as an **extension of physical memory**
- Can exist in:
  - Partitions
  - Logical volumes
- Swap spaces can be **activated/deactivated independently**

---
# File Systems & Types

## File Systems
- Logical containers for **files and directories**
- Created on **partitions, LVs, or VDO volumes**
- Must be **mounted** to be accessible

## Default RHEL Layout
- Mandatory: `/` , `/boot`
- Common optional FS:
  - `/home`, `/var`, `/usr`, `/opt`, `/tmp`

## Benefits of Separate File Systems
- Mount/unmount independently
- Isolated repairs
- Separate dissimilar data
- Independent tuning
- Grow/shrink independently

## File System Categories
- **Disk-based** – persistent local storage
- **Network-based** – shared over network
- **Memory-based** – temporary, non-persistent

## Common File System Types
- **Ext3** – journaling, reliable, up to 32k subdirs
- **Ext4** – larger FS/files, unlimited subdirs, journaling
- **XFS** – 64-bit, high performance, scalable, default in RHEL 9
- **VFAT** – removable/cross-platform media
- **ISO9660** – optical media (CD/DVD)
- **NFS** – network-shared FS
- **AutoFS** – auto-mounting NFS

---
# # Extended File Systems (ext3 / ext4)

- Longstanding Linux file system family
- ext1 obsolete; **ext2, ext3, ext4 supported**
- **ext4** is the latest and most feature-rich

## Structure
- Created on **partition or logical volume**
- Two parts:
  - **Metadata** (small)
  - **Data blocks** (majority of space)

## Metadata Components
- **Superblock**
  - FS type, size, status, block count
  - **Replicated** across FS
  - Primary + backup superblocks
- **Inode Table**
  - One inode per file
  - Stores metadata (type, perms, owner, size, timestamps)
  - Points to data blocks

## Journaling (ext3 / ext4)
- Logs metadata changes in a **journal**
- Enables **fast crash recovery**

## ext3 vs ext4
- **ext3**
  - FS ≤ 16 TiB
  - File ≤ 2 TiB
- **ext4**
  - FS ≤ **1 EiB**
  - File ≤ **16 TiB**
  - Uses **extents** (contiguous blocks)
  - Less fragmentation, better performance
  - Supports extended attributes, quota & metadata journaling

---
# XFS File System

- **64-bit**, high-performance, extent-based FS
- Default file system in **RHEL 9**
- Supports very large storage:
  - FS & files up to **8 EiB**

## Features
- **Metadata journaling** for crash consistency
- No automatic FS check at boot
  - Use `xfs_repair` for repairs
- Online **defragmentation**
- Extended attributes and mount options enabled by default
- Supports **snapshots** on mounted, active FS

## Caveat
- **Cannot shrink** XFS file systems

## Recovery
- Journal replayed on remount after crash/unmount

---
# VFAT File System

- Extension of legacy **FAT16**
- Introduced with **Windows 95**
- Supported by **Linux, Windows, macOS**

## Features
- Filenames up to **255 characters**
- Allows **spaces and periods**
- **Case-insensitive** (no upper/lower distinction)

## Usage
- Primarily used on **removable media**
  - USB flash drives
  - Floppy disks
- Common for **data exchange** between Linux and Windows

## Limitations
- No UNIX-style permissions
- Case-insensitive filenames

---
# ISO9660 File System

- Conforms to the **ISO 9660 standard**
- Used on **optical media**
  - CD-ROM
  - DVD-ROM

## Purpose
- Transport **software**, **patches**, and **OS images**
- Common format for **.iso files**

## Background
- Derived from **High Sierra File System (HSFS)**
- Later enhanced with additional features

## Characteristics
- Typically **read-only**
- Designed for **cross-platform compatibility**

---
# File System Management

## Common Tasks
- Create and remove file systems  
- Mount and unmount  
- Label and view  
- Grow or shrink file systems  

## Applicability
- **Ext3 / Ext4 / XFS**: full support
- **VFAT**: most operations
- **Optical (ISO9660)**: limited operations

## Context
- Partitions and LVM volumes were previously created but **not formatted**
- Unformatted devices cannot be mounted or used
- All partitions, LVs, and the VG were deleted

## Current State
- Disks **sdb–sdf** are unused and available for reuse

---
# File System Administration Commands

## Extended (Ext3/Ext4)
- **e2label**: change file system label  
- **tune2fs**: view/tune file system attributes  

## XFS
- **xfs_admin**: modify attributes  
- **xfs_growfs**: extend file system  
- **xfs_info**: display file system info  

## General (All FS Types)
- **blkid**: show UUIDs and labels  
- **df**: file system usage  
- **du**: directory disk usage  
- **fsadm**: resize file systems (used by `lvresize -r`)  
- **lsblk**: list block devices and file systems  
- **mkfs**: create file systems  
- **mount / umount**: mount or unmount file systems

---
# Mounting & Unmounting File Systems

## Mounting
- Connects a file system to the **directory hierarchy** at a **mount point** (empty directory)
- Examples: `/` (root), `/boot`
- Command: `mount <device|UUID|label> <mount_point>`  
  - Requires **root privileges**
  - Kernel entry added in `/proc/self/mounts`
- Mount point must be **empty** and **not in use**
- Can filter by FS type: `mount -t xfs`
- Common options:
  - `auto / noauto` – mount automatically with `-a` or not
  - `defaults` – default options (rw, async, auto)
  - `_netdev` – network required (e.g., NFS)
  - `remount` – modify options on mounted FS
  - `ro / rw` – read-only or read/write

## Unmounting
- Command: `umount <device|mount_point>`  
  - Detaches FS, removes entry from `/proc/self/mounts`
- Can unmount all or filter by FS type

---
# Determining File System UUIDs

- **UUID (Universally Unique Identifier)** uniquely identifies a file system
  - **Ext / XFS**: 128-bit (32 hex characters)  
  - **VFAT**: 32-bit (8 hex characters)
- Persistent across **reboots**
- Used by default in `/etc/fstab` for automatic mounting

## Commands to View UUID
- **Extended FS**: `tune2fs`, `blkid`, `lsblk`  
- **XFS**: `xfs_admin`, `blkid`, `lsblk`  
- Example: `/boot` FS UUID: `22d05484-6ae1-4ef8-a37d-abab674a5e35`

## Notes
- LVM / VDO volumes also get UUIDs  
- Not mandatory for fstab if device files are unique and persistent

---
# Automatically Mounting File Systems at Reboot

- File systems in **/etc/fstab** are mounted automatically at boot
- Must contain **accurate entries** to avoid unbootable system
- Using fstab allows **mount / umount** commands with just one attribute: device, UUID, label, or mount point

## /etc/fstab Columns
1. **Device**: physical/virtual path, UUID, or label  
2. **Mount Point**: directory to attach FS (`none` or `swap` for swap)  
3. **Type**: FS type (ext3, ext4, xfs, vfat, iso9660, swap, auto)  
4. **Options**: comma-separated mount options (see `mount` manual)  
5. **Dump**: 0 disables dump check (Extended FS only)  
6. **Fsck Order**: sequence for `e2fsck` at boot  
   - 0 = skip, 1 = root (`/`), 2 = other physical FS  
   - XFS, virtual, remote, removable FS ignore columns 5 & 6

**EXAM TIP:** Any missing/invalid fstab entry may prevent boot; fix via **emergency mode**. Edit carefully to avoid syntax errors.

---
# Monitoring File System Usage

- Use **`df`** to check file system usage (used, available, total space)  
  - Default: KB  
  - `-m`: MB, `-h`: human-readable  

## Example
```bash
df -h
```

---
# Create and Mount Ext4, VFAT, and XFS File Systems in Partitions

1. Apply the label msdos to the sdb disk using the parted command:
```bash
sudo parted /dev/sdb mklabel msdos
```
2. Create 2 x 100MB primary partiitons on sdb with the parted command:
```bash
sudo parted /dev/sdb mkpart primary 1 101m
sudo parted /dev/sdb mkpart primary 102 201m
```
3. Initialize the first partition (sdb1) with Ext4 file system type using the mkfs command:
```bash
 sudo mkfs -t ext4 /dev/sdb1
```
4. Initialize the second partition (sdb2) with VFAT file system type using the mkfs command:
```bash
sudo mkfs -t vfat /dev/sdb2
```
5. Initialize the whole disk (sdc) with the XFS file system type using the mkfs.xfs command. Add the -f flag to force the removal of any old partitioning or labelign information from the disk.
```bash
sudo mkfs.xfs /dev/sdc -f
```
6. Determine the UUIDs for all three file sstems using the lsblk command:

```bash
sudo lsblk -f /dev/sdb /dev/sdc
```

7. Open the /etc/fstab file, go to the end of the file, and append entries for the file systems for persistence using their UUIDs:
![](../RHCSA_Labs/attachment/Pasted%20image%2020260113172821.png)
8. Create mount points /ext4fs, /vfatfs1, and /xfsfs1
```bash
sudo mkdir /ext4fs1 /vfatfs1 /xfsfs1
```
9. Mount the new file systems using the mount command
```bash
sudo mount -a
```
10. Verify 
```bash
df -hT
```

---
# Create and Mount Ext4 and XFS File Systems in LVM Logical Volumes

1. Create a 172MB partition on the sdd disk using the parted command:
```bash
sudo parted /dev/sdd mkpart pri 1 172
```
2. Initialize the sdd1 partition for use in LVM using the pvcreate command:
```bash
sudo pvcreate /dev/sdd1
```
3. Create the volume group vgfs with a PE size of 16MB using the physical volume sdd1:
```bash
sudo vgcreate -s 16 vgfs /dev/sdd1
```
4. Create two logical volumes ext4vl and xfsvol of size 80MB
```bash
sudo lvcreate -n ext4vol -L 80 vgfs
sudo lvcreate -n xfsvol -L 80 vfgs
```
5. Format the ext4vol loggical volume with ext4 file system
```bash
sudo mkfs.ext4 /dev/vgfs/ext4vol
```
6. Format the xfsvol logical volume with the XFS file system type using mkfs.xfs command:
```bash
sudo mkfs.xfs /dev/vgfs/xfsvol
```
 7. Open the /etc/fstab file, go to the end of the file, and append entries for the file systems for persistence using their device files:
![](../RHCSA_Labs/attachment/Pasted%20image%2020260113181100.png)
8. Create the mount points
```bash
sudo mkdir /ext4fs2 /xfsfs2
```
9. Mount the new filesystem using mount command

```bash
sudo mount -a
```
10. Confirm mount 
![](../RHCSA_Labs/attachment/Pasted%20image%2020260113181337.png)

---
# Resize Ext4 and XFS File Systems in LVM Logical Volumes

1. Initialize the sde disk and add it to the vgfs volume group:
```bash
sudo pvcreate /dev/sde
sudo vgextend vgfs /dev/sde
```
2. Confirm the new size of vgfs using vgs
```bash
sudo vgs
```
3. Grow the logical volume ext4vol and the file system it holds 40MB
```bash
sudo lvextend -L +40 /dev/vgfs/ext4vol
sudo fsadm resize /dev/vgfs/ext4vol
```
4. Grow the logical volume xfsvol and the file systemm it holds by 40MB
```bash
sudo lvresize -r -L +40 /dev/vgfs/xfsvol
```
5. Verify the new extensions to both logical volumes using the lvs command
```bash
sudo lvs | grep vol
```
6. Check the new sizzes and the current mount status for the file systems 
```bash
df -hT | grep -E 'ext4vol|xfsvol'
lsblk /dev/sdd /dev/sde
```
![](../RHCSA_Labs/attachment/Pasted%20image%2020260114143156.png)

---
# Create and Mount XFS File System in LVM VDO Volume

1. Initialize the sdc disk using the pvcreate command:
```bash
sudo pvcreate /dev/sdc
```
2. Create vgvdo1 volume group using the vgcreate command:
```bash
sudo vgcreate vgvdo1 /dev/sdc
```
3. Display basic information aboutt the volume group:
```bash
sudo vgdisplay vgvdo1
```
4. Create a VDO voluume called lvvdo1 using lvcreate command
```bash
sudo lvcreate -n lvvdo1 -l 1279 -V 20G --type vdo vgvdo1
```






---
# Swap and Its Management

## Overview
- **Physical memory (RAM)** is limited and used by the kernel and running processes.
- **Swap space** is disk-based storage that extends RAM by holding idle memory pages.

## How Swap Works
- Memory is divided into small units called **pages**.
- The kernel tracks page locations using a **page table**.
- When free RAM is sufficient, no swapping occurs.
- When free RAM drops below a high threshold:
  - Idle pages are moved from RAM to swap (**page out**).
- When those pages are needed again:
  - They are moved back into RAM (**page in**).
- This process is called **demand paging**.

## Performance Impact
- Excessive paging in and out leads to **thrashing**.
- Thrashing consumes CPU cycles and degrades system performance.

## System Protection Mechanism
- If memory drops below a low threshold:
  - Idle processes are deactivated.
  - New processes are temporarily blocked.
- Normal operation resumes once free memory increases and thrashing stops.

---
# Determining Current Swap Usage

## Swap Size Guidelines
- Swap is often **equal to or larger than RAM**, depending on workload.
- Systems with **large RAM** may have **less swap** than physical memory.

## Viewing Memory and Swap Usage
- Use the `free` command to view RAM and swap utilization.
- Common options:
  - `-h` : human-readable units
  - `-k`, `-m`, `-g` : KB, MB, GB
  - `-t` : show totals
  - `-s <sec>` : refresh interval
  - `-c <count>` : number of updates

### Example
```bash
free -h
```

---

# Prioritizing Swap Spaces

- Systems may have **multiple active swap areas**.
- By default, RHEL uses swap **in activation order**.
- Swap priority is set in `/etc/fstab` using the `pri=` option.

## Priority Rules
- Range: **-2 to 32767**
- Default: **-2**
- **Higher value = higher priority**
- Swap areas with the **same priority are used alternately**

## Example `/etc/fstab` Entry
```text
/dev/sdb1  swap  swap  defaults,pri=10  0  0

```

---
# Swap Administration Commands

RHEL provides three main commands for swap management:

| Command   | Description |
|-----------|-------------|
| `mkswap`  | Initializes a partition or file for use as swap space. |
| `swapon`  | Activates a swap area so it can be used by the system. |
| `swapoff` | Deactivates a swap area, making it unavailable. |

## Usage Workflow
1. **Create swap area**
```bash
sudo mkswap /dev/sdb1
```

---
