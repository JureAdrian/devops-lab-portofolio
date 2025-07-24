 "# Prometheus Lab1 - Installation and Configuration" 

# Step 1 – Extracting Prometheus

Commands:
tar -xzf prometheus-3.5.0.linux-amd64.tar.gz
mv prometheus-3.5.0.linux-amd64 prometheus

Outcome:
- Unpacked binaries and default config.

# Step 2 - Move Binaries to system path

Commands: 
sudo mv prometheus/prometheus /usr/local/bin/
sudo mv prometheus/promtool /usr/local/bin/

Outcome:
- prometheus and promtool binaries are now globally accessible.

# Step 3 – Create Prometheus user and directories

Commands:
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

Outcome:
Dedicated user and folders for Prometheus created.

# Step 4 – Grant ownership over folders and binaries

Commands:
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /var/lib/prometheus

Outcome: 
File ownership assigned to Prometheus user.

# Step 5 – Create systemd Prometheus service

Commands:
sudo vi /etc/systemd/system/prometheus.service

Content: 
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
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

# Step 6 – Reload systemd and start Prometheus

Commands: 
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus

Outcome:
Prometheus service is running and enabled at startup.

# Step 7 – Customize port 

Context:
If port 9090 is already used, update the ExecStart line in prometheus.service to use port 9091, added the following line on the prometheus.service file:

--web.listen-address=":9091"




