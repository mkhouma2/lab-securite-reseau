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

---
---

## Etape 3 — Architecture complète + Règles Firewall

### Ce qui a été fait
- Installation Ubuntu Server 24.04 sur VMnet3 (DMZ)
- IP obtenue via DHCP pfSense : 192.168.20.100
- OpenSSH installé sur Ubuntu
- Règle firewall pfSense : LAN → DMZ autorisé (Protocol Any)
- Test ping Kali → Ubuntu : OK
- Connexion SSH Kali → Ubuntu : OK

### Architecture finale
- WAN  : 172.16.252.138/24 (NAT VMware)
- LAN  : 192.168.10.0/24 — Kali (192.168.10.100)
- DMZ  : 192.168.20.0/24 — Ubuntu (192.168.20.100)

### Prochaine étape
- Affiner les règles firewall (bloquer DMZ → LAN)
- Installer Apache sur Ubuntu (serveur web en DMZ)
- Tunnel VPN IPSec
- IDS Suricata

---
---

## Etape 4 — Serveur web DMZ + NAT

### Ce qui a été fait
- Activation NAT Outbound pfSense pour DMZ (Hybrid mode)
- Accès internet Ubuntu DMZ → installation Apache2
- Page web Apache accessible depuis Kali : http://192.168.20.100
- Architecture firewall finale :
  - LAN → DMZ : Pass (Any)
  - DMZ → LAN : Block
  - DMZ → WAN : Pass + NAT

### Architecture complète validée
- Kali ping Ubuntu : OK
- Kali SSH Ubuntu : OK  
- Kali HTTP Ubuntu (Apache) : OK
- Ubuntu ping Kali : BLOQUÉ ✅