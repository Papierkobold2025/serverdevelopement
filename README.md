# Serverdevelopment

Amateur Server Development Project

# General Configurations

Configuración general del espacio de trabajo, configuraciones no específicos a una de las integraciones.

## Instalación de Docker Engine

``` bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

# Nextcloud integration

Integración de Nextcloud dentro de un entorno de cinco nodos dentro de una arquitectura de Proxmox.
Meta de esta integración es crear un entorno personal, desconectado para guardar archivos sin necesidad de licencias.

## Punto de partida:

- Intel NUC

  - 64GB de RAM
  - 1.5TB de almacenamiento

##  Meta a alcanzar a primer instancia:

- Configuración de entorno con virtualización mediante Proxmox

- Envío de correos por medio de SMTP para reseteo de contraseñas, bienvenida etc.

- Fail2Ban con conexión por API con Telegram para alertar de IPs bloqueadas.

- Configuración de intentos de inicio de sesión en Nextcloud antes de bloquear usuario.

- Configuración de 2FA y políticas de contraseña como medida de seguridad.

- Configuración de sistema de Backups para seguridad de sistemas.

- Abrir puertos a nivel router para uso de Nextcloud.

- Dificultadess encontradas en el dessarrollo inicial.

# Configuración de entorno

- Proxmox instalado en NUC con almacenamiento ZFS.

- Dominio propio configurado y apuntando al servidor — requisito de AIO para emitir certificado válido vía Let's Encrypt y habilitar acceso público.

- Instalación de UBUNTU Live Server 24.04 TLS en máquina virtual.

- Instalación de Docker Container nextcloud_aio_nextcloud

``` bash
sudo docker run \
--sig-proxy=false \
--name nextcloud-aio-mastercontainer \
--restart always \
--publish 8080:8080 \
--env APACHE_PORT=11000 \
--env APACHE_IP_BINDING=0.0.0.0 \
--env NEXTCLOUD_DATADIR=/mnt/nextcloud_data \
--volume nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
--volume /var/run/docker.sock:/var/run/docker.sock:ro \
nextcloud/all-in-one:latest
```

# Envío de correos por SMTP

- Configuración de SMTP

| Campo | Valor |
|---|---|
| Protocolo | SMTP |
| Codificación | Keine / STARTTLS |
| Host | smtp.gmail.com |
| Puerto | 587 |
| Autenticación | Sí - Application Password de Gmail |

# Fail2Ban / Nextcloud Password / 2FA Policy

- Configuración para falla de contraseña y de 2FA

## Configuración de notificación a nivel Log

- Ruta de configuración /etc/fail2ban/filter.d/nextcloud.conf

``` bash
[Definition]
_groupsre = (?:.*);\s*
failregex = ^.*"remoteAddr":"<HOST>".*"message":"Login failed.*$
            ^.*"remoteAddr":"<HOST>".*"message":"Two-factor challenge failed.*$
ignoreregex =
```

## Configuración de cárcel a falla de Nextcloud

- Ruta de configuración /etc/fail2ban/jail.d/nextcloud.conf

``` bash
[nextcloud]
enabled = true
maxretry = 4
findtime = 600
bantime = 3600
port = http,https
filter = nextcloud
action = iptables-multiport[name=nextcloud, port="http,https", protocol=tcp]
         telegram
backend = auto
logpath = /var/lib/docker/volumes/nextcloud_aio_nextcloud/_data/data/nextcloud.log
```

## Configuración de mensaje a Telegram en caso de bloqueo a nivel IP

- Ruta de configuración: /etc/fail2ban/action.d/telegram.conf

``` bash
[Definition]
actionban = /usr/local/bin/telegram-alert.sh "Nextcloud Alerta: IP <ip> bloqueada por intentos fallidos."
```

## Configuración de API para mandar mensaje de alerta a Telegram

- Rutta de configuración /usr/local/bin/telegram-alert.sh

``` bash
#!/bin/bash

TOKEN="TOKEN"
CHAT_ID="CHAT_ID"
MESSAGE=$1
curl -s -X POST https://api.telegram.org/bot$TOKEN/sendMessage -d chat_id=$CHAT_ID -d text="$MESSAGE"
```

- **Importante**

- El Bot de Telegram a primeras no funcionó y ningún menssaje fue recibido, al mandar un mensaje manualmente e iniciar conversación el envío automático funcionó.

### Validación para probar funcionalidad

``` bash
# Probar el filtro contra el log real, sin arrancar el jail
sudo fail2ban-regex /var/lib/docker/volumes/nextcloud_aio_nextcloud/_data/data/nextcloud.log /etc/fail2ban/filter.d/nextcloud.conf

# Validar sintaxis completa de fail2ban sin arrancar nada
sudo fail2ban-client -d
```

### Comandos que resultaron útiles:

``` bash
# Estado del jail
sudo fail2ban-client status nextcloud

# Desbanear una IP manualmente
sudo fail2ban-client set nextcloud unbanip <IP>
```

# Configuración de seguridad

- Forzar mediante Nextcloud Admin que los nuevos usuarios creen una instancia de 2FA.

- Complejidad de contraseña:

    - 12 caracteres de contraseña
    - Símbolos, números, minúsculas y mayúsculas forzadas

- Reusabilidad de contraseñas
    
    - 3 Contraseñas pasadas no pueden ser usadas

- Máximos intentos de contraseña
    
    - Máximos intentos reducidos a 3 antes del bloqueo del usuario

# Backups (Proxmox Backup Server)

- Quinto nodo dedicado corriendo PBS, con conexión directa desde los otros cuatro nodos del clúster para respaldo automático de todas las VMs.

## Configuración del Backup Job

| Campo | Valor |
|---|---|
| Storage | pbs-homelab |
| Nodos incluidos | Todos (-- All --) |
| Horario | 2:30 AM |
| Modo | Snapshot |
| Compresión | ZSTD (fast and good) |
| Selección | Todas las VMs |

## Retención

| Tipo | Cantidad |
|---|---|
| Daily | 7 |
| Weekly | 4 |

Cobertura aproximada: último mes completo, sin acumulación indefinida.

## Verificación e integridad

- Verificación (Verify) configurada de forma inmediata tras cada backup, no en horario separado.

- Restauración probada exitosamente en al menos una ocasión (VM de prueba).

## Notificaciones

- Notificaciones de éxito/falla del job configuradas por correo, reutilizando el mismo SMTP configurado para Nextcloud (ver sección "Envío de correos por SMTP").

# Configuración de puertos

- Puerto 80 protocolo TCP abierto para acceso por medio de http a Nextcloud

- Puerto 443 protocolo TCP abierto para conexión https a Nextcloud

- Puerto 3478 protocolo TCP/UDP abierto para poder permitir Nextcloud Talk módulo de llamadas tipo Teams

# Dificultadess encontradas durante el desarrollo

- Bug en panel de administración de Nextcloud

    - Definir antigüedad de contraseñas usables no es mandado de Frontend a Backend

    - Definir máxima cantidad de contraseñas erróneas no es mandado de Frontend a Backend

- Workaround encontrado:
    
    - ``` bash
      sudo docker exec -u www-data nextcloud-aio-nextcloud php occ config:app:set password_policy maximumLoginAttempts --value=3
      sudo docker exec -u www-data nextcloud-aio-nextcloud php occ config:app:set password_policy historySize --value=3
      ```

# Monitoreo de nodos de servidor Prometheus + Proxmox Exporter + Grafana

Exporter, Grafana y Prometheus corren dentro de una VM dedicada en un Docker, separada de la integración de Nextcloud para no comprometer la seguridad del nodo.

## Preparación para exponer datos a Graphana

- Configurar Proxmox exporter para exponer todos los datos de los nodos de Proxmox.

- Configurar Prometheus para juntar todos los datos de manera centralizada.

- Configurar Graphana para exponer los datos de manera ordenada en un panel de visualización.

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

## Dificultades encontradas

- Al montar un volumen (`-v`) apuntando a un archivo que todavía no existe en el host, Docker crea automáticamente una carpeta vacía en su lugar en vez de fallar — causó `IsADirectoryError` en el exporter hasta corregirlo.

- Los flags de `docker run` no se pueden editar después de crear el contenedor — cualquier cambio (como el binding de puerto) requiere `docker stop` + `docker rm` + volver a crear el contenedor. Si la configuración vive en un archivo externo montado, no se pierde nada en el proceso.
