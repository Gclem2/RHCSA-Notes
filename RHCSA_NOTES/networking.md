# Networking Fundamentals

## Hostname
A hostname is a unique alphanumeric label (max 253 characters) identifying a system on the network. Allowed characters: letters, digits, hyphen (-), underscore (_), period (.). Stored in `/etc/hostname`. Can be viewed using `hostname`, `hostnamectl`, `uname`, `nmcli`, or by checking `/etc/hostname`.

## IPv4 Address
IPv4 is a 32-bit software address for network communication.  
- **Dynamic**: Temporary, leased from DHCP.  
- **Static**: Permanent, manually configured.  
View current IPv4 assignments with `ip addr`. Loopback interface `lo` always has `127.0.0.1`.

## Classful Network Addressing
IPv4 addresses: 32 bits = network (MSBs) + host (LSBs). Classes:  

- **Class A**: 1st octet = network, rest = host. Range 0–126, up to 16M hosts. Example: `10.121.51.209`.  
- **Class B**: 1st 2 octets = network, last 2 = host. Range 128–191, up to 65K hosts. Example: `161.121.51.209`.  
- **Class C**: 1st 3 octets = network, last = host. Range 192–223, up to 254 hosts. Example: `215.121.51.209`.  
- **Class D**: 224–239, multicast.  
- **Class E**: 240–255, experimental/reserved.  

**Reserved addresses**: Network (0) and broadcast (255) in each subnet. Classes D and E are not for standard hosts.

## Subnetting and CIDR
Subnetting divides a network into smaller sub-networks. CIDR (Classless Inter-Domain Routing) uses a `/n` suffix to denote network prefix length. Example: `192.168.1.0/24` → 24 bits network, 8 bits host.

## Protocols and Ports
- **TCP/UDP**: Transport protocols. TCP is reliable, connection-oriented; UDP is connectionless, faster.  
- **Well-known ports**: Ports 0–1023, e.g., HTTP 80, HTTPS 443, SSH 22.  
- **ICMP**: Used for diagnostics (ping, traceroute).

## Ethernet and MAC
- Ethernet address (MAC) uniquely identifies a network interface at the hardware level.  
- IPv6 uses 128-bit addresses, improved auto-configuration, and eliminates NAT.

## Network Device and Connection
- A **network device** (interface) connects a system to the network, e.g., `enp0s3`.  
- A **connection profile** links a device to network settings: IP, gateway, DNS, VLAN.  
- Managed via `nmcli`, `nmtui`, or editing configuration files in `/etc/NetworkManager/system-connections/`.

## Configuring Network
- **Static IP**: Assign manually in connection profile or config file.  
- **Dynamic IP**: Assigned via DHCP.  
- Change hostname: `hostnamectl set-hostname <name>`.  
- Test connectivity: `ping <IP/hostname>` or `hostname -I`.

## Hosts Table
- `/etc/hosts` maps hostnames to IPs locally. Format: `IP_address hostname [alias]`.

---
# Subnetting

Subnetting divides a large network into smaller, manageable subnets. It improves performance, reduces traffic, and simplifies administration. Subnetting uses node bits, not network bits.  

**Key Points:**  
- Reduces the number of usable addresses.  
- All nodes in a subnet share the same subnet mask.  
- Each subnet is isolated; a router is needed for inter-subnet communication.  
- First IP in a subnet = subnet address; last IP = broadcast address (both reserved).  

---
# Subnet Mask

A subnet mask separates the network/subnet bits from the node bits and helps routers identify network boundaries. Represented in decimal or binary, 1s indicate network/subnet bits, and 0s indicate node bits.  

**Default Masks:**  
- Class A: 255.0.0.0  
- Class B: 255.255.0.0  
- Class C: 255.255.255.0  

**Calculating Subnet Address:**  
1. Convert IP and subnet mask to binary.  
2. Perform a logical AND between them.  
3. Resulting 1s and 0s give the subnet address.  

Example: IP 192.168.12.72 with mask 255.255.255.224 → apply AND to get subnet address.

---
# Classless Network Addressing

Classless Inter-Domain Routing (CIDR) allows flexible allocation of IPv4 addresses to prevent address exhaustion and reduce routing table size. Unlike classful addressing, CIDR supports custom-sized network blocks and scalable routing.  

**CIDR Notation:**  
- Format: `IP_address/number_of_network_bits`  
- Example: IP `192.168.0.20` with mask `255.255.255.0` → `192.168.0.20/24`  
- Provides a concise way to represent an IP and its subnet mask.

---
# Protocol

A protocol is a set of rules governing data exchange between network entities, defining formatting, coding, error handling, speed matching, and packet sequencing. It acts as a common language understood by all nodes. Protocols are listed in `/etc/protocols` with details including name, port number, alias, and description.  

**Common Protocols:**  
- **TCP** – Connection-oriented, reliable transmission  
- **UDP** – Connectionless, low-overhead transmission  
- **IP** – Handles addressing and routing  
- **ICMP** – Used for network diagnostics and error messaging
---
# Network Devices and Connections

- **Network Interface Cards (NICs):** Hardware adapters providing one or more Ethernet ports for connectivity. Also called network adapters; individual ports are network interfaces or devices. Can be built-in or add-on; common designs include 1, 2, or 4 ports per adapter.  

- **Connection Profiles:** Each interface can have multiple profiles with unique names. Profiles include settings like device name, UUID, MAC address, IP address, etc. Only one profile per device can be active at a time. Configuration can be done via files or commands.

---
# The NetworkManager Service

- **NetworkManager:** Default service in RHEL 9 for configuring, administering, and monitoring network interfaces and connections.
- **Daemon:** The `NetworkManager` daemon maintains active network devices and connections.
- **Management Tools:**
  - **nmcli:** Powerful command-line tool for managing devices, connections, and the service.
  - **nmtui:** Text-based (TUI) interface for interactive configuration.
  - **nm-connection-editor:** Graphical tool for managing network connections.

---
# Understanding Interface Connection Profiles

- **Connection Profiles:** Each network connection has a configuration file defining IP settings and other parameters. These are read and applied when the connection is activated.
- **Location:** Stored under `/etc/NetworkManager/system-connections/`.
- **Naming:** Files are named after the connection with the `.nmconnection` extension (e.g., `enp0s3.nmconnection`, `ens160.nmconnection`).
- **Default Setup:** On systems like server10/server20, the primary interface `enp0s3` has a matching connection name and profile created during RHEL installation.

## Profile Structure
- Profiles are divided into sections, commonly:
  - `connection`
  - `ethernet`
  - `ipv4`
  - `ipv6`
  - `proxy`

## Common Properties
- **id:** Human-readable name of the connection (usually matches interface name).
- **uuid:** Unique identifier for the connection.
- **type:** Connection type (e.g., ethernet).
- **autoconnect-priority:** Preference when multiple autoconnect profiles exist (range -999 to 999).
- **interface-name:** Network device name.
- **timestamp:** Last successful activation time (auto-updated).
- **ipv4.addresses / method:** Static IPv4 address and method (`manual`, `/24` = subnet mask).
- **ipv6.addr-gen-mode / method:** IPv6 address generation method.

- **Note:** Many additional properties exist depending on interface type. See `man nm-settings` for full details.

---
# Network Device and Connection Administration Tools

- **NetworkManager Toolset:** RHEL 9 uses NetworkManager to configure and manage network interfaces, connections, and profiles.

## Key Commands
- **ip:** Versatile utility to display, monitor, and manage interfaces, connections, routing, and traffic.
- **nmcli:** NetworkManager CLI tool to create, modify, delete, activate, and deactivate connection profiles.

## Configuration Methods
- **Manual Profiles:** Administrators can manually create connection profiles and attach them to devices.
- **Alternative Methods:** RHEL provides additional methods covered later in the chapter.

## Deprecated Tools
- **Deprecated:** `ifup`, `ifdown`, `ifconfig`, and `/etc/sysconfig/network-scripts`.
- **Replacement:** NetworkManager and its tools (`nmcli`, `nmtui`, etc.).

- **Practice:** Exercise 15-3 uses the manual method and these tools to configure a new network device.

---
# The nmcli Command

`nmcli` is the NetworkManager command-line tool used to **create, view, modify, delete, activate, and deactivate** network connections, as well as **display and manage network device status**.

## Object Categories (Focus Areas)
nmcli operates on several object categories; the most commonly used are:

### Connection
Used to administer network connection profiles.
- **show** – List all connections (active and inactive)
- **up / down** – Activate or deactivate a connection
- **add** – Create a new connection
- **edit** – Edit an existing or new connection interactively
- **modify** – Change one or more connection properties
- **delete** – Remove a connection
- **reload** – Re-read all connection profiles
- **load** – Re-read a specific connection profile

### Device
Used to view and manage network interfaces.
- **status** – Show device state summary
- **show** – Display detailed information for devices

## Abbreviations
- Object categories and commands can be abbreviated:
  - `connection` → `c` or `con`
  - `device` → `d` or `dev`
  - `add` → `a`, `delete` → `d`, etc.
- Tab completion is supported for faster usage.

## Common Examples
- **Show all connections:** `nmcli c s`
- **Deactivate a connection:** `nmcli c down <connection>`
- **Activate a connection:** `nmcli c up <connection>`
- **Show device status:** `nmcli d s`

## Notes
- Output typically shows **connection name, UUID, type, device, and state**.
- The **loopback interface (lo)** is not managed by NetworkManager.
- See `man nmcli-examples` for additional usage patterns.

---
# Understanding Hosts Table

Each system IP should have an associated hostname to simplify access. Instead of repeatedly using IP addresses, hostname resolution allows systems to communicate using readable names.

## Hostname Resolution Methods
- **DNS:** Scalable solution for large networks and the Internet.
- **Hosts table (`/etc/hosts`):** Commonly used on small or isolated networks for local hostname-to-IP mapping.

## /etc/hosts File Format
Each entry consists of:
1. **IP address**
2. **Canonical (official) hostname**
3. **Optional aliases**

### Example:

This allows access to systems using either the full hostname or a short alias.

## Notes
- The hosts file must be updated **on each system** for consistency.
- **EXAM TIP:** If DNS is properly configured and resolving all hostnames, maintaining `/etc/hosts` is unnecessary.

---
# Testing Network Connectivity

RHEL provides the **ping** command to test network connectivity between systems. It sends 64-byte **ICMP** echo request packets to a destination IP and waits for replies. Successful responses confirm connectivity and basic network health.

## Usage
- `-c <count>`: Specify the number of packets to send.

### Example
Send two packets from `server10` to `server20` (192.168.0.120):

## Interpreting Results
- **Packets transmitted/received:** Should match.
- **Packet loss:** Ideally 0%.
- **Round-trip time (RTT):** Lower values indicate a healthier connection.

## Common Tests
- Local system IP
- Loopback address: `127.0.0.1`
- Default gateway
- Other local or remote network addresses

## Troubleshooting Failed Pings
- Check NIC seating and driver installation
- Verify network cable or link status
- Confirm IP address and subnet mask
- Ensure default gateway or static routes are correct
---
