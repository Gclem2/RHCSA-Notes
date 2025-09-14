# Installation and Initial Setup

This section describes setting up a **Red Hat Enterprise Linux (RHEL) 10 virtual machine** using VMware Workstation or VMware Player.

---

## 1. Download RHEL 10 ISO

1. Go to the [Red Hat Customer Portal](https://access.redhat.com/) and log in or create an account.
2. Navigate to the **Downloads** section and select **RHEL 10**.
3. Download the **RHEL 10 ISO** image for x86\_64 architecture.

---

## 2. Create a Virtual Machine in VMware

1. Open **VMware Workstation / Player**.
2. Click **Create a New Virtual Machine**.
3. Choose **Typical (recommended)**.
4. Select **Installer disc image file (ISO)** and browse to your downloaded RHEL 10 ISO.
5. Set the **guest operating system** to **Linux → Red Hat Enterprise Linux 10**.
6. Name your VM and choose a location for VM files.
7. Assign CPU, memory, and disk size:

   * CPU: 2 cores (or more if available)
   * Memory: 2–4 GB
   * Disk: 20 GB or more (preallocate disk for better performance)
8. Finish the VM creation wizard.

---

## 3. Install RHEL 10

1. Power on the VM and boot from the ISO.
2. Select **Install Red Hat Enterprise Linux 10** in the boot menu.
3. Choose **Language and Keyboard** preferences.
4. Select **Installation Destination**:

   * Select the virtual disk created earlier
   * Let the installer configure automatic partitioning (or manual for LVM practice)
5. Configure **Network & Hostname** if desired.
6. Set a **root password** and optionally create a user account.
7. Begin installation.

---

## 4. First Boot

1. After installation, remove the ISO from the virtual CD/DVD drive.
2. Reboot into your new RHEL 10 system.
3. Log in as root or your user account.
4. Update the system:

