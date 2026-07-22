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
- **Nota**: nodo más nuevo del clúster (CPU de generación más reciente, 16 hilos vs 4 de los demás) — es el nodo elegido para alojar la VM de k3s por ser el menos probable de fallar.

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

# VMs del clúster

Specs asignadas a cada VM (vCPU, RAM, disco), sin IPs.

| VM ID | Nombre | Nodo físico | vCPU | RAM | Disco | Notas |
|---|---|---|---|---|---|---|
| 100 | Minecraft | panel | 8 (1 socket) | 25.39 GiB | 300G | BIOS OVMF (UEFI) |
| 103 | Prometheus-Panel | api-panel | 3 (1 socket) | 9.77 GiB | 500G | |
| 104 | Wireguard | panel | 3 (1 socket) | 9.77 GiB | 400G | |
| 105 | Nginx | panel | 2 (1 socket) | 9.77 GiB | 400G | |
| 106 | Kubernetes | panel | 2 (1 socket) | 7.81 GiB | 40G | |
| 107 | pihole | api-panel | 2 (1 socket) | 5.86 GiB | 100G | |

## Notas sobre las VMs

- Todas comparten controlador SCSI VirtIO single, máquina i440fx (default), y arquitectura x86-64-v2-AES.
- La mayoría instaladas desde `ubuntu-24.04.4-live-server-amd64.iso`.
- **Minecraft** es la única con BIOS OVMF (UEFI) en vez de SeaBIOS — también la que más RAM y núcleos tiene asignados de todo el clúster.
- El nodo `panel` concentra la mayoría de las VMs de servicios "nuevos" (Minecraft, WireGuard, Nginx, Kubernetes) — consistente con ser el nodo más nuevo/potente, elegido para alojar servicios en crecimiento.
- `api-panel` aloja Prometheus/Grafana y Pi-hole — los dos servicios de infraestructura crítica (monitoreo y DNS/DHCP).

## Nota sobre nomenclatura

Los nombres de nodo (`nextcloud`, `nextcloud-prim`, `api-panel`, `panel`) no necesariamente corresponden 1:1 con los servicios que alojan — son nombres asignados al crear cada nodo en Proxmox, distintos de los hostnames de las VMs individuales que corren dentro de cada uno (ver `docs/pihole`, `docs/monitoring`, etc. para el detalle de qué VM vive en qué nodo).
