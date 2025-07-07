## 2. Setting up the Hypervisor - Proxmox VE Installation

**Goal:** Transform the chosen Firebat Mini PC into a dedicated bare-metal virtualization host by installing Proxmox Virtual Environment (PVE). This step is foundational for hosting all subsequent virtual machines.

**Key Technologies:**
* Proxmox VE 8.x ISO Installer
* Rufus (USB boot media creation tool)
* Physical Networking (Ethernet cabling, router interaction)
* BIOS/UEFI Firmware Configuration

**Process & Key Steps:**

1.  **Preparing the Installation Media:**
    * Downloaded the latest stable Proxmox VE ISO from the official Proxmox website.
    * Used Rufus to create a bootable USB flash drive from the ISO.
    * **What Went Wrong:** My initial attempt resulted in a "No device with valid ISO found" error during boot.
    * **How I Fixed It:** Troubleshooting involved re-downloading the ISO (to rule out corruption) and meticulously re-creating the bootable USB drive using Rufus, ensuring correct settings and proper writing process. This resolved the media read error.

2.  **Physical Setup & Connectivity:**
    * Connected monitor, USB keyboard, and USB mouse directly to the Firebat Mini PC.
    * **Critical Network Cabling:** Ensured proper network connectivity, which was a point of clarification. My Nighthawk router needed one Ethernet cable to connect to the modem (for internet access) and a **second Ethernet cable** to connect directly to the Firebat Mini PC. This ensures the Proxmox host has continuous network access and can be managed remotely.

3.  **BIOS/UEFI Firmware Configuration:**
    * Powered on the Firebat Mini PC and repeatedly pressed the appropriate key (e.g., `Del`, `F2`, `F7`, `F12`) to access the BIOS/UEFI setup utility or a one-time boot menu.
    * **Enabled Intel Virtualization Technology (VT-x):** Confirmed this crucial CPU feature was set to 'Enabled' in the BIOS. This allows the CPU to efficiently handle virtualization.
    * Set the boot order (or used the one-time boot menu) to prioritize booting from the Proxmox VE USB flash drive.

4.  **Proxmox VE Installation Wizard:**
    * Initiated the graphical Proxmox VE installer from the USB drive.
    * Accepted the End-User License Agreement.
    * Selected the Firebat's internal 512GB M.2 SSD (`/dev/sda`) as the installation target, understanding that this would erase all existing data.
    * Configured localization settings (country, time zone, keyboard layout).

**What Went Wrong (The Network Configuration Saga) & How I Fixed It:**
This was the most significant troubleshooting challenge during the Proxmox installation phase. The core issue was a mismatch between the Proxmox host's intended static IP subnet and the actual subnet of my home network (managed by the Nighthawk router).

* **Initial Problem:** After the first installation, the Proxmox console displayed `https://192.168.1.100:8006/` for web access, but attempts to reach it from my laptop failed with `ERR_CONNECTION_TIMED_OUT`.
* **Diagnosis - Part 1 (Client-Side):**
    * Verified the Firebat Mini PC was indeed displaying the `192.168.1.100` IP on its console.
    * Used `ipconfig` on my client laptop (Windows) to determine its actual network configuration. It revealed my laptop's IP was `10.0.0.19` and its `Default Gateway` (the Nighthawk router) was `10.0.0.1`.
    * This confirmed a **subnet mismatch**: my client was on `10.0.0.x`, but Proxmox was configured for `192.168.1.x`. The network "streets" didn't match.
* **Solution - Part 1 (Re-installation Attempt):**
    * Decision was made to re-install Proxmox VE, being extremely careful to input the correct network settings.
    * **Intended Static IP:** `10.0.0.100`
    * **Gateway:** `10.0.0.1`
    * **DNS Server:** `10.0.0.1`
    * **Hostname:** `projectblanco`
* **Problem 3: IP Still Incorrect After Re-installation:** Despite the careful input during the second installation attempt, the Proxmox console *still* showed `192.168.1.100` after reboot.
* **Diagnosis - Part 2 (Server-Side Deep Dive):**
    * Logged directly into the Proxmox console (using `root` and the set password).
    * Used the `ip a` command to check the *actual* active IP address. This confirmed `vmbr0` was indeed still `192.168.1.100`.
    * Used the `cat /etc/network/interfaces` command to inspect the network configuration file. This revealed the file still contained the old `192.168.1.x` settings, and was missing the `dns-nameservers` entry.
* **Solution - Part 2 (Manual Configuration File Edit):**
    * Utilized the `nano` text editor directly within the Proxmox console.
    * Opened `/etc/network/interfaces`.
    * Manually edited the `address`, `gateway`, and added the `dns-nameservers` lines to reflect the correct `10.0.0.x` subnet settings:
        ```
        address 10.0.0.100/24
        gateway 10.0.0.1
        dns-nameservers 10.0.0.1
        ```
    * Saved changes (`Ctrl+X`, `Y`, Enter) and rebooted the server (`reboot`).
* **Result:** Upon reboot, the Proxmox console successfully displayed `https://10.0.0.100:8006/`. The host was finally reachable and properly configured on the home network.

**Lessons Learned from Proxmox Installation:**
* **Network Fundamentals are Paramount:** A deep understanding of IP addresses, subnets, and gateways is crucial for setting up servers.
* **Trust, but Verify:** Always confirm active network configurations (`ip a`) and file contents (`cat`) after making changes, especially if issues persist.
* **Linux Command Line is Essential:** Tools like `nano`, `ip a`, `ping`, `cat`, `apt`, and `reboot` are fundamental for server administration and troubleshooting.
* **Patience and Persistence:** Troubleshooting requires a systematic approach and the willingness to retrace steps and try different solutions.
* **Documentation is Key:** This entire troubleshooting process highlighted the importance of documenting steps and observations.
