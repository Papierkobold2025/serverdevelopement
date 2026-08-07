# Portainer

- Portainer configuration to have control over the services running on the nodes.

## Decisions

- Create a new client in Keycloak to create two independent tokens for NetBird and Portainer login.

### Runbook

- Portainer configuration in /srv/portainer/compose.yaml

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

- After installing the container, the other environments must be configured (services to add).

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

- Add environments inside Portainer via IP:port.
