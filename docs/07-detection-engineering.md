# 07 — Detection Engineering

> Le fil rouge du projet : pour chaque attaque, sa détection et sa remédiation.

## Format standard par scénario

| Champ | Contenu |
|-------|---------|
| Technique MITRE ATT&CK | ID + nom (ex. T1558.003 — Kerberoasting) |
| Attaque | Commande / outil utilisé |
| Trace | Event ID / événement Sysmon généré |
| Détection | Règle Wazuh (extrait) + niveau |
| Remédiation | GPO / durcissement appliqué |

## Matrice de couverture (à remplir)

| Scénario | Technique | Détecté | Remédié |
|----------|-----------|:-------:|:-------:|
| LLMNR poisoning | T1557.001 | ⬜ | ⬜ |
| Kerberoasting | T1558.003 | ⬜ | ⬜ |
| Pass-the-Hash | T1550.002 | ⬜ | ⬜ |
| Recon AD (BloodHound) | T1087 | ⬜ | ⬜ |

_Section à compléter._
