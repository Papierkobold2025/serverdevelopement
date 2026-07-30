# Watchtower

- Integración de Watchtower dentro del cluster para poder mantener actualizados los dockers instalados

- Se trata de un servicio que se tiene que instalar en cada nodo, VM o contenedor que contiene dockers para pasar actualizaciones automáticas y borrar contenedores viejos

## Decisiones

- Quedará deprecado para la mayor parte del proyecto: k3s usa containerd por default (no Docker), y Watchtower depende del socket de Docker para funcionar.

### Runbook
```bash
services:
  watchtower:
    image: nickfedor/watchtower
    restart: unless-stopped
    environment:
      WATCHTOWER_NOTIFICATIONS: shoutrrr
      WATCHTOWER_NOTIFICATION_URL: '${TELEGRAM}'
      WATCHTOWER_SCHEDULE: "0 0 3 * * *"
      WATCHTOWER_NOTIFICATION_TITLE_TAG: "ansible"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
