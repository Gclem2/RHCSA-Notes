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
Lab 18-2: Test the Effect of PermitRootLogin Directive As user1 with sudo on server40, edit the /etc/ssh/sshd_config file and change the value of the directive PermitRootLogin to “no”. Use the systemctl command to activate the change. As root on server30, run ssh server40 (or use its IP). You’ll get permission denied message. Reverse the change on server40 and retry ssh server40. You should be able to log in. (Hint: The OpenSSH Service).