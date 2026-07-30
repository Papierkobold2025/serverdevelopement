# Semaphore

- Semaphore es una interfaz que permite automatizar tareas repetitivas a traves de varias VMs/Hipervisores.

- Herramienta integrada para poder automatizar de mejor manera el flujo de creación de la infraestructura

## Decisiones

- Utilización específica de Ansible en vez de usar scripts de bach que viene incluído con las herramientas de Semaphore, debido a falta de automatización completa

### Runnbook

- Instalación de docker compose de semaphore

``` bash
services:
  semaphore:
    ports:
      - 192.168.X.X:3000:3000
    image: semaphoreui/semaphore:v2.15.0
    environment:
      SEMAPHORE_DB_DIALECT: '${DB_DIALECT}'
      SEMAPHORE_ADMIN_PASSWORD: '${PASSWORD}'
      SEMAPHORE_ADMIN_NAME: '${ADMIN}'
      SEMAPHORE_ADMIN_EMAIL: '${EMAIL}'
      SEMAPHORE_ADMIN: '${ADMIN}'
      SEMAPHORE_SCHEDULE_TIMEZONE: '${TZ}'
      TZ: '${TZ}'
      SEMAPHORE_ACCESS_KEY_ENCRIPTION: '${ENCRIPTION}'
    volumes:
      - semaphore-data:/var/lib/semaphore
volumes:
  semaphore-data:

```

## Comandos útiles

- Al conectarse al contenedor en el que corre Semaphore, lo que Terraform genera no es accesible desde la consola misma de Semaphorm. Para entrar al entorno de terraform mismo: ```bash docker exec -it semaphore-semaphore-1 bash ```

- Una vez dentro del contenedor de Terraform cada playbook nuevo de Terraform genera un nuevo template, en este caso ni project ni repository cambian de mundo mientras el peoyecto no cambie, pero el template siempre cambia ```bash cd /tmp/semaphore/project_3/repository_1_template_6/terraform/landingpage ```

- Dentro del proyecto correcto despues se puede sacar el output de terraform para ese state con: ```bash terraform output -json``` o cualquier variable dentro de terraform. 

### Dificultades encontradas

- Docker Compose nombra volúmenes según el nombre de carpeta (project name) — renombrar la carpeta (ej. semaphore → semaphore-viejo) crea un volumen NUEVO vacío en vez de reutilizar el existente. Para forzar el volumen correcto sin importar el nombre de carpeta: docker compose -p semaphore up -d

- Typo en SEMAPHORE_ACCESS_KEY_ENCRYPTION (o falta total de esta variable) no truena el contenedor — genera una clave nueva silenciosamente en cada arranque, dejando ilegibles los secretos cifrados con la clave anterior (SSH keys, Variable Groups). Los Task Templates/Inventarios no van cifrados, así que siguen siendo visibles aunque los secretos ya no se puedan descifrar.

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
