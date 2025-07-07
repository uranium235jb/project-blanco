## 5. Initial Ubuntu VM Post-Installation & Remote Access

**Goal:** To perform essential post-installation configuration on the new Ubuntu Server VM and establish secure remote access, preparing it for future application deployments.

**Key Technologies:**
* Secure Shell (SSH) Client (from laptop/desktop/phone)
* `sudo` (for elevated privileges in Linux)
* `apt` package manager (for updates)
* `qemu-guest-agent`

**Process & Key Steps:**

1.  **Establishing Remote Access via SSH:**
    * After the Ubuntu Server VM successfully booted from its virtual hard disk, the primary method for administration was via SSH.
    * **From Laptop/Desktop:** Used the command `ssh jesse@10.0.0.101` in a command prompt/terminal.
        * Accepted the host key fingerprint on the first connection (`yes` prompt).
        * Authenticated using the `jesse` user's password.
    * **From Mobile (Android - Termux):** Successfully logged in from my Android phone using the Termux app via the same `ssh jesse@10.0.0.101` command.
    * **Lesson Learned:** SSH is the fundamental tool for secure, remote server management in IT, and it offers a full terminal experience. Mobile SSH clients enable administration on the go.

2.  **Post-Installation System Updates:**
    * Once logged into the Ubuntu VM via SSH, performed a full system update to ensure all software packages were current.
    * Commands executed:
        * `sudo apt update` (to refresh package lists)
        * `sudo apt upgrade -y` (to install all pending updates, `-y` for automatic confirmation)
    * **Lesson Learned:** Regular system updates are critical for security and stability on any Linux server. Understanding `sudo` is key for executing administrative commands as a regular user.

3.  **Installing QEMU Guest Agent:**
    * To enhance communication and management between the Proxmox hypervisor and the Ubuntu VM, the `qemu-guest-agent` was installed.
    * Command executed:
        `sudo apt install qemu-guest-agent -y`
    * **Lesson Learned:** Guest agents improve virtualization platform integration, enabling features like graceful shutdowns, better performance statistics, and IP address display in the hypervisor UI.

4.  **Graceful VM Reboot:**
    * After installing the guest agent, performed a graceful reboot of the Ubuntu VM to ensure all changes took effect.
    * Command executed:
        `sudo reboot`
    * Confirmed successful re-login via SSH after the VM came back online.
    * **Lesson Learned:** Always perform graceful shutdowns/reboots (`sudo reboot`, `sudo shutdown now`) on running VMs and physical servers to prevent data corruption.

**Current Status of `ubuntu-practice` VM:**
The `ubuntu-practice` VM is now fully installed, updated, and accessible via SSH, providing a robust and clean Linux environment ready for the deployment of specific services and further learning projects within Project Blanco.
