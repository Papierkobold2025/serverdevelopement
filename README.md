# Serverdevelopment

Infraestructura personal (homelab) documentada — portafolio técnico basado en Proxmox. Este repositorio demuestra habilidades en administración de sistemas, segmentación de red, automatización (Semaphore / Ansible / Terraform) y despliegue de servicios (K3s, Keycloak, Nextcloud). Está pensado para mostrar decisiones de diseño, runbooks y ejemplos de IaC.

---

## Visión rápida

- Stack: Proxmox · K3s · OPNSense · Nginx Proxy Manager · Pi-hole · Keycloak · Vaultwarden · NetBird
- IaC / Automatización: Terraform · Ansible · Semaphore · Portainer
- Observabilidad: Homarr · Zabbix

## Arquitectura general (resumen)

Infraestructura pensada en **aislamiento de servicios** para reducir la superficie de ataque y limitar movimientos laterales. La topología incluye un plano de red plana y una VLAN que aloja servicios de infraestructura, con OPNSense como puente/firewall entre planos.

### Nodos del clúster

| Nodo | Propósito |
|---|---|
| **Panel** | Hipervisor principal, hardware nuevo, almacena servicios críticos |
| **API-Panel** | Hipervisor dedicado a APIs, dashboards y visualizaciones de estado del clúster/servicios |
| **Nextcloud** | Hipervisor de almacenamiento personal (Nextcloud) |
| **Nextcloud-sec** | Hipervisor de High Availability y replicación |
| **i5** | Hipervisor de replicación y servicios no críticos |
| **PBS** | Backups diarios de todos los nodos y sus VMs/contenedores |

📄 Especificaciones de hardware completas en [cluster/nodes.md](cluster/nodes.md)

## Servicios

| Servicio | Dónde vive | Documentación |
|---|---|---|
| Nextcloud | VM | [Nextcloud](docs/nextcloud.md) |
| NPM | VM | [Nginx](docs/nginx.md) |
| Pi-hole | VM | [Pihole](docs/pihole.md) |
| Vaultwarden | VM | [Vaultwarden](docs/vaultwarden.md) |
| Keycloak | LXC | [Keycloak](docs/keycloak.md) |
| NetBird | LXC | [Netbird](docs/netbird.md) |
| K3s (clúster ligero) | VM | [K3s](docs/k3s.md) - aloja Portainer, Semaphore, Zabbix y Homarr |
| Backups (PBS) | VM dedicada | [cluster/backup.md](cluster/backup.md) |

## Automatización e Infraestructura como Código

| Herramienta | Propósito | Documentación |
|---|---|---|
| Semaphore | Pipelines de despliegue | [Semaphore](docs/semaphore.md) |
| Ansible | Playbooks para configuración y parches | [Ansible](docs/semaphore.md#ansible) |
| Terraform | Despliegue repetible de VMs | [Terraform](docs/semaphore.md#terraform) |
| Portainer | Despliegue y configuración de contenedores y cluster de Kubernetes | [Portainer](docs/portainer.md) |

## Políticas de red

| Herramienta | Propósito | Documentación |
|---|---|---|
| OPNSense | Firewall/Router de VLAN y red plana | [OPNSense](docs/opnsense.md) |
| K3s | Firewall dentro del Cluster de K3s | [K3s](docs/k3s.md#network-policies) |

## Roadmap / Pendientes

- [ ] Configuración de alertas con Zabbix a endpoint de Telegram
- [ ] Configuración de firewall en K3s
- [ ] Configuración de firewall en Proxmox
- [ ] Ampliación de tareas de automatización en Semaphore, Terraform y cronjobs en Linux
- [ ] Cloudflare Access como capa extra para servicios expuestos (Keycloak)
- [ ] ntopng — visibilidad de tráfico de red
- [ ] Wazuh — SIEM, centralización de logs de seguridad
- [ ] HA / replicación multi-nodo de k3s

## Estado actual / En progreso

> 🚧 **Segmentación de red en progreso.** Se ha creado una subred crítica y se está migrando parte de la infraestructura; las reglas en OPNSense y las validaciones DNS están en desarrollo. Detalles y runbooks en [Segmentación de red](docs/opnsense.md).

## Índice de documentación archivada

- Documentación histórica / archivada (referencia) — carpeta `Archive/Services/`:
    - [Automatización (migrado)](Archive/Services/automation.md)
    - [Homepage (histórico)](Archive/Services/homepage.md)
    - [Monitoreo (histórico)](Archive/Services/monitoring.md)
    - [Watchtower (histórico)](Archive/Services/watchtower.md)
    - [Portainer (migrado)](Archive/Services/portainer.md)

## Índice de referencias rápidas

- Documentación de apoyo de desarrollo
    - [Docker](ops/docker.md) — instalación base

---

Última actualización: 2026-08-07