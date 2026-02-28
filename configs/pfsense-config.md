# Configuration pfSense

## Statut : En cours

### Interfaces
| Interface | Nom  | IP              | Rôle          |
|-----------|------|-----------------|---------------|
| em0       | WAN  | 172.16.252.138  | NAT VMware    |
| em1       | LAN  | 192.168.10.1/24 | Réseau interne|
| em2       | DMZ  | 192.168.20.1/24 | Serveurs DMZ  |

### À documenter
- [ ] Règles firewall LAN → DMZ
- [ ] Règles firewall WAN → DMZ
- [ ] Configuration VPN IPSec
- [ ] Alias et objets réseau