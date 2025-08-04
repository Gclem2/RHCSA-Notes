
## Projects & Tools

- `portscanner.py` — Python-based port scanner using `socket`
- IDS — Python-based Intrusion Detection System using `scapy`, `python-nmap`, `numpy`, `sklearn` 
- `hashcracker.sh` — Simple Bash script using `john` and `hashcat`
- TryHackMe writeups

---

## Lab Architecture Update

Originally, this lab used shared folders between the host and Kali VM for file access. That approach has been deprecated in favor of a more secure and realistic setup:

- I now **host this repo directly inside the Kali Linux VM**
- I use **SSH with key-based authentication** to connect from my host system
- **VS Code Remote-SSH** enables seamless code editing from the host
- Networking uses a **dual-adapter setup**:  
  - **Bridged Adapter** provides full internet access inside the VM  
  - **Host-Only Adapter** offers a stable internal IP for SSH access from the host

This setup improves security, realism, and workflow integrity.

---

## Learning Areas

This repo reflects my progress and interests in:
- Network scanning and enumeration
- Exploitation and privilege escalation
- Web application testing
- Scripting tools for automation

---

## Goals

- Complete 25+ TryHackMe rooms
- Build personal tools and automation scripts
- Set up a fully air-gapped malware analysis VM
- Document everything clearly for portfolio use

---

## Screenshots

| Kali Desktop | Nmap Scan | TryHackMe Lab |
|--------------|-----------|----------------|
| ![](./screenshots/kali-desktop.png) | ![](./screenshots/nmap-output.png) | ![](./screenshots/tryhackme-lab.png) |

---

## Note

This lab is fully virtualized and safely isolated from my home network using host-only adapters and best practices. No live malware or attacks are conducted outside contained VMs.

---

## Connect

Feel free to open issues, suggest improvements, or connect with me:

- GitHub: [Gclem2](https://github.com/Gclem2)
- LinkedIn: [Bryce Clemenson](https://www.linkedin.com/in/bryce-clemenson-45a16230b/)