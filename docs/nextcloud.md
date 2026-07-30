# Nextcloud integration

- Integración de Nextcloud dentro de una VM en el cluster de nodos.

- Configuración y construcción de entorno de almacenaje de archivos personal.

## Decisiones

- Aislamiento de Nextcloud de los demas servicios dejándolo dentro de un solo nodo.

- Configuración forzada de 2FA como medida de seguridad adicional al login

- Configuración de Fail2Ban a nivel IP para evitar ataques de fuerza bruta

- Configuración de replicación de VM al nodo Nextcloud-sec y configuración de HA para evitar fallo de servicios o Downtime largos

### Runbook

- Instalación de Docker Container nextcloud_aio_nextcloud

``` bash
services:
  nextcloud_aio_mastercontainer:
    container_name: nextcloud-aio-mastercontainer
    image: nextcloud/all-in-one:latest
    restart: always
    ports:
      - "8080:8080"
    environment:
      APACHE_PORT: 11000
      APACHE_IP_BINDING: 0.0.0.0
    volumes:
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config
      - /var/run/docker.sock:/var/run/docker.sock:ro

volumes:
  nextcloud_aio_mastercontainer:
    external: true
```

- Configuración de SMTP

| Campo | Valor |
|---|---|
| Protocolo | SMTP |
| Codificación | Keine / STARTTLS |
| Host | smtp.gmail.com |
| Puerto | 587 |
| Autenticación | Sí - Application Password de Gmail |

- Ruta de configuración /etc/fail2ban/filter.d/nextcloud.conf

``` bash
[Definition]
_groupsre = (?:.*);\s*
failregex = ^.*"remoteAddr":"<HOST>".*"message":"Login failed.*$
            ^.*"remoteAddr":"<HOST>".*"message":"Two-factor challenge failed.*$
ignoreregex =
```

- Ruta de configuración /etc/fail2ban/jail.d/nextcloud.conf

``` bash
[nextcloud]
enabled = true
maxretry = 4
findtime = 600
bantime = 3600
filter = nextcloud
action = sshban
         telegram
backend = auto
logpath = /var/lib/docker/volumes/nextcloud_aio_nextcloud/_data/data/nextcloud.log
```

- Ruta de configuración: /etc/fail2ban/action.d/telegram.conf

``` bash
[Definition]
actionban = /usr/local/bin/telegram-alert.sh "Nextcloud Alerta: IP <ip> bloqueada por intentos fallidos."
```

- Ruta de configuración: /etc/fail2ban/action.d/sshban.conf

``` bash
[Definition]
actionban = /usr/local/bin/nginx-ban.sh "<ip>"
actionunban = /usr/local/bin/nginx-unban.sh "<ip>"
```

- Rutta de configuración /usr/local/bin/telegram-alert.sh

``` bash
#!/bin/bash

TOKEN="TOKEN"
CHAT_ID="CHAT_ID"
MESSAGE=$1
curl -s -X POST https://api.telegram.org/bot$TOKEN/sendMessage -d chat_id=$CHAT_ID -d text="$MESSAGE"
```

- Ruta de configuración: /usr/local/bin/nginx-ban.sh

 ``` bash
#!/bin/bash
ssh="ssh -i /root/.ssh/nginx_ban_key -p 2226 nginx@192.168.1.133"
ban="sudo ipset add blocklist $1"
$ssh "$ban"
```

- Ruta de configuración: /usr/local/bin/nginx-unban.sh

 ``` bash
#!/bin/bash
ssh="ssh -i /root/.ssh/nginx_unban_key -p 2226 nginx@192.168.1.133"
ban="sudo ipset add blocklist $1"
$ssh "$ban"
```

- Configuración crontab para hacer la configuración persistente

``` bash
0 3 * * * /usr/sbin/netfilter-persistent save
```

## Dificultadess encontradas durante el desarrollo

- Bug en panel de administración de Nextcloud

    - Definir antigüedad de contraseñas usables no es mandado de Frontend a Backend

    - Definir máxima cantidad de contraseñas erróneas no es mandado de Frontend a Backend

- Workaround encontrado:

    - ``` bash
      sudo docker exec -u www-data nextcloud-aio-nextcloud php occ config:app:set password_policy maximumLoginAttempts --value=3
      sudo docker exec -u www-data nextcloud-aio-nextcloud php occ config:app:set password_policy historySize --value=3
      ```

# Validación para probar funcionalidad

``` bash
# Probar el filtro contra el log real, sin arrancar el jail
sudo fail2ban-regex /var/lib/docker/volumes/nextcloud_aio_nextcloud/_data/data/nextcloud.log /etc/fail2ban/filter.d/nextcloud.conf

# Validar sintaxis completa de fail2ban sin arrancar nada
sudo fail2ban-client -d
```

# Comandos que resultaron útiles:

``` bash
# Estado del jail
sudo fail2ban-client status nextcloud

# Desbanear una IP manualmente
sudo fail2ban-client set nextcloud unbanip <IP>
```
