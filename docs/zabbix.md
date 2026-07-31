# Zabbix

- Instalación de monitoreo de nodos, VMs y contenedores de entornos tipo Proxmox

## Decisiones

- Reemplazo de Grafana/prometheus por alcance de escaneo de red nativo

- Integración de Zabbix dentro del cluster k3s

- PostgreSQL con persistencia habilitada, instalado como sub-chart dentro del mismo release de Helm

### Dificultades encontradas

- Dificultad de configuración de Dashboards e importación de Templates

# Runbook

- Instalación de Helm y clonación de repositorio oficial de Zabbix

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
git clone https://github.com/zabbix-community/helm-zabbix.git
```

- Configuración de archivo .yaml de zabbix

```bash
zabbixServer.service.type: ClusterIP
postgresql.persistence.enabled: true # Valor true viene configurado por defecto
```

- Instalación de cliente Zabbix

```bash
helm install zabbix ./charts/zabbix --dependency-update -f $HOME/zabbix_values.yaml -n monitoring
```

- Acceso al frontend: Service `zabbix-zabbix-web`, NodePort puerto 31080 (interno 80)

```bash
kubectl get svc zabbix-zabbix-web -n monitoring
```
