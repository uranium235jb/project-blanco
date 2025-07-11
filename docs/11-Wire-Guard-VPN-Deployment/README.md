Project Blanco - 11 WireGuard VPN Deployment & Advanced Troubleshooting

Overview

This document details the challenging yet ultimately successful deployment of a WireGuard VPN server on my Ubuntu VM, enabling secure remote access to the Project Blanco home lab. This project was a deep dive into advanced networking, requiring meticulous configuration across multiple devices and persistent troubleshooting to overcome significant obstacles.

As I continue my journey towards a BS in Cloud Computing at WGU, having earned my AWS Cloud Practitioner certification, this project serves as a critical demonstration of my ability to diagnose and resolve complex network connectivity issues, a foundational skill set directly applicable to cloud environments and highly sought after by employers.

Why a VPN? The Fun, Importance, and Challenges

Remote access to a home lab is essential for flexibility, allowing me to manage and utilize my services (like Plex, Nextcloud, and the monitoring stack) from anywhere. A VPN provides a secure, encrypted tunnel for this access.

The importance of this project extended beyond basic remote access:

🔒 Secure Remote Access: Encrypts all traffic between my external devices and the home lab, protecting data when using public Wi-Fi.

<span style="font-size: 1.1em;">💥 BYPASSING ISP RESTRICTIONS: THIS WAS THE MOST SIGNIFICANT HURDLE AND A CRITICAL LEARNING EXPERIENCE, INVOLVING OVERCOMING POTENTIAL CARRIER-GRADE NAT (CGNAT) AND ISP-LEVEL PORT BLOCKING, A COMMON AND FRUSTRATING CHALLENGE FOR HOME LAB ENTHUSIASTS.</span>

🏠 Accessing Internal Resources: Allows seamless access to services hosted on private IP addresses within my home network.

🕵️ Privacy & Security: All internet traffic from the client can be routed through the home lab, masking the client's public IP address.

🧠 Deep Learning Opportunity: This project provided an invaluable, real-world scenario to apply and deepen my understanding of networking fundamentals, routing, and advanced troubleshooting.

Key Components Utilized

This project involved careful configuration across several layers of my home network:

✅ WireGuard: A modern, fast, and secure VPN protocol used for both the server (on Ubuntu VM) and client (laptop/mobile).

✅ Ubuntu VM (10.0.0.101): The dedicated virtual machine hosting the WireGuard VPN server.

✅ Netgear Nighthawk R8700: My personal router, now directly connected to the internet, responsible for port forwarding.

✅ AT&T BGW210 Gateway: The ISP-provided modem, successfully configured for IP Passthrough and later completely bypassed.

✅ Public IP Address (23.127.217.2): The external endpoint for the VPN tunnel, provided by the ISP.

The Deployment & Troubleshooting Journey

The path to a fully functional VPN was a comprehensive learning experience, characterized by systematic troubleshooting and persistence.

Initial Setup & The Problem

After an initial attempt to set up WireGuard, the primary issue encountered was a complete lack of incoming traffic to the VPN server. Despite the client showing an "active" tunnel, there was no internet access, and crucially, journalctl -kf on the server showed ABSOLUTELY NO INCOMING CONNECTION ATTEMPTS.

Initial diagnostics confirmed:

➡️ canyouseeme.org reported "Connection timed out" for the WireGuard UDP port (51820).

➡️ sudo tcpdump -i any udp port 51820 on the server showed 0 packets captured during external connection attempts.

This strongly suggested an upstream block, either at the AT&T gateway or the ISP level.

Troubleshooting Phase 1: Linux Server Internal Checks

Before contacting the ISP, a thorough check of the Ubuntu VM's WireGuard server configuration was performed:

✅ WireGuard Listening Port: Confirmed WireGuard was actively listening on UDP port 51820 (sudo wg show wg0 | grep "listening port").

✅ UFW Firewall: Verified the Uncomplicated Firewall (UFW) was Status: inactive, ruling out local server-side blocking (sudo ufw status verbose).

✅ IP Forwarding: Confirmed net.ipv4.ip_forward=1 was permanently enabled in /etc/sysctl.conf.

✅ iptables Rules: Checked the FORWARD and POSTROUTING (NAT) chains. The necessary ACCEPT rules for wg0 and the MASQUERADE rule for ens18 were present, indicating the server was configured to forward and translate traffic.

At this point, the server configuration appeared correct, reinforcing the suspicion of an external block.

Troubleshooting Phase 2: AT&T BGW210 Gateway Deep Dive

Given the persistent external block, a meticulous review of the AT&T BGW210 gateway's settings was undertaken:

✅ IP Passthrough: Confirmed "Allocation Mode" was "Passthrough" and "Passthrough Mode" was "DHCP-fixed," with the Netgear Nighthawk selected as the "Passthrough Fixed Allocation" device. "Default Server Internal Address" was correctly empty/grayed out.

✅ Firewall Advanced: Verified "Drop incoming ICMP Echo requests" and ALGs (ESP, SIP) were all "Off," indicating minimal interference.

✅ Packet Filter: Confirmed all custom "Drop packets that match" rules were disabled.

✅ CRITICAL FINDING (BGW210 NAT/Gaming): Discovered a conflicting WireGuard port forwarding rule on the BGW210's "NAT/Gaming" section. This rule was attempting to forward UDP 51820 to an internal IP on the BGW210's own 192.168.1.x subnet, effectively intercepting traffic before it could reach the Nighthawk. This rule was promptly deleted.

✅ Reboot: Both the AT&T BGW210 and Netgear Nighthawk were rebooted after this change.

➡️ Result: Re-testing with canyouseeme.org and tcpdump still showed no incoming packets. This led to the conclusion that an ISP-level block (CGNAT or blanket port blocking) was the most likely remaining cause.

AT&T Call & Modem Bypass

After exhausting all self-troubleshooting on the AT&T gateway, a call to AT&T was made. The AT&T representative was able to completely disable the AT&T modem, allowing the Netgear Nighthawk to receive the internet connection directly. This was a significant breakthrough, bypassing any potential hidden ISP-level restrictions.

The "Fresh Start" Decision

Despite the AT&T modem bypass, initial VPN connection attempts still showed "no internet access" on the client. To eliminate any lingering configuration issues or subtle conflicts from previous attempts, a decision was made to completely remove and re-deploy WireGuard on the Ubuntu VM from scratch.

✅ Deletion Steps: WireGuard package was purged, /etc/wireguard directory removed, and all iptables chains were flushed.

✅ Unexpected Challenge: During re-generation of keys, unusual "Permission denied" errors occurred even for the root user, indicating a deeper system anomaly. This required a full reboot of the Ubuntu VM to clear the transient filesystem/permission issue.

Fresh WireGuard Deployment (Meticulous Steps)

With a clean slate on the Ubuntu VM, WireGuard was re-installed meticulously:

✅ WireGuard Installation: wireguard and qrencode packages were installed.

✅ Key Generation: Fresh server and client private/public key pairs were successfully generated and saved to /etc/wireguard/ with correct permissions.

✅ Server Configuration (wg0.conf): The wg0.conf file was created with the following critical elements:

➡️ Server Private Key.

➡️ Server's VPN IP (10.8.0.1/24).

➡️ Listening Port (51820).

➡️ PostUp and PostDown iptables rules to enable IP forwarding and NAT (masquerading) for internet access via ens18.

➡️ Client's Public Key.

➡️ Client's VPN IP (10.8.0.2/32).

✅ Service Management: wg0.conf permissions were set, and the wg-quick@wg0 service was enabled and started, with status and iptables rules verified.

Troubleshooting Phase 3: Netgear Nighthawk Router Final Deep Dive

With the server confirmed perfect, the focus returned to the Nighthawk for any remaining interference:

✅ Port Forwarding: Re-confirmed the port forwarding rule for UDP 51820 to 10.0.0.101.

✅ Access Control: Verified the Ubuntu VM was "Allowed."

✅ VPN Service: Confirmed the Nighthawk's built-in VPN service was disabled to prevent conflicts.

✅ WAN Setup / NAT Filtering / SPI Firewall / MTU: Checked these advanced settings for any potential interference.

✅ THE ULTIMATE FIX (Nighthawk Port Forwarding Typo): Discovered a critical typo in the Nighthawk's port forwarding rule. The external and internal port ranges were incorrectly set to 15820-15820 instead of the correct 51820-51820. This was immediately corrected.

✅ Reboot: The Netgear Nighthawk was rebooted after the correction.

Final Success!

Following the correction of the port forwarding typo on the Netgear Nighthawk, the WireGuard VPN client was able to connect successfully, and full internet access was restored! Internal home lab resources (Plex, Nextcloud) were also confirmed accessible via the VPN tunnel.

Key Configuration Snippets

<div style="background-color: #2d2d2d; padding: 15px; border-radius: 8px; margin-bottom: 20px; color: #f8f8f2;">
<h3 style="color: #f8f8f2; margin-top: 0; padding-top: 0;">WireGuard Server (/etc/wireguard/wg0.conf)</h3>
<pre style="background-color: #3c3c3c; padding: 10px; border-radius: 5px; overflow-x: auto; color: #f8f8f2;"><code class="language-conf">[Interface]
Address = 10.8.0.1/24
ListenPort = 51820
PrivateKey = [YOUR_SERVER_PRIVATE_KEY] # Replace with actual key
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o ens18 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o ens18 -j MASQUERADE

Client Peer Configuration

[Peer]
PublicKey = [YOUR_CLIENT_PUBLIC_KEY] # Replace with actual key
AllowedIPs = 10.8.0.2/32
</code></pre>

</div>

<div style="background-color: #2d2d2d; padding: 15px; border-radius: 8px; margin-bottom: 20px; color: #f8f8f2;">
<h3 style="color: #f8f8f2; margin-top: 0; padding-top: 0;">WireGuard Client (client.conf)</h3>
<pre style="background-color: #3c3c3c; padding: 10px; border-radius: 5px; overflow-x: auto; color: #f8f8f2;"><code class="language-conf">[Interface]
PrivateKey = [YOUR_CLIENT_PRIVATE_KEY] # Replace with actual key
Address = 10.8.0.2/24
DNS = 8.8.8.8
MTU = 1420

[Peer]
PublicKey = [YOUR_SERVER_PUBLIC_KEY] # Replace with actual key
Endpoint = 23.127.217.2:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
</code></pre>

</div>

Skills Learned

This project provided an unparalleled opportunity to develop and strengthen a wide array of technical and professional skills, particularly relevant for cloud computing and IT infrastructure roles:

🌐 Advanced Networking: Gained deep practical understanding of Network Address Translation (NAT), IP forwarding, iptables firewall rules, port forwarding, and Maximum Transmission Unit (MTU) concepts.

🛡️ VPN Protocols (WireGuard): Proficiently deployed, configured, and troubleshot a modern VPN server and client.

🔍 Systematic Troubleshooting: Executed a methodical, multi-layered diagnostic process across ISP equipment, personal router, Linux server, and client devices. This included utilizing tools like canyouseeme.org, tcpdump, wg show, and iptables -L.

💪 Problem Solving & Persistence: Successfully navigated and resolved highly complex and frustrating connectivity issues, demonstrating resilience and a commitment to finding solutions.

🐧 Linux System Administration: Enhanced skills in command-line operations, service management (systemctl), file permissions (chmod), user management (sudo -i), and configuration file editing (nano).

⚙️ Router Configuration: Detailed experience with advanced settings on consumer-grade routers (Netgear Nighthawk) and ISP-provided gateways (AT&T BGW210).

🎯 Attention to Detail: Identified subtle but critical configuration errors (e.g., port number typos, conflicting rules) that were the root cause of complex problems.

📝 Documentation & Communication: Developed clear, structured, and visually enhanced technical documentation for complex deployments, demonstrating the ability to articulate technical processes and troubleshooting steps effectively.

These skills are foundational for understanding and operating robust network infrastructures in any environment, especially in the context of cloud-native systems.

Conclusion

Successfully deploying a fully functional WireGuard VPN to my home lab has been an incredibly rewarding experience. This project, while challenging, significantly deepened my practical understanding of networking, security, and systematic troubleshooting. It reinforces the importance of meticulous configuration and persistent problem-solving in complex IT environments.

This achievement directly complements my AWS Cloud Practitioner certification and provides a strong foundation for my upcoming studies in the BS in Cloud Computing at WGU. The hands-on experience gained from diagnosing and resolving real-world network issues is invaluable for building a robust portfolio of practical skills.

Next Steps for Project Blanco:

📊 Consider implementing a centralized logging solution (e.g., Loki with Grafana) to complement the existing monitoring stack.

☸️ Explore setting up a lightweight Kubernetes cluster (e.g., K3s) for container orchestration.

🔒 Investigate network segmentation or VLANs for enhanced home lab security.
