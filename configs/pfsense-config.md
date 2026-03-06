# Configuration pfSense — Lab Sécurité Réseau

## Statut : En cours
**Version pfSense :** CE 2.7.2-RELEASE (amd64)
**Hyperviseur :** VMware Fusion — macOS Intel

---

## 1. Interfaces Réseau

| Interface | Nom  | IP              | VMnet  | Rôle                        |
|-----------|------|-----------------|--------|-----------------------------|
| em0       | WAN  | 172.16.252.138/24 | NAT  | Accès internet (NAT VMware) |
| em1       | LAN  | 192.168.10.1/24 | VMnet2 | Réseau interne (attaquant)  |
| em2       | OPT1 | 192.168.20.1/24 | VMnet3 | DMZ (serveurs exposés)      |

**DHCP activé sur :**
- LAN : 192.168.10.100 → 192.168.10.200
- DMZ : 192.168.20.100 → 192.168.20.200

---

## 2. Machines du Lab

| Machine       | Zone | IP               | OS                    | Rôle              |
|---------------|------|------------------|-----------------------|-------------------|
| pfSense       | —    | 192.168.10.1 (LAN) / 192.168.20.1 (DMZ) | FreeBSD | Firewall/Routeur |
| Kali Linux    | LAN  | 192.168.10.100   | Kali Linux            | Attaquant         |
| Ubuntu Server | DMZ  | 192.168.20.100   | Ubuntu Server 24.04   | Serveur cible (Apache) |

---

## 3. Règles Firewall

### 3.1 Interface LAN (em1)

| # | Action | Protocol | Source      | Destination      | Description           |
|---|--------|----------|-------------|------------------|-----------------------|
| 1 | Pass   | Any      | LAN subnet  | 192.168.20.0/24  | LAN vers DMZ          |
| 2 | Pass   | Any      | LAN subnet  | Any              | LAN vers Internet     |

**Logique :** Le LAN (Kali) peut accéder à la DMZ et à internet.
Les connexions non explicitement autorisées sont bloquées par la règle implicite de pfSense.

### 3.2 Interface DMZ / OPT1 (em2)

| # | Action | Protocol | Source      | Destination      | Description           |
|---|--------|----------|-------------|------------------|-----------------------|
| 1 | Pass   | Any      | OPT1 subnet | Any              | DMZ vers Internet     |
| 2 | Block  | Any      | OPT1 subnet | LAN subnet       | Bloquer DMZ vers LAN  |

**Logique de sécurité :**
- La DMZ peut accéder à internet (pour mises à jour, apt)
- La DMZ **ne peut pas** attaquer le réseau interne (LAN)
- C'est le principe fondamental d'une DMZ : isoler les serveurs exposés du réseau interne
- L'ordre des règles est critique : Pass internet AVANT Block LAN

### 3.3 Interface WAN (em0)
Aucune règle entrante configurée — pfSense bloque tout le trafic entrant non sollicité par défaut.

---

## 4. NAT Outbound (Sortant)

**Mode :** Hybrid Outbound NAT (Automatic + règles manuelles)

| Interface | Source           | Destination | NAT Address | Description       |
|-----------|------------------|-------------|-------------|-------------------|
| WAN       | 192.168.20.0/24  | Any         | WAN address | NAT DMZ vers WAN  |

**Pourquoi Hybrid ?**
Le mode Automatic gère déjà le NAT du LAN. On ajoute manuellement la règle DMZ
sans toucher aux règles automatiques existantes.

**Règles automatiques générées par pfSense :**
- LAN (192.168.10.0/24) → WAN : NAT automatique
- DMZ (192.168.20.0/24) → WAN : NAT manuel ajouté

---

## 5. Services Configurés

### DNS
- Serveurs DNS WAN : 8.8.8.8 / 8.8.4.4 (Google)
- DNS Resolver activé sur pfSense (port 53)
- Les clients DHCP reçoivent pfSense comme résolveur DNS

### DHCP Server
- LAN : actif, plage 192.168.10.100-200
- DMZ : actif, plage 192.168.20.100-200

### WebConfigurator
- Protocole : HTTP (lab uniquement)
- Port : 80
- Accessible depuis : LAN uniquement (192.168.10.0/24)

---

## 6. Tests de Connectivité Validés

| Source | Destination      | Protocole | Résultat  | Attendu |
|--------|------------------|-----------|-----------|---------|
| Kali   | 192.168.10.1     | ICMP      | ✅ OK     | Oui     |
| Kali   | 192.168.20.100   | ICMP      | ✅ OK     | Oui     |
| Kali   | 192.168.20.100   | SSH (22)  | ✅ OK     | Oui     |
| Kali   | 192.168.20.100   | HTTP (80) | ✅ OK     | Oui     |
| Ubuntu | 192.168.10.100   | ICMP      | ❌ Bloqué | Non     |
| Ubuntu | 8.8.8.8          | ICMP      | ✅ OK     | Oui     |
| Kali   | 8.8.8.8          | ICMP      | ✅ OK     | Oui     |

---

## 7. À Configurer (Roadmap)

- [ ] VPN IPSec site-à-site
- [ ] IDS Suricata sur interface LAN/DMZ
- [ ] Règles firewall WAN → DMZ (port forwarding HTTP)
- [ ] Durcissement pfSense (désactiver services inutiles)
- [ ] Alias et groupes d'objets réseau
- [ ] Logs et monitoring

---

## 8. Choix Techniques et Leçons Apprises

### Pourquoi pfSense CE 2.7.2 et non 2.8.x ?
Les versions 2.8.x sont récentes et les dépôts n'étaient pas stables au moment de l'installation.
2.7.2 est la dernière version LTS stable avec une documentation abondante.

### Pourquoi UFS et non ZFS ?
ZFS nécessite un accès aux dépôts pendant l'installation sur cette version.
UFS s'installe entièrement depuis l'ISO — plus fiable en environnement lab sans internet stable.

### Pourquoi Hybrid NAT et non Automatic ?
Le mode Automatic ne couvre que les interfaces avec une gateway configurée.
La DMZ étant une interface OPT sans gateway dédiée, elle nécessite une règle NAT manuelle.
Hybrid permet d'ajouter cette règle sans casser le NAT automatique du LAN.

### Ordre des règles firewall DMZ
pfSense évalue les règles de haut en bas et s'arrête à la première correspondance.
La règle "Pass DMZ → Internet" doit être AVANT "Block DMZ → LAN",
sinon le trafic vers internet serait bloqué par la règle Block (qui matche "Any destination").