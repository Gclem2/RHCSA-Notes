# Chapter 15: Networking, Network Devices, and Network Connections

## Major Topics

This chapter covers:

- **Networking fundamentals**: hostname, IPv4, network classes, subnetting, subnet mask, CIDR, protocols, TCP/UDP, well-known ports, ICMP, Ethernet address, IPv6, IPv4/IPv6 differences, consistent device naming.  
- **Hostname management**: changing and verifying system hostname.  
- **Network devices and connections**: understanding device types, connection profiles, and their anatomy.  
- **Network management tools**: commands and techniques to manage devices and connections.  
- **Network configuration**: manually or via commands for both IPv4 and IPv6.  
- **Hosts table**: purpose and use.  
- **Connectivity testing**: using hostnames and IP addresses.  
## Overview

A computer network consists of two or more computers connected to share resources and data. Connections may be wired or wireless, often linked via switches for communication. Understanding network concepts and tools is essential for configuring, managing, and troubleshooting network devices and connections.

### Key Points:

- Each system must have at least **one network device** with a **connection profile**.  
- A connection profile contains:
  - IP address (static or DHCP)
  - Hostname
  - Other essential network parameters  
- Configuration can be applied manually (editing files) or via commands.  
- After configuration, connectivity testing ensures proper communication.

---
## Hostname

A **hostname** is a unique alphanumeric label assigned to a node to identify it on a network. Allowed characters include:

- Letters and numbers
- Hyphen (`-`)
- Underscore (`_`)
- Period (`.`)  

Hostnames can be up to **253 characters** long and are usually chosen based on the system's **purpose** or **role**.  

In RHEL, the hostname is stored in:


---
# Change a hostname

```bash
sudo vim /etc/hostname
sudo systemctl restart systemd-hostnamed.service
```

---
# IPv4 Address

**IPv4 (Internet Protocol version 4)** is a **32-bit unique software address** assigned to every network entity to enable communication. It is the first IP version released for public use. IPv4 addresses are also called **dotted-quad addresses**.

### Types of IPv4 Addresses
- **Dynamic**: Temporary addresses leased from a DHCP server for a specific period.
- **Static**: Permanent addresses manually assigned to the system.

### Viewing IPv4 Addresses
Use the `ip` command with the `addr` argument:

---
