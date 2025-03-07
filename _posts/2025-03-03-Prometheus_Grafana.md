---
layout: post
title: Prometheus/Grafana - Quick Setup & Test
date: 03-03-2025
categories: [lab]
tag: [linux]
---

## Install Prometheus

Create a prometheus dedicated user

```sh
sudo useradd --no-create-home --shell /bin/false prometheus
```

Create required directories

```sh
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

Download latest version of Prometheus

```sh
cd /tmp
curl -LO https://github.com/prometheus/prometheus/releases/download/v3.2.0/prometheus-3.2.0.linux-amd64.tar.gz
tar xvf prometheus-3.2.0.linux-amd64.tar.gz
cd prometheus-3.2.0.linux-amd64
```

Move binaries & Change ownerships

```sh
sudo mv prometheus /usr/local/bin/
sudo mv promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

Move config files & Set ownership

```sh
sudo mv prometheus.yml /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

Create a systemD service

`sudo vim /etc/systemd/system/prometheus.service`

```
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
--storage.tsdb.path=/var/lib/prometheus \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries \
--web.listen-address=0.0.0.0:9090

[Install]
WantedBy=multi-user.target
```

Reload systemd manager configuration, then enable/start the new service

```sh
sudo systemctl daemon-reload
sudo systemctl enable prometheus --now
sudo systemctl status prometheus
```

Prometheus is now accesible at: `http://<ip>:9090`

## Install Grafana

[Source: Install Grafana on Debian or Ubuntu](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/)

Install pre-requisites

`sudo apt-get install -y apt-transport-https software-properties-common wget`

Import GPG key

```sh
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```

Add stable release repo

```sh
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Update list of available packages and install Grafana

```sh
sudo apt-get update
sudo apt-get install grafana
```

Enable/start the service, then check status

```sh
sudo systemctl enable --now grafana-server
sudo systemctl status grafana-server
```

Grafana is now accesible at: `http://<ip>:3000`

## Install Node Exporter

Prometheus collects metrics from exporters.
We can get the system metrics with Node Exporter

Download/Install it

```sh
cd /tmp/
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvfz node_exporter-1.8.2.linux-amd64.tar.gz
cd node_exporter-1.8.2.linux-amd64/
sudo mv node_exporter /usr/local/bin/
```

Create a systemD service

`sudo vim /etc/systemd/system/node_exporter.service`

```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```

Reload systemd manager configuration, enable/start the new service and check its status

```sh
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
sudo systemctl status node_exporter
```

## Configure Prometheus to collect metrics

Edit Prometheus configuration

`sudo vim /etc/prometheus/prometheus.yml`

```
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

Restart it

`sudo systemctl restart prometheus`

For each system from which you want to collect metrics, install Node Exporter only on that system.

Then, edit /etc/prometheus/prometheus.yml and add new IPs:

```
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100', '<IP_srv1>:9100, '<IP_srv2>:9100]
```

and restart the service

`sudo systemctl restart prometheus`

## Display metrics in Grafana

```
Step 1: Add Prometheus as a Data Source
Open Grafana: http://<ip>:3000
Click on "Configuration" → "Data Sources".
Click "Add Data Source".
Select Prometheus.
Set URL to: http://localhost:9090
Click "Save & Test".
```
```
Step 2: Import a Node Exporter Dashboard
In Grafana, click "Create" → "Import".
Enter Dashboard ID: 1860 (Official Node Exporter Dashboard).
Click "Load".
Select your Prometheus data source.
Click "Import".
```