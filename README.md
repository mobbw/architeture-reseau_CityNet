# City_Net75 : Architecture réseau multi-sites

Projet partiel **Architecture des réseaux** (T1 : 1A1&2&6&7 – 2026/2027).
Conception et simulation (Cisco Packet Tracer) de l'infrastructure réseau de l'entreprise fictive **City_Net75**, qui étend son siège parisien vers trois villes : Nantes, Strasbourg et Marseille.

## Contexte

City_Net75 est spécialisée dans le développement web, les ERP et les jeux vidéo. Son réseau est aujourd'hui centralisé à Paris (La Défense). L'entreprise souhaite s'étendre à l'ouest, à l'est et au sud tout en garantissant **sécurité, traçabilité et disponibilité** des services.

### Besoins

| Site | Sous-réseaux | Postes |
|---|---|---|
| Paris (siège) | 2 LAN utilisateurs + 1 LAN serveurs | 250 et 300 postes, 200 serveurs |
| Nantes | 2 LAN | 380 et 155 postes |
| Strasbourg | 1 LAN | 222 postes |
| Marseille | 1 LAN | 280 postes |

### Contraintes

- Les sites distants sont reliés au siège par des liaisons **WAN**.
- **Aucun lien direct** entre sites distants : tout transite par Paris (topologie en étoile).
- Adressage privé selon la RFC 1918 pour les LAN et une adresse publique pour la sortie Internet depuis Paris.
- Matériel : switchs Cisco 2960 et routeurs Cisco 2911.

## Architecture

Topologie **en étoile** centrée sur le routeur du siège de Paris.

- Paris : 3 LAN (postes A, postes B, serveurs) + liens WAN vers Nantes, Strasbourg et Marseille + lien vers le FAI.
- Nantes : 1 routeur 2911, 2 LAN (2 switchs 2960).
- Strasbourg : 1 routeur 2911, 1 LAN.
- Marseille : 1 routeur 2911, 1 LAN.
- Un routeur « FAI » simule l'accès à Internet.

> Schéma : voir `schema/` (capture) et le fichier Packet Tracer `.pkt`.

## Plan d'adressage

**Adresse privée :** `172.16.0.0` (classe B, RFC 1918).
**Découpage :** 10 sous-réseaux nécessaires (7 LAN + 3 liens WAN) → emprunt de 4 bits (2⁴ = 16 ≥ 10) → masque **/20** (`255.255.240.0`), soit 4 094 hôtes par sous-réseau. Il reste 6 sous-réseaux disponibles pour l'évolution du réseau.

**Adresse publique :** `203.0.0.0/30` (classe C), liaison point à point Paris ↔ FAI.

### LAN

| Site | Réseau | Passerelle |
|---|---|---|
| Paris – Serveurs | 172.16.0.0/20 | 172.16.0.254 |
| Paris – Postes A | 172.16.16.0/20 | 172.16.16.254 |
| Paris – Postes B | 172.16.32.0/20 | 172.16.32.254 |
| Strasbourg | 172.16.96.0/20 | 172.16.96.254 |
| Nantes – LAN A | 172.16.112.0/20 | 172.16.112.254 |
| Nantes – LAN B | 172.16.144.0/20 | 172.16.144.254 |
| Marseille | 172.16.128.0/20 | 172.16.128.254 |

### Liens WAN

| Liaison | Réseau | Paris | Site distant |
|---|---|---|---|
| Paris ↔ Strasbourg | 172.16.48.0/20 | 172.16.48.1 | 172.16.48.2 |
| Paris ↔ Nantes | 172.16.64.0/20 | 172.16.64.1 | 172.16.64.2 |
| Paris ↔ Marseille | 172.16.80.0/20 | 172.16.80.1 | 172.16.80.2 |
| Paris ↔ FAI | 203.0.0.0/30 | 203.0.0.1 | 203.0.0.2 |

## Équipements et câblage

| Équipement | Modèle | Quantité | Répartition |
|---|---|---|---|
| Routeurs | Cisco 2911 | 4 (+ 1 routeur FAI simulé) | Paris, Nantes, Strasbourg, Marseille |
| Switchs | Cisco 2960-24TT | 7 | Paris 3, Nantes 2, Strasbourg 1, Marseille 1 |
| Postes / serveurs | PC, laptops, Server-PT | – | Selon les LAN |

| Liaison | Câble |
|---|---|
| PC / serveur ↔ switch | Cuivre droit (Cat5e/Cat6) |
| Switch ↔ routeur | Cuivre droit (Cat5e/Cat6) |
| Routeur ↔ routeur (WAN) | Série DCE/DTE |

**Normes de câblage :** paires torsadées en câble droit, Fast Ethernet (100 Mb/s, IEEE 802.3u) et Gigabit Ethernet (1 Gb/s, IEEE 802.3ab), câblage structuré selon TIA/EIA-568.

## Routage

Routage **statique** *(à adapter si vous utilisez OSPF)*.

- Paris : une route statique vers chaque LAN distant via le routeur du site (next hop en `.2` du lien WAN) et une route par défaut vers le FAI (`203.0.0.2`).
- Sites distants : route par défaut vers Paris (next hop en `.1` du lien WAN).

Exemple :

```
Paris(config)# ip route 172.16.96.0 255.255.240.0 172.16.48.2
Strasbourg(config)# ip route 0.0.0.0 0.0.0.0 172.16.48.1
```

## Utilisation

1. Ouvrir le fichier `.pkt` avec Cisco Packet Tracer.
2. Attendre la convergence des liens.
3. Tester la connectivité de bout en bout :
   - `ping` entre PC de sites différents (ex. Nantes ↔ Marseille, via Paris) ;
   - `show ip route` et `show ip interface brief` sur les routeurs.

## Contenu du dépôt

```
.
├── README.md
├── projet.pkt        # simulation Packet Tracer
├── dossier.pdf       # dossier conceptuel
└── schema/           # captures du schéma réseau
```

## Pistes d'amélioration

- VLSM pour limiter le gaspillage d'adresses (notamment /30 sur les liens WAN).
- Redondance (second routeur central, HSRP) pour la haute disponibilité.
- Sécurité : VLANs, ACL, DMZ pour les serveurs, NAT/PAT vers Internet, VPN.

## Auteurs

- Mathias WELMANT
- Endrit SHURDHANI
- Mohamed ANZOUM

Projet réalisé dans le cadre du cursus *Architecture des réseaux*, encadré par Céline OULMI.
