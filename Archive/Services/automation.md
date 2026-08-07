# Semaphore

- Semaphore is an interface that allows repetitive tasks to be automated across multiple VMs/hypervisors.

- An integrated tool to automate the infrastructure creation workflow more effectively.

## Decisions

- Specific use of Ansible instead of using bash scripts that come included with Semaphore tools, due to a lack of full automation.

### Runbook

- Installation of the Semaphore Docker Compose setup.

```bash
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

## Useful commands

- When connecting to the container where Semaphore runs, the Terraform-generated environment is not accessible from the Semaphore console itself. To enter the Terraform environment directly: ```bash docker exec -it semaphore-semaphore-1 bash ```

- Once inside the Terraform container, each new Terraform playbook generates a new template; in this case, neither the project nor the repository changes while the project remains the same, but the template always changes: ```bash cd /tmp/semaphore/project_3/repository_1_template_6/terraform/landingpage ```

- From the correct project, the Terraform output for that state can be retrieved with: ```bash terraform output -json``` or any variable inside Terraform.

### Issues encountered

- Docker Compose names volumes according to the folder name (project name) — renaming the folder (for example, semaphore → semaphore-old) creates a NEW empty volume instead of reusing the existing one. To force the correct volume regardless of the folder name: docker compose -p semaphore up -d

- A typo in SEMAPHORE_ACCESS_KEY_ENCRYPTION (or the variable being completely absent) does not crash the container — it silently generates a new key on each startup, making previously encrypted secrets unreadable (SSH keys, Variable Groups). Task Templates/Inventories are not encrypted, so they remain visible even though the secrets can no longer be decrypted.

- Because several parts have to work together, the configuration was a bit difficult.

  - On the other hand, there must be SSH connectivity to all machines that need to be automated.

  - Semaphore cannot create a connection to external nodes if the SSH key has a passphrase, which appears to be a limitation of the environment.

    - Found solution: remove the passphrase from the SSH key.

  - Migrating YAML environments can cause difficulties due to the encryption Semaphore applies to the data.

    - Found solution: re-enter the SSH keys in the Semaphore GUI so that it would encrypt them again.

  - Terraform, in the context of Semaphore, cannot read files configured inside the server where Docker is running.

    - Found solution: create secrets inside the Semaphore environment and reference the variable inside main.tf.

  - Terraform needs the configuration to remove privilege separation inside Proxmox, because otherwise it does not have the right to create a container.

  - By default, Terraform saves its state inside the /tmp/ directory of the container, so sessions do not persist.

    - Found solution: reference an explicit directory in main.tf to save the state persistently.
