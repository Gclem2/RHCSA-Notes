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
# Chapter 20: Security Enhanced Linux

## Overview
SELinux is a kernel-level mandatory access control (MAC) mechanism that provides security beyond traditional DAC (user/group/permissions)

## Purpose
- Control **who** can access **what** on the system
- Limit damage from unauthorized user or program access
- Enforce security policies beyond standard file permissions

## Traditional Security vs SELinux
**Traditional (DAC - Discretionary Access Control)**:
- File/directory permissions (rwx)
- User and group ownership
- Shadow passwords and password aging

**SELinux (MAC - Mandatory Access Control)**:
- Additional layer on top of DAC
- Context-based access control
- Policy enforcement at kernel level

## RHCSA Objectives Covered
- **55**: Set enforcing and permissive modes
- **56**: List and identify file and process contexts
- **57**: Restore default file contexts
- **58**: Manage SELinux port labels
- **59**: Use Boolean settings to modify SELinux
- **60**: Diagnose and address SELinux policy violations

## Topics Covered
1. SELinux terminology and concepts
2. Contexts for users, processes, files, and ports
3. Copy/move/archive files with SELinux context
4. Domain transitioning
5. SELinux Booleans
6. Query and manage SELinux (tools)
7. Modify contexts for files and ports
8. Add SELinux rules to policy database
9. View and analyze SELinux alerts

---
# Security Enhanced Linux

## What is SELinux?
Mandatory Access Control (MAC) developed by NSA, integrated into kernel via LSM framework

## DAC vs MAC
- **DAC** (Traditional): File permissions, ownership, setuid/setgid, su/sudo
- **MAC** (SELinux): Additional layer limiting subject (user/process) access to object (file/device/port)

## How It Works
- **Policy**: Authorization rules
- **Context/Label**: Security attributes on subjects and objects
- **AVC (Access Vector Cache)**: Stores decisions for performance

### Access Flow
Subject → Check AVC → Check policy (if not cached) → Allow/Deny

## Fine-Grained Control
Compromised service (e.g., HTTP) only damages what that process can access, not other processes/objects

## Default
Enabled at install, confines processes to minimum required privileges

---
# SELinux Terminology

## Core Components

### Subject
User or process accessing an object  
**Examples**: `system_u` (SELinux system user), `unconfined_u` (not bound by policy)  
**Location**: Field 1 of context

### Object
Resource being accessed (file, directory, device, port, socket, etc.)  
**Examples**: `object_r` (general), `system_r` (system-owned), `unconfined_r` (not bound by policy)

### Access
Action performed by subject on object (create, read, update file; access port)

### Policy
Ruleset enforced system-wide to analyze security attributes and decide access  
**Default behavior**: Deny if no rule exists

**Policy Types**:
- **targeted** (default): Targeted processes run confined, others unconfined (e.g., httpd confined, users unconfined)
- **mls**: Tight security at deeper levels
- **minimum**: Light version protecting only selected processes

## Security Attributes

### Context (Label)
Tag storing security attributes for subjects/objects  
**Format**: `SELinux_user:role:type(domain):level`

### Labeling
Mapping files with their stored contexts

### SELinux User
Predefined identities authorized for specific roles  
Maps Linux users to SELinux users to restrict role/level access  
**Example**: User mapped to `user_u` cannot run su/sudo or execute programs in home directory

### Role (RBAC)
Classifies who can access what domains/types  
**Examples**: `user_r` (users), `sysadm_r` (admins), `system_r` (system processes)  
**Location**: Field 2 of context

### Type Enforcement (TE)
Limits subject access to domains (processes) and types (files) using contexts

### Type & Domain
**Type**: Group of objects with common security requirements  
- **Examples**: `user_home_dir_t` (user home dirs), `usr_t` (/usr objects)  
- **Location**: Field 3 of file context

**Domain**: Defines process access; groups processes with common security needs  
- **Examples**: `init_t` (systemd), `firewalld_t` (firewalld), `unconfined_t` (not bound by policy)  
- **Location**: Field 3 of process context

### Level (MLS/MCS)
Pair of `sensitivity:category` values  
**RHEL 9 default**: MCS with one sensitivity (s0) and 0-1023 categories (e.g., `s0:c0.c4`)

---
# SELinux Contexts for Users

## Viewing User Context
```bash
id -Z
# Example output for user1:
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

**Format**: `user:role:type:level`

## Default Behavior
- All Linux users (including root) run **unconfined** by default
- **unconfined_u**: No SELinux restrictions, full system access

## SELinux User Types

### Unconfined User
`unconfined_u` - Unlimited privileges (default for all users)

### Confined Users (7 types)
Restricted access mapped to Linux users via policy to protect system from user damage

## Listing SELinux Users
```bash
# Requires setools-console package
seinfo -u
```
Shows 8 predefined SELinux users

## User Mapping
```bash
semanage login -l
```

**Output columns**:
1. **Login Name**: Linux user (or `__default__` for all non-root users)
2. **SELinux User**: Mapped SELinux identity
3. **MLS/MCS Range**: Security level
4. **Service**: Context (usually `*` for all services)

**Default mapping**: `__default__` → `unconfined_u`

---
# SELinux Contexts for Processes

## Viewing Process Context
```bash
ps -eZ
# Example output (first two lines):
system_u:system_r:init_t:s0    1  ?  00:00:02 systemd
```

**Format**: `user:role:domain:level`

## Context Fields
- **User** (`system_u`): SELinux username (mapped to Linux user, e.g., root)
- **Role** (`system_r`): Object/role
- **Domain** (`init_t`): Type of protection applied to process
- **Level** (`s0`): Security level

## Unprotected Processes
Run in **`unconfined_t`** domain (no SELinux restrictions)

---
# SELinux Contexts for Files

## Viewing File Context
```bash
ls -Z /etc/passwd
# Output:
system_u:object_r:passwd_file_t:s0 /etc/passwd
```

**Format**: `user:role:type:level`

## Context Fields
- **User** (`system_u`): Subject
- **Role** (`object_r`): Object
- **Type** (`passwd_file_t`): File type classification
- **Level** (`s0`): Security level

## Context Storage
- **System files**: `/etc/selinux/targeted/contexts/files/file_contexts`
- **User-created files**: `/etc/selinux/targeted/contexts/files/file_contexts.local`
- **Management**: Use `semanage` command to update policy files

---
# Copying, Moving, and Archiving Files with SELinux Contexts

## Default Behavior
- New files inherit **parent directory's context**
- All RHEL files labeled with SELinux context by default

## File Operation Rules

### 1. Copy to Different Directory
```bash
cp file /dest/
```
- Destination file gets **destination directory's context**
- Preserve original: `cp --preserve=context file /dest/`

### 2. Copy Overwriting Existing File
```bash
cp file /dest/existing_file
```
- Copied file gets **overwritten file's context**
- Preserve original: `cp --preserve=context file /dest/existing_file`

### 3. Move File
```bash
mv file /dest/
```
- File **retains original context** (may differ from destination directory)

### 4. Archive with tar
```bash
tar --selinux -czf archive.tar.gz files/
```
- Use `--selinux` option to preserve contexts

## Key Points
- **Copy**: Gets destination context (unless `--preserve=context`)
- **Move**: Keeps original context
- **Archive**: Use `--selinux` flag

---
# SELinux Contexts for Ports

## Viewing Port Contexts
```bash
semanage port -l
```

**Output columns**:
1. **SELinux Type**: Port type label
2. **Protocol**: tcp/udp
3. **Port Number(s)**: Assigned port(s)

## Default Behavior
SELinux allows services to listen only on **restricted set of network ports**

## Example Output
```
ssh_port_t         tcp    22
http_port_t        tcp    80, 443, 488, 8008, 8009, 8443
```

Services confined to their designated ports by default

---
# SELinux Booleans

## Overview
On/off switches that activate/deactivate specific SELinux policy rules **immediately** (no recompile/reload needed)

## Purpose
Control permissions dynamically  
**Example**: `ftpd_anon_write` Boolean enables/disables anonymous FTP uploads

## Storage Locations
- **Runtime values**: `/sys/fs/selinux/booleans/` (virtual files, values: `1` or `0`)
- **Permanent values**: SELinux policy database

## Documentation
```bash
# Install manual pages
yum install selinux-policy-doc

# View Boolean documentation
man -K boolean_name
# Example: man -K abrt_anon_write
```

## Boolean Operations
- **View**: Check current Boolean values
- **Temporary change**: Modify runtime (stored in `/sys/fs/selinux/booleans/`)
- **Permanent change**: Update policy database (survives reboot)
- **Effect**: Changes take effect immediately

## Typical System
Hundreds of Boolean files available for fine-grained control

---
# SELinux Administration

## Management Tasks
- Control activation mode
- Check operational status
- Set security contexts on subjects/objects
- Switch Boolean values

## Essential Packages & Commands

| Package | Commands | Purpose |
|---------|----------|---------|
| `libselinux-utils` | `getenforce`, `setenforce`, `getsebool` | Check/set enforcement mode, view Booleans |
| `policycoreutils` | `sestatus`, `setsebool`, `restorecon` | Status, set Booleans, restore contexts |
| `policycoreutils-python-utils` | `semanage` | Manage contexts, ports, users |
| `setools-console` | `seinfo`, `sesearch` | Query policy information, search rules |
| `setroubleshoot-server` | SELinux Alert Browser (GUI) | View alerts, debug issues |

## Installation
Ensure all packages installed for full SELinux management capability

## Additional Tools
Other utilities available for specific tasks (less frequently used)

---
# SELinux Management Commands

## Mode Management
| Command | Description |
|---------|-------------|
| `getenforce` | Display current mode (enforcing/permissive/disabled) |
| `setenforce` | Switch between enforcing/permissive temporarily |
| `sestatus` | Show runtime status and Boolean values |
| `grubby` | Update/display grub2 boot loader config |

## Context Management
| Command | Description |
|---------|-------------|
| `chcon` | Change file contexts (lost on relabeling) |
| `restorecon` | Restore default contexts from `/etc/selinux/targeted/contexts/files` |
| `semanage fcontext` | Change file contexts permanently (survives relabeling) |

## Policy Management
| Command | Description |
|---------|-------------|
| `seinfo` | Display policy component information |
| `semanage` | Manage policy database |
| `sesearch` | Search policy rules |

## Boolean Management
| Command | Description |
|---------|-------------|
| `getsebool` | Display Booleans and current settings |
| `setsebool` | Modify Boolean values (temporary or permanent with `-P`) |
| `semanage boolean` | Modify Boolean values in policy database |

## Troubleshooting
| Command | Description |
|---------|-------------|
| `sealert` | Graphical troubleshooting tool (SELinux Alert Browser) |

## Key Distinction
- **Temporary changes**: `chcon`, `setenforce`, `setsebool` (without `-P`)
- **Permanent changes**: `semanage`, `setsebool -P`, `restorecon`

---
# Viewing and Controlling SELinux Operational State

## Configuration File
**Location**: `/etc/selinux/config`

### Key Directives
```bash
SELINUX=enforcing    # enforcing, permissive, or disabled
SELINUXTYPE=targeted # Policy type (default: targeted)
```

## Operating Modes
| Mode | Behavior |
|------|----------|
| **enforcing** | Enabled, allows/denies based on policy rules |
| **permissive** | Enabled, allows all but logs violations (troubleshooting/tuning) |
| **disabled** | SELinux completely off |

## View Current Mode
```bash
getenforce
# Output: Enforcing
```

## Temporary Mode Change (Runtime Only)
```bash
# Switch to permissive
setenforce 0    # or: setenforce permissive

# Switch to enforcing
setenforce 1    # or: setenforce enforcing

# Verify
getenforce
```

**Lost on reboot** - edit `/etc/selinux/config` for persistence

## Persistent Disable
```bash
# Disable via bootloader
grubby --update-kernel ALL --args selinux=0

# Re-enable
grubby --update-kernel ALL --remove-args selinux=0
```

Modifies `/boot/loader/entries/` bootloader config

## EXAM TIP
Switch to **permissive** for troubleshooting non-functioning services, then **change back to enforcing** when resolved

---
# Modify SELinux File Context

1. Create the hierarchy sedir1/sefile1 under /tmp:
```bash
cd /tmp
mkdir sedir1
touch sedir1/sefile1
```
2. Determine the context on the new directory and file:
```bash
ls -ldZ sedir1
ls -ldZ sedir1/sefile1
```

3. Modify the SELinux user (-u) on the directory to user_u and type (-t) to public_content_t recursively (-R) with the chcon command:
```bash
sudo chcon -vu user_u -t public_content_t sedir1 -R
```
4. Validate the new context:
```bash
ls -ldZ sedir
```

---
# Add and Apply File Context

1. Determine the current context:
```bash
ls -ldZ sedir1
```
2. Add (-a) the directory recursively to the policy database using the semanage command with the fcontext suubcommand:
```bash
sudo semanage fcontext -a -s user_u -t public_content_t '/tmp/sedir1(/.*)?'
```
3. Validate the addition by listing (-l) the recent changes (-C) in the policy database:
```bash
sudo semanage fcontext -Cl | grep sedir
```
4. Change the current context on sedir1 to something random (staff_u/etc_t) with the chcon command:
```bash
sudo chcon -vuu staff_u -t etc_t sedir1 -R
```
5. The secuurity context is changed successfully. Confirm with the ls command:
```bash
ls -ldZ sedir1 ; ls -lZ sedir1/sefile1
```
![](../RHCSA_Labs/attachment/Pasted%20image%2020260126140757.png)
6. Reinstate the context on sedir1 direcotr recursively (-R) as stored in the policy database using the restorecon command:
```bash
sudo restorecon -Rv sedir1
```

---
# Add and Delete Network Ports

1. List (-l) the ports for the httpd service as defined in the SELinux policy database:
```bash
sudo semanage port -l | grep ^http_port
```
2. Add (-a) port 8010 with type (-t) http_port_t and protocol (-p) tcp to the policy:
```bash
sudo semanage port -at http_port_t -p tcp 8010
```
3. Confirm the addition:
```bash
sudo semanage port -l | grep ^http_port
```
4. Delete (-d) port 8010 form the policy and confirm:
```bash
sudo semanage port -dp tcp 8010
sudo semanage port -l | grep ^http_port
```

---
# Copy Files with and without Context
1. Create file sefile2 under /tmp and show context:
```bash
touch /tmp/sefile2
ls -lZ /tmp/sefile2
```
2. Copy this file to the /etc/default directory, and check the context again:
```bash
sudo cp /tmp/sefile2 /etc/default/
ls -lZ /etc/default sefile2
```
3. Erase the /etc/default/sefile2 file, and copy it again with the --preserve=context option:
```bash
sudo rm /etc/default sefile2
```

---
# View and Toggle SELinux Boolean Values

1. Display the current setting of the Boolean nfs_export_all_rw using three different commands--getsebool, sestatus, and semanage
```bash
sudo getsebool -a | grep nfs_export_all_rw
sudo sestatus -b | grep nfs_export _all_rw
sudo semanage boolean -l | grep nfs_export
```
2. Turn off the value of nfs_export_all_rw using the seetsebool command by simply furnishhing "off" or "0" with it and confirm:
```bash
sudo setsebool nfs_export_all_rw 0
sudo getsebool -a | grep nfs_export_all_rw
```
3. Reboot the system and rerun the ggetsebool command to check the Boolean state:
```bash
sudo getseboool -a | grep nfs_export_all_rw
```
4. Set the value of the Boolean persistently -P or -m as needed using either of the following:
```bash
sudo setsebool -P nfs_export_all_rw off
sudo semanage boolean -m -0 nfs_export_all_rw
```
5. Validate the new value using the gesebool, sestatus, or semanage command:
```bash
sudo getsebool fs_export_all_rw
```

---
# Monitoring and Analyzing SELinux Violations

## Log Locations
- **Primary**: `/var/log/audit/audit.log` (if auditd running)
- **Fallback**: `/var/log/messages` (via rsyslog if auditd absent)

## Denial Messages
- Tagged with **AVC** (Access Vector Cache) type
- Includes message ID and viewing instructions
- **Troubleshooting tip**: If works in permissive but not enforcing → SELinux needs adjustment

## Analysis Process

### 1. SELinux Access Request Flow
```
Subject → Access Request → SELinux Policy Check → Allow/Deny → Target Object
                                ↓
                         Log to audit.log (AVC)
```

### 2. setroubleshoot Service
- **Daemon**: `setroubleshootd` (background analysis)
- **Client**: `sealert` command (text/GUI interface)
- **Package**: `setroubleshoot-server` (must be installed)
- Analyzes denials, provides recommendations

## Sample Records

### Allowed Access (audit.log)
```
type=USER_AUTH ... user1 successfully su to root on server10
```

### Denied Access (audit.log)
```
type=AVC msg=audit(...): avc: denied { write } 
  scontext=unconfined_u:unconfined_r:passwd_t:s0-s0:c0.c1023
  tcontext=system_u:object_r:etc_t:s0
  tclass=file comm="passwd" name="nshadow"
  permissive=0
```

**Key fields**:
- `scontext`: Source context (passwd command)
- `tcontext`: Target context (shadow file with wrong type `etc_t`)
- `tclass`: Object class (file)
- `permissive=0`: Enforcing mode

## Analyzing Denials
```bash
# Analyze all AVC records
sealert -a /var/log/audit/audit.log
```

Produces formatted report with:
- Root cause
- Recommended fixes
- Relevant details

## Example Scenario
1. **Problem**: Changed `/etc/shadow` type to `etc_t` (incorrect)
2. **Result**: `passwd` command denied writing to shadow file
3. **Fix**: `restorecon /etc/shadow` (restored correct type)
4. **Verify**: Password change successful

## Key Point
Check SELinux logs when service fails - often context mismatch issue

---
# Monitoring and Analyzing SELinux Violations

## Log Locations
- **Primary**: `/var/log/audit/audit.log` (if auditd running)
- **Fallback**: `/var/log/messages` (via rsyslog)
- **Denials**: Tagged with **AVC** (Access Vector Cache)

**Troubleshooting tip**: Works in permissive but not enforcing → adjust SELinux

## Analysis Tools

### setroubleshoot Service
- **Daemon**: `setroubleshootd` (background analysis)
- **Client**: `sealert` (text/GUI)
- **Package**: `setroubleshoot-server`
- Analyzes denials, provides fix recommendations

## Sample Log Entries

### Allowed Access
```
type=USER_AUTH ... user1 successfully su to root
```

### Denied Access (AVC)
```
type=AVC avc: denied { write }
  scontext=unconfined_u:unconfined_r:passwd_t:...
  tcontext=system_u:object_r:etc_t:s0
  tclass=file comm="passwd" name="nshadow"
  permissive=0
```

**Key fields**: `scontext` (source), `tcontext` (target), `tclass` (object type), `permissive=0` (enforcing)

## Analyze Denials
```bash
sealert -a /var/log/audit/audit.log
```
Produces formatted report with cause and recommendations

## Example Fix
**Problem**: `/etc/shadow` has wrong type (`etc_t`)  
**Result**: `passwd` command denied  
**Solution**: `restorecon /etc/shadow`  
**Verify**: Password change works

---
