# User Management and System Info Commands

This document summarizes key commands and files for **user account management and system information** in RHEL.

---

## 1. User and Login Monitoring

* **w**: Displays who is logged in and CPU time, system load (similar info to `uptime`).
* **who**: Shows logged-in users.
* **load average**: Numbers from `w`/`uptime` show CPU load; 0.00 = idle, 1.00 = full load, >1 = overloaded.
* **last**: Reports successful logins and reboots (`last reboot` for only reboots). Consults `/var/log/wtmp`.
* **lastb**: Reports unsuccessful login attempts (root only).
* **lastlog**: Shows most recent login info for every user.

---

## 2. User Identification

* **id**: Shows UID, username, GID, primary and secondary groups, SELinux context. Can check other users with `id username`.
* **groups**: Lists all groups a user belongs to.

---

## 3. User Account Types

* **Root**: Superuser (UID 0).
* **Normal**: Standard user.
* **Service**: System accounts.

Important files:

* `/etc/passwd`, `/etc/shadow`, `/etc/group`, `/etc/gshadow`
* `shadow` and `gshadow` have no access for regular users.

---

## 4. /etc/passwd File

Structure (colon-separated fields):

1. **Login Name**: Username (max 255 chars, avoid special chars and uppercase).
2. **Password**: `x` (points to `/etc/shadow`), `*` (disabled), or hashed password.
3. **UID**: 0=root, 1–200=core services, 201–999=non-core services, 1000+=normal users.
4. **GID**: Primary group, matches `/etc/group`.
5. **GECOS**: Optional comments (name, phone, location).
6. **Home Directory**: Default `/home/username`.
7. **Shell**: Default `/bin/bash`.

```bash
head -3 /etc/passwd; tail -3 /etc/passwd
```

---

## 5. /etc/shadow File

Stores hashed passwords and aging info:

1. Login Name
2. Encrypted Password
3. Last Change (days since epoch)
4. Minimum days before change
5. Maximum password age
6. Warning days before expiry
7. Password inactivity period
8. Account expiry date
9. Reserved

Permissions: 000 (root only). Updated automatically on user changes.

---

## 6. Group Files

* **/etc/group**: Plaintext file listing groups and members. Permissions 644.
* **/etc/gshadow**: Stores hashed group passwords, admins, and members. Permissions 000 (root only).

---

## 7. User Account Commands

**useradd**: Add new user, picks defaults from `/etc/default/useradd` and `/etc/login.defs`.

Options:

```text
-b (--base-dir)    Base directory for home
-c (--comment)     User info
-d (--home-dir)    Home directory path
-D (--defaults)    Show defaults
-e (--expiredate)  Expire account on date
-f (--inactive)    Max days of inactivity
-g (--gid)         Primary GID
-G (--groups)      Supplementary groups
-k (--skel)        Skeleton directory for new accounts
-m (--create-home) Create home directory
-o (--non-unique)  Allow duplicate UID
-r (--system)      Create system account
-s (--shell)       Login shell
-u (--uid)         Specify UID
login              Login name
```

**usermod**: Modify existing user. Similar options, plus:

```text
-a (--append)      Add to supplementary groups
-l (--login)       Change login name
-m (--move-home)   Move home directory
```

---
**chage**: Configure password aging.
-d (--lastday)     Last password change
-E (--expiredate)  Expire account on date
-I (--inactive)    Days after expiry before lock
-l                 List password aging attributes
-m (--mindays)     Min days before password change
-M (--maxdays)     Max password age
-W (--warndays)    Days before expiry to warn user


## 8. Group Management

**groupadd**: Create a group.
-g (--gid)      Specify GID
-o (--non-unique) Use existing GID
-r              Create system group (<1000)
groupname       Name of the group

**groupmod**: Modify group attributes -n to rename

**groupdel**: Remove group entries from /etc/group and /etc/gshadow


## 8. Switing Users

**su**: Switch to another user (requires password).
**whoami**: Shows current effective username.
**logname**: Shows original username.
**su -c 'command**': Run a command as another user without switching shell.
**nologin shell**: Assign to users that should not log in. Located in /sbin/nologin or /usr/sbin/nologin. Can display a custom message via /etc/nologin.txt.


## 8. Sudo and Priviledged Access
**sudo**: Execute commands with elevated privileges. Sudoers configuration edited via visudo.
Full root access:
user1    ALL=(ALL) ALL
%dba     ALL=(ALL) ALL

No password prompt:
user1    ALL=(ALL) NOPASSWD:ALL
%dba     ALL=(ALL) NOPASSWD:ALL

Restrict for specific commands:
user1    ALL=/usr/bin/cat
%dba     ALL=/usr/bin/cat

Alias for multiple users/commands:
Cmnd_Alias  PKGCMD = /usr/bin/yum, /usr/bin/rpm
User_Alias  PKGADM = user1, user100, user200
PKGADM      ALL = PKGCMD



