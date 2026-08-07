# Monitoring server nodes with Prometheus + Proxmox Exporter + Grafana

The Exporter, Grafana, and Prometheus run inside a dedicated VM in Docker, separated from the Nextcloud integration so as not to compromise the security of the node.

## Preparation to expose data to Grafana

- Configure the Proxmox exporter to expose all data from the Proxmox nodes.

- Configure Prometheus to collect all data centrally.

- Configure Grafana to present the data in an orderly way in a visualization panel.

## Proxmox exporter configuration (`prometheus-pve-exporter`)

- Configuration path: `/srv/prometheus-exporter/pve.yml`

```yaml
default:
    user: prometheus@pve
    password: <YOUR_REAL_PASSWORD>
    verify_ssl: false
```

- Configuration path: `/srv/prometheus-exporter/compose.yml`

```bash
services:
  prometheus-pve-exporter:
    container_name: prometheus-pve-exporter
    image: prompve/prometheus-pve-exporter
    restart: always
    ports:
      - "192.168.X.X:9221:9221"
    volumes:
      - /srv/prometheus-exporter/pve.yml:/etc/prometheus/pve.yml
```

## Pihole exporter configuration (`pihole6_exporter`)

- Exporter installation:

```bash
sudo apt install python3-prometheus-client python3-requests -y
sudo curl -o /usr/local/bin/pihole6_exporter https://raw.githubusercontent.com/bazmonk/pihole6_exporter/main/pihole6_exporter
sudo chmod +x /usr/local/bin/pihole6_exporter
sudo curl -o /etc/systemd/system/pihole6_exporter.service https://raw.githubusercontent.com/bazmonk/pihole6_exporter/main/pihole6_exporter.service
```

- Configuration path: `/etc/systemd/system/pihole6_exporter.service`

```bash
[Unit]
Description=Pihole 6 Prometheus Exporter
After=pihole-FTL.service

[Service]
ExecStart=/usr/local/bin/pihole6_exporter -H localhost -k password
Type=exec
Restart=always

[Install]
WantedBy=default.target
```

- Note: unlike the other components in this document, pihole6_exporter does not run in Docker — it is a Python script installed as a direct systemd service on the Pi-hole VM (192.168.X.X), not on the monitoring VM.

- Important: the port binding uses the VM's internal IP (not `127.0.0.1` or `0.0.0.0`), so that it is accessible from the LAN but not from the internet.

## Prometheus configuration

- Configuration path: `/etc/prometheus/prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'pve'
    static_configs:
      - targets:
        - 192.168.X.X
    metrics_path: /pve
    params:
      module: [default]
      cluster: ['1']
      node: ['1']
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.X.X:9221
  - job_name: 'pihole'
    static_configs:
      - targets:
        - 192.168.X.X:9666
```

- Configuration path: `/srv/prometheus/compose.yml`

```bash
services:
  prometheus:
    container_name: prometheus
    image: prom/prometheus
    restart: always
    command:
      - '--config.file=/srv/prometheus/prometheus.yml'
    ports:
      - "192.168.X.X:9090:9090"
    volumes:
      - /srv/prometheus/prometheus.yml:/srv/prometheus/prometheus.yml
      - /srv/prometheus/alert_rules.yml:/srv/prometheus/alert_rules.yml
```

## Grafana configuration

- Configuration path: `/srv/grafana/compose.yaml`

```yaml
services:
  grafana:
    container_name: grafana
    image: grafana/grafana:latest
    restart: unless-stopped
    ports:
      - "192.168.X.X:3000:3000"
    environment:
      GF_SMTP_ENABLED: 'true'
      GF_SMTP_HOST: 'smtp.gmail.com:587'
      GF_SMTP_USER: 'your_email@gmail.com'
      GF_SMTP_PASSWORD: 'your_app_password_from_gmail'
      GF_SMTP_FROM_ADDRESS: 'your_email@gmail.com'
    volumes:
      - './data:/var/lib/grafana'
```

- Important: the ./data folder must belong to UID 472 (Grafana's internal user): sudo chown -R 472:472 /srv/grafana/data — otherwise the container will enter a restart loop (Permission denied).

- Boot confirmation

    - sudo docker ps | grep grafana

- Connection of the Prometheus database to Grafana through the Prometheus IP:9090 address.

- Importation of the Grafana dashboard template for Proxmox with ID: 10347.

- Importation of the Grafana dashboard template for Pi-hole with ID: 21043.

## Issues encountered

- When mounting a volume (`-v`) pointing to a file that does not yet exist on the host, Docker automatically creates an empty directory in its place instead of failing — this caused an `IsADirectoryError` in the exporter until it was corrected.

- Docker `run` flags cannot be edited after creating the container — any change (such as the port binding) requires `docker stop` + `docker rm` + recreating the container. If the configuration lives in an externally mounted file, nothing is lost in the process.
