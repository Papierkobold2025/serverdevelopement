# Nginx

- Configuración de Reverse Proxy dentro de la infraestructura para utilizar subdominios dentro del dominio.

- Configuración de dominio con certificados SSL.

## Runbook

Configuración de documento /srv/nginx/compose.yml:

```bash
services:
  npm:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: npm
    restart: unless-stopped
    ports:
      - '80:80'
      - '443:443'
      - '81:81'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt

```

- Configuración de puerto para Apache en Nextcloud a puerto 11000 para poder usar los puertos 80 y 443 dirigidos a nginx

- Apertura de puertos a nivel de router, 80 && 443 dirigidos a la IP de nginx y 11000 a Nextcloud

# Dificultades encontradas

- Contenedor de Apache no actualizaba puerto por guardado en Cache por parte del Master Container

   - Solución encontrada:

   - Desinstalación de contenedor apache y reinstalación con puerto especificado

- Certificado SSL no prendido a nivel de Nginx, respuesta: Página rechazó la conexión

   - Solución encontrada:

   - Instalación de certificado SSL forzando SSL

