# Rundown de nodos del clúster Proxmox

Specs de hardware de cada nodo físico, sin direcciones IP (documentación interna, no expuesta en registros DNS públicos).

## nextcloud-prim
- **CPU(s)**: 4 x Intel Core i7-7567U @ 3.50GHz (1 Socket)
- **RAM total**: 62.69 GiB
- **Almacenamiento total**: 1.69 TiB
- **Kernel**: 7.0.2-6-pve
- **Manager**: pve-manager/9.2.2
- **Boot Mode**: EFI

## nextcloud
- **CPU(s)**: 4 x Intel Core i7-7567U @ 3.50GHz (1 Socket)
- **RAM total**: 62.68 GiB
- **Almacenamiento total**: 1.26 TiB
- **Kernel**: 7.0.2-6-pve
- **Manager**: pve-manager/9.2.4
- **Boot Mode**: EFI

## api-panel
- **CPU(s)**: 4 x Intel Core i7-7567U @ 3.50GHz (1 Socket)
- **RAM total**: 62.68 GiB
- **Almacenamiento total**: 1.28 TiB
- **Kernel**: 7.0.2-6-pve
- **Manager**: pve-manager/9.2.4
- **Boot Mode**: EFI

## panel
- **CPU(s)**: 16 x Intel Core i7-13620H, 13th Gen (1 Socket)
- **RAM total**: 62.44 GiB
- **Almacenamiento total**: 1.71 TiB
- **Kernel**: 7.0.2-6-pve
- **Manager**: pve-manager/9.2.2
- **Boot Mode**: EFI

## k3s
- **Producto**: Nodo dedicado a Kubernetes (k3s), sexto miembro del clúster
- **Notas**: hardware de laptop reutilizada; specs pendientes de documentar

## pbs-homelab
- **CPU(s)**: 8 x Intel Core i7-8559U @ 2.70GHz (1 Socket)
- **RAM total**: 62.67 GiB
- **Almacenamiento (root)**: 956.93 GB
- **Kernel**: 6.17.2-1-pve
- **Producto**: Proxmox Backup Server (no Proxmox VE — nodo dedicado a backups)
- **Boot Mode**: EFI

## Resumen comparativo

| Nodo | CPU | Núcleos/Hilos | RAM total | Almacenamiento total |
|---|---|---|---|---|
| nextcloud-prim | i7-7567U | 4 | 62.69 GiB | 1.69 TiB |
| nextcloud | i7-7567U | 4 | 62.68 GiB | 1.26 TiB |
| api-panel | i7-7567U | 4 | 62.68 GiB | 1.28 TiB |
| panel | i7-13620H | 16 | 62.44 GiB | 1.71 TiB |
| k3s | *(pendiente)* | *(pendiente)* | *(pendiente)* | *(pendiente)* |
| pbs-homelab | i7-8559U | 8 | 62.67 GiB | 956.93 GB (root) |

---

# VMs del cluster

Specs asignadas a cada VM (vCPU, RAM, disco), sin IPs.

| VM ID | Nombre | Nodo físico | vCPU | RAM | Disco | Notas |
|---|---|---|---|---|---|---|
| 100 | Minecraft | panel | 8 (1 socket) | 25.39 GiB | 300G | BIOS OVMF (UEFI) |
| 104 | Wireguard | panel | 3 (1 socket) | 9.77 GiB | 400G | |
| 105 | Nginx | panel | 2 (1 socket) | 9.77 GiB | 400G | |
| 107 | pihole | api-panel | 2 (1 socket) | 5.86 GiB | 100G | |
| 102 | nextcloud | nextcloud-prim | 2 (1 socket) | 7.81 GiB | 1.62 TB | |
| 112 | k3s | k3s | *(pendiente)* | *(pendiente)* | *(pendiente)* | VM que corre el clúster de Kubernetes |

### VMs eliminadas
| VM ID | Nombre | Motivo |
|---|---|---|
| 103 | Prometheus-Panel | Servicios migrados a k3s (Prometheus, Grafana, exporter) |
| 101 | landingpage (original) | Reemplazada por CT 109, luego CT 109 también eliminado |

# Contenedores del cluster

Specs asignadas a cada CT (vCPU, RAM, disco), sin IPs.

| CT ID | Nombre | Nodo físico | vCPU | RAM | Disco | Notas |
|---|---|---|---|---|---|---|
| 108 | keycloak | panel | 2 (1 socket) | 3.91 GiB | 30G | |
| 110 | vaultwarden | panel | 1 (1 socket) | 512 MiB | 10G | |
| 111 | portainer | panel | 1 (1 socket) | 512 MiB | 10G | Servidor central de Portainer |
| 101 | netbird | panel | 1 (1 socket) | 2G | 10G | |

### Contenedores eliminados
| CT ID | Nombre | Motivo |
|---|---|---|
| 106 | ansible / Semaphore | Migrado completo a k3s |
| 109 | landingpage | Homepage migrado a k3s; el LXC solo tenía Portainer (agente), NetBird-agent y Watchtower |
