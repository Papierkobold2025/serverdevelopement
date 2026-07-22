# Pi-hole configuration

Integración de Pi-hole para poder registrar listas DNS para conexión interna mediante DHCP.
Instalación mediante Docker Compose en vez de Docker run para documentación mas sencilla.

##  Meta a alcanzar a primer instancia:

- Configuración de docker compose

- Registro de DNS records.

- Utilizar pi-hole como servidor DHCP/DNS en vez de router para resolucion de la lista DNS.

- Configuración de pihole para reenviar todo tráfico a nginx.

- Dificultadess encontradas en el dessarrollo inicial.

### Configuración de docker compose

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

- Levantar y bajar docker con sudo docker compose down y sudo docker compose up -d

# Registro de DNS records

- Registros DNS configurados reenviando todo trafico a Nginx

- Configuración fue hecha en base al servicio de cada uno de los dominios

- *.proxmox.moralesdario.com para nodos Proxmox

- *.backup.moralesdario.com para servidores Backup

- *.apps.moralesdario.com para servicios

- *.monitoring.moralesdario.com para servicios de monitoreo

- *.network.moralesdario.com para servicios de red

- *.storage.moralesdario.com para servicios de almacenamiento

# Configuración DHCP en pihole y DNS en gninx

- Pi-hole fue configurado como servidor DHCP, desactivando por completo el DHCP del router

- El DNS del router se mantiene como servidor DNS secundario en la configuración manual de algunos dispositivos, como respaldo si Pi-hole llegara a fallar

- Reenvío de peticiones de pihole a nginx, ya que en nginx se configurarion los certificados SSL, finalmente reenviando la petición a la IP correcta con el subdominio correcto

- Configuración de subdominios a nivel de pihole haciendo Match con la configuración en nginx para que el reenvío funcionara correctamente

## Configuración de rango DHCP y reservas estáticas

- Rango repartido: 192.168.1.101 - 192.168.1.220

- Gateway: 192.168.1.1

- Reservas estáticas (MAC → IP) configuradas para todas las VMs con IP fija, evitando que pierdan su dirección al renovar DHCP — ver Settings → DHCP → Static DHCP configuration en Pi-hole

# Dificultadess encontradas durante el desarrollo

- Ciertos Dockers no es suficiente con configurar reenvío de IP de nginx a la IP meta

- Solución encontrada:
    
    - El dominio al que se le estaba agregando tenía que ser agregado manualmente a los trusted domains del docker meta

    - Ciertos dockers como Nextcloud y Homepage necesitan que su subdominio sea agregado a trusted domains, ya que cada llamada la comparan con una whitelist de dominios

- Problema encontrado:
    - Al ser Pi-hole el propio servidor DHCP, la VM recibía su IP dinámicamente de sí misma — dependencia circular arriesgada ante reinicios
- Solución encontrada:
    - Configuración de IP estática a nivel de sistema operativo (Netplan, /etc/netplan/*.yaml), independiente de cualquier DHCP

- Problema encontrado:
    - Con network_mode por defecto (bridge), el servidor DHCP de Pi-hole no funcionaba: el broadcast DHCP no cruza la red bridge de Docker, por lo que los dispositivos nunca recibían respuesta y caían en IP de emergencia (APIPA, 169.254.x.x)
- Solución encontrada:
    - Cambiar a ``` bash network_mode: host ```, permitiendo que el contenedor comparta la red directamente con el host y reciba los broadcasts correctamente
