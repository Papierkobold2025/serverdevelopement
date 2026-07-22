# Homepage configuration

Integración de Homepage para poder tener una Landing Page desde donde poder abrir los servicios instalados

##  Meta a alcanzar a primer instancia:

- Instalar Docker Compose para Homepage

- Configuración de páginas como Landing Page

### Configuración de docker compose

- Configuración de compose.yaml en ruta /srv/homepage/compose.yaml

- ``` yaml
  services:
    homepage:
      container_name: homepage
      image: ghcr.io/gethomepage/homepage:latest
      restart: unless-stopped
      ports:
        - "192.168.1.128:3000:3000"
      working_dir: /app
      volumes:
        - /srv/homepage/homepage-config:/app/config
        - /var/run/docker.sock:/var/run/docker.sock:ro
      environment:
        HOMEPAGE_ALLOWED_HOSTS: homepage.apps.moralesdario.com
        PUID: 1000
        PGID: 1000
   ```

- Levantar y bajar docker con sudo docker compose down y sudo docker compose up -d
