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

- Configuración de primer Manifest para migración de homepage

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: homepage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: homepage
  template:
    metadata:
      labels:
        app: homepage
    spec:
      containers:
        - name: homepage
          image: ghcr.io/gethomepage/homepage:latest
          ports:
            - containerPort: 3000
          env:
            - name: HOMEPAGE_ALLOWED_HOSTS
              value: homepage.apps.midominio.com
          volumeMounts:
            - name: homepage-volume
              mountPath: /app/config
      volumes:
        - name:  homepage-volume
          hostPath:
            path: /srv/homepage/homepage-config
---
apiVersion: v1
kind: Service
metadata:
  name: homepage
spec:
  selector:
    app: homepage
  ports:
    - port: 3001
      targetPort: 3000
  type: NodePort
```

- Importar configuraciones del servicio de Homepage
