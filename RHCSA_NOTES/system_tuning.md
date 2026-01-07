# Chapter 12: System Initialization, Logging, and Tuning (Exam Focus)

## Overview
RHEL uses **systemd** for system initialization, service management, logging, and tuning. Administrators manage system states using **units** and **targets**, analyze logs for troubleshooting, and apply **tuning profiles** to optimize performance or power usage.
### systemd
- Default init system in RHEL
- Manages services, boot process, and system states
- Uses **units** (resources) and **targets** (operational states)
### Units
- Configuration files defining system resources
- Common types:
  - `service` – background services
  - `target` – system state (runlevel replacement)
- Stored in `/usr/lib/systemd/system` and `/etc/systemd/system`
### Targets
- Group of units representing a system mode
- Examples:
  - `multi-user.target` – text-based, network enabled
  - `graphical.target` – GUI
  - `rescue.target`, `emergency.target` – recovery
- Default target controls boot behavior
## Service & Target Management
- List units and their status
- Start, stop, restart services
- Enable/disable services at boot
- Switch targets temporarily or permanently
## Logging

### Traditional Logging
- Logs stored under `/var/log`
- Configured via rsyslog
- Log rotation handled by **logrotate**

### systemd Journal
- Managed by `systemd-journald`
- Stores boot-time and runtime logs
- Query using `journalctl`
- Can be volatile (memory) or persistent (disk)
## System Tuning
- Managed by the **tuned** service
- Uses tuning profiles for:
  - Performance
  - Power saving
  - Virtualization
- Profiles can be recommended and activated

---
## System Initialization & Service Management (systemd — Exam Focus)

- **systemd** is the default init and service manager in RHEL.
- It is the **first process (PID 1)** started at boot and the **last to stop** at shutdown.
- Key benefits:
  - **Parallel service startup** → faster boot
  - **Dependency management**
  - **On-demand (lazy) service activation**
  - **State snapshotting**
  - Automatic mount handling

### Resource Management
- systemd organizes processes into **control groups (cgroups)**.
- cgroups allow:
  - Resource limiting (CPU, memory, I/O, network)
  - Isolation and prioritization of services
- Improves overall system performance and stability.

### Service Activation Methods
- **Socket-based activation**:
  - systemd creates sockets first, then starts services in parallel.
  - Client requests are queued until the service starts.
  - Services do not need to be running—only the socket must exist.
- **D-Bus activation**:
  - Service starts when a client communicates with it.
- **Device-based activation**:
  - Service starts when specific hardware (e.g., USB) is detected.
- **Path-based activation**:
  - Service starts when a file or directory changes.

### Communication Mechanisms
- **Sockets**: IPC or network communication between processes.
- **D-Bus**: Message bus for communication between local or remote services.

### On-Demand Services
- Services like **Bluetooth** and **printing** start only when needed.
- Saves boot time and system resources.

### Filesystem Mount Optimization
- systemd uses **autofs** during boot:
  - Temporarily mounts filesystems to avoid boot delays.
  - Remounts them normally after filesystem checks complete.
- Root (`/`) and virtual filesystems are unaffected.

**Bottom line:**  
systemd accelerates boot time, improves reliability, and efficiently manages services using parallelism, cgroups, and on-demand activation.

---
# Units
### What Are Units
- **Units** are systemd objects that manage boot and runtime tasks:
  - Services, mounts, sockets, devices, targets, etc.
- Units can be **active, inactive, activating, deactivating, or failed**.
- Units can be **enabled** (can start at boot) or **disabled**.

### Unit Naming & Types
- Format: `unitname.type`  
  - Examples: `sshd.service`, `tmp.mount`, `syslog.socket`, `graphical.target`
- systemd defines **11 unit types**:

| Unit Type | Purpose                             |
| --------- | ----------------------------------- |
| Automount | On-demand filesystem mounting       |
| Device    | Kernel device representation        |
| Mount     | Mount/unmount filesystems           |
| Path      | Start service on file/dir change    |
| Scope     | Manage externally started processes |
| Service   | Manage daemons                      |
| Slice     | Group units for resource control    |
| Socket    | IPC/network sockets                 |
| Swap      | Swap partitions                     |
| Target    | Logical grouping of units           |
| Timer     | Time-based activation               |

### Unit File Locations (Priority Order)
1. `/etc/systemd/system` → **Admin/user-defined (highest priority)**
2. `/run/systemd/system` → Runtime-generated
3. `/usr/lib/systemd/system` → Package-provided (lowest priority)

> Unit files replace legacy `/etc/rc.d/init.d` scripts.

### Unit File Structure
- **[Unit]**: Description, dependencies, ordering, conflicts
- **[Install]**: Enable/disable behavior (targets)
- **Type-specific section**:
  - `[Service]`, `[Socket]`, `[Mount]`, etc.

### Dependencies & Ordering
- **Before / After** → startup order
- **Requires** → must be running or unit fails
- **Wants** → preferred but not required
- **Conflicts** → cannot run together

Example:
- `graphical.target` **requires** `multi-user.target`
- `rescue.target` **conflicts** with `graphical.target`

### Key Exam Notes
- systemd usually manages dependencies automatically
- Manual dependency tuning is possible if needed
- Use: `man systemd.unit` for details
---
# Targets 
### What Are Targets
- **Targets** are logical collections of systemd units (`.target`)
- Used to bring the system into a specific **operational state** (run level)
- Targets may **inherit units** from other targets and add more
- Stored in the same locations as other unit files:
  - `/usr/lib/systemd/system`
  - `/etc/systemd/system`
  - `/run/systemd/system`


| Target              | Purpose                                              |
| ------------------- | ---------------------------------------------------- |
| `halt.target`       | Shut down and halt system                            |
| `poweroff.target`   | Shut down and power off                              |
| `shutdown.target`   | System shutdown                                      |
| `rescue.target`     | Single-user mode, filesystems mounted, no networking |
| `emergency.target`  | Minimal shell, root FS read-only, no services        |
| `multi-user.target` | Multi-user, networking enabled, no GUI               |
| `graphical.target`  | Multi-user with networking and GUI                   |
| `reboot.target`     | Shut down and reboot                                 |
| `default.target`    | Symlink to default boot target                       |
| `hibernate.target`  | Save system state and power off                      |

### Target Files
- Contain **only a `[Unit]` section**
- Define:
  - Dependencies (`Requires`, `Wants`)
  - Ordering (`After`)
  - Conflicts (`Conflicts`)

Example (`graphical.target`):
- **Requires**: `multi-user.target`
- **Wants**: `display-manager.service`
- **Conflicts**: `rescue.target`
- **After**: `multi-user.target`

### Key Exam Notes
- `default.target` determines **boot target**
- `multi-user.target` ≈ runlevel 3
- `graphical.target` ≈ runlevel 5
- `rescue` vs `emergency`:
  - **rescue**: usable system, no network
  - **emergency**: minimal shell, read-only root
- Reference: `man systemd.target`
---
# Systemctl Command 
### Purpose
- `systemctl` is the **primary management tool** for `systemd`
- Used to **start, stop, enable, disable, query, and inspect** units and targets

### Common systemctl Subcommands

| Command                         | Function                              |
| ------------------------------- | ------------------------------------- |
| `daemon-reload`                 | Reload unit files after changes       |
| `enable` / `disable`            | Enable or disable autostart at boot   |
| `get-default` / `set-default`   | Show or set default boot target       |
| `start` / `stop`                | Start or stop a unit                  |
| `restart`                       | Stop and start a unit                 |
| `reload`                        | Reload unit config without restarting |
| `status`                        | Show detailed unit status             |
| `is-active`                     | Check if unit is running              |
| `is-enabled`                    | Check if unit starts at boot          |
| `is-failed`                     | Check if unit is failed               |
| `list-units`                    | List active units (default behavior)  |
| `list-unit-files`               | List all installed unit files         |
| `list-dependencies`             | Show unit dependency tree             |
| `list-sockets`                  | List socket units                     |
| `isolate`                       | Switch system to another target       |
| `mask` / `unmask`               | Prevent or allow unit activation      |
| `kill`                          | Terminate all processes of a unit     |
| `show`                          | Display unit properties               |
| `get-property` / `set-property` | Read or modify unit properties        |

### Key Exam Notes
- Always run `systemctl daemon-reload` after editing unit files
- `enable` ≠ `start` (boot vs runtime)
- `mask` fully blocks a service (stronger than disable)
- `isolate` changes targets immediately
- `status` is the **most commonly tested** command
---
# Listing and Viewing Units
### Default Listing
- `systemctl`
  - Lists **active units** loaded in memory
  - Columns:
    - **UNIT**: Unit name
    - **LOAD**: Config load status (loaded, error, masked, etc.)
    - **ACTIVE**: High-level state (active, inactive, failed, activating)
    - **SUB**: Low-level, unit-specific state
    - **DESCRIPTION**: Unit purpose

### Common Listing Commands (Exam-Relevant)

- List **all units** (active + inactive):
  ```bash
  systemctl --all

```

---
# Managing Service Units

Check the detailed view of a service:
```bash
systemctl status atd
```

Service states
active (running): Service running
active (exited): One-time task completed
active (waiting): Running, waiting for event
inactive: Not running
activating / deactivating: Transitioning
failed: Crashed or failed to start

Disable autostart:
```bash
systemctl disable atd
```

Enable autostart:
```bash
systemctl enable atd
```

Check enabled state:
```bash
systemctl is-enabled atd
```

Start/Stop/Restart
Check if running:
```bash
systemctl is-active atd
```

Stop or Restart:
```bash
systemctl stop atd
systemctl restart atd
```

Masking Services
```bash
systemctl mask atd
systemctl unmask atd
```

---

# Managing Target Units

View active loaded target units
```bash
systemctl -t target
```

Command to view default boot target and set it

```bash
systemctl get-default
sudo systemctl set-default multi-user
```

EXAM TIP: set-default makes the change PERSISTENT

## Switch Targets on a Running System

```bash
sudo systemctl isolate multi-user.target
```
Switch back to graphical:
```bash
sudo systemctl isolate graphical.target
```

Power State Targets

```bash
sudo systemctl poweroff
sudo systemctl reboot
sudo systemctl halt
```

---
# System Logging

## System Logging

System logging (syslog) captures messages from the kernel, daemons, commands, applications, and user activities for troubleshooting, auditing, or informational purposes.

### Logging Daemon: rsyslogd
- **Daemon:** `rsyslogd` (rocket-fast system for log processing)
- **Features:** Multi-threaded, filtering, encryption, modular configuration
- **Configuration Files:**  
  - `/etc/rsyslog.conf`  
  - `/etc/rsyslog.d/` directory
- **Default Log Location:** `/var/log` (subdirectories for services like Apache, audit, GNOME, etc.)
- **Modules:** Dynamically loaded as needed to extend functionality
- **Control via systemctl:**  
  ```bash
  sudo systemctl start|stop|restart|reload|status rsyslog
  ```

---
# Syslog Configuration File

The primary syslog configuration file is `/etc/rsyslog.conf`. It has three main sections: **Global Directives**, **Modules**, and **Rules**.

### Global Directives
These directives control overall rsyslog behavior:
- **`$WorkDirectory /var/lib/rsyslog`** – location for auxiliary files  
- **`$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat`** – sets traditional file formatting for logs  
- **`$IncludeConfig /etc/rsyslog.d/*.conf`** – loads additional configuration files

### Modules
- **`imuxsock`** – enables local system logging (e.g., `logger` command)  
- **`imjournal`** – allows access to the systemd journal  

Modules are loaded on demand.

### Rules
Each rule has two fields:
1. **Selector** – left field; defines **facility.priority**  
   - **Facility**: system process categories (auth, cron, daemon, kern, mail, user, local0–local7, `*` for all)  
   - **Priority**: message severity (emerg, alert, crit, error, warning, notice, info, debug, none; `*` for all)  
   - Multiple `facility.priority` entries are separated by a semicolon `;`
2. **Action** – right field; destination for messages (file, terminal, etc.)

#### Examples of Rules
- Log informational messages from all services to `/var/log/messages` (except mail, auth, cron)  
- Log authentication messages to `/var/log/secure`  
- Log mail messages to `/var/log/maillog`  
- Log cron messages to `/var/log/cron`  
- Display emergency messages to all logged-in users (omusrmsg)  
- Log critical messages from `uucp` and `news` to `/var/log/spooler`  
- Record boot-time startup messages to `/var/log/boot.log`

### Validating Changes
After editing `/etc/rsyslog.conf`:
```bash
rsyslogd -N1   # Check configuration for syntax errors with verbosity level 1
```

---
# Rotating Log Files

RHEL stores system logs under `/var/log`. Logs can grow quickly, potentially filling the filesystem or making them difficult to analyze. To prevent this, log files are **rotated** regularly using `logrotate`.

## Logrotate Service

A systemd timer unit, `logrotate.timer` (`/usr/lib/systemd/system/logrotate.timer`), triggers the logrotate service daily:

- **Service file**: `/usr/lib/systemd/system/logrotate.service`  
- **Configuration**: `/etc/logrotate.conf` and `/etc/logrotate.d/*`

## Default Log Rotation Configuration

`/etc/logrotate.conf` defines global rotation settings:

- **Frequency**: weekly  
- **Retention**: 4 weeks  
- **Replacement**: new empty file created with date suffix  
- **Compression**: optional (gzip)  
- **rsyslog restart**: rotates logs and restarts rsyslog  
- **Overrides**: settings in `/etc/logrotate.d/*` take priority for specific logs

## Example: Additional Log Configuration Files

`/etc/logrotate.d/` contains service-specific log rules:

- `chrony`, `dnf`, `rsyslog`, `samba`, etc.

Example: `/etc/logrotate.d/btmp` (failed login attempts)

- **Rotation**: monthly  
- **Permissions**: read/write for owner `root` and group `utmp`  
- **Retention**: 1 rotated copy  
- Ensures `/var/log/btmp` is rotated safely without losing critical audit data

---
# The Boot Log File

During system startup, services generate logs showing their **startup sequence** and status. This helps with **post-boot troubleshooting**.  

- **Location**: `/var/log/boot.log`  
- **Format**: Each service shows `OK` or `FAILED` in square brackets `[ ]` to indicate whether it started successfully.  

Example excerpt from `boot.log`:

---
# The System Log File

The primary system log for most activities is located at `/var/log/messages`, as defined in `rsyslog.conf`.  

- **Format**: Plain text; viewable with utilities like `cat`, `more`, `less`, `head`, or `tail`.  
- **Real-time monitoring**: Use `tail -f /var/log/messages` to watch events as they occur.  
- **Contents**: Each entry typically includes:  
  - Date and time of the event  
  - Hostname  
  - Service name and PID  
  - Short description of the event  

**EXAM TIP**: Use `tail -f` when starting or restarting services to troubleshoot issues.  

Example entries from `/var/log/messages`:

---
# Logging Custom Messages

You can add manual notes to the system log to mark events, track script execution, or debug application startup. The `imuxsock` module in `rsyslog.conf` supports recording custom messages via the `logger` command.  

### Example: Log a Reboot Event
```bash
logger "User $(whoami) has rebooted the system"
```

View the Message
```bash
tail -f /var/log/messages
```

Setting Priority 

```bash
logger -p local0.info "Custom messages"
```

---
# The systemd Journal

RHEL provides a systemd-based logging service via the `systemd-journald` daemon. It collects, stores, and displays logs from sources such as the kernel, `rsyslog`, other services, the initial RAM disk, and early boot alerts.  

Logs are stored in binary **journal files** under `/run/log/journal`, which are indexed for fast searches and managed with the `journalctl` command. Since `/run` is a memory-based filesystem, these logs are **non-persistent** by default, but persistent storage can be enabled.  

Both `rsyslogd` and `systemd-journald` run concurrently. `systemd-journald` can forward logs to `rsyslogd` for persistent text storage.  

The main configuration file is `/etc/systemd/journald.conf`, where default settings can be modified as needed.

---
# Retrieving and Viewing Messages

RHEL provides the `journalctl` command to retrieve and view messages from the systemd journal. Running it without options displays all messages since the last system reboot. Each entry shows a **timestamp**, **hostname**, **process name (with PID if available)**, and the **message content**.  

## Common `journalctl` Options

- **Verbose output**:  
```bash
sudo journalctl -o verbose
```

View all events since last system reboot:
```bash
sudo journalctl -b
sudo journalctl -0 # Last reboot
sudo journalctl -1 # the previous system reboot
```

View only kernel-generated alerts sine last reboot:
```bash
sudo journalctl -kb0
```

Limit the output to view a specific number
```bash
sudo journalctl -n3
```

Show all alerts from a particular service
```bash
sudo journalctl /usr/sbin/crond
```
To retreive messages logged for a ertain process associated with a PID
```bash
sudo journalctl PID=$(pgrep chronyd)
```

Reveall all messages from a particular system unit:
```bash
sudo journalct _SYSTEMD_UNIT=sshd.service
```

To view all error messages logged between a date range:
```bash
sudo journalctl --since 2023-01-25 --until 2023-01-31 -p err
```

To get all warning messages that have appeared today and display them in reverse chronological order:
```bash
sudo journalctl --since today -p warning -r
```

Similar to the -f (follow) option that is used with the tail command for real-time viewing of a log file, you can use the same switch with journalctl as well.
```bash
sudo journalctl -f
```

---
# Preserving Journal Information

By default, systemd journals are stored in the **/run/log/journal** directory, which is **volatile** and lost upon reboot. The `rsyslogd` daemon reads from this temporary location and stores messages persistently in **/var/log/messages**.  

To make journal logs persistent, create the directory:  
```bash
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
sudo systemctl restart systemd-jjournald
```

---
# Exercise 12-1: Configure Persistent Store for Journal Information

1. Create a subdirectory called journal under the /var/log directory and confirm:
```bash
sudo mkdir /var/log/journal
```
```bash
ls -ld /var/log/journal
```

Restart the systemd-journald service and confirm:

```bash
sudo systemctl restart systmemd-journald
sudo systemctl status systemd-journald
```

List the new directory and observe a sub-directory matching the machine ID of the system as defined in the /etc/machine-id file is created:
```bash
ll/vaar/log/journal/
```

```bash
cat /etc/machine-id
```

---
# System Tuning

RHEL uses the **tuned** service to monitor system components—such as storage, networking, CPU, audio, and video devices—and adjusts their parameters to optimize **performance** or **power consumption**.  

## Tuning Profiles

RHEL ships with several predefined **tuning profiles** for common scenarios. Profiles can be applied:

- **Statically** – A profile is activated at service startup and remains in use until manually switched. This is the **default behavior**.
- **Dynamically** – System settings are adjusted automatically based on real-time activity from monitored components. For example:  
  - CPU and disk usage increase during program execution.  
  - Network bandwidth utilization rises during large file transfers.  
Dynamic tuning ensures optimal **performance** while minimizing **power consumption** during varying workloads.

---
# Tuning Profiles

# Tuning Profiles

The **tuned** service in RHEL includes **nine predefined profiles** to optimize for performance, power saving, or a balance of both. Custom profiles can also be created from scratch or using an existing profile as a template. Custom profiles must be stored under `/etc/tuned` to be recognized by the service.

## Profile Groups

Tuning profiles fall into three main groups:

### 1. Optimized for Better Performance
- **Desktop** – Based on balanced profile for desktops; improves throughput for interactive apps.  
- **Latency-performance** – Reduces latency for time-sensitive workloads.  
- **Network-latency** – Derived from latency-performance; improves network response.  
- **Network-throughput** – Maximizes network throughput.  
- **Virtual-guest** – Optimized for virtual machines.  
- **Virtual-host** – Optimized for hosts running virtual machines.  

### 2. Optimized for Power Saving
- **Powersave** – Minimizes power usage at the cost of performance.  

### 3. Balanced/Max Profiles
- **Balanced** – Default choice; balances performance and power saving.  
- **Throughput-performance** – Maximizes system performance; consumes maximum power.  

## Locations

- Predefined profiles are in `/usr/lib/tuned/<profile-name>`.  
- The default active profile on virtualized systems like VirtualBox is typically **virtual-guest**.

---
# Exercise 12-2: Manage Tuning Profiles

1. Install tuned package if it is not already installed:
```bash
sudo dnf install -y tuned
```

2. Start the tuned service and set it to autostart at reboots:
```bash
sudo systemctl --now enable tuned
```

3. Confirm startup
```bash
sudo systemctl status tuned
```

4. Display the list of available tuning profiles:
```bash
sudo tuned-adm list
```

5. List only the current active profile:
```bash
sudo tuned-adm active
```

Switch to powersave profile and confirm:
```bash
sudo tuned-adm profile powersave
sudo tuned-adm active
```

7. Determine the recommended profile for server1 and switch to it:
```bash
sudo tuned-adm recommend
sudo tuned-adm profile virtual-guest
```

8. Turn off tuning and confirm:
```bash
sudo tuned-adm off
sudo tuned-adm active
```

9. Reactivate tuning and confirm:
```bash
sudo tuned-adm profile virtual-guest
sudo tuned-adm active
```

---

