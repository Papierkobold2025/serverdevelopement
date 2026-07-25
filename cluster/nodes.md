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

## pbs-homelab
- **CPU(s)**: 8 x Intel Core i7-8559U @ 2.70GHz (1 Socket)
- **RAM total**: 62.67 GiB
- **Almacenamiento (root)**: 956.93 GB
- **Kernel**: 6.17.2-1-pve
- **Producto**: Proxmox Backup Server (no Proxmox VE — nodo dedicado a backups, quinto nodo del clúster)
- **Boot Mode**: EFI

## Resumen comparativo

| Nodo | CPU | Núcleos/Hilos | RAM total | Almacenamiento total |
|---|---|---|---|---|
| nextcloud-prim | i7-7567U | 4 | 62.69 GiB | 1.69 TiB |
| nextcloud | i7-7567U | 4 | 62.68 GiB | 1.26 TiB |
| api-panel | i7-7567U | 4 | 62.68 GiB | 1.28 TiB |
| panel | i7-13620H | 16 | 62.44 GiB | 1.71 TiB |
| pbs-homelab | i7-8559U | 8 | 62.67 GiB | 956.93 GB (root) |

---

# VMs del cluster

Specs asignadas a cada VM (vCPU, RAM, disco), sin IPs.

| VM ID | Nombre | Nodo físico | vCPU | RAM | Disco | Notas |
|---|---|---|---|---|---|---|
| 100 | Minecraft | panel | 8 (1 socket) | 25.39 GiB | 300G | BIOS OVMF (UEFI) |
| 103 | Prometheus-Panel | api-panel | 3 (1 socket) | 9.77 GiB | 500G | |
| 104 | Wireguard | panel | 3 (1 socket) | 9.77 GiB | 400G | |
| 105 | Nginx | panel | 2 (1 socket) | 9.77 GiB | 400G | |
| 107 | pihole | api-panel | 2 (1 socket) | 5.86 GiB | 100G | |
| 102 | nextcloud | nextcloud-prim | 2 (1 socket) | 7.81 GiB | 1.62 TB | |
| 101 | landingpage | api-panel | 2 (1 socket) | 9.77 GiB | 32 GiB | Pendiente de eliminar, reemplazada por CT 109 |

# Contenedores del cluster

Specs asignadas a cada VM (vCPU, RAM, disco), sin IPs.

| VM ID | Nombre | Nodo f  sico | vCPU | RAM | Disco | Notas |
|---|---|---|---|---|---|---|
| 106 | ansible | api-panel | 1 (1 socket) | 1.46 GiB | 8 GiB | |
| 109 | landingpage | api-panel | 1 (1 socket) | 512 MiB | 4 GiB | Creado con Terraform, reemplaza a VM 101 |

