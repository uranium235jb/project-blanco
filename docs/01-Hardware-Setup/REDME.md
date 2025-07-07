## 1. Initial Planning & Hardware Selection

**Goal:** My journey into building a home lab began with a clear objective: to bridge the gap between theoretical knowledge gained from my Cloud Computing studies and practical, hands-on experience. I aimed to create a robust, yet budget-friendly, environment for learning virtualization, Linux administration, networking, and eventually, self-hosting services like a media server and a VPN.

**Hardware Evaluation Process:**
Before committing to specific hardware, I explored several options, weighing their pros, cons, and suitability for a dedicated home lab server:

* **Existing Desktop/Laptop:**
    * **Pros:** Zero immediate cost, readily available.
    * **Cons:** Often noisy, higher power consumption, not ideal for 24/7 operation, ties up a daily-use machine.
* **Raspberry Pi (e.g., Raspberry Pi 5):**
    * **Pros:** Very low power, extremely quiet, excellent for learning basic Linux and specific lightweight services.
    * **Cons:** ARM-based architecture (not x86_64), which means it cannot run Proxmox VE (a bare-metal hypervisor) or traditional x86_64 virtual machines. Limited RAM and I/O for multiple demanding VMs.
    * **Lesson Learned:** Understanding CPU architecture compatibility is critical for hypervisor selection.
* **Small Form Factor (SFF) / Mini PCs (e.g., Firebat Mini PC - Intel N150):**
    * **Pros (Why Chosen):**
        * **Budget-Friendly:** Excellent value for performance (acquired for ~$128).
        * **x86_64 Architecture:** Fully compatible with Proxmox VE and standard Linux/Windows VMs.
        * **Hardware Virtualization (VT-x):** Essential for running Proxmox efficiently.
        * **Sufficient RAM (16GB):** Provides ample memory for running multiple virtual machines concurrently.
        * **Fast Storage (512GB M.2 SSD):** Crucial for snappy VM performance.
        * **Low Power Consumption:** Efficient for 24/7 operation in a home environment.
        * **Compact & Quiet:** Ideal for discreet placement in a home lab.
    * **Cons:** Limited internal expansion compared to larger desktops.

**Key Decision & Rationale:**
The **Firebat Mini PC (Intel N100, 16GB RAM, 512GB SSD)** was selected as the core hardware for Project Blanco. It struck the perfect balance between cost, performance, power efficiency, and compatibility requirements for a robust entry-level home lab hypervisor. It serves as the dedicated "bare metal" foundation for all virtualized services.

**Lessons Learned from Planning:**
* **Match Hardware to Purpose:** A server requires specific features (VT-x, ample RAM/fast storage) that consumer devices or older consoles do not offer.
* **Importance of Research:** Thoroughly evaluating options prevents compatibility headaches and ensures resources are wisely invested.
* **Understanding Core Components:** Gained a deeper appreciation for CPU architecture, RAM demands of virtualization, and the impact of storage type (SSD vs. HDD) on performance.
