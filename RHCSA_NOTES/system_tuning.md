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
