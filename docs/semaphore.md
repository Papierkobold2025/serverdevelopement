# Semaphore

- Semaphore es una interfaz que permite automatizar tareas repetitivas a través de varias VMs/Hipervisores.

- Herramienta integrada para poder automatizar de mejor manera el flujo de creación de la infraestructura

## Decisiones

- Utilización específica de Ansible en vez de usar scripts de bash que viene incluído con las herramientas de Semaphore, debido a falta de automatización completa

- Migración de Semaphore a K3s para mantener servicios de infraestructura en la misma instancia

## Runbook

- Configuración de entorno se encuentra en [K3s](../k3s/manifests/deployment/automation/semaphore.yaml)

## Comandos útiles

- Acceso al entorno de ejecución:
```bash
  kubectl exec -it <nombre-del-pod> -n <namespace> -- bash
```

## Dificultades encontradas

- Typo en SEMAPHORE_ACCESS_KEY_ENCRYPTION (o falta total de esta variable) no truena el contenedor — genera una clave nueva silenciosamente en cada arranque, dejando ilegibles los secretos cifrados con la clave anterior (SSH keys, Variable Groups). Los Task Templates/Inventarios no van cifrados, así que siguen siendo visibles aunque los secretos ya no se puedan descifrar.

- Al haber varias partes que tienen que ir en conjunto la configuración se dificultó un poco

  - Tiene que haber conexión ssh con todas las máquinas que tienen que ser automatizadas

  - Semaphore no puede crear conexión con nodos externos si la clave SSH tiene un Passphrase, parece ser una limitación del entorno

    - Solución encontrada: eliminación de Passphrase para clave ssh

  - Migración de entornos yaml puede traer dificultades por la encriptación que Semaphore hace a los datos

    - Solución encontrada: reingresar claves ssh dentro de la GUI de Semaphore para que volviera a cifrarlas

## Ansible

- Escogí Ansible como solución para poder automatizar tareas repetitivas dentro de contenedores, VMs y nodos de proxmox

- Ansible actualmente lo utilizo para de manera automatizada actualizar *apt* de los contenedores, VMs y nodos y la instalación automatizada de Docker Compose

- Los playbooks de ansible se encuentran en [ansible](../semaphore/ansible/playbooks/)

## Terraform

- Escogí Terraform para poder levantar de manera automatizada VMs y contenedores con ciertas especificaciones como CPU, RAM y almacenamiento

- Los manifiestos de Terraform se encuentran en [terraform](../semaphore/terraform/)

### Comandos útiles

- Una vez dentro del contenedor de Terraform cada playbook nuevo genera un nuevo template; ni project ni repository cambian mientras el proyecto no cambie, pero el template siempre cambia:
```bash
  cd /tmp/semaphore/project_3/repository_1_template_6/terraform/landingpage
```

- Para sacar el output de Terraform de ese state: `terraform output -json` (o cualquier variable dentro de terraform)

### Dificultades encontradas

- No puede leer archivos configurados dentro del servidor donde está el docker

  - Solución encontrada: creación de secretos dentro del entorno de Semaphore y referencia a variable dentro de main.tf

- Necesita quitar privilege separation dentro de Proxmox, si no, no tiene derecho de crear contenedor

- Por defecto guarda su state dentro del directorio /tmp/ del contenedor, por lo que no persiste sesiones

  - Solución encontrada: referenciar directorio explícito en main.tf para guardar el state de forma persistente