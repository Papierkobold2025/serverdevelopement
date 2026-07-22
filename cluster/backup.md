# Backups configuration

Integración de Proxmox Backup Server (PBS) como quinto nodo dedicado del clúster, para respaldo automático centralizado de todas las VMs — separado en su propio documento ya que respalda todo el clúster, no solo Nextcloud.

## Meta a alcanzar a primer instancia:
- Nodo dedicado corriendo PBS, con conexión directa desde los demás nodos del clúster.
- Configuración de Backup Job centralizado para todas las VMs.
- Configuración de retención y verificación de integridad.
- Notificaciones de éxito/falla por correo.
- Dificultades encontradas en el desarrollo inicial.

## Especificaciones del nodo (pbs-homelab)
- CPU(s): 8 x Intel Core i7-8559U @ 2.70GHz (1 Socket)
- RAM total: 62.67 GiB
- Almacenamiento (root): 956.93 GB
- Kernel: 6.17.2-1-pve
- Producto: Proxmox Backup Server (no Proxmox VE)

## Configuración del Backup Job

| Campo | Valor |
|---|---|
| Storage | pbs-homelab |
| Nodos incluidos | Todos (-- All --) |
| Horario | 2:30 AM |
| Modo | Snapshot |
| Compresión | ZSTD (fast and good) |
| Selección | Todas las VMs |

## Retención

| Tipo | Cantidad |
|---|---|
| Daily | 7 |
| Weekly | 4 |

- Cobertura aproximada: último mes completo, sin acumulación indefinida.

## Verificación e integridad
- Verificación (Verify) configurada de forma inmediata tras cada backup, no en horario separado.
- Restauración probada exitosamente en al menos una ocasión (VM de prueba).

## Notificaciones
- Notificaciones de éxito/falla del job configuradas por correo, reutilizando el mismo SMTP configurado para Nextcloud (Gmail, puerto 587, Application Password — ver `docs/nextcloud`).
