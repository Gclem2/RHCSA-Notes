# Lab 20-1: Disable and Enable SELinux Operating Mode

## Objective
Toggle SELinux between disabled and enforcing modes on server30

## Steps

### 1. Check Current Mode
```bash
# As user1 with sudo
sudo getenforce
# Note the output (likely: Enforcing)
```

### 2. Disable SELinux
```bash
# Edit config file
sudo vi /etc/selinux/config

# Change to:
SELINUX=disabled

# Reboot to apply
sudo reboot
```

### 3. Verify Disabled Mode
```bash
# After reboot
sudo getenforce
# Output: Disabled
```

### 4. Re-enable SELinux
```bash
# Edit config file
sudo vi /etc/selinux/config

# Change to:
SELINUX=enforcing

# Reboot to apply
sudo reboot
```

### 5. Verify Enforcing Mode
```bash
# After reboot
sudo getenforce
# Output: Enforcing
```

## Key Points
- Changes to `/etc/selinux/config` require reboot
- Use `getenforce` to verify current mode
- Valid modes: `enforcing`, `permissive`, `disabled`

---
# Lab 20-2: Modify Context on Files

## Objective
Change and persist SELinux context on directory hierarchy

## Steps

### 1. Create Directory Hierarchy
```bash
# As user1 with sudo on server30
mkdir -p /tmp/d1/d2
```

### 2. Check Current Contexts
```bash
ls -Zd /tmp/d1
ls -Zd /tmp/d1/d2
# Note the current type (likely: tmp_t)
```

### 3. Change Context Recursively
```bash
# Change type to etc_t
sudo chcon -R -t etc_t /tmp/d1
```

### 4. Verify Context Change
```bash
ls -Zd /tmp/d1
ls -Zd /tmp/d1/d2
# Should show: ...etc_t...
```

### 5. Make Context Persistent
```bash
# Add to policy database
sudo semanage fcontext -a -t etc_t "/tmp/d1(/.*)?"

# Apply persistent context
sudo restorecon -Rv /tmp/d1
```

### 6. Verify Persistence
```bash
ls -Zd /tmp/d1 /tmp/d1/d2
# Context survives relabeling
```

## Key Points
- **chcon**: Temporary change (lost on relabeling)
- **semanage fcontext**: Adds to policy database (persistent)
- **restorecon**: Applies persistent context from policy
- Regex `"/tmp/d1(/.*)?"` applies to directory and all contents
---
# Lab 20-3: Add Network Port to Policy Database

## Objective
Add custom port 9005 to SELinux policy for HTTPS service

## Steps

### 1. Add Port to Policy
```bash
# As user1 with sudo on server30
sudo semanage port -a -t http_port_t -p tcp 9005
```

**Options**:
- `-a`: Add
- `-t http_port_t`: SELinux type for HTTP/HTTPS
- `-p tcp`: Protocol
- `9005`: Port number

### 2. Verify Addition
```bash
# List all HTTP ports
sudo semanage port -l | grep http_port_t

# Or check specific port
sudo semanage port -l | grep 9005
```

**Expected output**: Should show `9005` in `http_port_t` list

## Key Points
- Custom ports must be added to policy for SELinux-confined services
- Change is persistent (survives reboot)
- Use `semanage port -d` to delete port
- Common types: `http_port_t`, `ssh_port_t`, `smtp_port_t`

---
# Lab 20-4: Copy Files with and without Context

## Objective
Compare SELinux context behavior when copying files with/without --preserve=context

## Steps

### 1. Copy Without Preserving Context
```bash
# As user1 with sudo on server30
# Create file
touch /tmp/sef1

# Check source context
ls -Z /tmp/sef1
# Note: likely tmp_t type

# Copy to /usr/local
sudo cp /tmp/sef1 /usr/local/

# Check destination context
ls -Z /usr/local/sef1
# Note: inherits /usr/local context (likely usr_t)

# Compare contexts
ls -Z /tmp/sef1 /usr/local/sef1
```

### 2. Copy Preserving Context
```bash
# Create file
touch /tmp/sef2

# Check source context
ls -Z /tmp/sef2
# Note: tmp_t type

# Copy with --preserve=context
sudo cp --preserve=context /tmp/sef2 /var/local/

# Check destination context
ls -Z /var/local/sef2
# Note: retains tmp_t from source

# Compare contexts
ls -Z /tmp/sef2 /var/local/sef2
```

## Key Points
- **Without --preserve=context**: Destination inherits target directory's context
- **With --preserve=context**: Destination retains source file's original context
- Contexts differ based on directory location (`tmp_t`, `usr_t`, `var_t`, etc.)

---
# Lab 20-5: Flip SELinux Booleans

## Objective
Toggle SELinux Boolean value and verify changes

## Steps

### 1. Check Current Boolean Value
```bash
# As user1 with sudo on server30
# Method 1: getsebool
getsebool ssh_use_tcpd

# Method 2: sestatus
sudo sestatus -b | grep ssh_use_tcpd

# Note current value (on or off)
```

### 2. Toggle Boolean Value
```bash
# If currently off, turn on (and vice versa)
sudo setsebool -P ssh_use_tcpd on
# Or: sudo setsebool -P ssh_use_tcpd off
```

**Options**:
- `-P`: Make change persistent (saved to policy database)

### 3. Verify New Value
```bash
# Method 1: getsebool
getsebool ssh_use_tcpd

# Method 2: sestatus
sudo sestatus -b | grep ssh_use_tcpd

# Method 3: semanage
sudo semanage boolean -l | grep ssh_use_tcpd
```

## Key Points
- **Without -P**: Temporary (lost on reboot)
- **With -P**: Persistent (survives reboot)
- Changes take effect immediately
- Boolean values: `on` (1) or `off` (0)
---
