# Lab 15-1: Add New Interfface and Configure Connection Profile with nmcli
![](attachment/Pasted%20image%2020260115150534.png)
1. Verify the new interface
```bash 
ip a
```
![](attachment/Pasted%20image%2020260115144821.png)
2. List existing connections on NetworkManager
```bash
nmcli connection show
```
![](attachment/Pasted%20image%2020260115144957.png)
3. Create a new connection  for the new interface
```bash
sudo nmcli connection add type ethernet ifname enp0s8
con-name server30-3
```

4. Assign ip address and gateway:
```bash
sudo nmcli connection modify server30-3 \
ipv4.method manual \
ivp4.addresses 192.168.0.230/24 \
ipv4.gateway 192.168.0.1
```
5. Deactivate and Reactivate the connection
```bash
sudo nmcli connection down server30-3
sudo nmcli connection up server30-3
```
6. Verify network configuration
```bash
ip a show enp0s8
```
![](attachment/Pasted%20image%2020260115150154.png)
7. Add to host table
```
sudo vim /etc/hosts
```
![](attachment/Pasted%20image%2020260115150344.png)
8. Verify host table
```bash
getent hosts server30-3
```
![](attachment/Pasted%20image%2020260115150443.png)

---
# Lab 15-2: Add New Interface and Configure Connection Profile Manually

![](attachment/Pasted%20image%2020260115150556.png)

1. Verify the new interface
```bash
ip a
```
![](attachment/Pasted%20image%2020260115173555.png)
2. Go to NetworkManager connection profiles
```bash
cd /etc/NetworkManager/system-connections
ls
```
3. Copy the existing profile for the new interface
```bash
sudo cp enp9s0.nmconnection enp1s0.nmconnection
sudo chmod1 600 enp1s0.nmconnection
```
4. Edit the new connection file
```bash

sudo vim enp1s0.nmconnection
```

5. Reload NetworkManager connection files
```bash
sudo nmcli connection reload
```
6. Deactivate and reactive connection
```bash

sudo nmcli connection down enp1s0
sudo nmcli connection up enp1s0
```
7. Verify ip assignment
```bash
ip a show enp1s0
```
8. Add entry to host table
```bash
sudo vim /etc//hosts
```
9. Ping
```bash
ping -c 3 192.168.240
ping -c 3 sever30-3
```