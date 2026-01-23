# Chapter 19: The Linux Firewall

## Overview
Host-based security control to restrict network access to systems and services

## Key Concepts

### Firewall Purpose
- Tighten system access in networked/Internet-facing environments
- Control access to individual services
- Restrict inbound traffic to:
  - Allowed ports only
  - Valid source IP addresses only

### Main Components
- **firewalld service** - Dynamic firewall daemon
- **firewall-cmd** - Command-line management tool
- **Zones** - Logical groupings with different trust levels
- **Services** - Predefined port/protocol configurations

## RHCSA Objectives Covered
- **Objective 47**: Restrict network access using firewall-cmd/firewall
- **Objective 52**: Configure firewall settings using firewall-cmd/firewalld

## Topics Covered
1. Linux firewall for host-based security
2. firewalld service overview
3. Zones and services concepts
4. Zone and service configuration files
5. Control access to services and ports
6. firewall-cmd command usage

## Solution Type
**Host-based firewall** - Protection at the individual system level

---
# Firewall Overview

## Firewall Types

### Network-Level Firewalls
- Use dedicated hardware or software appliances
- Form protective shield around entire network

### Host-Based Firewalls
- Run in computer operating system
- Monitor and manage traffic in/out of individual server
- **RHEL uses host-based firewall solution**

## How Firewalls Work

### Packet Filtering
1. **Encapsulation**: Message (payload) + header information = data packet
2. **Header contains**:
   - Source IP address
   - Destination IP address
   - Port number
   - Data type
3. **Firewall inspects** each packet header against predefined rules
4. **Decision**: Allow or block packet

## Common Service Ports

| Service | Port | Description |
|---------|------|-------------|
| FTP | 21 | File Transfer Protocol |
| SSH | 22 | Secure Shell |
| Postfix | 25 | Email service |
| HTTP | 80 | HyperText Transfer Protocol |
| NTP | 123 | Network Time Protocol |

> Port definitions standardized in `/etc/services`

## RHEL Firewall Components

### Core Technology
- **netfilter** - Kernel module for traffic policing
- **nftables** - Filtering and packet classification framework

### Features
- Packet inspection, modification, dropping, or routing
- Network Address Translation (NAT)
- Port forwarding
- Rule-based control for: incoming, outgoing, and forwarded packets
---
# firewalld Zones

## What are Zones?
Logical policies based on trust level of network connections and source IP addresses

## Zone Characteristics
- Network connection can be in **one zone at a time**
- Zone can have **multiple network connections**
- Each zone configured independently

## Zone Configuration Includes
- Services (open/closed)
- Ports and protocols (open/closed)
- Advanced features:
  - Masquerading
  - Port forwarding
  - NAT
  - ICMP filters
  - Rich language rules

## Packet Processing Logic

1. **Check source IP** against zone configurations
2. **If match found** → Apply matched zone rules
3. **If no match** → Use zone with defined network connection
4. **If still no match** → Use

---
# Zone Configuration Files

## File Locations

| Location | Purpose | Format |
|----------|---------|--------|
| `/usr/lib/firewalld/zones/` | System-defined (default) rules | XML |
| `/etc/firewalld/zones/` | User-defined (custom) rules | XML |

## Configuration Workflow

### Option 1: Using Management Tools
- Modify zone using `firewall-cmd` or web console
- Modified system file **automatically copied** to `/etc/firewalld/zones/`

### Option 2: Manual Configuration
1. Copy zone file from `/usr/lib/firewalld/zones/` to `/etc/firewalld/zones/`
2. Edit the custom copy
3. firewalld reads and applies rules from `/etc/firewalld/zones/`

## Zone File Structure (XML)

### Example: public.zone
```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>...</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
</zone>
```

### Key Elements
- **Name**: Zone identifier
- **Description**: Zone purpose
- **Services**: List of allowed services (e.g., ssh, dhcpv6-client, cockpit)

## Zone File Precedence
- Custom rules in `/etc/firewalld/zones/` **override** system defaults
- System files in `/usr/lib/firewalld/zones/` used as **templates**

> **Reference**: `man firewalld.zone` for zone configuration details

---
# firewalld Services

## What are firewalld Services?
Preconfigured firewall rules for specific network services stored in individual files

## Service Components
A service file defines:
- **Port number(s)**
- **Protocol** (TCP/UDP)
- **Helper modules** (if needed for service loading)

## How Services Work
- Services can be **added to zones**
- Easier activation/deactivation of specific rules
- Simplifies management vs. manual port/protocol configuration

## Default Behavior
**firewalld blocks all traffic** unless:
- A service is explicitly opened, OR
- A port is explicitly opened

## Usage Pattern
```bash
# Add service to zone (allows that service's traffic)
firewall-cmd --zone=public --add-service=http

# Remove service from zone (blocks that service's traffic)
firewall-cmd --zone=public --remove-service=http
```

## Benefits
- **Predefined rules** - No need to remember port numbers
- **Consistency** - Standard service definitions
- **Simplicity** - Enable/disable by service name vs. port/protocol

---
# Service Configuration Files

## File Locations

| Location | Purpose | Format |
|----------|---------|--------|
| `/usr/lib/firewalld/services/` | System-defined (default) service rules | XML |
| `/etc/firewalld/services/` | User-defined (custom) service rules | XML |

## Configuration Workflow

### Option 1: Using Management Tools
- Modify service using `firewall-cmd` or web console
- Modified system file **automatically copied** to `/etc/firewalld/services/`

### Option 2: Manual Configuration
1. Copy service file from `/usr/lib/firewalld/services/` to `/etc/firewalld/services/`
2. Edit the custom copy
3. firewalld reads and applies rules from `/etc/firewalld/services/`

## Service File Structure (XML)

### Example: ssh.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>SSH</short>
  <description>Secure Shell (SSH)</description>
  <port protocol="tcp" port="22"/>
</service>
```

### Key Elements
- **Name**: Service identifier
- **Description**: Service purpose
- **Port**: Port number
- **Protocol**: TCP or UDP

## Service File Precedence
- Custom rules in `/etc/firewalld/services/` **override** system defaults
- System files in `/usr/lib/firewalld/services/` used as **templates**

> **Reference**: `man firewalld.service` for service configuration details

---
# Firewalld Management

## Management Operations
- List, query, add, change, remove:
  - Zones
  - Services
  - Ports
  - IP sources
  - Network connections

## Management Methods

| Method | Tool | Interface |
|--------|------|-----------|
| Command-line | `firewall-cmd` | CLI |
| Graphical | Web console | GUI |
| Manual | Edit XML templates | Text editor |

---
# The firewall-cmd Command

Powerful CLI tool to manage firewalld service. Supports runtime and persistent rule changes.

## Common Options

### General
| Option | Description |
|--------|-------------|
| `--state` | Display firewalld running status |
| `--reload` | Reload rules from zone files (loses runtime changes) |
| `--permanent` | Store change persistently (active after reload/restart) |

### Zones
| Option               | Description                                        |
| -------------------- | -------------------------------------------------- |
| `--get-default-zone` | Show default/active zone                           |
| `--set-default-zone` | Change default zone (runtime + persistent)         |
| `--get-zones`        | List available zones                               |
| `--get-active-zones` | Show active zones and assigned interfaces          |
| `--list-all`         | List all settings for a zone                       |
| `--list-all-zones`   | List settings for all zones                        |
| `--zone=<name>`      | Specify zone to work on (defaults to default zone) |

### Services
| Option | Description |
|--------|-------------|
| `--get-services` | List predefined services |
| `--list-services` | List services in a zone |
| `--add-service=<svc>` | Add service to zone |
| `--remove-service=<svc>` | Remove service from zone |
| `--query-service=<svc>` | Query if service exists |

### Ports
| Option | Description |
|--------|-------------|
| `--list-ports` | List ports in zone |
| `--add-port=<port/proto>` | Add port(s) to zone |
| `--remove-port=<port/proto>` | Remove port from zone |
| `--query-port=<port/proto>` | Query if port exists |

### Network Connections
| Option | Description |
|--------|-------------|
| `--list-interfaces` | List interfaces in zone |
| `--add-interface=<if>` | Bind interface to zone |
| `--change-interface=<if>` | Move interface to different zone |
| `--remove-interface=<if>` | Unbind interface from zone |

### IP Sources
| Option | Description |
|--------|-------------|
| `--list-sources` | List IP sources in zone |
| `--add-source=<ip>` | Add IP source to zone |
| `--change-source=<ip>` | Change IP source |
| `--remove-source=<ip>` | Remove IP source from zone |

## Persistence
- Use `--permanent` with `--add`/`--remove` options to save to `/etc/firewalld/zones/`
- Without `--permanent`: runtime only (lost on reload/restart)

> **Reference**: `man firewall-cmd`

---
# Querying the Operational Status of firewalld

## Check Status

### Method 1: systemctl
```bash
sudo systemctl status firewalld
```
Shows: running state + enabled/disabled for autostart

### Method 2: firewall-cmd
```bash
firewall-cmd --state
```
Output: `running` or `not running`

## Enable and Start (if needed)
```bash
sudo systemctl --now enable firewalld
```
- Starts service immediately
- Enables autostart on reboot

## Expected State
- **Running**: Service is active
- **Enabled**: Service starts automatically at boot

---
# Exercise 19-1: Add Services and Ports, and Manage Zones

1. Determine the name of the current default zone:
```bash
sudo firewal-cmd --get-default-zone
```

2. Add a permanent rle to allow HTTP traffic on its default port:
```bash
sudo firewall-cmd --permanent --add-service http
```
3. Activate the new rule:

```bash
sudo firewall-cmd --reload
```
4. Confirm the activation of the new rule:
```bash
sudo firewall-cmd --list-services
```
![](../RHCSA_Labs/attachment/Pasted%20image%2020260123152734.png)
5. Display the content of the default zone file to confirm the addition of the permanent rule:
```bash
sudo cat /etc/firewalld/zones/public.xml
```
![](../RHCSA_Labs/attachment/Pasted%20image%2020260123152850.png)
6. Add a runtime rule to allow traffic on  TCP port 443 and verify:
```bash
sudo firewall-cmd --add-port 443/tcp
```

7. Add a permanent rule to the internal zone for TCP port range 5901 to 5910:
```bash
sudo firewall-cmd --add-port 5901-5910/tcp --permanent --zone internal
```
8. Display the content of the internal zone file to confirm the addition of the permanent rule:
```bash
sudo cat /etc/firewalld/zones/internal.xml
```
9. Switch the default zone to internal and confirm:
```bash
sudo firewall-cmd --set-default-zone interal
```

---
# Exercise 19-2: Remove Services and ports, and Manage Zones

1. Remove the permanent rule for HTTP from the public zone:
```bash
sudo firewall-cmd --remove-service=http --zone public --permanent 
```
2. Remove the permanenet rule for ports 5901 to 5910 from the internal zone

```bash
sudo firewall-cmd --remove-port 5901-5910/tcp --permanent 
```
3. Switch the default zone to public and validate:
```bash
sudo firewall-cmd --set-default-zone=public
sudo firewall-cmd --get-default-zone
```

4. Activate the public zone rules and list the current services:

```bash
sudo firewall-cmd --reload
sudo firewall-cmd --list-servieces
```
![](../RHCSA_Labs/attachment/Pasted%20image%2020260123154325.png)

---
# Exercise 19-3: Test the Effect of Firewall Rule

1. Remove the rule for the sshd service on server10:
```bash
sudo firewall-cmd --remove-service ssh
```
2. Issue the ssh command on server20 to access server10:
```bash
ssh server10
```
3. Add the rule back for sshd one server10:
```bash
sudo firewall-cmd --add-service=ssh
```
4. Issue the ssh command on server20 to access server10. Enter es if prompted and the password for user1
```
ssh server10
```

---
# Chapter 19 Summary

## Key Topics Covered

### Firewall Fundamentals
- Host-based firewall solution for system protection
- How firewalls work (packet filtering, header inspection)

### firewalld Service
- Dynamic firewall management (no service restart needed)
- Configuration locations: `/usr/lib/firewalld/` (defaults), `/etc/firewalld/` (custom)

### Zones
- Trust-based policies for network connections
- Predefined zones: trusted → internal → home → work → dmz → external → **public (default)** → block → drop
- Zone configuration files (XML format)

### Services
- Preconfigured rules for specific services
- Service configuration files (XML format with port/protocol)
- Default: all traffic blocked unless service/port explicitly opened

### Management with firewall-cmd
- Check status: `--state`, `systemctl status firewalld`
- Zones: `--get-zones`, `--set-default-zone`, `--list-all`
- Services: `--add-service`, `--remove-service`, `--list-services`
- Ports: `--add-port`, `--remove-port`, `--list-ports`
- Interfaces: `--add-interface`, `--list-interfaces`
- Sources: `--add-source`, `--list-sources`
- Persistence: `--permanent` flag + `--reload`

## Exercises Performed
- Changed/checked firewalld operational state
- Added/removed services and ports (runtime and persistent)
- Managed zones
- Tested port deletion and restoration

---
