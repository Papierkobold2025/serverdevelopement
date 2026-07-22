# Monitoreo de nodos de servidor Prometheus + Proxmox Exporter + Grafana

Exporter, Grafana y Prometheus corren dentro de una VM dedicada en un Docker, separada de la integración de Nextcloud para no comprometer la seguridad del nodo.

## Preparación para exponer datos a Grafana

- Configurar Proxmox exporter para exponer todos los datos de los nodos de Proxmox.

- Configurar Prometheus para juntar todos los datos de manera centralizada.

- Configurar Grafana para exponer los datos de manera ordenada en un panel de visualización.

## Configuración de exporter para Proxmox (`prometheus-pve-exporter`)

- Ruta de configuración: `/etc/prometheus/pve.yml`

``` yaml
default:
    user: prometheus@pve
    password: <TU_CONTRASEÑA_REAL>
    verify_ssl: false
```

``` bash
sudo docker run \
--init \
--name prometheus-pve-exporter \
--restart always \
-d \
-p 192.168.1.127:9221:9221 \
-v /etc/prometheus/pve.yml:/etc/prometheus/pve.yml \
prompve/prometheus-pve-exporter
```

## Configuración de exporter para pihole (`pihole6_exporter`)

- instalación de exporter:

``` bash
sudo apt install python3-prometheus-client python3-requests -y
sudo curl -o /usr/local/bin/pihole6_exporter https://raw.githubusercontent.com/bazmonk/pihole6_exporter/main/pihole6_exporter
sudo chmod +x /usr/local/bin/pihole6_exporter
sudo curl -o /etc/systemd/system/pihole6_exporter.service https://raw.githubusercontent.com/bazmonk/pihole6_exporter/main/pihole6_exporter.service
```

- Ruta de configuración: `/etc/systemd/system/pihole6_exporter.service`

``` bash
[Unit]
Description=Pihole 6 Prometheus Exporter
After=pihole-FTL.service

[Service]
ExecStart=/usr/local/bin/pihole6_exporter -H localhost -k Dape72duro_2026!!
Type=exec
Restart=always


[Install]
WantedBy=default.target
```

- Nota: a diferencia de los demás componentes de este documento, pihole6_exporter no corre en Docker — es un script Python instalado como servicio systemd directo en la VM de Pi-hole (192.168.1.138), no en la VM de monitoreo.

- **Importante**: el binding de puerto usa la IP interna de la VM (no `127.0.0.1` ni `0.0.0.0`), para que sea accesible desde la LAN pero no desde internet.

## Configuración de Prometheus

- Ruta de configuración: `/etc/prometheus/prometheus.yml`

``` yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'pve'
    static_configs:
      - targets:
        - 192.168.1.123
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
        replacement: 192.168.1.127:9221
  - job_name: 'pihole'
    static_configs:
      - targets:
        - 192.168.1.138:9666
```

``` bash
sudo docker run \
--name prometheus \
--restart always \
-d \
-p 192.168.1.127:9090:9090 \
-v /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
prom/prometheus
```

## Configuración de Grafana

``` bash

sudo docker run \
--name grafana \
--restart always \
-d \
-p 192.168.1.127:3000:3000 \
grafana/grafana

```

- Migración de Grafana a Docker Compose para mejor documentación

- Ruta de configuración: `/srv/grafana/compose.yaml`

``` yaml
services:
  grafana:
    container_name: grafana
    image: grafana/grafana:latest
    restart: unless-stopped
    ports:
      - "192.168.1.127:3000:3000"
    environment:
      GF_SMTP_ENABLED: 'true'
      GF_SMTP_HOST: 'smtp.gmail.com:587'
      GF_SMTP_USER: 'tu_correo@gmail.com'
      GF_SMTP_PASSWORD: 'tu_app_password_de_gmail'
      GF_SMTP_FROM_ADDRESS: 'tu_correo@gmail.com'
    volumes:
      - './data:/var/lib/grafana'
```

- Importante: la carpeta ./data debe pertenecer al UID 472 (usuario interno de Grafana): sudo chown -R 472:472 /srv/grafana/data — de lo contrario el contenedor entra en loop de reinicio (Permission denied).

- Confirmación de arranque

    - sudo docker ps | grep grafana

- Conexión de base de datos de Prometheus con Grafana por medio de la dirección IP:9090 de Prometheus

- Importación de template de pantalla de Grafana para Proxmox con la ID: 10347

- Importación de template de pantalla de Grafana para Pihole con la ID: 21043

## Dificultades encontradas

- Al montar un volumen (`-v`) apuntando a un archivo que todavía no existe en el host, Docker crea automáticamente una carpeta vacía en su lugar en vez de fallar — causó `IsADirectoryError` en el exporter hasta corregirlo.

- Los flags de `docker run` no se pueden editar después de crear el contenedor — cualquier cambio (como el binding de puerto) requiere `docker stop` + `docker rm` + volver a crear el contenedor. Si la configuración vive en un archivo externo montado, no se pierde nada en el proceso.
