# Nextcloud integration

- Integración de Nextcloud dentro de una VM en el cluster de nodos.

- Configuración y construcción de entorno de almacenaje de archivos personal.

## Decisiones

- Aislamiento de Nextcloud de los demás servicios, en su propio nodo dedicado (con replicación a Nextcloud-sec para HA, ver siguiente punto)

- Configuración forzada de 2FA como medida de seguridad adicional al login

- Configuración de replicación de VM al nodo Nextcloud-sec y configuración de HA para evitar fallo de servicios o Downtime largos

## Runbook

- Instalación de Docker Container nextcloud_aio_nextcloud

```bash
services:
  nextcloud_aio_mastercontainer:
    container_name: nextcloud-aio-mastercontainer
    image: nextcloud/all-in-one:latest
    restart: always
    ports:
      - "8080:8080"
    environment:
      APACHE_PORT: 11000
      APACHE_IP_BINDING: 0.0.0.0
    volumes:
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config
      - /var/run/docker.sock:/var/run/docker.sock:ro

volumes:
  nextcloud_aio_mastercontainer:
    external: true
```

- Configuración de SMTP

| Campo | Valor |
|---|---|
| Protocolo | SMTP |
| Codificación | Ninguna / STARTTLS |
| Host | smtp.gmail.com |
| Puerto | 587 |
| Autenticación | Sí - Application Password de Gmail |

## Dificultades encontradas durante el desarrollo

- Bug en panel de administración de Nextcloud

  - Definir antigüedad de contraseñas usables no es mandado de Frontend a Backend

  - Definir máxima cantidad de contraseñas erróneas no es mandado de Frontend a Backend

- Workaround encontrado:

```bash
  sudo docker exec -u www-data nextcloud-aio-nextcloud php occ config:app:set password_policy maximumLoginAttempts --value=3
  sudo docker exec -u www-data nextcloud-aio-nextcloud php occ config:app:set password_policy historySize --value=3
```