# Pi-hole

- Configuración de pihole como reemplazo principal de DNS y DHCP del router, dejando al Router como servidor DNS secundario por si ocurriera algun tipo de error

- Reserva de direcciones IP mediante DHCP lease, de maner que los nodos principales que necesitan ser alcanzados como servicio tengan IP fija.


## Decisiones

- Mantener router como servidor DNS secundario por si contenedor fuera a tener un error

- Apuntar todos los dominios como servidor DNS a la IP de NPM para permitir resolución limpia de subdominios

- Configuración de subdominios con el formato: aplicación.servicio.midominio.com

- Rango de IP: 192.168.X.101 - 192.168.X.220 con Gateway estandar

### Runbook

- Configuración de compose.yaml en ruta /srv/pihole/compose.yaml

- ``` bash
  services:
    pihole:
      container_name: pihole
      image: pihole/pihole:latest
      network_mode: host
      environment:
        TZ: 'Europe/Zurich'
        FTLCONF_webserver_api_password: 'contraseña'
        FTLCONF_dns_listeningMode: 'ALL'
      volumes:
        - './etc-pihole:/etc/pihole'
      cap_add:
        - NET_ADMIN
      restart: unless-stopped
   ```

# Dificultades encontradas durante el desarrollo

- Una serie de servicios no pueden resolver subdominios DNS a menos que el dominio este dentro de la whitelist del servicio

- Solución encontrada:
    
    - El dominio al que se le estaba agregando tenía que ser agregado manualmente a los trusted domains del servicio

    - Ejemplos de servicios que necesitan configuración por whitelist: Nextcloud y Homepage

- Problema encontrado:

    - Al ser Pi-hole el propio servidor DHCP, la VM recibía su IP dinámicamente de sí misma — dependencia circular arriesgada ante reinicios

- Solución encontrada:

    - Configuración de IP estática a nivel de sistema operativo (Netplan, /etc/netplan/*.yaml), independiente de cualquier DHCP

- Problema encontrado:

    - Con network_mode por defecto (bridge), el servidor DHCP de Pi-hole no funcionaba: el broadcast DHCP no cruza la red bridge de Docker, por lo que los dispositivos nunca recibían respuesta y caían en IP de emergencia (APIPA, 169.254.x.x)

- Solución encontrada:

    - Cambiar a ``` bash network_mode: host ```, permitiendo que el contenedor comparta la red directamente con el host y reciba los broadcasts correctamente
