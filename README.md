# Serverdevelopment

Desarrollo de infraestructura con base de **Proxmox**, priorizando seguridad y resiliencia.

El propósito del proyecto es crear un **portafolio de sysadmin**, instalando y administrando servicios de uso personal.

## Arquitectura general

Infraestructura pensada en **aislamiento de servicios** para evitar movimientos laterales entre máquinas, además de aislar servicios críticos para evitar vulneración de múltiples servicios simultáneamente.

### Nodos del clúster

| Nodo | Propósito |
|---|---|
| **Panel** | Hipervisor principal, hardware nuevo, almacena servicios críticos |
| **API-Panel** | Hipervisor dedicado a APIs, dashboards y visualizaciones de estado del clúster/servicios |
| **Nextcloud** | Hipervisor de almacenamiento personal (Nextcloud) |
| **Nextcloud-sec** | Hipervisor de High Availability y replicación |
| **K3s** | Hipervisor de Kubernetes, en proceso de concentrar servicios |
| **PBS** | Backups diarios de todos los nodos y sus VMs/contenedores |

📄 Especificaciones de hardware completas en [Nodes](cluster/nodes.md)

## Servicios

| Servicio | Dónde vive | Documentación |
|---|---|---|
| Nextcloud | VM | [Nextcloud](docs/nextcloud.md) |
| NPM | VM | [Nginx](docs/nginx.md) |
| Pi-hole | VM | [Pihole](docs/pihole.md) |
| Vaultwarden | VM | [Vaultwarden](docs/vaultwarden.md) |
| Keycloak | LXC | [Keycloak](docs/keycloak.md) |
| NetBird | LXC | [Netbird](docs/netbird.md) |
| Portainer | LXC | [Portainer](docs/portainer.md) |
| Zabbix, Homepage, Prometheus, Grafana, Semaphore | **k3s** | [K3s](docs/k3s.md) |
| Backups (PBS) | VM dedicada | [Backup](cluster/backup.md) |

## Kubernetes (k3s)

> 🚧 **En progreso** — adición más nueva al clúster de Proxmox, para empezar a administrar todo dentro de Kubernetes.

- En proceso de migración de servicios a Kubernetes
- Proceso de segmentación de red interna (Network Policies) para aislamiento de servicios críticos

## Automatización e Infraestructura como Código

- **Semaphore** — automatización de deployment de nuevos contenedores/VMs
- **Ansible** — playbooks para actualizaciones periódicas de sistema operativo e instalación de software
- **Terraform** — deployment de nuevas VMs de forma normalizada

## Roadmap / Pendientes

- [ ] Network Policies en K3s
- [ ] Distribución de almacenamiento vía Ceph
- [ ] Migración del resto de servicios a K3s
- [ ] Helm (gestor de paquetes para Kubernetes)
- [ ] Evaluación de Keel / GitOps (Renovate + Flux/ArgoCD) para auto-actualización de imágenes
- [ ] Cloudflare Access como capa extra para servicios expuestos (Keycloak)
- [ ] Segmentación de red física (VLANs) / OPNsense
- [ ] ntopng — visibilidad de tráfico de red
- [ ] Wazuh — SIEM, centralización de logs de seguridad
- [ ] HA / replicación multi-nodo de k3s

## Índice de documentación

- [Automatización (Semaphore/Ansible/Terraform)](docs/automation.md)
- [Docker](docs/docker.md)
- [Homepage](docs/homepage.md)
- [K3s / Kubernetes](docs/k3s.md)
- [Keycloak](docs/keycloak.md)
- [Monitoreo (Prometheus/Grafana)](docs/monitoring.md)
- [NetBird](docs/netbird.md)
- [Nextcloud](docs/nextcloud.md)
- [Nginx Proxy Manager](docs/nginx.md)
- [Pi-hole](docs/pihole.md)
- [Portainer](docs/portainer.md)
- [Vaultwarden](docs/vaultwarden.md)
- [Watchtower](docs/watchtower.md)
- [Backups (PBS)](cluster/backup.md)
- [Zabbix](docs/zabbix.md)
- [Nodos del clúster](cluster/nodes.md)
