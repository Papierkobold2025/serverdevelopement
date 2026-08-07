# K3s

- Configuración de K3s para empezar a aprender kubernetes

## Decisiones

- Aislamiento de kubernetes dentro de un nuevo hipervisor especializado

- Instalación K3s sobre una VM de Proxmox porque K3s tiene como requisito acceso a módulos de Kernel y manejo directo de iptables.

- Usé hostPath (apunta directo a una carpeta del disco del nodo) en vez de PVC (donde Kubernetes decide y gestiona el almacenamiento por su cuenta) — es el mismo patrón que ya conocía de Docker Compose, más fácil de entender para un primer manifiesto. Revisar PVC si más adelante quiero que Kubernetes administre el disco en vez de mí

## Dificultades encontradas

- kubectl resultó ser el mismo binario de k3s (symlink), y por diseño busca su config en /etc/rancher/k3s/k3s.yaml (solo lectura root), ignorando la copia que hice en ~/.kube/config

  - Solución: forzar la ruta correcta con la variable KUBECONFIG, agregada a ~/.bashrc para que sea permanente

## Runbook

- Instalación de servidor k3s

```bash
curl -sfL https://get.k3s.io | sh -
```

## Network Policies

- Aislamiento de trafico entre pods dentro de k3s, y de pods hacia red interna.

### Decisiones

- Patrón de aislamiento deny-all por namespace y apertura de tráfico aislado mediante allow.

- Las reglas actuales permiten tráfico desde/hacia el rango completo de la red plana (`192.168.1.0/24`) para Portainer y su agente, en vez de restringir a IPs específicas — mismo enfoque de aceptación de riesgo inicial que en la segmentación de red general (ver [network.md](network.md)), pendiente de endurecer (ver Roadmap: "Configuración de firewall en K3s")

### Dificultades encontradas

- Dos Network Policies no pueden tener el mismo nombre en el mismo namespace, la segunda siempre sobreescribirá la primer regla.

- ipBlock interno apuntando a rango de IP de un Service 10.43.X.X nunca funciona por como k3s enruta tráfico (Necesita IPs específicas, no rangos)

### Nota

- `portainer-agent` corre en su propio namespace (`portainer`), creado automáticamente al desplegar el agente — no es parte del árbol `automation` a nivel de Kubernetes, solo a nivel organizacional del repositorio.

### Workloads

| Servicio | Manifiesto / Instalación |
|---|---|
| Portainer | [portainer.yaml](../k3s/manifests/deployment/automation/portainer/portainer.yaml) |
| Semaphore | [semaphore.yaml](../k3s/manifests/deployment/automation/semaphore/semaphore.yaml) |
| Zabbix | [docs/zabbix.md](../docs/zabbix.md) (Helm chart) |
| Homarr | [homarr.yaml](../k3s/manifests/deployment/monitoring/homarr/homarr.yaml) |

### Network Policy Rules

**Capa general** — baseline deny-all (Ingress/Egress) y reglas DNS/allow compartidas por namespace:

- [general.yaml](../k3s/manifests/network-policies/general/general.yaml)

**Reglas específicas por servicio:**

| Servicio | Namespace(s) | Archivo |
|---|---|---|
| Portainer + Portainer-Agent | automation, portainer | [portainer-network.yaml](../k3s/manifests/network-policies/namespaces/automation/portainer/portainer-network.yaml) |
| Semaphore | automation | [semaphore-network.yaml](../k3s/manifests/network-policies/namespaces/automation/semaphore/semaphore-network.yaml) |
| Zabbix | monitoring | [zabbix-network.yaml](../k3s/manifests/network-policies/namespaces/monitoring/zabbix/zabbix-network.yaml) |
| Homarr | monitoring | [homarr-network.yaml](../k3s/manifests/network-policies/namespaces/monitoring/homarr/homarr-network.yaml) |