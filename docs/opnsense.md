# Segmentación de red

- Segmentación de red para mantener servicios críticos aislados mediante VLANs

- Aislamiento y reglas en OPNSense aún en progreso

## Terminología (importante)

- **VLAN**: separación lógica en Capa 2 (dominio de broadcast). Una VLAN permite segregar tráfico en el switch y controlar dominios de difusión.
- **Subred**: segmentación en Capa 3 (rango de direcciones IP). Una subred puede asignarse a una VLAN, pero no son lo mismo.
- En este proyecto uso VLANs para aislar servicios a nivel de capa 2 y asigno subredes IP por VLAN. Las políticas de OPNSense (ruteo/Firewall) operan a nivel L3 y pueden aplicar NAT o reglas entre subredes/VLANs según sea necesario.

## Estado actual

- Creación de VLAN para poder aislar servicios críticos y evitar movimientos laterales a nivel de red en caso de compromiso

- Servicios seleccionados para migración son servicios de infraestructura y no servicios de usuario final

- Migración de servicios a VLAN: 
  - Vaultwarden
  - Keycloak
  - Kubernetes
  - Nginx
  - Nextcloud

- Pi-hole no fue migrado debido a que actúa como servidor DHCP para la red plana y debe mantener alcance sobre todos los clientes

- Netbird no fue migrado porque el orquestador para acceso fuera de la red local es más accesible desde la red plana

- OPNSense naturalmente tampoco fue migrado debido a que es el punto de comunicación entre VLAN y red plana

## Próximos pasos

- Creación de reglas restrictivas a nivel de OPNSense de deny-by-default para evitar tráfico libre entre servicios críticos, VLAN y red plana

- Creación de un servidor DHCP mínimo para permitir conexión con dispositivos desde red plana para debug

## Decisiones

- Segmentación de red solamente con una VLAN, para aislar servicios críticos sin segmentar a nivel de servicios

- Decidí colocar el reverse proxy dentro de la VLAN; las consultas DNS internas pasan por Pi-hole y se reenvían a NPM, mientras OPNSense actúa como puente para devolver la resolución al cliente

- Servicios internos que no tienen acceso a red externa se metieron a la VLAN, dejando solamente los servicios de usuario final en la red plana para evitar movimientos laterales

- Aceptación de riesgo inicial al no tener ninguna regla explícita de flujo de red, manejando un enfoque de ir haciéndolo más prohibitivo en vez de cerrar todo de principio e ir abriendo después

- Decidí usar los registros DNS locales en Pi-hole por encima de reenvíos condicionales, ya que mantengo esa lista siempre actualizada. Al crear la VLAN, bastó con apuntar los registros existentes a la nueva IP de NPM para que la resolución de dominios siguiera funcionando sin necesidad de configurar reenvío condicional.

### Dificultades encontradas

- VLAN aún con tráfico abierto sin reglas estrictas de Firewall; actualmente uso NAT/PUENTE en OPNSense para permitir comunicación controlada con la red plana cuando es necesario



