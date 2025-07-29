# Prometheus Lab2 - Secure Node Exporter on node02 (TLS + Basic Auth)

# Step 1 – Generate Self-Signed TLS Certificates

Commands:
bash
openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 \
  -keyout node_exporter.key -out node_exporter.crt \
  -subj "/C=US/ST=California/L=Oakland/O=MyOrg/CN=localhost" \
  -addext "subjectAltName = DNS:localhost"

mkdir -p /etc/node_exporter
mv node_exporter.crt node_exporter.key /etc/node_exporter


Outcome:
- TLS certificate and key created and moved to /etc/node_exporter`.

# Step 2 – Create Node Exporter Configuration with TLS and Basic Auth

Commands:
Create and edit `/etc/node_exporter/config.yml` with the following content:

yaml
tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key

basic_auth_users:
  prometheus: $2y$10$TJOdwozyUEqEmiAMPkb2Q0F.17d/3568IZ..zXQnLdI0Wn=5JhSKty


Outcome:
- TLS encryption and HTTP basic authentication configured.

# Step 3 – Set Directory Permissions

Commands:
bash
chmod 700 /etc/node_exporter
chmod 600 /etc/node_exporter/config.yml
chown -R nodeusr:nodeusr /etc/node_exporter

Outcome:
- Secure permissions and ownership set on the config directory.

# Step 4 – Create systemd Service File

Commands:
vi /etc/systemd/system/node_exporter.service` with the following content:

[Unit]
Description=Node Exporter
After=network.target

[Service]
User=nodeusr
Group=nodeusr
Type=simple
ExecStart=/usr/local/bin/node_exporter \
  --web.config=/etc/node_exporter/config.yml

[Install]
WantedBy=multi-user.target

Outcome:
- Node Exporter will run as a systemd service with TLS and auth.

# Step 5 – Reload systemd and Start Node Exporter

Commands:
bash
systemctl daemon-reload
systemctl restart node_exporter
systemctl status node_exporter

Outcome:
- Node Exporter service is running with TLS enabled.

# Step 6 – Test HTTP-to-HTTPS Error (Expected)

Commands:
From Prometheus server:
bash
curl -u prometheus:$2y$10$TJOdwozyUEqEmiAMPkb2Q0F.17d/3568IZ..zXQnLdI0Wn=5JhSKty \
  http://node02:9100/metrics


Outcome:
- Expected error: `Client sent an HTTP request to an HTTPS server.`

# Step 7 – Add node02 to Prometheus Scrape Config

Commands:
vi /etc/prometheus/prometheus.yml` and update the scrape config:

yaml
- job_name: "nodes"
  static_configs:
    - targets: ["node01:9100", "node02:9100"]
      basic_auth:
        username: prometheus
        password: $2y$10$TJOdwozyUEqEmiAMPkb2Q0F.17d/3568IZ..zXQnLdI0Wn=5JhSKty


Outcome:
- Prometheus now scrapes metrics from node02 securely using TLS and authentication.
