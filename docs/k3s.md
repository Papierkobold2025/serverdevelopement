# K3s

- Configuración de K3s para empezar a aprender kubernetes

## Decisiones

- Aislamiento de kubernetes dentro de un nuevo hipervisor especializado

- Instalación K3s sobre una VM de Proxmox porque K3s tiene como requisito acceso a módulos de Kerne y manejo directo de iptables.

- Usé hostPath (apunta directo a una carpeta del disco del nodo) en vez de PVC (donde Kubernetes decide y gestiona el almacenamiento por su cuenta) —es el mismo patrón que ya conocía de Docker Compose, más fácil de entender para un primer manifiesto. Revisar PVC si más adelante quiero que Kubernetes administre el disco en vez de mí

## Dificultades

- kubectl resultó ser el mismo binario de k3s (symlink), y por diseño busca su config en /etc/rancher/k3s/k3s.yaml (solo lectura root), ignorando la copia que hice en ~/.kube/config
  
  - Solución: forzar la ruta correcta con la variable KUBECONFIG,agregada a ~/.bashrc para que sea permanente

### Runbook

- Instalación de servidor k3s

```bash
curl -sfL https://get.k3s.io | sh -
```

- Los manifiestos específicos de los servicios migrados estan en [K3s](../k3s)
