Project Blanco: Building a Home Lab for Cloud & IT Skills Development

This document details the foundational build of "Project Blanco," a personal home lab designed to provide hands-on experience in virtualization, Linux administration, networking, and self-hosting. This project directly supports my academic pursuits in Cloud Computing and aims to develop practical, real-world IT skills.

Key Technologies Utilized:

•	Hypervisor: Proxmox VE 8.x (Debian Linux-based)

•	Hardware: Intel N100 Mini PC (Firebat) with 16GB RAM, 512GB NVMe SSD

•	Guest Operating System: Ubuntu Server 24.04 LTS

•	Networking: Ethernet, DHCP, Static IP Configuration, Subnetting

•	Remote Access: Secure Shell (SSH)

Key Outcomes Achieved (Phase 1: Foundation):
•	Successful bare-metal deployment of Proxmox VE on dedicated mini PC hardware.

•	Robust network configuration and troubleshooting, ensuring stable connectivity for the hypervisor and virtual machines.

•	Deployment and initial configuration of a foundational Linux virtual machine (ubuntu-practice), ready for hosting various services.

•	Established secure remote access to the Linux VM via SSH from both laptop and mobile devices.

### Current Status: Personal Cloud Operational!

Successfully deployed a robust personal cloud solution using **Nextcloud Hub**. This involved comprehensive setup of Apache web server, MariaDB database, and PHP, alongside meticulous troubleshooting of configuration and database privilege issues. The Nextcloud instance is fully functional, supporting file file storage, collaboration, and seamless access via web and mobile applications (iOS/Android).

Skills Demonstrated:
•	Hardware Selection & Setup

•	Operating System Installation (Hypervisor & Guest OS)

•	Network Configuration & Troubleshooting (including subnet conflicts and static IP assignment)

•	Linux Command Line Administration (e.g., nano, apt, ip a)

•	Virtual Machine Management (creation, configuration, lifecycle)

•	Problem Solving & Debugging (systematic approach to network and repository issues)

•	Technical Documentation (ongoing)

## Lessons Learned & Skills Obtained

Project Blanco has been an intensive and incredibly rewarding hands-on journey, transforming theoretical knowledge into practical expertise. Beyond just deploying services, this endeavor has profoundly enhanced my abilities in systematic problem-solving and strategic self-directed learning within complex IT environments.

### Key Technical Skills Developed:

* **Virtualization & Hypervisor Management:**
    * Deployment and administration of Proxmox VE (Type-1 hypervisor).
    * Virtual Machine (VM) creation, configuration, and lifecycle management.
    * Understanding virtual hardware (BIOS/UEFI, VirtIO drivers, SCSI vs. VirtIO Block).
    * VM integration with host via QEMU Guest Agent.
* **Linux System Administration:**
    * Core command-line interface (CLI) proficiency (`ssh`, `sudo`, `ls`, `cd`, `mkdir`, `rm`, `nano`, `cat`, `systemctl`, `apt`).
    * Package management and repository configuration (`apt update/upgrade/dist-upgrade`, managing `sources.list.d`).
    * User and group management fundamentals.
    * Basic shell scripting (e.g., custom login banners with `figlet`/`toilet`).
* **Networking & Connectivity:**
    * Static IP configuration and understanding IP addressing, subnets, and gateways.
    * Router administration (Netgear Nighthawk, AT&T router) including DHCP, WAN/LAN concepts.
    * **Advanced Troubleshooting:** Diagnosing and resolving complex connectivity issues like **Double NAT** and meticulous **Port Forwarding** across multiple router layers.
    * VPN deployment (WireGuard server setup fundamentals, though still troubleshooting external access).
* **Web Server & Database Administration:**
    * Installation and configuration of Apache2 web server (virtual hosts, modules).
    * Installation and secure configuration of MariaDB database server (`mysql_secure_installation`, user/database creation, granting privileges).
    * PHP installation and `php.ini` tuning for web applications.
* **Application Deployment & Management:**
    * Successful deployment and configuration of **Plex Media Server** (including remote access).
    * Successful deployment and configuration of **Nextcloud Hub** (personal cloud).
    * Troubleshooting application-specific issues (Plex media permissions, video encoding, Nextcloud database connectivity).
* **Tools Proficiency:**
    * GitHub (version control, documentation).
    * WinSCP (SFTP file transfer).
    * HandBrake (video encoding).
    * WireGuard application (client VPN).
    * Terminal emulators (Windows Terminal, PuTTY, Termux).

### **Key Professional Learnings:**

* **Systematic Problem Solving:** Developed a methodical approach to diagnosing and resolving complex technical issues, often involving multiple layers (hardware, OS, network, application).
* **Persistence & Resilience:** Learned to persevere through frustrating roadblocks (e.g., persistent 404s, VPN connectivity, permission errors) by re-evaluating, researching, and re-attempting solutions.
* **Attention to Detail:** Recognized the critical importance of meticulous syntax, exact file paths, and precise configuration settings.
* **Self-Directed Learning:** Demonstrated the ability to independently acquire new skills and deploy complex technologies without formal instruction.
* **Technical Documentation:** Understood the value of clear, concise, and structured documentation for personal learning, troubleshooting, and professional showcasing.

---
