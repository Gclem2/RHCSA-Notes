# Chapter 18 – The Secure Shell (SSH) Service

## Overview
Secure Shell (SSH) is a network service that provides **secure, encrypted communication** between systems over untrusted networks. It replaces legacy insecure tools by ensuring confidentiality, integrity, and authentication.

## Key Topics
- OpenSSH service, versions, and cryptographic algorithms
- Encryption techniques and authentication methods
- OpenSSH administration commands and configuration files
- Private/public key-based authentication
- Remote access to Linux systems using SSH
- Secure file transfer and remote synchronization over SSH

## RHCSA Objectives
- Access remote systems using SSH  
- Securely transfer files between systems  
- Configure key-based authentication for SSH  

## OpenSSH Capabilities
- Generates **public/private key pairs** for trusted, passwordless authentication
- Enables secure **remote login**, **command execution**, and **file transfer**
- Uses encrypted network channels to protect data in transit
- Widely adopted in enterprise environments as a replacement for insecure protocols

## Common SSH Utilities
- `ssh` – Secure remote login and command execution
- `scp` / `sftp` – Secure file transfer
- `rsync` (over SSH) – Remote file synchronization
- `ssh-keygen` – Key pair generation

SSH is a foundational tool for secure system administration and remote management in Linux environments.

---

## The OpenSSH Service

Secure Shell (SSH) delivers a secure mechanism for data transmission between source and destination systems over IP networks. It was designed to replace older remote login programs that transmitted user passwords in clear text and left data unencrypted.

SSH employs digital signatures for user authentication along with encryption to secure the communication channel. As a result, it is extremely difficult for unauthorized individuals to gain access to passwords or data in transit. SSH also monitors data transferred during a session to ensure integrity.

The OpenSSH suite includes utilities such as `ssh` and `sftp`, which allow remote users to log in to systems, execute commands, and transfer files securely over encrypted network connections.

---
## Common Encryption Techniques

Encryption is a method of scrambling information to conceal its true meaning from unauthorized access. OpenSSH can use various encryption techniques during an end-to-end communication session between two entities (client and server). The two most common techniques are symmetric and asymmetric encryption, also referred to as secret key encryption and public key encryption.

### Symmetric Technique
This technique uses a single key, called a *secret key*, which is generated through a negotiation process between two entities at the time of their initial contact. Both sides use the same secret key for encrypting and decrypting data during subsequent communication.

### Asymmetric Technique
This technique uses a pair of keys: a *private key* and a *public key*. These keys are randomly generated and mathematically related strings of alphanumeric characters attached to messages being exchanged. The client encrypts information using the public key, and the server decrypts it using the corresponding private key. The private key must be kept secure, as it belongs to a single entity, while the public key is distributed to clients. This technique is used for both secure channel establishment and user authentication.

---
## Authentication Methods

Once an encrypted channel is established between the client and server, additional negotiations take place to authenticate the user attempting to access the server. OpenSSH supports several authentication methods, which are attempted in the following order during the authentication process:

- GSSAPI-based (Generic Security Service Application Program Interface) authentication  
- Host-based authentication  
- Public key-based authentication  
- Challenge-response authentication  
- Password-based authentication  

Each method is described below.

### GSSAPI-Based Authentication
GSSAPI provides a standard interface that allows security mechanisms such as Kerberos to be integrated. OpenSSH uses this interface along with the underlying Kerberos infrastructure for authentication. With this method, an exchange of authentication tokens occurs between the client and the server to validate the user’s identity.

### Host-Based Authentication
Host-based authentication allows a single user, a group of users, or all users on a client system to be authenticated on a server. A user may be configured to log in with the same username on the server or as a different existing user. For individual users, a `~/.shosts` file is created containing the client hostname or IP address and, optionally, an alternate username.

For a group of users or all users on a client system, access is configured on the server using the `/etc/ssh/shosts.equiv` file.

### Private/Public Key-Based Authentication
This method uses a private and public key pair for user authentication. The client user possesses a private key, while the server stores the corresponding public key. During login, the server validates the key, and the user provides the passphrase associated with the private key. If both are verified, access is granted.

### Challenge-Response Authentication
Challenge-response authentication requires the user to correctly answer one or more predefined challenge questions. Access is granted only if the responses match the expected answers.

### Password-Based Authentication
This is the final fallback authentication method. The server prompts the user for their password and verifies it against the stored entry in the shadow password file. If the password matches, the user is authenticated.

Among these methods, password-based authentication is the most common and requires no additional configuration. GSSAPI-based, host-based, and challenge-response authentication methods are beyond the scope of this discussion. Public/private key-based authentication and encryption methods are the primary focus of the remainder of this chapter.

---
## OpenSSH Protocol Version and Algorithms

OpenSSH has evolved significantly over time. The latest and default version included with RHEL 9 is **SSH protocol version 2**, which introduces numerous enhancements, improved security, and more advanced configuration capabilities compared to earlier versions.

OpenSSH supports multiple cryptographic algorithms for **data encryption** and **user authentication (digital signatures)**, including **RSA**, **DSA**, and **ECDSA**. Among these, **RSA** is the most widely used because it supports both encryption and authentication. In contrast, **DSA** and **ECDSA** are limited to authentication only.

These

---
# OpenSSH Packages

## Three Main Packages

**openssh**
- Provides `ssh-keygen` command
- Includes library routines

**openssh-clients**
- Commands: `sftp`, `ssh`, `ssh-copy-id`
- Client config: `/etc/ssh/ssh_config`

**openssh-server**
- Service daemon: `sshd`
- Server config: `/etc/ssh/sshd_config`
- Library routines

> All three packages installed by default during RHEL installation
---
# OpenSSH Server Daemon and Client Commands

## Server Daemon (sshd)

- Preconfigured and operational on new RHEL installations
- Listens on **TCP port 22** (defined in `/etc/ssh/sshd_config` with `Port` directive)
- Allows remote login via SSH clients (PuTTY, ssh command, etc.)

> **Note**: `scp` command is deprecated due to security flaws - use `sftp` instead

## Client Commands

| Command       | Description                                    |
| ------------- | ---------------------------------------------- |
| `sftp`        | Secure remote file transfer program            |
| `ssh`         | Secure remote login command                    |
| `ssh-copy-id` | Copies public key to remote systems            |
| `ssh-keygen`  | Generates and manages private/public key pairs |

---

# Server Configuration File

## Location and Purpose

- **File**: `/etc/ssh/sshd_config`
- Defines default global settings for sshd operation
- **Log file**: `/var/log/secure` (captures authentication messages)
- Most directives work as-is for common use cases

## Key Directives

| Directive | Default | Description |
|-----------|---------|-------------|
| `Port` | 22 | Port number to listen on |
| `Protocol` | - | Default protocol version |
| `ListenAddress` | All | Local addresses sshd listens on |
| `SyslogFacility` | AUTHPRIV | Facility code for logging to `/var/log/secure` |
| `LogLevel` | INFO | Criticality level for logged messages |
| `PermitRootLogin` | yes | Allow/disallow direct root login |
| `PubKeyAuthentication` | yes | Enable/disable public key auth |
| `AuthorizedKeysFile` | `~/.ssh/authorized_keys` | Location of user's authorized keys |
| `PasswordAuthentication` | yes | Enable/disable password auth |
| `PermitEmptyPasswords` | no | Allow/disallow null passwords |
| `ChallengeResponseAuthentication` | yes | Enable/disable challenge-response auth |
| `UsePAM` | yes | Enable/disable PAM authentication (requires root to run sshd) |
| `X11Forwarding` | yes | Allow/disallow remote graphical application access |

> **Reference**: `man 5 sshd_config` for complete directive list

---
# Client Configuration File

## Location and Purpose

- **File**: `/etc/ssh/ssh_config`
- Directs how SSH client behaves on outbound connections
- Preset directives work as-is for most use cases

## Key Directives

| Directive | Default | Description |
|-----------|---------|-------------|
| `Host` | `*` (all hosts) | Container for directives applicable to specific host(s). Ends at next `Host` or `Match` |
| `ForwardX11` | no | Enable/disable automatic X11 traffic redirection over SSH |
| `PasswordAuthentication` | yes | Allow/disallow password authentication |
| `StrictHostKeyChecking` | ask | Controls host key handling:<br>• `no` - adds new keys, ignores mismatches<br>• `yes` - adds new keys, blocks mismatches<br>• `accept-new` - adds new keys, blocks mismatches<br>• `ask` - prompts for new keys, blocks mismatches |
| `IdentityFile` | `id_rsa`, `id_dsa`, `id_ecdsa` | Location of user's private key file (public key: same location with `.pub` extension) |
| `Port` | 22 | Port number to connect on |
| `Protocol` | - | Default protocol version |

## ~/.ssh Directory

- **Not created by default**
- Created when:
  - User runs `ssh-keygen` for first time (generates key pair)
  - User connects to remote SSH server and accepts host key
- **known_hosts file**: Stores server host keys with hostname/IP for authentication verification on subsequent connections

> **Reference**: `man 5 ssh_config` for complete directive list

---
# System Access and File Transfer

## Overview

- Users must log in to use the system or transfer files
- Login process identifies user to system
- Remote access requires resolvable hostname or IP address

## Remote Access Commands

| Command | Purpose | Security |
|---------|---------|----------|
| `ssh` | Remote system access | Secure (works over secure/insecure networks) |
| `sftp` | Secure file transfer | Secure (works over secure/insecure networks) |

## Common Scenarios

1. RHEL to RHEL access (server10 → server20)
2. Windows to RHEL access
3. SSH key-based authentication
4. File transfer using sftp

---
# Executing Commands Remotely Using ssh

## Overview

- `ssh` allows secure login OR remote command execution without logging in
- Can run any command on remote system without interactive login

## Syntax
```bash
ssh <user>@<hostname> <command>
```

## Examples

**Execute hostname command on server20:**
```bash
ssh user1@server20 hostname
```

**Show active network connections on server20:**
```bash
ssh user1@server20 nmcli c s
```

> Any command can be executed remotely this way without an interactive login session

---
# Executing Commands Remotely Using ssh

## Overview

- `ssh` allows secure login OR remote command execution without logging in
- Can run any command on remote system without interactive login

## Syntax
```bash
ssh <user>@<hostname> <command>
```

## Examples

**Execute hostname command on server20:**
```bash
ssh user1@server20 hostname
```

**Show active network connections on server20:**
```bash
ssh user1@server20 nmcli c s
```

> Any command can be executed remotely this way without an interactive login session

---
# Transferring Files Remotely Using sftp

## Launching sftp
```bash
sftp user1@server20
```

## Common Commands (Run on Remote Server)

| Command | Description |
|---------|-------------|
| `?` | List available commands with descriptions |
| `cd` | Change directory |
| `get` | Download file from remote to local |
| `put` | Upload file from local to remote |
| `ls` | List files |
| `pwd` | Print working directory |
| `mkdir` | Create directory |
| `rename` | Rename file |
| `rm` | Remove file |
| `bye` / `quit` / `exit` | Exit sftp and return to shell |

## Local Commands (Run on Source Server)

Commands prefixed with `l` operate on the **local** system:

| Command | Description |
|---------|-------------|
| `lcd` | Change local directory |
| `lls` | List local files |
| `lpwd` | Print local working directory |
| `lmkdir` | Create local directory |

## Usage

- Remote commands operate on **server20** (target)
- Local commands (with `l` prefix) operate on **server10** (source)
- Standard Linux commands available for basic file management
- Type `quit` to exit when done

> **Reference**: `man sftp` for options and details
# Exercise 18-2: Generate, Distribute, and Use SSH Keys
1. Generate RSA keys without a password  and without detailed output
```bash
ssh-keygen -N "" -q
```

2. Show public and private keys with cat
```bash
cat /.ssh/id_rsa
cat /.ssh/id_rsa.pub
```
3. Copy the public key file to server20 
```bash
ssh-copy-id server20
```
4. On server10, run ssh command as user1 to connect to server20
```bash
ssh server20
```