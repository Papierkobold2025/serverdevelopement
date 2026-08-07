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

- Apertura de puertos a nivel de router, 80 && 443 dirigidos a la IP de nginx y 11000 a Nextcloud

## Dificultades encontradas

- Contenedor de Apache no actualizaba puerto por guardado en Cache por parte del Master Container

  - Solución encontrada: desinstalación del contenedor apache y reinstalación con puerto especificado
