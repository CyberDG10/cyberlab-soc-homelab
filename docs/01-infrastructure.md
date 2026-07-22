# 01 — Infrastructure & Réseau

> Hyperviseur, réseaux virtuels, pare-feu pfSense.
> Voir [`00-architecture.md`](00-architecture.md) pour le plan d'adressage cible.

## Hyperviseur
- VMware Workstation Pro.
- Réseaux : VMnet8 (NAT / Internet), VMnet2 (Host-Only / LAN du lab).
- Serveur DHCP VMware désactivé (le DHCP sera fourni par le DC).

## pfSense (à déployer — Phase 1)
- Rôle : routeur, pare-feu, segmentation, source de logs.
- Interfaces : WAN sur VMnet8 (NAT), LAN sur VMnet2 (192.168.209.1).
- Objectif : fournir un accès Internet maîtrisé et un point de filtrage.

_Section à compléter au déploiement._
