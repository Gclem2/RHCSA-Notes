# Chapter 16: Network File System (NFS)

This chapter covers the concepts and configuration of **Network File System (NFS)** and **AutoFS** on RHEL systems.

## Topics Covered
- Overview of the NFS service and its key components  
- Benefits of NFS and supported versions  
- Exporting directories (shares) on an NFS server  
- Mounting NFS shares on a client using the standard `mount` command  
- Understanding the **AutoFS** service, its benefits, and how it works  
- AutoFS configuration maps  
- Mounting NFS shares using AutoFS  
- Sharing and mounting user home directories with NFS and AutoFS  
## Overview
NFS allows remote file systems to be mounted and accessed on RHEL clients just like local file systems. These mounts can be handled manually using standard mount tools or automatically using **AutoFS**.

AutoFS dynamically mounts file systems when they are accessed and unmounts them after inactivity, reducing resource usage and administrative overhead. In real-world environments, NFS is commonly paired with AutoFS to provide scalable, efficient file sharing, including centralized user home directories.

This chapter walks through both server-side and client-side configurations and demonstrates practical use cases for NFS and AutoFS.

---
# Network File System (NFS)

**NFS** is a protocol for sharing files over a network using a **client/server architecture**:

- **NFS Server:** Provides shares (files, directories, or entire file systems) for remote access. The process of making shares available is called **exporting**.  
- **NFS Client:** Accesses exported shares from the server as if they were local. The process of making them available locally is called **mounting**.  

A system can act as both **server and client** simultaneously.  
- Exported shares include the full directory tree beneath them.  
- A subdirectory or parent of an exported share cannot be re-exported within the same file system.  
- Mounted shares cannot be re-exported.  
- Each exported share is mounted to a **single mount point** on the client.

---
# NFS Versions

RHEL 9 supports **NFSv3, NFSv4.0, NFSv4.1, and NFSv4.2** (default: 4.2):

- **NFSv3:**  
  - Supports TCP & UDP  
  - Asynchronous writes  
  - 64-bit file sizes (files >2GB)  

- **NFSv4.x (4.0, 4.1, 4.2):**  
  - Built on NFSv3 features  
  - Internet-friendly, firewall-transit capable  
  - Enhanced security with optional encrypted transfers  
  - Uses usernames/groups instead of UIDs/GIDs  
  - Improved scalability, cross-platform support, and crash handling  
  - Protocol defaults:  
    - 4.0 & 4.1 → TCP (UDP optional)  
    - 4.2 → TCP only

---
# Export Share on NFS Server

1. Install the NFS software called nfs-utils:
```bash
sudo dnf -y install nfs-utils
```
2. Create /common directory to be exported as a share:
```bash
sudo mkdir /common
```
3. Add permissions to common
```bash
sudo chmod 777 /common
```
4. Add the NFS service persistently to the Linux firewall to allow NFS traffic
```bash
sudo firewall-cmd --permanent -add-service nfs
```
5. Start the NFS service and enable it to autostart
```bash
sudo systemctl --now enable nfs-server
```
6. Verify the operational status of the NFS service:
```bash
sudo systemctl status nfs-server
```
7. Open the /etc/exports file in a text editor and add an entry for /common to export it to server10 with read/write
```bash
sudo vim /etc/exports
#In file
/common server10(rw)
```
8. Export the entry defined in /etc/exports file. The -a option exports all entries
```bash
sudo exportfs -av
```
9. Unexport the entry 
```bash
sudo exportfs -u server10:/common
```

---
# Mount Share on FS Client
`
![](../RHCSA_Labs/attachment/Pasted%20image%2020260116165712.png)
```bash
sudo dnf -y install nfs-utils
```