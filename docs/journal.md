# Journal de bord — CyberLab

> Journal chronologique de la construction du laboratoire. Chaque étape indique
> l'objectif, les actions réalisées et le raisonnement (« pourquoi »).

---

## Phase 1 — Préparation du laboratoire

### Étape 1 — Installation de VMware
**Objectif :** installer l'hyperviseur hébergeant l'ensemble des VM.
**Résultat :** VMware Workstation Pro installé et vérifié.

### Étape 2 — Création des réseaux virtuels
**Actions :** création de VMnet2 (Host-Only), sous-réseau `192.168.209.0/24`.
**Pourquoi :** isoler le laboratoire du réseau physique tout en permettant la
communication entre VM.

### Étape 3 — Configuration réseau
**Action :** désactivation du serveur DHCP de VMware.
**Pourquoi :** le DHCP sera hébergé par Windows Server, comme en entreprise (un seul
serveur DHCP par réseau).

### Étape 4 — Préparation des ISO
ISO disponibles : Windows Server 2022, Windows 11, Ubuntu Server, Kali Linux.

### Étapes 5-6 — Création et dimensionnement de SRV-DC01
**Configuration retenue :** Windows Server 2022 Standard · 6 Go RAM · 2 vCPU · disque
80 Go · réseau VMnet2.
**Pourquoi :** ressources adaptées à un DC de lab (AD DS + DNS + DHCP) tout en préservant
l'hôte.

**Ce que j'ai appris :** rôle d'un hyperviseur ; création d'un réseau virtuel dédié ;
différence NAT / Host-Only ; dimensionnement d'une VM ; principe d'un seul DHCP par réseau.

**Bonnes pratiques retenues :** préparer l'architecture réseau avant de créer les VM ;
vérifier le matériel avant le premier démarrage ; documenter au fil de l'eau.

### Étapes 7-10 — Installation de Windows Server
- Édition : Windows Server 2022 Standard Evaluation (Expérience de bureau), pour
  faciliter la découverte des rôles avant de passer à PowerShell.
- Installation personnalisée (disque neuf).
- Mot de passe Administrateur complexe.
- Première connexion réussie, Server Manager lancé.

### Étapes 11-13 — Configuration initiale du serveur
- Renommage : `WIN-J3T47SLTB5F` → `SRV-DC01`.
- IP fixe : `192.168.209.10/24`, DNS préféré `192.168.209.10` (lui-même).
- **Pourquoi une IP fixe :** un DC doit rester joignable en permanence.

### Étapes 14-17 — Promotion en contrôleur de domaine
- Installation des rôles **AD DS** et **DNS**.
- Création d'une nouvelle forêt : domaine `cyberlab.local`, NetBIOS `CYBERLAB`.
- Niveau fonctionnel forêt/domaine : Windows Server 2016.
- Catalogue global activé, RODC désactivé, mot de passe DSRM configuré.
- Avertissements non bloquants : délégation DNS impossible (normal pour une nouvelle
  forêt), compatibilité anciens algorithmes NT 4.0.
- **Résultat :** domaine `cyberlab.local` opérationnel (AD DS + DNS).

### Étape 18 — Structure des unités d'organisation (OU)
Création des OU : Administration, Serveurs, Postes, Utilisateurs, Groupes, Comptes
privilégiés. Toutes protégées contre la suppression accidentelle.
**Pourquoi :** structure claire et évolutive facilitant l'application des GPO.

### Étape 19 — Sous-OU par service
Sous l'OU Utilisateurs : IT, RH, Comptabilité, Direction.

### Étape 20 — Groupes de sécurité
Création de `GG_IT`, `GG_RH`, `GG_COMPTA`, `GG_DIRECTION`.
**Pourquoi :** attribuer les permissions aux groupes, pas aux utilisateurs.

### Étapes 21-24 — Comptes utilisateurs et appartenances
| Nom | Service | Identifiant | Groupe |
|-----|---------|-------------|--------|
| Dylan Gaillard | IT | dgaillard | GG_IT |
| Alice Martin | RH | amartin | GG_RH |
| Thomas Bernard | Comptabilité | tbernard | GG_COMPTA |
| Sarah Dupont | Direction | sdupont | GG_DIRECTION |

Configuration : mot de passe temporaire, changement imposé à la première connexion.

---

## Poste client

### Étape 26 — Création de la VM WIN11-01
Windows 11 Pro · 6 Go RAM · 2 vCPU · disque 80 Go · VMnet2 · TPM virtuel activé · Easy
Install désactivé.

### Étape 27 — Installation de Windows 11
Installation personnalisée, disque entier, partitionnement automatique.

### Étape 28 — Contournement de l'obligation de compte Microsoft
**Problème :** Windows 11 imposait une connexion Internet + compte Microsoft.
**Solution :** `Maj + F10` → commande `OOBE\BYPASSNRO` → après redémarrage, option
« Je n'ai pas Internet » disponible, installation avec compte local.
**Pourquoi :** en entreprise, les postes sont intégrés à un domaine, pas à un compte
Microsoft personnel.

### Étapes 29-30 — Configuration du poste
- Renommage : `WIN11-01`.
- IP : `192.168.209.20/24`, DNS préféré `192.168.209.10`.
- **Pourquoi ce DNS :** les postes du domaine doivent utiliser le DNS du DC pour localiser
  les services `cyberlab.local`.

### Étape 31 — Vérification de connectivité
`ping 192.168.209.10` (OK) et `ping SRV-DC01` (résolution DNS OK).

### Étapes 32-34 — Intégration au domaine
- Jointure de `WIN11-01` à `cyberlab.local` (compte `CYBERLAB\Administrator`).
- Déplacement de l'objet ordinateur du conteneur `Computers` vers l'OU `Postes`.
- Première ouverture de session avec `CYBERLAB\dgaillard`, changement de mot de passe.
- **Résultat :** authentification de domaine fonctionnelle.

### Étape 35 — Première GPO
**Objectif :** empêcher l'utilisation des périphériques de stockage amovibles.
- GPO dédiée, liée à l'OU `Postes`.
- *Configuration ordinateur → Modèles d'administration → Système → Accès au stockage
  amovible → Tous les types : Refuser tout accès*.
- Validation : `gpupdate /force` puis `gpresult /scope computer /r`.
- **Résultat :** GPO correctement appliquée sur `WIN11-01`.

---

> 📝 Note : ce journal reprend les étapes réalisées avant la refonte du dépôt. La suite
> intègre pfSense, le durcissement, le SIEM et les scénarios de détection.
