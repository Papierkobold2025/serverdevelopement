# Semaphore

Semaphore es una interfaz que permite automatizar tareas repetitivas a traves de varias VMs/Hipervisores.

La meta es poder actualizar, instalar y crear entornos de una manera mas documentada, controlada y eficiente entre varias maquinas.

##Actualizacion de nodos

- Primer proyecto creado dentro del entorno de Semaphore por medio de un Playbooks.

- Ansible playbook vive en `semaphore/playbooks/ansible/nodes-update.yml`.

- Corre via Semaphore (Task Template "Nodes-Update", inventario "Nodes"),

- Esta programado para ejecutarse los domingos a las 2:30 AM hora Europe/Zurich.

## Decision: update + reboot en un solo playbook

- Evalue si separar la actualizacion del reinicio condicionado para detectar si algo se rompe (ej. Wireguard tras reinicio ya paso antes).

- Se descarto porque ninguna alternativa cambiaba el tiempo real de deteccion.

## Decision: Separacion de claves y contraseñas de la configuracion del compose.yaml

- Evaluacion del peligro de dejar contraseñas dentro del contexto del compose.yaml al hacer el push a repositorio publico de github

## Creacion de primer codigo de Teraform

- Configuracion del primer codigo de Terraform dentro de Semaphore

- Configuracion para crear contenedores en vez de VMs

### Dificultades encontradas

- Al haber varias partes que tienen que ir en conjunto la configuracion se dificulto un poco

  - Por otra parte tiene que haber conexion ssh con todas las maquinas que tienen que ser automatizadas

  - Semaphore no puede crear conexion con nodos externos si la clave SSH tiene un Passphrase, parece ser una limitacion del entorno
  
    - Solucion encontrada: Eliminacion de Passphrase para clave ssh

  - Migracion de entornos yaml puede traer dificultades por la encriptacion que Semaphore hace a los datos
  
    - Solucion encontrada: Reingresar claves ssh dentro de la GUI de Semaphore para que volviera a cifrarlas

  - Terraform dentro del contexto de Semaphore no puede leer archivos configurados dentro del servidor donde esta el docker
    
    - Solucion encontrada: Creacion de secretos dentro del entorno de Semaphore y referencia a variable dentro de main.tf 

  - Terraform necesita la configuracion de quitar privilege separation dentro de Proxmox porque si no no tiene derecho de crear contenedor

  - Terraform por defecto guarda su state dentro del directorio /tmp/ del contenedor, por lo que no persiste sesiones

    - Solucion encontrada: Referenciar directorio explicito en main.tf para guardar el state de forma persistente
