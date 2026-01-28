![](attachment/Pasted%20image%2020260127142519.png)
```bash
#!/bin/bash                                          
DISKS=(/dev/vdb /dev/vdc)                             # Create 800MB partition on each disk                                                                  
for DISK in "${DISKS[@]}"                                                                              
do                                                                                                     
    echo "Creating partition on $DISK"                                                                 
    sudo parted "$DISK" --script mklabel gpt                                                           
    sudo parted "$DISK" --script mkpart primary 1MiB 801MiB                                            
    sudo parted "$DISK" --script set 1 lvm on                                                          
done                                                                                                   
# Initialize physical volumes                                                                          
echo "Creating physical volumes"                                                                       
sudo pvcreate /dev/vdb1 /dev/vdc1                                                                      
# Create volume group                                                                                  
echo "Creating volume group vgscript"            
```
---
# Lab 21-2: Write Script to Create File Systems

## Objective
Create file systems on LVs from Lab 21-1, mount them, and verify

## Script: create_filesystems.sh
```bash
#!/bin/bash

# Define arrays for LVs, filesystem types, and mount points
LVS=(lvscript1 lvscript2 lvscript3)
FSTYPES=(xfs ext4 vfat)
MOUNTS=(/mnt/xfs /mnt/ext4 /mnt/vfat)

# Create file systems
for i in {0..2}
do
    echo "Creating ${FSTYPES[$i]} filesystem on /dev/vgscript/${LVS[$i]}"
    sudo mkfs.${FSTYPES[$i]} /dev/vgscript/${LVS[$i]}
done

# Create mount points and mount
for i in {0..2}
do
    echo "Creating mount point ${MOUNTS[$i]}"
    sudo mkdir -p ${MOUNTS[$i]}
    
    echo "Mounting /dev/vgscript/${LVS[$i]} on ${MOUNTS[$i]}"
    sudo mount /dev/vgscript/${LVS[$i]} ${MOUNTS[$i]}
done

# Display mounted file systems
echo "Mounted file systems:"
df -h | grep vgscript
```

## Execute Script
```bash
chmod +x create_filesystems.sh
./create_filesystems.sh
```

## Expected Output
```
Creating xfs filesystem on /dev/vgscript/lvscript1
Creating ext4 filesystem on /dev/vgscript/lvscript2
Creating vfat filesystem on /dev/vgscript/lvscript3
...
Mounted file systems:
/dev/mapper/vgscript-lvscript1  500M  ... /mnt/xfs
/dev/mapper/vgscript-lvscript2  500M  ... /mnt/ext4
/dev/mapper/vgscript-lvscript3  500M  ... /mnt/vfat
```

## Verify
```bash
# Check mount points
mount | grep vgscript

# Check file systems
lsblk -f | grep vgscript

# Check disk usage
df -h /mnt/{xfs,ext4,vfat}
```

## Key Script Elements
- **Indexed arrays**: Access elements by position `${ARRAY[$i]}`
- **{0..2}**: Loop through indices 0, 1, 2
- **mkdir -p**: Create parent directories if needed
- **mkfs.${FSTYPE}**: Dynamic filesystem creation
- **grep vgscript**: Filter df output for our LVs

## Array Index Matching
```
Index 0: lvscript1 → xfs   → /mnt/xfs
Index 1: lvscript2 → ext4  → /mnt/ext4
Index 2: lvscript3 → vfat  → /mnt/vfat
```
---
# Lab 21-3: Write Script to Configure New Network Connection Profile
![](attachment/Pasted%20image%2020260127170904.png)
# Lab 21-3: Write Script to Configure New Network Connection Profile

## Objective
Configure new network interface with custom IP, backup /etc/hosts, add hostname mapping

## Prerequisites
- New network interface presented to server40 (e.g., enp0s8)
- Run as user1 with sudo

## Script: network_setup.sh
```bash
#!/bin/bash

# Variables
DEVICE="enp0s8"
CONN_NAME="lab-connection"
IP_ADDR="192.168.100.50/24"
GATEWAY="192.168.100.1"
DNS="8.8.8.8"
HOSTNAME="labserver.example.com"

# Backup /etc/hosts
echo "Backing up /etc/hosts"
sudo cp /etc/hosts /etc/hosts.backup.$(date +%Y%m%d)

# Configure network connection
echo "Creating network connection $CONN_NAME on $DEVICE"
sudo nmcli connection add \
    type ethernet \
    con-name $CONN_NAME \
    ifname $DEVICE \
    ip4 $IP_ADDR \
    gw4 $GATEWAY

# Set DNS
echo "Setting DNS server"
sudo nmcli connection modify $CONN_NAME ipv4.dns $DNS

# Set manual method
sudo nmcli connection modify $CONN_NAME ipv4.method manual

# Activate connection
echo "Activating connection"
sudo nmcli connection up $CONN_NAME

# Add hostname mapping to /etc/hosts
echo "Adding hostname mapping to /etc/hosts"
echo "$IP_ADDR $HOSTNAME" | sudo tee -a /etc/hosts

# Verify configuration
echo "Network configuration:"
sudo nmcli connection show $CONN_NAME | grep -i ip4
echo ""
echo "/etc/hosts content:"
cat /etc/hosts

echo "Network setup complete"
```

## Execute Script
```bash
chmod +x network_setup.sh
./network_setup.sh
```

## Verify Setup
```bash
# Check connection
nmcli connection show

# Check IP assignment
ip addr show enp0s8

# Test connectivity
ping -c 3 192.168.100.1

# Verify /etc/hosts
cat /etc/hosts

# Check backup
ls -l /etc/hosts.backup.*
```

## Expected Results
- **Connection**: lab-connection active on enp0s8
- **IP**: 192.168.100.50/24
- **Gateway**: 192.168.100.1
- **DNS**: 8.8.8.8
- **/etc/hosts**: Contains new hostname mapping
- **Backup**: /etc/hosts.backup.YYYYMMDD created

## Key Script Elements
- **Variables**: Store configuration values at top
- **date +%Y%m%d**: Timestamp for backup file
- **tee -a**: Append to file (preserves existing content)
- **nmcli connection add**: Create new connection profile
- **nmcli connection modify**: Update connection settings
- **Backslash (\)**: Line continuation for readability

## Notes
- Adjust DEVICE name based on your system (check with `ip link`)
- Choose IP settings that match your network
- Script creates timestamped backup of /etc/hosts
- `-a` flag in `tee` appends without overwriting