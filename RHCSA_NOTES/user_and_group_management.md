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

## 8. Password Hashing and Security

* Default hashing: **SHA-512**.
* Password aging, min/max days, warning period configurable in `/etc/login.defs`.
* Encrypted passwords protect against unauthorized access.

---

✅ **Practice Tips**

* Use `id` and `groups` to inspect users.
* Check `/etc/passwd`, `/etc/shadow`, `/etc/group`, `/etc/gshadow`.
* Create, modify, and delete users using `useradd`, `usermod`, `userdel`.
* Monitor logins with `w`, `who`, `last`, `lastb`, and `lastlog`.
