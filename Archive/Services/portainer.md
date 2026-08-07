# Portainer

- Configuración de Portainer para poder tener control sobre los seervicios corriendo en los nodos

## Decisiones

- Creación de nuevo cliente en Keycloak para crear dos Tokens independientes para Login de Netbird y Portainer

### Runbook

- Configuración de Portainer en /srv/portainer/compose.yaml

```bash
services:
  portainer:
    image: portainer/portainer-ce:lts
    container_name: portainer
    restart: unless-stopped
    ports:
      - "192.168.X.X:9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer-data:/data
volumes:
  portainer-data:
```

- Despues de instalación de contenedor se tienen que configurar los demas entornos (Servicios a agregar)

```bash
services:
  agent:
    image: portainer/agent:latest
    container_name: portainer_agent
    restart: unless-stopped
    ports:
      - "192.168.X.X:9001:9001"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
```

- Agregar entornos dentro de Portainer mediante IP:puerto
