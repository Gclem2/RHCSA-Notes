# Lab 18-1: Establish Key-Based Authentication

## Objective
Set up passwordless SSH authentication for user20 from server40 to server30

## Steps

### 1. Create User Account (on both servers)
```bash
# As user1 with sudo on server30 and server40
sudo useradd user20
sudo passwd user20
```

### 2. Generate SSH Key Pair (on server40)
```bash
# As user20 on server40
ssh-keygen
# Press Enter for all prompts (no passphrase, default location)
```

### 3. Copy Public Key to server30 (from server40)
```bash
# As user20 on server40
ssh-copy-id user20@server30
# Enter user20's password when prompted
# Accept server fingerprints if presented
```

### 4. Test Passwordless Login
```bash
# As user20 on server40
ssh user20@server30
# Should login WITHOUT password prompt
```

## Key Points
- Private key stored: `~/.ssh/id_rsa` (server40)
- Public key stored: `~/.ssh/id_rsa.pub` (server40)
- Authorized keys: `~/.ssh/authorized_keys` (server30)
- First login requires accepting host fingerprint
- Subsequent logins are passwordless

---
# Lab 18-2: Test the Effect of PermitRootLogin Directive

## Objective
Test how `PermitRootLogin` directive controls root SSH access

## Steps

### 1. Disable Root Login (on server40)
```bash
# As user1 with sudo on server40
sudo vi /etc/ssh/sshd_config
```
Change:
```
PermitRootLogin no
```

### 2. Apply Changes
```bash
# Restart sshd service
sudo systemctl restart sshd
```

### 3. Test Root Login (from server30)
```bash
# As root on server30
ssh root@server40
# Result: Permission denied
```

### 4. Re-enable Root Login (on server40)
```bash
# As user1 with sudo on server40
sudo vi /etc/ssh/sshd_config
```
Change:
```
PermitRootLogin yes
```

### 5. Apply and Retest
```bash
# Restart sshd service
sudo systemctl restart sshd

# From server30 as root
ssh root@server40
# Result: Login successful
```

## Key Points
- `PermitRootLogin no` blocks direct root SSH access
- Changes require `systemctl restart sshd` to take effect
- Security best practice: disable root login, use sudo instead