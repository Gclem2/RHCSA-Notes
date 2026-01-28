# Chapter 22: Containers

## Overview
Package, deploy, and run isolated applications using Linux container technology

## Key Concepts
- **Containers**: Isolated, self-contained applications with all dependencies
- **Images**: Templates for creating containers
- **Registries**: Repositories storing container images
- **Containerization**: Docker, Kubernetes, Podman technologies

## Benefits
- Consistent deployment across environments
- Application isolation on same/different servers
- Leverages native Linux kernel virtualization
- Encapsulates app + dependencies (libraries, configs, binaries, services)

## RHCSA Objectives
- **61**: Find and retrieve images from remote registry
- **62**: Inspect container images
- **63**: Container management (podman, skopeo commands)
- **64**: Build container from Containerfile
- **65**: Basic management (run, start, stop, list containers)
- **66**: Run service inside container
- **67**: Configure container as systemd service (auto-start)
- **68**: Attach persistent storage to container

## Topics Covered
1. Container technology fundamentals
2. Linux kernel features enabling containers
3. Container benefits analysis
4. Container images and registries
5. Root vs rootless containers
6. Registry configuration
7. Image management (find, inspect, pull, list, delete)
8. Build images with Containerfile
9. Basic container administration (start, stop, remove, interact)
10. Advanced features (port mapping, environment variables, persistent storage)
11. systemd integration for container control

## Tools
- **podman**: Container management
- **skopeo**: Image operations
- **systemd**: Container service controlhhh
---
# Introduction to Containers

## Traditional Deployment Challenges
- Multiple apps on single server cause conflicts:
  - Shared library incompatibilities
  - Package dependency conflicts
  - Software version mismatches
- OS updates/patches may break applications
- Requires analysis before collocating new apps

## Container Solution

### What is a Container?
**Isolated set of processes** running in complete seclusion on Linux system

### Container Contents
Single image file packaging:
- Application
- Dependencies
- Shared libraries
- Environment variables
- Configuration specifics

### Key Characteristics
- **Isolation**: Each container runs independently
- **Portability**: Portable across servers without impact
- **Density**: Single system supports tens/hundreds of containers
- **Whole unit**: Start, stop, restart, tag, transport as complete entity
- **Conflict avoidance**: No app/component/OS conflicts

## Deployment Flexibility
- **Platform**: Linux (also Windows)
- **Location**: Bare metal, VM, on-premises, cloud

## Terminology
- **Containerized applications**: Apps packaged to run in containers
- **Containerization**: Architecture/deployment trend for apps, components, databases

## Benefits Summary
- Eliminates dependency conflicts
- Consistent deployment
- Easy migration between servers
- Isolated runtime environments
- No impact on other containers
---
# Benefits of Using Containers

## Isolation
Run fully isolated from host OS and other apps - unaffected by external changes

## Loose Coupling
Self-contained with minimal OS dependency

## Maintenance Independence
Individual container maintenance without affecting others

## Less Overhead
Require fewer system resources than bare metal/VMs

## Transition Time
**Seconds** to start/stop (vs minutes for VMs)

## Transition Independence
Start/stop without affecting other containers or host OS services (no restart required)

## Portability
Migrate between servers without modification:
- Bare metal ↔ VM
- On-premises ↔ Cloud

## Reusability
Same image runs identically across:
- Development
- Test
- Preproduction
- Production
**No rebuild needed**

## Rapidity
Accelerated development, testing, deployment, patching, scaling - minimal testing required

## Version Control
Multiple image versions available - choose appropriate version for deployment

## Key Advantages Summary
**Fast** • **Portable** • **Isolated** • **Lightweight** • **Consistent**

---
# Container Home: Bare Metal or Virtual Machine

## Virtual Machine Architecture
**Hypervisor layer** (VMware ESXi, VirtualBox, Hyper-V, KVM) on bare metal:
- Multiple VMs share physical hardware
- Each VM runs isolated guest OS
- Applications share VM's virtualized resources

**Drawbacks**:
- Management/operational overhead from hypervisor services
- OS updates may require reboot
- Potential application failures from updates

## Container Architecture
Run **directly on host OS** (bare metal or VM):
- Share hardware and OS resources securely
- Share same Linux kernel
- No hypervisor layer needed (if on bare metal)

**Advantages**:
- Lightweight and isolated
- Parallel execution
- Far fewer hardware resources than VMs
- Speedy start/stop times
- No hypervisor overhead (on bare metal)

## Three Deployment Models

### 1. Traditional Bare Metal
Hardware → OS → Applications

### 2. Virtual Machines
Hardware → Hypervisor → VMs (each with guest OS) → Applications

### 3. Containers on Bare Metal
Hardware → Host OS → Containers → Applications

## Comparison
| Feature | VMs | Containers |
|---------|-----|------------|
| **Extra layer** | Hypervisor | None (on bare metal) |
| **OS instances** | Multiple guest OS | Single shared kernel |
| **Resource usage** | Higher | Lower |
| **Start time** | Minutes | Seconds |
| **Overhead** | Hypervisor management | Minimal |

## Recommendation
Running containers on **bare metal servers** most beneficial and economical (eliminates hypervisor layer)

**Note**: Requires use case study to determine best option

---
# Rootful vs. Rootless Containers

## Rootful Containers (Root Privileges)

### Launched As
- Root user (directly)
- With `sudo`

### Capabilities
- Full administrative access
- Map privileged network ports (**1024 and below**)
- Perform all administrative functions

### Security Risk
**Vulnerability**: If container compromised → potential unauthorized access to host

## Rootless Containers (Recommended)

### Launched As
Normal Linux users (unprivileged)

### Characteristics
- Run without root privileges
- **Cannot** perform privileged tasks
- **Cannot** map privileged ports (<1024)

### Security Benefit
**Isolation**: Container compromise does NOT grant host access

## Best Practice
**Use rootless containers** to secure:
- Containers themselves
- Underlying operating system

## Key Distinction
| Type         | User             | Privileged Ports | Admin Functions | Security |      |
| ------------ | ---------------- | ---------------- | --------------- | -------- | ---- |
| **Rootful**  | root/sudo        | Yes (<1024)      | Yes             | Lower    |      |
| **Rootless** | Regular user<br> | No               | No              | Higher   | <br> |

---
# Install Necessary Container Support

1. Install the container-tools package:
```bash
sudo dnf install -y container-tools
```
2. Verify the package installation:
```bash
dnf list container-tools
```
---
# The podman Command

## Overview
Primary tool for managing images and containers

## Image Management Subcommands

| Subcommand | Description |
|------------|-------------|
| `build` | Build image using Containerfile instructions |
| `images` | List downloaded images in local storage |
| `inspect` | Examine image and display details |
| `login`/`logout` | Authenticate to/from registry (required for private registries) |
| `pull` | Download image from registry to local storage |
| `rmi` | Remove image from local storage |
| `search` | Search for image in registries |
| `tag` | Add name/version to image (default: `latest`) |

### search Options
- **Partial name**: Returns all matching images
- `--no-trunc`: Show full output without truncation
- `--limit <number>`: Limit results to specified count

## Container Management Subcommands

| Subcommand | Description |
|------------|-------------|
| `attach` | Attach to running container |
| `exec` | Run process in running container |
| `generate` | Generate systemd unit file (use `--new` option) |
| `info` | Show system info including defined registries |
| `inspect` | Display container configuration |
| `ps` | List running containers (add `-a` for stopped containers) |
| `rm` | Remove container |
| `run` | Launch new container from image |
| `start`/`stop`/`restart` | Control container state |

### run Options
- `-d`: Detached mode (background)
- `-i`: Interactive mode
- `-t`: Allocate pseudo-TTY (terminal)

## Common Usage Patterns
```bash
# Image operations
podman search <image>
podman pull <FQIN>
podman images
podman rmi <image>

# Container operations
podman run -dit <image>
podman ps -a
podman exec <container> <command>
podman stop <container>
podman rm <container>
```

## EXAM TIP
**Solid understanding of podman command usage is key** to completing container tasks

> **Reference**: `man podman` for complete subcommand details
---
# The skopeo Command

## Overview
Interact with local and remote images and registries

## Key Subcommand: inspect
Examine details of image stored in **remote registry** (without downloading)

## Usage
```bash
skopeo inspect docker://<registry>/<image>:<tag>
```

### Example
```bash
skopeo inspect docker://registry.redhat.io/rhel9/httpd-24:latest
```

## Key Difference from podman inspect
- **skopeo inspect**: Inspects image in **remote registry** (no download)
- **podman inspect**: Inspects image in **local storage** (must be pulled first)

## Other Subcommands
Available but beyond RHCSA scope

> **Reference**: `man skopeo` for complete subcommand details
> ---
# The registries.conf File

## File Locations

| Location | Scope | Precedence |
|----------|-------|------------|
| `/etc/containers/registries.conf` | System-wide | Default |
| `~/.config/containers/registries.conf` | Per-user | Overrides system-wide |

**Use case**: Per-user file useful for rootless containers

## Configuration

### Default Searchable Registries
```ini
unqualified-search-registries = [
    "registry.access.redhat.com",
    "registry.redhat.io",
    "docker.io"
]
```

**Search order**: Left to right (top to bottom)

### Add Private Registry (Highest Priority)
```ini
unqualified-search-registries = [
    "registry.private.myorg.io",
    "registry.access.redhat.com",
    "registry.redhat.io",
    "docker.io"
]
```

### Use Only Private Registry
```ini
unqualified-search-registries = ["registry.private.myorg.io"]
```

## Features
- **Searchable registries**: Registries podman searches for images
- **Blocked registries**: Registries to exclude (can be configured)
- **Order matters**: First match wins

## Customization
1. Edit file to add/remove/reorder registries
2. Place preferred registry first in list
3. Remove unwanted registries

## EXAM TIP
**No Internet access during Red Hat exams** - may need to access network-based registry to download images

## Key Points
- Controls which registries podman searches
- Per-user config overrides system-wide
- Search order determined by list position
- Default config works for most use cases
---
# Viewing Podman Configuration and Version

## Display System Configuration
```bash
podman info
```

### Shows
- Runtime and configuration file locations
- Registry definitions
- Storage configuration
- Memory info (from `/proc/meminfo`)
- Kernel version (via `uname -r`)
- Container/image counts
- Host information

## Root vs. Rootless Differences

### As Normal User (rootless)
```bash
podman info
```

**Key fields**:
- `rootless: true`
- `ConfigFile: ~/.config/containers/...`
- `ImageStore: ~/.local/share/containers/...`

### As Root
```bash
sudo podman info
```

**Key fields**:
- `rootless: false`
- `ConfigFile: /etc/containers/...`
- `ImageStore: /var/lib/containers/...`

### Differences
| Aspect | Root | Rootless |
|--------|------|----------|
| **rootless** | false | true |
| **ConfigFile** | `/etc/containers/` | `~/.config/containers/` |
| **ImageStore** | `/var/lib/containers/` | `~/.local/share/containers/` |
| **Local images** | Separate storage | Separate storage |

## Check Podman Version
```bash
podman version
```

**Example output**: Version 4.2.0

## Key Points
- Root and rootless users have **separate storage locations**
- Root and rootless users have **separate image stores**
- Configuration data stored in different locations
- Use `podman info` to verify configuration
- Use `podman version` to check utility version
---
# Search, Examine, Download, and Remove an Image

1. Log in to the specified Red Hat registry:
```bash
podman login registry.redhat.io
```

2. Confirm a succesful login:
```bash
podman login registry.redhat.io
```
3. Find the mysql-80 image in the specified registry
```bash
podman search registryy.redhat.io/mysql-80 --no-trunc
```

4. Select the second image rhel9/mysql80
```bash
skopeo inspect docker://registry.redhat.io/rhel9/myysql-80
```
5. Pull the image with podman
```bash
podman pull docker://registry.redhat.io/rhel9/mysql-80  
```
6. List the image to confirm the retrieval using podman images:
```bash
podman images
```
7. Display the image's details using podman inspect
```bash
podman inspect mysql-80
```
8. Remove the mysql-80 image from local storage using podman rmi:
```bash
podman rmi mysql-80
```
9. Confirm the removal using podman images:
```bash
podman images
```

---
# Viewing Podman Configuration and Version

## Display Configuration
```bash
podman info
```
Shows: registries, storage, memory, kernel version, container/image counts

## Root vs. Rootless Differences

| Aspect | Root | Rootless |
|--------|------|----------|
| **rootless** | false | true |
| **ConfigFile** | `/etc/containers/` | `~/.config/containers/` |
| **ImageStore** | `/var/lib/containers/` | `~/.local/share/containers/` |

**Note**: Root and rootless use **separate storage and configs**

## Check Version
```bash
podman version
# Example: 4.2.0
```
---
# Containerfile

## Overview
Text file with instructions to build custom container images using `podman build`

**File name**: `Containerfile` (or any name)

## Common Instructions

| Instruction | Description |
|-------------|-------------|
| `FROM` | Base image to use |
| `RUN` | Execute commands during build |
| `COPY` | Copy files to image |
| `WORKDIR` | Set working directory (auto-created if missing) |
| `USER` | Define non-root user for commands |
| `ENV` | Define environment variables |
| `EXPOSE` | Port to open when container launches |
| `CMD` | Default command to run |

## Example Containerfile
```dockerfile
# Use RHEL 9 UBI as base
FROM registry.access.redhat.com/ubi9/ubi:latest

# Install Apache
RUN yum install -y httpd && yum clean all

# Copy custom index file
COPY index.xhtml /var/www/html/

# Set working directory
WORKDIR /var/www/html

# Run as non-root user
USER apache

# Expose port 80
EXPOSE 80

# Start Apache
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
```

## Build Image
```bash
podman build -t custom-httpd:latest .
```

## EXAM TIP
**Solid understanding of Containerfile instructions and usage is important**

---
# Use Containerfile to Build Image

1. Log in to the specified Red Haat registry:
```bash
podman login registryy.redhat.io
```
2. Confirm a successful login:
```bash
podman login registry.redhat.io --get-login
```
3. Create a file called containerfile with the following code:
```bash
cat containerfile
```
4. Create a file called testfile with some random text in it and place it in the same direcotry as the containerfile
5. Build an image by specifying the containerfile name and the image tagg suchh as ub9-simple-image.
```bash
podman image build -f containerfile -t ubi9-simple-image .
```
6. Confirm image creation:
```
podman image ls
```
---
# Basic Container Management

## Common Tasks
Start, stop, list, inspect, delete containers

## Launch Options
- Named or anonymous
- Interactive terminal (`-it`)
- Detached/background (`-d`)
- Auto-remove after exit (`--rm`)
- Entry point command (custom command at launch)

## Container Lifecycle
```bash
# Launch
podman run [options] <image>

# List running
podman ps

# List all (including stopped)
podman ps -a

# Stop
podman stop <container>

# Start
podman start <container>

# Restart
podman restart <container>

# Remove
podman rm <container>

# Inspect
podman inspect <container>
```

## Key Point
**podman command** used for all container lifecycle operations

---
# Run, Interact with, and Remove a Named Container

1. Launch a container using ubi8. Name this container rhel8-base os and open a terminal session -t for interaction -i
```bash
podman run -ti --name rhel8-base-os ubi8
```
2. Run some commands to verify
```bash
ls 
pwd
date
```
3. Close the terminal session when done:
```bash
exit
```
4. Delete the container using podman rm:
```bash
podman rm rhel8-base-os
```
---
# Run a Nameless Container and Auto-Remove it After Entry Point Command Execution

1. Start a container using ubi7 (RHEL 7) and run ls as an entry point command. Use podman run with the -rm option to remove the container
```bash
podman run --rm ubi7 ls
```
2. Confirm the container removal with podman ps:
```bash
podman ps
```
---
# Basic Container Management

## Common Tasks
Start, stop, list, inspect, delete containers

## Launch Options
- Named or anonymous
- Interactive terminal (`-it`)
- Detached/background (`-d`)
- Auto-remove after exit (`--rm`)
- Entry point command (custom command at launch)

## Container Lifecycle
```bash
# Launch
podman run [options] <image>

# List running
podman ps

# List all (including stopped)
podman ps -a

# Stop
podman stop <container>

# Start
podman start <container>

# Restart
podman restart <container>

# Remove
podman rm <container>

# Inspect
podman inspect <container>
```

## Key Point
**podman command** used for all container lifecycle operations

---
# Containers and Port Mapping

## Purpose
Enable communication:
- **Container ↔ Container**: E.g., Apache web server ↔ MySQL database
- **Container ↔ Outside world**: E.g., web traffic on port 80/8080

## Port Mapping
Establish mappings between host system ports and container ports

### Syntax
```bash
podman run -p <host_port>:<container_port> <image>
```

### Example
```bash
# Map host port 8080 to container port 80
podman run -p 8080:80 httpd

# Access from outside: http://host_ip:8080 → container port 80
```

## Multiple Mappings
```bash
podman run -p 8080:80 -p 8443:443 <image>
```

## EXAM TIP
**Rootless users CANNOT map host ports below 1024**

**Valid (rootless)**: `podman run -p 8080:80 httpd` ✓  
**Invalid (rootless)**: `podman run -p 80:80 httpd` ✗ (requires root)

---
# Configure Port Mapping

1. Search for an Apache web server image for RHEL 7 using podman search
```bash
podman search registry.redhat.io/rhel7/httpd
```
2. Log in to registry.redhat.io using the RedHat credentials to access the image:
```bash
podman login registry.redhat.io
```
3. Download the latest version of the Apache image using podman pull:
```bash
podman pull registry.redhat.io/rhscl/httpd-24-rhel7
```
4. Verify the download using podman images:

```bash
podman images
```
5. Launch a container named(--name) rhel7-port-map in detatched mode -d to run the containerized Apache web server -p 8000
```bash
podman run -dp 10000:8000 --name rhel7-port-map httpd-24-rhel7
```
6. Verifyy that the container was launched successfully using podman ps:
```bash
podman ps
```
7. You can also use podman port to view the mapping:
```bash
podman port rhel7-port-map 8000/tcp -> 0.0.0.0:10000
```
---
# Stop, Restart and Remove a Container

1. Verify the current operational state of the container rhel7-port-map:
```bash
podman ps
```
2. Stop the container and confirm using the stop and ps subcommands
```bash 
podman stop rhel7-port-map
podman ps -a
```
3. Start the container and confirm with the start and ps subcommands:
```bash
podman start rhel7-port-map
```
4. Stop the container and remove it using the stop and rm submcommands:
```bash
podman stop rhel7-port-map
```
5. Confirm the removal using the -a option with podman ps to also include any stopped containers
```bash
podman ps -a
`
```bash
podman ps -a
`
```bash
podman ps -a
`
```bash
podman ps -a
`
```bash
podman ps -a
```
---
# Containers and Environment Variables

## Purpose
- Pass host environment variables to container (e.g., `PATH`)
- Set new variables for container use:
  - Debugging flags
  - Sensitive data (passwords, access keys, secrets)

## Syntax
```bash
podman run -e VARIABLE=value <image>
```

## Examples

### Pass Host Variable
```bash
podman run -e PATH=$PATH <image>
```

### Set New Variable
```bash
podman run -e DB_PASSWORD=secret123 <image>
```

### Multiple Variables
```bash
podman run -e VAR1=value1 -e VAR2=value2 -e VAR3=value3 <image>
```

## EXAM TIP
**Use -e option for EACH variable** (multiple `-e` flags allowed)

## Verification Inside Container
```bash
podman exec <container> env
# or
podman exec <container> printenv VARIABLE
```

> See Chapter 07 for details on variable types and management

---
# Pass and Set Environment Variables
1. Launch a container with an interactive terminal session (-it) and inject (-e) variables HISTSIZE and SECRET
```bash
podman run -it -e HISTSIZE -e SECRET="secret123" --name rhel9-env-vars ubi9
```
2. Verify both variables using the echo command:
```bash
echo $HISTSIZE $SECRET
```
3. Disconnected from the container using the exit command, and stop and remove it using the stop and rm subcommands:
```bash
exit
podman stop rhel9-env-vars
podman rm rhel9-env-vars
```
---
# Containers and Persistent Storage

## Problem
Container data lost on restart/failure/termination (containers are ephemeral)

## Solution
Attach host directory to container for persistent data storage

## How It Works
1. Container sees attached host directory as local directory
2. Application stores data in attached directory
3. Data persists after container reboot/removal
4. Directory can be re-attached to other containers
5. Host directory can be on local or remote filesystem

## Attach Host Directory
```bash
podman run -v <host_dir>:<container_dir>[:options] <image>
```

### Example
```bash
podman run -v /data/mysql:/var/lib/mysql:Z mysql
```

## Preparation Steps (Host Directory)

### 1. Create Directory
```bash
mkdir -p /data/myapp
```

### 2. Set Ownership
```bash
chown user:group /data/myapp
```

### 3. Set Permissions
```bash
chmod 755 /data/myapp  # or appropriate permissions
```

### 4. Set SELinux Type
```bash
semanage fcontext -a -t container_file_t "/data/myapp(/.*)?"
restorecon -Rv /data/myapp
```

## SELinux Context Options
- **:z**: Shared between multiple containers
- **:Z**: Private to single container (recommended)
```bash
podman run -v /data/myapp:/app:Z <image>
```

## EXAM TIP
**Proper ownership, permissions, and SELinux type (container_file_t) must be set** for persistent storage to work without issues

## Key Points
- Host directory must have correct ownership/permissions
- SELinux type `container_file_t` prevents unauthorized access
- Protects host and other containers if container compromised
- Data survives container lifecycle
---
# Attach Persistent Storage and Access Data Across Containers

1. Create a directory called /host_date set full permissions it, and confirm:
```bash
sudo mkdir /host_data
sudo chmod 777 /host_data/
ll -d /host_data/
```
2. Launch a root container called rhel9-persisten-data
```bash
sudo podman run --name rhel9-persistent-data -v /host_data:/container_data:Z -it ubi9
```
3. Confirm the presence of the directory inside the container with ls on /container_data:
```bash
ls -ldZ /container_data
```
4. Create a file called testfile with the echo command under /container_data:
```bash
echo "This is persistent storage." > /container_data/testfile
```
5. Verify the file creation and the SELinux tyype
```bash
ls -lZ /container_data/
```
6. Exit out of the container and check the presence of the file in the host directory:
```bash
exit
ls -lz /host_data/
```
7. Stop and remove the container using the stop and rm subocommands:
```bash
sudo podman stop rhel9-persistent-data
sudo podman rm rhel9-persistent-data
```
8. Launch a new root container called rhel8-persistent-data in interactive mode
```bash
sudo podman run -it --name rhel8-persistent-data
-v/host_data:/container_data2:Z ubi8
```