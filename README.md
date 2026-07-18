# Serverdevelopment
Amateur Server Development Project
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
- Abrir puertos a nivel router para uso de Nextcloud.
-  Dificultadess encontradas en el dessarrollo inicial.
# Configuración de entorno
- Proxmox instalado en NUC con almacenamiento ZFS.
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
