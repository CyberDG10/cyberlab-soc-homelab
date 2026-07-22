# 05 — Détection & SOC

> Collecte de logs et SIEM. **Cœur du projet (focus Blue Team).**

## Objectif
Déployer la chaîne de détection **avant** toute activité offensive, afin de disposer
d'une visibilité complète lors des scénarios d'attaque.

## Composants prévus
- **Sysmon** sur les postes/serveurs Windows, avec une configuration reconnue
  (base type SwiftOnSecurity / Olaf Hartong), versionnée dans `../detections/`.
- **Collecte** des journaux Windows (Sécurité, Système, Sysmon).
- **Wazuh** (manager sur `SRV-WAZUH01`, agents sur les hôtes) : centralisation,
  corrélation, alerting.

## Livrables
- Configuration Sysmon commentée.
- Agents Wazuh déployés et remontant les logs.
- Tableau de bord de supervision fonctionnel.

_Section à compléter._
