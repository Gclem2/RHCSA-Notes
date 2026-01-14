# Lab 13-1: Create and Remove Partitions with parted

![](attachment/Pasted%20image%2020260113121114.png)
1. Identify the 250 MB disk
```bash
lsblk
```
![](attachment/Pasted%20image%2020260113121147.png)

2. I'll use sdb as the disk
```bash
sudo parted /dev/sdb
print
mklabel msdos
mkpart primary 1MB 101MB
```
![](attachment/Pasted%20image%2020260113121703.png)k
3. Make the second partition
```bash
sudo parted /dev/sdb
print
mkpart primary 101MB 201MB
quit
```
![](attachment/Pasted%20image%2020260113121859.png)
```bash
lsblk
```
![](attachment/Pasted%20image%2020260113121952.png)
4. Remove both partitions
```bash
sudo parted /dev/sdb
print
rm 1
rm 2
```

5. Final check
```bash
lsblk
```
![](attachment/Pasted%20image%2020260113122609.png)

---
# Lab 13-2: Create and Remove Partittions with gdisk
![](attachment/Pasted%20image%2020260113123024.png)
1. Go into gdisk on the disk you need
```bash
sudo gdisk /dev/sdb
```
![](attachment/Pasted%20image%2020260113124020.png)
2. Repeat for the second partiton
3. Then list the partitions
![](attachment/Pasted%20image%2020260113124112.png)
4. To remove the partitions
![](attachment/Pasted%20image%2020260113124307.png)
5. To check run lsblk
```bash
lsblk
```

---
# Lab 13-3: Create Volume Group and Logical Volumes
![](attachment/Pasted%20image%2020260113124410.png)

1. Identify an unusued 250MB disk
```bash
lsblk
``` 
2. Initialize the disk as an LVM Physical Volume
```bash
sudo pvcreate /dev/sdb
sudo pvs
```
3. Create Volume Group vg100 with 16MB PE size
```bash
sudo vgcreate -s 16M vg100 /dev/sdb
sudo vgs
sudo vgdisplay1 vg100
```

4. Create Logical Volume
```bash
sudo lvcreate -n lvol0 -L 90M vg100
```
5. Create another one
```bash
sudo lvcreate1 -n swapvol -L 120M vg100
```
6. Verify  all LVM components
```bash
sudo pvs
sudo vgs
sudo lvs
sudo vgdisplay vg100
```

---
# Lab 13-4: Expaand Volume Group and Logical Volume

![](attachment/Pasted%20image%2020260113133147.png)
1. View the disk layout
```bash
lsblk
```
2. Created a partition for LVM
```bash
sudo parted /dev/sdc
mklabel gpt \
mkpart primary 1MiB 100%
```
3. Verify with command
```bash
lsblk
```
4. Initialize the partition as a phsyical volume
```bash
sudo pvcreate /dev/sdc1
sudo pvs
```

5. Add the new PV to the volume group
```bash
sudo vgextend1 vg100 /dev/sdc1
sudo vgs
```
6. Expand logical volume lvol0 to 300 MB
```bash
sudo lvextend -L 300M /dev/vg100/lvol0
sudo lvs
```
7. Final Verifcation
```bash
sudo pvs
sudo vgs
sudo lvs
sudo vgdisplay vg100
```

---
# Lab 13-5: Add a VDO Logical Volume

![](attachment/Pasted%20image%2020260113143507.png)
1. Initialize sde as an LVM physical volume
```bash
sudo pvcreate /dev/sde
sudo pvs
```
2. Add the PV to volume group vg100
```bash
sudo vgextend vg100 /dev/sde
sudo vgs
```
3. Create a VDO logical volume using all the free space
```bash
sudo lvcreate \
--type vdo \
-n vdovol \
-l 100%FREE \
vg100

#Verify volume 
sudo lvs
```
4. Final Verification 
```bash
sudo pvs
sudo vgs
sudo lvs
```

