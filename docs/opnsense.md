# Network segmentation

- Network segmentation to keep critical services isolated through VLANs.

- Isolation and rules in OPNsense are still in progress.

## Terminology (important)

- **VLAN**: logical separation at Layer 2 (broadcast domain). A VLAN allows traffic to be segmented on the switch and broadcast domains to be controlled.
- **Subnet**: segmentation at Layer 3 (range of IP addresses). A subnet can be assigned to a VLAN, but they are not the same.
- In this project, I use VLANs to isolate services at Layer 2 and assign IP subnets per VLAN. OPNsense policies (routing/firewall) operate at Layer 3 and can apply NAT or rules between subnets/VLANs as needed.

## Current status

- Creation of VLANs to isolate critical services and avoid lateral movement at the network level in case of compromise.

- The selected services for migration are infrastructure services, not end-user services.

- Migration of services to the VLAN:
  - Vaultwarden
  - Keycloak
  - K3s
  - Nginx
  - Nextcloud

- Pi-hole was not migrated because it acts as the DHCP server for the flat network and must keep scope over all clients.

- Netbird was not migrated because the orchestrator for access outside the local network is more accessible from the flat network.

- OPNsense was also not migrated because it is the communication point between the VLAN and the flat network.

## Next steps

- Creation of restrictive rules at the OPNsense level using a deny-by-default approach to prevent free traffic between critical services, VLANs, and the flat network.

- Creation of a minimal DHCP server to allow connection from devices on the flat network for debugging.

## Decisions

- Network segmentation only with a single VLAN, to isolate critical services without segmenting by service.

- I decided to place the reverse proxy inside the VLAN; internal DNS queries pass through Pi-hole and are forwarded to NPM, while OPNsense acts as the bridge to return the resolution to the client.

- Internal services that do not have access to the external network were placed in the VLAN, leaving only end-user services in the flat network to avoid lateral movement.

- Initial risk acceptance due to the absence of any explicit network-flow rules, taking an approach of making it more restrictive over time instead of closing everything at the outset and then opening it up later.

- I decided to use the local DNS records in Pi-hole rather than conditional forwarding, since I keep that list updated. When the VLAN was created, it was enough to point the existing records to the new NPM IP so domain resolution continued to work without needing to configure conditional forwarding.

### Issues encountered

- The VLAN still has open traffic without strict firewall rules; I currently use NAT/bridge mode in OPNsense to allow controlled communication with the flat network when necessary.



