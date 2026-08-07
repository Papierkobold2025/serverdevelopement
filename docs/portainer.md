# Portainer

- Herramienta de gestión visual para clusters de contenedores (Docker/Kubernetes).

## Decisiones

- Autenticación vía Keycloak: cliente dedicado con tokens independientes para Netbird y Portainer (heredado de la instalación anterior, sigue vigente)

## Estado actual

- Gestiona actualmente el cluster de K3s. La instalación anterior (Docker standalone) está documentada en [Archive/Services/portainer.md](../Archive/Services/portainer.md) como referencia histórica.

## Playbook

- [manifiesto/config actual de Portainer sobre K3s]