# Vaultwarden

- Password manager self-hosted, propiedad de Bitwarden en formato OpenSource

## Decisiones

- Acceso solo interno para evitar comprometer todas las contraseñas para entrar a servicios

- HA replicado a nextcloud-sec por la criticalidad de tener acceso a contraseñas

- Watchtower instalado para asegurar que el contenedor docker siempre estuviera en la version mas actual para evitar versiones deprecadas

## Dificultades encontradas

- Creación de usuario vía "Invite User" del panel admin falla (bug conocido con organizationId en el link) — hay que crear el usuario entrando directo a la URL raíz

## Runbook

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
