# Chapter 17 — Hostname Resolution and Time Synchronization

This chapter covers **DNS-based hostname resolution** and **time synchronization** on Linux systems, both critical for reliable network operations and system integrity.

## Key Topics
- Overview of DNS and hostname resolution
- DNS roles and resolution process
- Resolver configuration files and entries
- Name resolution tools
- Time synchronization concepts
- NTP roles, stratum levels, and time sources
- Chrony configuration and operation
- Configuring and verifying NTP/Chrony clients
- Viewing and setting system date and time

## RHCSA Objectives
- Configure hostname resolution
- Configure time service clients

## Domain Name System (DNS)
DNS is a platform-independent network service that resolves **hostnames to IP addresses and vice versa**. It enables users and applications to communicate using human-readable names instead of numeric addresses. DNS operates as the standard hostname resolution mechanism on the Internet and enterprise networks, with clients querying one or more DNS servers using dedicated lookup utilities.

## Time Synchronization (Chrony / NTP)
Chrony is an implementation of the **Network Time Protocol (NTP)** used to keep system clocks accurate and synchronized with trusted time sources. Accurate time is essential for:
- Monitoring and backup systems
- Job scheduling and billing
- File sharing and authentication
- Logging, auditing, and security analysis

Consistent time across systems ensures correct event ordering, reliable authentication, and precise logging throughout the network.

---
## DNS and Name Resolution

The **Domain Name System (DNS)** is a hierarchical, inverted tree–like naming system used on the Internet and private networks to resolve **hostnames to IP addresses**. It is platform-independent and integrated into all modern operating systems, making it the standard mechanism for hostname resolution.

DNS is commonly referred to as **BIND (Berkeley Internet Name Domain)**, the most widely used DNS implementation.  
**Name resolution** is the process by which systems use DNS/BIND to translate hostnames into numeric IP addresses.

Understanding DNS requires familiarity with its **core components and roles**, as well as the **client-side configuration files and lookup tools** used to perform hostname resolution.

---
## DNS Name Space and Domains

The **DNS name space** is a hierarchical structure of all domains on the Internet, with the **root represented by a period (.)**. Directly below the root are the **top-level domains (TLDs)**, such as `.com`, `.net`, `.edu`, `.org`, `.gov`, `.ca`, and `.de`.  

A **DNS domain** is a collection of systems, and **subdomains** reside under parent domains, separated by periods. For example:  
- `redhat.com` → second-level domain under `.com`  
- `bugzilla.redhat.com` → third-level domain under `redhat.com`  

At the leaf level are individual systems or devices with IP addresses. For instance, a network switch `net01` in the `travel.gc.ca` subdomain has the name `net01.travel.gc.ca`. Appending a trailing period, `net01.travel.gc.ca.`, makes it a **Fully Qualified Domain Name (FQDN)**.

---
## DNS Roles

A system can operate as a **primary server**, **secondary server**, or **client** in DNS terminology. A DNS server is also called a **nameserver**.  

- **Primary Server:** Maintains the master database of hostnames and IP addresses for its domain. All changes are made here. Each domain must have **one primary server**.  
- **Secondary Server:** Stores an updated copy of the master database for redundancy and load balancing. Provides name resolution if the primary server is unavailable.  
- **DNS Client:** Queries nameservers to resolve hostnames to IP addresses. Every system with network access typically has DNS client functionality configured via specific text files.

---
## Understanding Resolver Configuration File

The `/etc/resolv.conf` file is the DNS **resolver configuration file** used for hostname lookups. It is referenced by resolver utilities and can be edited manually. Key directives include:

- **domain:** Sets the default domain name for queries.  
- **nameserver:** Lists up to three DNS server IPs to query in order. Can be on one line or separate lines.  
- **search:** Specifies up to six domain names for query searches; the first must be the local domain. Overrides `domain` if used.

**Sample syntax:**
domain example.com
search example.net example.org example.edu example.gov
nameserver 192.168.0.1 8.8.8.8 8.8.4.4

NetworkManager typically populates this file automatically. If absent, resolvers use the local host, derive the domain from the hostname, and build the search path accordingly.

---
## Viewing and Adjusting Name Resolution Sources and Order

The `/etc/nsswitch.conf` file directs lookup utilities to the correct source for hostname resolution and specifies the order in which sources are consulted. Four keywords govern behavior:

- **success:** Information found; return immediately (do not try next source)  
- **notfound:** Information not found; continue to next source  
- **unavail:** Source down or unavailable; continue to next source  
- **tryagain:** Source busy; retry later; continue to next source  

**Sample entry for hostname resolution:**


This searches `/etc/hosts` first, then DNS (`/etc/resolv.conf`).  

**Modified behavior to ignore DNS if not found locally:**


Once `resolv.conf` and `nsswitch.conf` are configured, use resolver tools like `dig`, `host`, `nslookup`, or `getent` to perform lookups.

---
# Performing Name Resolution with dig

`dig` (Domain Information Groper) is a DNS lookup utility. It queries a specified nameserver or consults `/etc/resolv.conf` to determine which nameservers to query. It is flexible and verbose, making it useful for troubleshooting DNS issues.

## Examples

### Forward Lookup
To get the IP address of `redhat.com` using the nameservers listed in `resolv.conf`:

```bash
dig redhat.com
```
Reverse Lookup

To perform a reverse DNS lookup on the IP address 52.200.142.250:
```bash
dig -x 52.200.142.250
```

This queries the PTR record and returns the associated hostname.

Reference the dig manual pages (man dig) for additional options and usage details.

---
# Performing Name Resolution with host

`host` is a simple DNS lookup utility that operates like `dig` in determining nameservers. By default, it produces minimal output, but adding the `-v` flag increases verbosity.

## Forward Lookup

Lookup the IP address of `redhat.com`:

```bash
host redhat.com
```

```bash
host -v redhat.com
```

----
# Performing Name Resolution with getent

`getent` (get entries) is a basic tool that retrieves matching entries from databases defined in `/etc/nsswitch.conf`. For DNS resolution, it queries the `hosts` database.

## Forward Lookup

Resolve the IP address of a hostname:

```bash
getent hosts redhat.com
```


---
# Time Synchronization

Network Time Protocol (NTP) is used to synchronize the system clock with remote time servers for accuracy and reliability. Proper time synchronization is critical for applications like authentication, email, backups, scheduling, financial systems, logging, monitoring, and file sharing.

NTP communicates with configured time servers and selects the one with the least delay for accuracy. The client keeps a drift file to correct gradual time deviations.

RHEL 9 implements NTP using **Chrony**, which uses UDP over port 123. Chrony starts at system boot and continuously synchronizes the system clock. It performs well on systems that are intermittently connected, on busy networks, or on systems with temperature-related clock drift.

---
# NTP Roles

From an NTP perspective, systems can function as **primary servers**, **secondary servers**, **peers**, or **clients**:

- **Primary server:** Obtains time from a reliable time source and provides it to secondary servers or clients.  
- **Secondary server:** Receives time from a primary server and can serve clients to reduce load or provide redundancy. Optional but recommended.  
- **Peer:** Exchanges time with other NTP servers at the same stratum level; all peers are equally reliable.  
- **Client:** Receives time from a primary or secondary server and synchronizes its system clock accordingly.

---
# Stratum Levels

Time sources in NTP are organized into hierarchical **stratum levels** based on their distance from highly accurate reference clocks (atomic, radio, GPS).

- **Stratum 0:** Reference clocks (most accurate). Not directly accessible on the network.
- **Stratum 1:** Servers directly connected to stratum 0 devices; considered highly accurate time servers.
- **Stratum 2–15:** Servers that synchronize from the next lower stratum (e.g., stratum 2 syncs from stratum 1). Accuracy decreases as stratum number increases.
- Servers at the same stratum can be configured as **peers** to exchange time updates.

Most public NTP servers operate at **stratum 2 or 3**, providing reliable time synchronization for general use.

---
# Chrony Configuration File

The primary configuration file for the Chrony NTP service is **/etc/chrony.conf**. It is read by the Chrony daemon at startup to determine time sources, logging behavior, and clock adjustment settings. The file can be edited manually.

## Common Directives

- **driftfile**: Specifies the file that records clock drift information, allowing Chrony to correct time more accurately.
- **logdir**: Defines the directory where Chrony stores its log files.
- **pool**: Points to a pool of NTP servers. Chrony automatically selects and switches servers as needed.  
  - **iburst**: Sends rapid initial requests (every 2 seconds) at startup to synchronize time quickly.
- **server**: Defines a specific NTP server by hostname or IP.  
  - `127.127.1.0` represents the local system clock.
- **peer**: Configures a peer time server at the same stratum level for mutual time exchange.

Chrony supports many additional directives for advanced configurations. Refer to `man chrony.conf` for full details.

---
# Chrony Daemon and Command

The Chrony service runs as a background daemon called **chronyd**, which synchronizes the system clock based on settings in **/etc/chrony.conf**. Instead of making abrupt time changes, Chrony gradually adjusts the clock in small steps to minimize disruption. Additional runtime options can be supplied to fine-tune daemon behavior.

Chrony also provides a command-line utility called **chronyc** to monitor and control the service while it is running. Common subcommands include:

- **sources**: Displays the current time sources and their status.
- **tracking**: Shows detailed performance statistics and clock accuracy information.

---
# Configure NTP Client

1. Install the Chronyy package using the dnf command:
```bash
sudo dnf -y install chrony
```
2. Ensure that preconfigured public time server entries are present in /etc/chrony.conf
```bash
 grep -E 'pool|server' /etc/chrony.conf  | grep -v ^#^C   
```
3. Start the Chrony sreveice and set it to autostart
```bash
sudo systemctl enable --now chrony
```
4. Examine the operational status of Chrony:
```bash
 sudo systemctl status chronyd --no-pager -l  
```
5. Inspect the binding status using the sources subcommand with chroncy:
```bash
chronyc sources
```
6. Display the clock performance using the trarcking subcommand with chronyc:
```bash
chronyc tracking
```
7. 
EXAM TIP: You will not have access to the outside network during the exam. You will need to point your system to an NTP server available on the exam network. Simply comment the default server/pool directive(s) and add a single directive “server hostname” to the file. Replace hostname with the NTP server name or its IP address as provided.

---

