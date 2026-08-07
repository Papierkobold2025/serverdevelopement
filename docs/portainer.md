# Portainer

- Visual management tool for container clusters (Docker/Kubernetes).

## Decisions

- Authentication through Keycloak: dedicated client with independent tokens for Portainer (inherited from the previous installation and still in use).

## Current status

- It currently manages the K3s cluster. The previous installation (Docker standalone) is documented in [Archive/Services/portainer.md](../Archive/Services/portainer.md) as a historical reference.

## Playbook

- [portainer.yaml](../k3s/manifests/deployment/automation/portainer/portainer.yaml)