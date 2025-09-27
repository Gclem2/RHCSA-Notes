![alt text](Images/image.png)
# Lab 6-1 Create User and Configure Password Aging  
You can use groupadd and man -h if you need to remember the flags
``` bash
groupadd -g 6000 lnxgrp
```
``` bash
useradd -g 6000 -u 5000 user5000
```
``` bash
passwd user5000
```
``` bash
chage -m 4 -M 30 -W 10 user5000
```
``` bash
chage -E 2023-12-20 user5000
```
``` bash
chage -l user5000
```
![alt text](Images/6-1_image1.PNG)

# Lab 6-2 Lock and Unlock user
![alt text](Images/6-2_image2.PNG)
``` bash
passwd -l user5000
```
``` bash
sudo cat /etc/shadow
```

![alt text](image.png)
The ! infront of the hash indicates the account is locked

![alt text](Images/6-2_image3.PNG)

The authentication failure is because the account was properly locked

``` bash
usermod -U user5000
```

``` bash
grep user5000 /etc/shadow
```

![alt text](Images/6-2_image4.PNG)

# Lab 6-3 Modify group
``` bash
groupmod -g 7000 lnxgrp
```
``` bash
usermod -aG lnxgrp user1000
usermod -aG lnxgrp user2000
```
``` bash
groupmod -n  dbagrp lnxgrp
```
``` bash
grep dbagrp /etc/group
```
![alt text](Images/6-3_image1.PNG)

# Lab 6-4 Configuring Sudo Acess
![alt text](Images/6-4_image2.PNG)
```bash 
visudo
```
Put this text in the visudo file
![alt text](Images/6-4_image1.PNG)
You can see after running the vgs command in sudo there is no password prompt
![alt text](Images/6-4_image3.PNG)
