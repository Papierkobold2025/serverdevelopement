# Vaultwarden

- Password manager self-hosted, propiedad de Bitwarden en formato OpenSource

## Decisiones

- Acceso solo interno

- HA replicado a nextcloud-sec

- Watchtower instalado

### Dificultades encontradas

- Creación de usuario vía "Invite User" del panel admin falla (bug conocido con organizationId en el link) — hay que crear el usuario entrando directo a la URL raíz

#### Runbook

``` bash 
services:
  vaultwarden:
    container_name: vaultwarden
    image: vaultwarden/server:latest
    restart: always
    ports:
      - "192.168.X.X:1620:80"
    volumes:
      -  /srv/vaultwarden:/data/
    environment:
      ADMIN_TOKEN: '${VAULTWARDEN_TOKEN}'
      DOMAIN: '${DOMAIN_VAULTWARDEN}'
      SIGNUPS_ALLOWED: "false"
      SMTP_HOST: '${SMTP_DOMAIN}'
      SMTP_FROM: '${SMTP_SEND}'
      SMTP_PORT: 587
      SMTP_SECURITY: starttls
      SMTP_USERNAME: '${SMTP_SEND}'
      SMTP_PASSWORD: '${SMTP_PASSWORD}'
```
