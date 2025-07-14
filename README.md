🔍 DevOps Monitoring Project with Prometheus and Grafana

📌 Project Overview

This project sets up a comprehensive monitoring system on an AWS EC2 instance using:

Prometheus for collecting metrics

Node Exporter for system metrics

Grafana for data visualization

The setup helps monitor CPU, memory, disk, network, and system health in real time.

💡 Technologies Used

AWS EC2 (Ubuntu 24.04 LTS)

Prometheus v2.52.0

Node Exporter v1.8.1

Grafana OSS

Systemd for service management

🚀 Setup Instructions

✅ 1. Launch EC2 Instance

Use official Ubuntu Server 24.04 LTS AMI

Open inbound ports in Security Group: 22, 9090, 9100, 3000

SSH into instance:

ssh -i path/to/your-key.pem ubuntu@<EC2_PUBLIC_IP>

⚙️ 2. Install and Configure Node Exporter

# Download and install Node Exporter
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
tar xvf node_exporter-1.8.1.linux-amd64.tar.gz
sudo mv node_exporter-1.8.1.linux-amd64/node_exporter /usr/local/bin/

# Create user and systemd service
sudo useradd -rs /bin/false node_exporter
sudo nano /etc/systemd/system/node_exporter.service

Paste:

[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target

# Start Node Exporter
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

Test: curl http://localhost:9100/metrics

📈 3. Install and Configure Prometheus

curl -LO https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar xvf prometheus-2.52.0.linux-amd64.tar.gz
cd prometheus-2.52.0.linux-amd64
sudo mv prometheus promtool /usr/local/bin/
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo cp -r consoles console_libraries /etc/prometheus/
sudo cp prometheus.yml /etc/prometheus/
sudo useradd -rs /bin/false prometheus
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus

Update config:

sudo nano /etc/prometheus/prometheus.yml

Paste:

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

Create service:

sudo nano /etc/systemd/system/prometheus.service

Paste:

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

Start Prometheus:

sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus

Visit: http://<EC2_PUBLIC_IP>:9090

📊 4. Install and Configure Grafana

sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install grafana -y

sudo systemctl enable grafana-server
sudo systemctl start grafana-server

Visit: http://<EC2_PUBLIC_IP>:3000

Login: admin / admin

Set a new password

Add Prometheus Data Source:

Go to ⚙️ → Data Sources → Add Prometheus

Set URL: http://localhost:9090

Click Save & Test

Import Dashboard:

Go to 📊 Dashboards → Import

Use ID: 1860

Click Load → Choose Prometheus → Click Import

✅ Final URLs

Prometheus: http://<EC2_PUBLIC_IP>:9090

Node Exporter: http://<EC2_PUBLIC_IP>:9100/metrics

Grafana: http://<EC2_PUBLIC_IP>:3000

<img width="1440" height="900" alt="Screenshot 2025-07-14 at 9 07 23 PM" src="https://github.com/user-attachments/assets/5f0f8a0c-2938-4290-acd9-0e9a84811a16" />
