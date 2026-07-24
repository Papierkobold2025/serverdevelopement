# Ansible 

Ansible es una interfaz que permite automatizar tareas repetitivas a traves de varias VMs/Hipervisores.

La meta es poder actualizar, instalar y crear entornos de una manera mas documentada, controlada y eficiente entre varias maquinas.

##Actualizacion de nodos

- Primer proyecto creado dentro del entorno de Ansible por medio de un Playbooks.

- Ansible playbook vive en `semaphore/playbooks/ansible/nodes-update.yml`.

- Corre via Semaphore (Task Template "Nodes-Update", inventario "Nodes"),

- Esta programado para ejecutarse los domingos a las 2:30 AM hora Europe/Zurich.

## Decision: update + reboot en un solo playbook

- Evalue si separar la actualizacion del reinicio condicionado para detectar si algo se rompe (ej. Wireguard tras reinicio ya paso antes).

- Se descarto porque ninguna alternativa cambiaba el tiempo real de deteccion.

## Decision: Separacion de claves y contraseñas de la configuracion del compose.yaml

- Evaluacion del peligro de dejar contraseñas dentro del contexto del compose.yaml al hacer el push a repositorio publico de github

### Dificultades encontradas

- Al haber varias partes que tienen que ir en conjunto la configuracion se dificulto un poco

  - Por una parte se tiene que crear una conexion con un entorno externo en este caso github, donde se guarda el playbook

  - Por otra parte tiene que haber conexion ssh con todas las maquinas que tienen que ser automatizadas

  - Ansible no puede crear conexion con nodos externos si la clave SSH tiene un Passphrase, parece ser una limitacion del entorno
  
    - Solucion encontrada: Eliminacion de Passphrase para clave ssh

  - Migracion de entornos yaml puede traer dificultades por la encriptacion que ansible hace a los datos
  
    - Solucion encontrada: Reingresar claves ssh dentro de la GUI de ansible para que volviera a cifrarlas

