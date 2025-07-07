## 4. Deploying Your First Virtual Machine - Ubuntu Server

**Goal:** To create and configure a foundational Linux virtual machine (VM) within Proxmox VE, serving as a dedicated environment for learning Linux administration, hosting applications, and practicing server management skills.

**Key Technologies:**
* Proxmox VE Virtual Machine Creation Wizard
* Ubuntu Server 24.04 LTS (Guest Operating System)
* VirtIO Drivers (for virtual hardware optimization)
* Network Configuration (Static IP within VM)
* Secure Shell (SSH)

**Process & Key Steps:**

1.  **Uploading Ubuntu Server ISO:**
    * Uploaded the `ubuntu-24.04.2-live-server-amd64.iso` file (downloaded in earlier steps) to Proxmox's `local (projectblanco)` storage under the "ISO Images" section. This ISO serves as the virtual installation CD for the new VM.

2.  **Creating the Virtual Machine (`ubuntu-practice`) via Proxmox UI:**
    * Initiated the "Create VM" wizard in the Proxmox web interface.

    * **General Tab:**
        * **Node:** `projectblanco` (default)
        * **VM ID:** Auto-assigned (e.g., `100`)
        * **Name:** `ubuntu-practice` (a descriptive name for the VM)

    * **OS Tab:**
        * **Type:** `Linux` (default)
        * **ISO image:** Selected the uploaded `local (projectblanco): ubuntu-24.04.2-live-server-amd64.iso`.

    * **System Tab (Configuring Virtual Hardware):**
        * **Graphic card:** `Default (std)`
        * **SCSI Controller:** `VirtIO SCSI single` (selected for optimal disk performance with Linux guests)
        * **Qemu Agent:** **Checked** (essential for graceful shutdowns and Proxmox-VM communication)
        * **BIOS:** **`OVMF (UEFI)`**
            * **Why I Changed It:** The default was `SeaBIOS` (legacy). Changed to `OVMF (UEFI)` as it's the modern standard aligning with cloud environments and offers better features.
        * **EFI Storage:** `local (projectblanco)`
            * **What Went Wrong:** After selecting `OVMF (UEFI)`, this field initially turned red, indicating a required input.
            * **How I Fixed It:** Selected `local (projectblanco)` from the dropdown to designate where the UEFI firmware's virtual disk would be stored.

    * **Disks Tab (Configuring Virtual Hard Drive):**
        * **Bus/Device:** `SCSI`
            * **What Went Wrong:** Initially, this was set to `VirtIO Block`, which caused "SSD Emulation" to be grayed out.
            * **How I Fixed It:** Changed `Bus/Device` to `SCSI` to ensure compatibility with `SSD Emulation` and leverage the `VirtIO SCSI controller` selected earlier.
        * **Storage:** `local-lvm (projectblanco)` (preferred for flexibility like snapshots)
        * **Disk Size (GiB):** `32` (ample for a learning server)
        * **Cache:** `No cache` (chosen for data safety in case of unexpected power loss on the host)
        * **Discard:** Checked (enables TRIM for SSD optimization)
        * **IO Thread:** Checked (improves disk performance)
        * **SSD Emulation:** Checked (after changing `Bus/Device` to `SCSI`, this option became available, telling the guest OS it's on an SSD).
        * **Note:** Enabled "Advanced" checkbox at the bottom of the wizard to reveal these advanced disk options.

    * **CPU Tab:**
        * **Sockets:** `1`
        * **Cores:** `2` (a good balance for the N100 CPU, leaving resources for host and other VMs)
        * **Type:** `host` (for optimal performance by exposing host CPU features directly to the VM)

    * **Memory Tab:**
        * **Memory (MiB):** `2048` (2GB - sufficient for Ubuntu Server and basic tasks)
        * **Minimum Memory (MiB):** `2048`
            * **What Went Wrong:** This field turned red if left blank when "Ballooning Device" was enabled.
            * **How I Fixed It:** Set it to the same value as "Memory" (`2048`) to satisfy the requirement.
        * **Ballooning Device:** Checked (for dynamic memory management).

    * **Network Tab:**
        * **Bridge:** `vmbr0` (connects to the physical network)
        * **Firewall:** Unchecked (managed within guest OS or host firewall later)
        * **Model:** `VirtIO (paravirtualized)` (for best network performance)

    * **Confirm Tab:**
        * Reviewed all settings.
        * **Unchecked "Start after created"** (to allow manual console access for installation).
        * Clicked "Finish" to create the VM.

3.  **Ubuntu Server Operating System Installation (within the VM Console):**
    * After VM creation, started the VM manually from the Proxmox UI.
    * Accessed the VM's console (`100 (ubuntu-practice)` -> Console) to interact with the Ubuntu installer.
    * **Key Installation Steps:**
        * Language & Keyboard Layout selection.
        * Installation Type (Standard `Ubuntu Server` - not minimized).
        * **Crucial Network Configuration:**
            * Navigated to `ens18` interface.
            * Changed IPv4 method from `DHCP` to `Manual`.
            * Set **Subnet:** `10.0.0.0/24`
            * Set **Address:** `10.0.0.101` (static IP for this VM, distinct from Proxmox host)
            * Set **Gateway:** `10.0.0.1`
            * Set **Name servers:** `10.0.0.1`
            * Confirmed consistency with home network.
        * Proxy (left blank), Mirror (default).
        * **Storage Configuration:** Selected "Use an entire disk" (the 32GB virtual disk), confirmed partition summary.
        * Profile Setup: Created primary user account (`jesse`), set hostname (`ubuntu-practice`), and a strong password.
        * **SSH Setup:** **Enabled `Install OpenSSH server`** (essential for remote management).
        * Ubuntu Pro (skipped), Featured Server Snaps (skipped).
        * Final installation process.

4.  **Post-Installation VM Setup & Boot:**
    * After installation complete, selected "Reboot Now" in the Ubuntu console.
    * **Critical Step:** Immediately switched back to Proxmox UI (`100 (ubuntu-practice)` -> Hardware), selected the `CD/DVD Drive (ide2)`, and clicked "Remove" to virtually "unplug" the ISO. This ensured the VM booted from its virtual hard disk instead of the installer.
    * Monitored VM console for successful boot to `ubuntu-practice login:` prompt.

**Lessons Learned from VM Deployment:**
* **VM Wizard Details:** Understanding each setting (BIOS, SCSI vs. VirtIO Block, cache, discard, SSD emulation, core allocation) and its impact on VM performance and functionality.
* **Linux Guest OS Installation:** Familiarity with the Ubuntu Server installer flow.
* **Consistent Networking:** Reinforcement of static IP assignment and subnet matching, both for the host and the guest VM.
* **Importance of SSH:** Recognizing SSH as the primary tool for server administration.
* **Guest Agent Integration:** Understanding the role of `qemu-guest-agent` for better hypervisor-VM communication.
