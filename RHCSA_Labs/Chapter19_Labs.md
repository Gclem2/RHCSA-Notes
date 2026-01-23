# Labs 19-1: Add service to Firewall

![](attachment/Pasted%20image%2020260123155243.png)
1. Add the HTTPS Service persistently
```bash
sudo firewall-cmd --permanent --add-serverice=https
```
2. Reload firewall
```bash
sudo firewall-cmd --reload
```
3. Verify with firewall-cmd
```bash
sudo firewall-cmd --list-services
```
4. Verify in the XML file
```bash
sudo cat /etc/firewalld/zones/public.xml
```
![](attachment/Pasted%20image%2020260123155658.png)

---
# Lab 19-2: Add Port Range to Firewall

## Objective
Add permanent UDP port range 8000-8005 to trusted zone on server30

## Steps

### 1. Add Port Range (Permanent)
```bash
# As user1 with sudo on server30
sudo firewall-cmd --permanent --zone=trusted --add-port=8000-8005/udp
```

### 2. Reload Firewall (Activate Change)
```bash
sudo firewall-cmd --reload
```

### 3. Verify with firewall-cmd
```bash
# List ports in trusted zone
firewall-cmd --zone=trusted --list-ports

# Or list all settings for trusted zone
firewall-cmd --zone=trusted --list-all
```
Expected output: `8000-8005/udp`

### 4. Verify in XML File
```bash
# Check trusted zone configuration file
sudo cat /etc/firewalld/zones/trusted.xml
```
