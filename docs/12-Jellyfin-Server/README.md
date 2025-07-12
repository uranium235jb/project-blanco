# Project Blanco – Jellyfin Home Lab Documentation

## Overview

This guide describes the end-to-end deployment of Jellyfin on an Ubuntu VM as part of Project Blanco. You will learn:

- What Jellyfin is and why it’s a compelling home-lab project  
- Hardware and software requirements  
- A step-by-step installation and configuration process  
- Tips for securing, maintaining, and troubleshooting your media server  

Having recently earned my AWS Cloud Practitioner certification and preparing for a BS in Cloud Computing at WGU, this project demonstrates containerized application deployment, network architecture design, and systematic troubleshooting—skills directly applicable to modern cloud environments.

---

## 1. Why Jellyfin?

**Jellyfin** is a free, open-source media server that lets you host and stream your own movies, music, and photos without subscriptions or vendor lock-in. Key benefits:

- **Privacy & Control**  
  You retain full ownership of your media and metadata; no external tracking.  
- **Cost-Effectiveness**  
  Completely free to use—no fees or premium tiers.  
- **Extensibility**  
  Supports plugins, hardware transcoding, and active community contributions.  
- **Learning Opportunity**  
  Involves Docker networking, volume management, reverse-proxy setup, and SSL.

---

## 2. Architecture & Components

- **Jellyfin Container**  
  Runs the media server software.  
- **Ubuntu VM (10.0.0.102)**  
  Host for Docker Engine & Docker Compose.  
- **Host Volumes**  
  - `/config` (server settings)  
  - `/cache` (transcode cache)  
  - `/media` (your media files)  
- **Optional Reverse Proxy**  
  NGINX with Let’s Encrypt for HTTPS and custom domain routing.  
- **Router Configuration**  
  Port forwarding (TCP 8096/8920) or VPN for secure remote access.

---

## 3. Hardware & Software Requirements

| Component           | Minimum              | Recommended                          |
|---------------------|----------------------|--------------------------------------|
| CPU                 | Dual-core (i3 class) | Quad-core with QuickSync/AMF support |
| RAM                 | 2 GB                 | 4 GB+                                |
| Storage             | 50 GB                | 100 GB+ for media library            |
| OS                  | Ubuntu 20.04 LTS     | Ubuntu 22.04 LTS                     |
| Software            | Docker, Compose      | NGINX, Certbot                       |

---

## 4. Installation Steps

### 4.1. Prepare the OS

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release \
  software-properties-common
4.2. Install Docker & Compose
bash
Copy
Edit
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
   https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
4.3. Create Directories & Set Permissions
bash
Copy
Edit
mkdir -p ~/jellyfin/{config,cache,media}
sudo chown -R 1000:1000 ~/jellyfin
Ensure the UID/GID (1000:1000) matches your primary user account.

4.4. Define the Docker Compose File
Create ~/jellyfin/docker-compose.yml:

yaml
Copy
Edit
version: "3.8"

services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    ports:
      - "8096:8096"      # HTTP
      - "8920:8920"      # HTTPS (if configured)
    environment:
      TZ: America/Chicago
    volumes:
      - ./config:/config
      - ./cache:/cache
      - ./media:/media
    restart: unless-stopped
4.5. Launch the Container
bash
Copy
Edit
cd ~/jellyfin
docker compose up -d
docker compose ps
docker compose logs -f jellyfin
Access the UI at http://<HOST_IP>:8096.

5. First-Time Configuration
Browser Setup
Navigate to http://<HOST_IP>:8096.

Administrator Account
Create your first user.

Library Paths
Point to /media/Movies, /media/Music, etc.

Transcoding
(Optional) Enable hardware acceleration under Playback > Transcoding.

Remote Access

Forward TCP ports 8096 & 8920 on your router

Or configure VPN access for enhanced security

6. (Optional) NGINX Reverse Proxy
Edit your NGINX site configuration:

nginx
Copy
Edit
server {
    listen 80;
    server_name jellyfin.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name jellyfin.example.com;

    ssl_certificate     /etc/letsencrypt/live/jellyfin.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/jellyfin.example.com/privkey.pem;

    location / {
        proxy_pass          http://127.0.0.1:8096;
        proxy_set_header    Host              $host;
        proxy_set_header    X-Real-IP         $remote_addr;
        proxy_set_header    X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto $scheme;
    }
}
Obtain SSL certificates with Certbot:

bash
Copy
Edit
sudo certbot --nginx -d jellyfin.example.com
7. Maintenance & Troubleshooting
Update Jellyfin

bash
Copy
Edit
cd ~/jellyfin
docker compose pull
docker compose up -d
Backups

Regularly archive ~/jellyfin/config and ~/jellyfin/cache.

Logs & Diagnostics

docker compose logs jellyfin

Check directory permissions and firewall settings.

8. Skills Developed
Container Orchestration with Docker Compose

Linux Administration: package management, permissions, services

Networking: port mapping, reverse proxy, SSL termination

Troubleshooting: log analysis, permission fixes, firewall rules

9. Next Steps
Implement centralized logging with Loki & Grafana

Automate daily backups of configuration and cache

Explore Kubernetes (K3s) for container orchestration

Integrate OAuth for multi-user authentication

Test Jellyfin mobile clients via secure endpoints

