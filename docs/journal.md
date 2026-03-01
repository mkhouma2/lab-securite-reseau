# Journal de bord

##  Etape 1 - Installation pfSense


### Ce qui a été fait
- Téléchargement ISO pfSense CE 2.7.2 (874 Mo)
- Création VM VMware Fusion : 2 CPU, 1 Go RAM, 16 Go disque
- Ajout de 3 interfaces réseau : NAT (WAN), VMnet2 (LAN), VMnet3 (DMZ)
- Installation pfSense avec partitionnement UFS/GPT
- Configuration des interfaces :
  - WAN em0 : 172.16.252.138/24 (DHCP NAT VMware)
  - LAN em1 : 192.168.10.1/24 (DHCP 100-200)
  - DMZ em2 : 192.168.20.1/24 (DHCP 100-200)

### Difficultés rencontrées
- Première tentative ZFS échouée : dépôts 2.7.2 inaccessibles
- Solution : ISO complet 874 Mo + partitionnement UFS/GPT

### Prochaine étape
- Importer Kali Linux sur VMnet2
- Importer Ubuntu Server sur VMnet3
- Accéder à l'interface web pfSense depuis Kali

---
---

## Etape 2 — Connexion Kali & Accès WebConfigurator

### Ce qui a été fait
- Import VM Kali Linux dans VMware Fusion
- Connexion Kali sur VMnet2 (LAN 192.168.10.0/24)
- IP obtenue via DHCP pfSense : 192.168.10.100
- Accès interface web pfSense : http://192.168.10.1
- Setup Wizard complété : hostname, DNS, password admin

### Difficultés rencontrées
- Interfaces VMware mal assignées sur pfSense → corrigé
- WebConfigurator inactif → résolu par redémarrage pfSense

### Prochaine étape
- Installer Ubuntu Server sur VMnet3 (DMZ)
- Configurer règles firewall LAN → DMZ