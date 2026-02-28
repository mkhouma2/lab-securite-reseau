# Lab Sécurité Réseau — Architecture pfSense & DMZ

## Objectif
Construire un lab de sécurité réseau complet simulant une 
architecture d'entreprise avec segmentation réseau, firewall, 
IDS/IPS et tunnel VPN.

## Architecture

![image](docs/image/archi.png)

## Réseaux

| Zone | Interface | Subnet           | Rôle                  |
|------|-----------|------------------|-----------------------|
| WAN  | em0       | 172.16.252.0/24  | Accès internet (NAT)  |
| LAN  | em1       | 192.168.10.0/24  | Réseau interne        |
| DMZ  | em2       | 192.168.20.0/24  | Serveurs exposés      |

## Stack technique
- **Hyperviseur** : VMware Fusion (macOS Intel)
- **Firewall** : pfSense CE 2.7.2
- **Attaquant** : Kali Linux
- **Serveur DMZ** : Ubuntu Server 22.04
- **Virtualisation réseau** : GNS3

## Progression

- [x] Installation pfSense 2.7.2
- [x] Configuration WAN / LAN / DMZ
- [ ] Connexion VMs Kali et Ubuntu
- [ ] Règles firewall LAN → DMZ
- [ ] Tunnel VPN IPSec
- [ ] IDS Suricata
- [ ] Tests d'intrusion depuis Kali