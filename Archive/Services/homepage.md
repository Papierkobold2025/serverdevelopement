# Homepage configuration

Integration of Homepage to have a Landing Page from which to open the installed services.

## Goal to be achieved at first instance:

- Install Docker Compose for Homepage.

- Configure pages as a Landing Page.

### Docker Compose configuration

- Configuration of compose.yaml at path /srv/homepage/compose.yaml.

- ```yaml
  services:
    homepage:
      container_name: homepage
      image: ghcr.io/gethomepage/homepage:latest
      restart: unless-stopped
      ports:
        - "192.168.X.X:3000:3000"
      working_dir: /app
      volumes:
        - /srv/homepage/homepage-config:/app/config
        - /var/run/docker.sock:/var/run/docker.sock:ro
      environment:
        HOMEPAGE_ALLOWED_HOSTS: homepage.apps.mydomain.com
        PUID: 1000
        PGID: 1000
  ```

- Start and stop Docker with sudo docker compose down and sudo docker compose up -d.
