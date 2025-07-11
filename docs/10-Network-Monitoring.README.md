Project Blanco - 10 Network Monitoring with Grafana, Prometheus, & Uptime Kuma
Overview
This document details the setup of a robust network and server monitoring solution within the Project Blanco home lab. Leveraging a powerful combination of Grafana, Prometheus, and Uptime Kuma, this project establishes real-time visibility into system performance and service availability. This is a crucial step in building a resilient and observable home lab, providing insights that are invaluable for proactive maintenance and troubleshooting.

As I continue my journey towards a BS in Cloud Computing at WGU, having earned my AWS Cloud Practitioner certification, this project serves as a practical demonstration of my ability to deploy, configure, and manage critical monitoring infrastructure – a foundational skill set directly applicable to cloud environments and highly sought after by employers.

Why Monitor? The Fun and Importance
In a dynamic home lab environment, things can and will go wrong. A server might run out of disk space, a service might crash, or the internet connection might drop. Without proper monitoring, these issues can go unnoticed until they impact usability, leading to frustration and downtime.

This monitoring stack provides:

Proactive Issue Detection: Catch problems before they become critical. Is a hard drive nearly full? Is CPU usage consistently high? Get alerted.

Performance Insight: Understand how your systems are behaving over time. Identify bottlenecks, analyze trends, and plan for future resource needs.

Service Availability Assurance: Know immediately if a critical service (like your Plex server, Nextcloud, or even your home internet) goes offline.

Learning & Skill Development: Working with these industry-standard tools deepens understanding of observability, data visualization, and containerization (with Docker). It's incredibly satisfying to see your infrastructure's "heartbeat" laid out visually.

Resume Building: This hands-on experience with Grafana, Prometheus, and Docker is highly valuable. It demonstrates practical skills in system administration, DevOps, and cloud-native technologies.

The Monitoring Stack Components
This project integrates three key open-source tools, each playing a distinct yet complementary role:

Node Exporter: A lightweight agent that runs on your Linux server (Ubuntu VM in this case). Its sole purpose is to collect a wide array of hardware and OS metrics (CPU, memory, disk I/O, network stats, etc.) and expose them in a format that Prometheus can easily scrape.

Prometheus: A powerful open-source monitoring system and time-series database. Prometheus pulls (or "scrapes") metrics from configured targets (like Node Exporter). It stores this data efficiently and provides a flexible query language (PromQL) for analysis.

Grafana: An open-source platform for monitoring and observability. Grafana is where all the collected data comes to life. It connects to various data sources (like Prometheus) and allows you to create highly customizable, interactive dashboards to visualize your metrics in beautiful and insightful ways.

Uptime Kuma: A "fancy" self-hosted monitoring tool that focuses specifically on service availability. It performs simple checks (ping, HTTP/S, TCP port) to determine if a service is up or down, sends instant notifications on status changes, and can generate public (or private) status pages.

Together, these tools provide a comprehensive view: Prometheus and Grafana show you how your server is performing, while Uptime Kuma tells you if your services are available.

Deployment Steps
This section outlines the step-by-step process used to deploy and configure the monitoring stack on the Ubuntu VM (10.0.0.101).

1. Node Exporter Installation
Node Exporter was installed directly on the Ubuntu VM to collect system-level metrics.

# Download Node Exporter (check GitHub releases for latest version if needed)
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz

# Extract the archive
tar xvfz node_exporter-*.tar.gz

# Move the executable to a common binary path
sudo mv node_exporter-1.9.1.linux-amd64/node_exporter /usr/local/bin/

# Create a dedicated system user for Node Exporter
sudo useradd --no-create-home --shell /bin/false node_exporter

# Create a systemd service file for Node Exporter to manage it
sudo nano /etc/systemd/system/node_exporter.service
# Paste the following content:
# [Unit]
# Description=Node Exporter
# Wants=network-online.target
# After=network-online.target
#
# [Service]
# User=node_exporter
# Group=node_exporter
# Type=simple
# ExecStart=/usr/local/bin/node_exporter
#
# [Install]
# WantedBy=multi-user.target
# Save and exit (Ctrl+X, Y, Enter)

# Reload systemd, enable, and start Node Exporter
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

# Verify status
sudo systemctl status node_exporter
# Expected: Active: active (running)


2. Prometheus Installation & Configuration
Prometheus was installed directly on the Ubuntu VM to scrape metrics from Node Exporter and store them.

# Create a system user for Prometheus
sudo useradd --no-create-home --shell /bin/false prometheus

# Create directories for Prometheus configuration and data
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

# Download Prometheus (check GitHub releases for latest stable version)
cd ~
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz

# Extract and move Prometheus binaries
tar xvfz prometheus-*.tar.gz
sudo mv prometheus-2.52.0.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-2.52.0.linux-amd64/promtool /usr/local/bin/

# Move Prometheus console templates and libraries
sudo mv prometheus-2.52.0.linux-amd64/consoles /etc/prometheus
sudo mv prometheus-2.52.0.linux-amd64/console_libraries /etc/prometheus

# Set ownership for Prometheus directories and binaries
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool

# Create the Prometheus configuration file
sudo nano /etc/prometheus/prometheus.yml
# Paste the following content:
# global:
#   scrape_interval: 15s
#   evaluation_interval: 15s
#
# scrape_configs:
#   - job_name: 'prometheus'
#     static_configs:
#       - targets: ['localhost:9090'] # Prometheus itself
#
#   - job_name: 'node_exporter'
#     static_configs:
#       - targets: ['localhost:9100'] # Node Exporter
# Save and exit (Ctrl+X, Y, Enter)

# Create a systemd service file for Prometheus
sudo nano /etc/systemd/system/prometheus.service
# Paste the following content:
# [Unit]
# Description=Prometheus
# Wants=network-online.target
# After=network-online.target
#
# [Service]
# User=prometheus
# Group=prometheus
# Type=simple
# ExecStart=/usr/local/bin/prometheus \
#     --config.file /etc/prometheus/prometheus.yml \
#     --storage.tsdb.path /var/lib/prometheus/data \
#     --web.console.templates=/etc/prometheus/consoles \
#     --web.console.libraries=/etc/prometheus/console_libraries
#
# [Install]
# WantedBy=multi-user.target
# Save and exit (Ctrl+X, Y, Enter)

# Reload systemd, enable, and start Prometheus
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus

# Verify status
sudo systemctl status prometheus
# Expected: Active: active (running)

# Access Prometheus UI: http://10.0.0.101:9090 and verify targets are UP.


3. Grafana Installation & Data Source Setup
Grafana was installed directly on the Ubuntu VM for data visualization.

# Install prerequisite packages
sudo apt install -y ca-certificates curl gnupg

# Add Grafana's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
wget -q -O - https://apt.grafana.com/gpg.key | sudo gpg --dearmor | sudo tee /usr/share/keyrings/grafana.gpg > /dev/null

# Add the Grafana repository to Apt sources
echo "deb [signed-by=/usr/share/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

# Update package list
sudo apt update

# Install Grafana
sudo apt install grafana -y

# Start and enable Grafana
sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

# Verify status
sudo systemctl status grafana-server
# Expected: Active: active (running)

# Access Grafana UI: http://10.0.0.101:3000
# Default login: admin / admin (you will be prompted to change it)

# Add Prometheus as a Data Source in Grafana UI:
# 1. Navigate to Configuration (gear icon) -> Data sources.
# 2. Click "Add data source" and select "Prometheus".
# 3. Set URL to: http://localhost:9090
# 4. Click "Save & test".

# Import Node Exporter Dashboard (ID: 1860) in Grafana UI:
# 1. Navigate to Dashboards icon -> Import.
# 2. Enter "1860" in the "Import via grafana.com dashboard" field.
# 3. Click "Load".
# 4. Select your Prometheus data source from the dropdown.
# 5. Click "Import".


4. Docker & Docker Compose Installation (Prerequisite for Uptime Kuma)
Docker and Docker Compose were installed to containerize Uptime Kuma, ensuring clean deployment and easy management.

# Install prerequisite packages
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the Docker repository to Apt sources
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package index
sudo apt update

# Install Docker Engine and Docker Compose plugin
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add your user to the 'docker' group to run Docker commands without sudo
sudo usermod -aG docker ${USER}

# IMPORTANT: Log out of your SSH session and log back in for group changes to take effect.
# If still facing permission issues after re-login, a full server reboot might be necessary.
# As a last resort workaround (less secure for multi-user systems):
# sudo chmod 666 /var/run/docker.sock

# Verify Docker installation (should work without sudo after re-login)
docker --version
docker run hello-world


5. Uptime Kuma Deployment with Docker Compose
Uptime Kuma was deployed as a Docker container for service availability monitoring.

# Create a directory for Uptime Kuma project files
mkdir uptime-kuma
cd uptime-kuma

# Create the docker-compose.yml file
nano docker-compose.yml
# Paste the following content:
# version: '3.8'
# services:
#   uptime-kuma:
#     image: louislam/uptime-kuma:1 # The recommended stable version
#     container_name: uptime-kuma
#     volumes:
#       - ./data:/app/data # Persistent storage for Uptime Kuma data
#     ports:
#       - "3001:3001" # Host Port:Container Port
#     restart: always # Ensures automatic restart on crash or reboot
# Save and exit (Ctrl+X, Y, Enter)

# Start Uptime Kuma container
docker compose up -d

# Verify Uptime Kuma container is running
docker ps
# Expected: uptime-kuma listed with status 'Up ...'

# Access Uptime Kuma UI: http://10.0.0.101:3001
# Follow the on-screen prompts to create your admin account.

# Add your first monitor in Uptime Kuma UI (e.g., Ping your router or Google.com)


What You Can Do With These Applications
With this integrated monitoring stack, your home lab gains significant capabilities:

Real-time Server Health: The Grafana dashboard (powered by Prometheus and Node Exporter) provides immediate insights into your Ubuntu VM's CPU, RAM, disk, and network usage. This allows you to identify resource bottlenecks, unusual activity, or potential failures at a glance.

Service Uptime & Latency: Uptime Kuma actively pings or checks your specified services (e.g., Plex, Nextcloud, router, external websites). You'll see their current status (up/down), response times, and historical uptime percentages.

Instant Notifications: Configure Uptime Kuma to send alerts via various channels (email, Discord, Telegram, etc.) the moment a service goes down or recovers. This eliminates the need to constantly check dashboards.

Custom Status Pages: Uptime Kuma's ability to generate attractive status pages is invaluable. You can create a public or private page (e.g., accessible only within your home network) that shows the current operational status of all your critical home lab services. This is fantastic for communicating with family members or anyone else who uses your services.

Historical Data Analysis: Both Grafana and Uptime Kuma store historical data, allowing you to review past performance, identify recurring issues, and understand long-term trends.

Foundation for Advanced Monitoring: This setup is highly extensible. You can add more Prometheus exporters for other services (e.g., database exporters, application-specific exporters), integrate more data sources into Grafana, and expand Uptime Kuma's checks as your home lab grows.

Conclusion
Successfully deploying this network and server monitoring solution with Grafana, Prometheus, and Uptime Kuma has been a rewarding experience. It not only provides critical visibility into the health and availability of my Project Blanco home lab but also significantly enhances my practical skills in observability, containerization, and system administration.

This project directly complements my AWS Cloud Practitioner certification and provides a strong foundation for my upcoming studies in the BS in Cloud Computing at WGU. The ability to set up, manage, and interpret data from such a stack is a core competency for any cloud professional, demonstrating a proactive approach to infrastructure management and problem-solving. This documentation, along with the others in my project-blanco repository, serves as a tangible portfolio of my hands-on learning and dedication to mastering cloud and infrastructure technologies.

Next Steps for Project Blanco:

Configure Uptime Kuma notifications (e.g., Discord webhook).

Create a public/private Uptime Kuma status page.

Add more monitors to Uptime Kuma for critical home lab services.

Explore advanced Grafana features like custom dashboards and alerting rules.
