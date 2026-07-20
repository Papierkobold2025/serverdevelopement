# Monitoreo de nodos de servidor Prometheus + Proxmox Exporter + Grafana

Exporter, Grafana y Prometheus corren dentro de una VM dedicada en un Docker, separada de la integración de Nextcloud para no comprometer la seguridad del nodo.

## Preparación para exponer datos a Grafana

- Configurar Proxmox exporter para exponer todos los datos de los nodos de Proxmox.

- Configurar Prometheus para juntar todos los datos de manera centralizada.

- Configurar Grafana para exponer los datos de manera ordenada en un panel de visualización.

## Configuración del exporter (`prometheus-pve-exporter`)

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

- Confirmación de arranque

    - sudo docker ps | grep grafana

- Conexión de base de datos de Prometheus con Grafana por medio de la dirección IP:9090 de Prometheus

- Importación de template de pantatlla de Grafana con la ID: 10347

## Dificultades encontradas

- Al montar un volumen (`-v`) apuntando a un archivo que todavía no existe en el host, Docker crea automáticamente una carpeta vacía en su lugar en vez de fallar — causó `IsADirectoryError` en el exporter hasta corregirlo.

- Los flags de `docker run` no se pueden editar después de crear el contenedor — cualquier cambio (como el binding de puerto) requiere `docker stop` + `docker rm` + volver a crear el contenedor. Si la configuración vive en un archivo externo montado, no se pierde nada en el proceso.
