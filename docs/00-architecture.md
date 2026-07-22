# 00 — Architecture & Réseau

## Vue d'ensemble

CyberLab reproduit l'infrastructure d'une petite entreprise, segmentée par un pare-feu
pfSense. L'objectif est double : réalisme (une vraie entreprise ne met pas tout à plat sur
un seul réseau) et pertinence SOC (la segmentation crée des points de contrôle et des
sources de logs).

## Schéma réseau (cible)

![Architecture réseau CyberLab](../diagrams/network-diagram.png)

<details>
<summary>Version texte (schéma ASCII)</summary>

```
                          ┌─────────────────────────┐
                          │        INTERNET          │
                          └────────────┬─────────────┘
                                       │  VMnet8 (NAT)
                                 ┌─────┴──────┐
                                 │  pfSense   │  Routeur / Pare-feu
                                 │   -FW      │  WAN : DHCP (VMware NAT)
                                 └─────┬──────┘
                     LAN (VMnet2 – Host-Only) 192.168.209.0/24
                                       │  GW : 192.168.209.1
        ┌──────────────────────────────┼──────────────────────────────┐
        │                              │                              │
  ┌─────┴──────┐              ┌────────┴────────┐            ┌────────┴────────┐
  │  SRV-DC01  │              │  SRV-WAZUH01    │            │    WIN11-01     │
  │ AD/DNS/DHCP│              │  SIEM (Wazuh)   │            │  Poste client   │
  │ .209.10    │              │  .209.20        │            │  .209.30 (DHCP) │
  └────────────┘              └─────────────────┘            └─────────────────┘
                                                              ┌─────────────────┐
                                                              │     KALI-01     │
                                                              │  Poste attaquant│
                                                              │  .209.40 (DHCP) │
                                                              └─────────────────┘
```

</details>


## Plan d'adressage

| Élément | Valeur |
|---------|--------|
| Réseau LAN | `192.168.209.0/24` |
| Passerelle (pfSense LAN) | `192.168.209.1` |
| Contrôleur de domaine `SRV-DC01` | `192.168.209.10` (fixe) |
| SIEM `SRV-WAZUH01` | `192.168.209.20` (fixe) |
| Étendue DHCP (clients) | `192.168.209.30` → `192.168.209.199` |
| Réservations / statique | `192.168.209.200` → `192.168.209.254` |
| Serveur DNS (pour tous les postes) | `192.168.209.10` (le DC) |

## Décisions d'architecture (et pourquoi)

### Pourquoi ajouter pfSense ?
L'infrastructure initiale était en Host-Only **sans passerelle** : aucune machine n'avait
accès à Internet, ce qui bloque Windows Update, `apt`, la mise à jour des outils et le
téléchargement de Wazuh. pfSense résout ce problème **proprement** :
- Il fournit un accès Internet maîtrisé (WAN sur le NAT VMware) pour les mises à jour.
- Il devient le point de segmentation et de filtrage (base pour de futures règles).
- Il constitue lui-même une **source de logs** exploitable par le SIEM (bonus SOC).

### Pourquoi conserver `192.168.209.0/24` ?
On ne ré-adresse pas un contrôleur de domaine sans raison métier : cela impose de
reconfigurer DNS et les enregistrements AD, pour un gain purement cosmétique. Le plan
d'adressage existant est valide ; on documente ce choix plutôt que d'introduire du risque.

### Pourquoi un seul segment LAN pour commencer ?
La segmentation clients/serveurs (VLAN séparés + relais DHCP) est prévue en **phase
ultérieure**, où elle deviendra un chapitre documenté à part entière (« segmentation
réseau »). Commencer simple, puis segmenter de façon justifiée, est plus lisible pour un
recruteur qu'une usine à gaz montée d'un coup.

### Pourquoi `.local` malgré tout ?
`cyberlab.local` est conservé pour la continuité du lab existant. **Limite connue** :
Microsoft déconseille `.local` (conflit potentiel avec le mDNS/Bonjour) ; en production on
utiliserait un sous-domaine d'un domaine routable (ex. `ad.cyberlab.fr`). Ce choix est
assumé et documenté.

## Budget ressources (hôte)

Toutes les VM ne tournent pas simultanément. Prévoir les scénarios :

| Scénario | VM actives | RAM approx. |
|----------|-----------|-------------|
| Admin AD | pfSense + DC01 + WIN11 | ~13 Go |
| Détection | pfSense + DC01 + WAZUH01 + WIN11 | ~17 Go |
| Attaque/défense | pfSense + DC01 + WAZUH01 + WIN11 + KALI | ~19 Go |

> ⚙️ Si l'hôte est limité en RAM, réduire DC01 à 4 Go et WIN11 à 4 Go, et n'allumer Kali
> que pendant les scénarios offensifs.
