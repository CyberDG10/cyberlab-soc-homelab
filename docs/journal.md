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

## Phase 2 — Déploiement du pare-feu pfSense

> Objectif global de la phase : introduire un routeur/pare-feu entre le LAN isolé du
> laboratoire et Internet, afin de fournir un accès aux mises à jour tout en gardant la
> maîtrise du trafic. pfSense devient la passerelle du réseau et une future source de logs
> pour le SOC.

### Étape 36 — Choix de la solution de pare-feu

**Décision :** pfSense Community Edition (CE) 2.8.1.
**Pourquoi ?**
- Solution open source (licence Apache 2.0), gratuite et largement reconnue en entreprise.
- La CE couvre l'intégralité des besoins du laboratoire (routage, pare-feu, segmentation) ;
  la version Plus (payante) n'apporterait rien d'utile ici.
- Compétence valorisable : pfSense est un standard du marché pour la sécurité périmétrique.

### Étape 37 — Création de la machine virtuelle pfSense

**Configuration retenue :**
- Nom : `pfSense-FW`
- Système invité : FreeBSD 64 bits (pfSense est basé sur FreeBSD)
- Mémoire : 1 Go — Processeur : 1 vCPU — Disque : 20 Go (fichier unique, allocation à la demande)
- **Adaptateur réseau 1 : NAT (VMnet8)** → futur WAN
- **Adaptateur réseau 2 : VMnet2 (Host-Only)** → futur LAN

**Pourquoi deux cartes réseau ?**
Un pare-feu sépare deux zones de confiance : une patte « externe » (WAN, côté Internet) et
une patte « interne » (LAN, côté réseau protégé). L'ordre des adaptateurs détermine leur
nom dans pfSense : adaptateur 1 → `em0` (WAN), adaptateur 2 → `em1` (LAN).

### Étape 38 — Installation de pfSense

**Choix d'installation :**
- Système de fichiers : **ZFS** (recommandé par Netgate : intégrité des données, snapshots).
- Schéma de partition : **GPT** (standard moderne).
- Topologie : **stripe** (un seul disque, pas de redondance nécessaire en environnement virtuel).
- Édition installée : **pfSense CE** (aucun abonnement Plus, ce qui est attendu).

**Note :** l'ISO a été retirée du lecteur après l'installation pour éviter de redémarrer
sur l'installateur.

### Étape 39 — Assignation des interfaces et adressage

| Interface | Rôle | Mode | Adresse |
|-----------|------|------|---------|
| `em0` | WAN | Client DHCP | 192.168.201.128/24 (fournie par le NAT VMware) |
| `em1` | LAN | Statique | **192.168.209.1/24** |

**Serveur DHCP de pfSense : désactivé.**
**Pourquoi ?** Un seul serveur DHCP doit exister par réseau. Ce rôle sera assuré par le
contrôleur de domaine (SRV-DC01) en phase suivante, conformément aux pratiques d'entreprise
où DHCP est couplé à l'AD/DNS. Les machines actuelles restent en IP fixe.

**Pourquoi l'adresse .1 pour le LAN ?**
Par convention, la première adresse d'un réseau est réservée à la passerelle. Toutes les
machines du LAN utiliseront `192.168.209.1` comme porte de sortie vers Internet.

### Étape 40 — Connexion du contrôleur de domaine à la passerelle

**Action réalisée sur SRV-DC01 :** ajout de la passerelle par défaut `192.168.209.1` dans
les propriétés IPv4 de la carte réseau. Le serveur DNS préféré reste `192.168.209.10`
(le DC lui-même).

**Pourquoi ne pas changer le DNS du DC ?**
Un contrôleur de domaine doit rester son propre serveur DNS, car il héberge la zone
`cyberlab.local`. Le faire pointer vers un DNS externe casserait la résolution du domaine et
tout l'Active Directory. **Règle : on ne modifie jamais le DNS d'un DC vers un DNS externe.**

**Tests de validation :**
```
ping 192.168.209.1   → réponses (le DC joint pfSense)
ping 8.8.8.8         → réponses (routage Internet fonctionnel)
```

### Étape 41 — Configuration du redirecteur DNS

**Action :** dans la console DNS (`dnsmgmt.msc`), ajout du redirecteur `192.168.209.1`
(pfSense) dans les propriétés du serveur.

**Pourquoi un redirecteur alors que la résolution fonctionnait déjà ?**
Sans redirecteur, le DC résolvait déjà les noms Internet via les *root hints* (interrogation
directe des serveurs racine), possible dès qu'une route Internet existe. Le redirecteur vers
pfSense est néanmoins préférable dans un contexte SOC : il fait transiter **toutes** les
requêtes DNS par le pare-feu, créant un point unique de visibilité et de contrôle
(journalisation, filtrage, détection future).

**Test de validation :**
```
nslookup google.com  → réponse non autoritaire avec adresses IPv4/IPv6 (résolution OK)
```

### Vérification de l'isolement

Malgré l'accès Internet, l'infrastructure reste isolée : pfSense autorise le trafic
**sortant** du LAN mais bloque toute connexion **entrante** depuis Internet, et le LAN
(VMnet2, host-only) derrière le NAT n'est pas joignable de l'extérieur. Les machines sortent,
mais restent injoignables depuis Internet.

### Ce que j'ai appris

- Différence entre **passerelle** (routage : comment sortir du réseau) et **pare-feu**
  (filtrage : ce qui est autorisé à passer) — deux fonctions assurées par un seul boîtier.
- Rôle du **WAN** (patte qui reçoit) et du **LAN** (patte qui distribue) sur un pare-feu.
- Principe du **DHCP unique** par réseau et raison de le centraliser sur le DC.
- Résolution DNS : **root hints** vs **redirecteur (forwarder)**, et pourquoi router était le
  seul chaînon manquant à l'accès Internet.
- Pourquoi le **DNS d'un contrôleur de domaine** ne doit jamais pointer vers l'extérieur.
- Notions d'installation système : **ZFS / GPT / stripe** et quand choisir la redondance.

### Bonnes pratiques retenues

- Vérifier la correspondance carte réseau ↔ interface (ordre des adaptateurs) avant de
  démarrer une VM multi-interfaces.
- Prendre un **snapshot** avant toute modification réseau sensible (fait avant et après la
  configuration de la passerelle).
- Tester chaque couche séparément : d'abord le routage (`ping IP`), puis la résolution de
  noms (`nslookup`).

  ## Phase 3 — Mise en place du service DHCP

> Objectif global de la phase : automatiser l'attribution des adresses IP du laboratoire.
> Jusqu'ici, chaque machine était configurée manuellement en IP fixe — une approche qui ne
> passe pas à l'échelle. Le contrôleur de domaine devient le serveur DHCP du réseau,
> conformément aux pratiques d'entreprise où DHCP est couplé à l'AD et au DNS.

### Étape 42 — Définition du plan d'adressage

Avant tout déploiement, le réseau `192.168.209.0/24` (254 adresses utilisables, de `.1` à
`.254`) a été découpé **par usage** :

| Plage | Usage | Mode d'attribution |
|-------|-------|--------------------|
| `.1` → `.29` | Passerelle et serveurs | Manuel (IP fixe) |
| `.30` → `.199` | Postes clients | Automatique (DHCP) |
| `.200` → `.254` | Réserve pour besoins futurs | — |

Attribution actuelle :

| Adresse | Machine | Rôle |
|---------|---------|------|
| `192.168.209.1` | pfSense-FW | Passerelle / pare-feu |
| `192.168.209.10` | SRV-DC01 | AD DS, DNS, DHCP |
| `192.168.209.20` | SRV-WAZUH01 | SIEM (à déployer) |
| `192.168.209.30`+ | WIN11-01, KALI-01… | Postes clients (DHCP) |

**Pourquoi ce découpage ?**
- **Les serveurs restent en IP fixe** : si l'adresse d'un contrôleur de domaine changeait,
  plus aucun poste ne pourrait s'authentifier, résoudre les noms ou recevoir ses GPO. De
  même, l'adresse de la passerelle est inscrite en dur dans la configuration des clients.
- **La plage DHCP démarre à `.30`, pas à `.2`** : cette zone tampon garantit que le serveur
  ne pourra jamais attribuer à un client une adresse déjà utilisée par un serveur (ce qui
  provoquerait un conflit d'adresses). Les adresses `.2` à `.29` constituent une marge pour
  les futurs serveurs.
- **La plage s'arrête à `.199`** : les adresses hautes restent en réserve. Redécouper un
  plan d'adressage sur un réseau déjà en production est une opération lourde ; prévoir de
  la marge dès le départ ne coûte rien.

**Principe général retenu :** adresses basses = infrastructure stable, adresses médianes =
clients dynamiques, adresses hautes = réserve.

### Étape 43 — Installation du rôle DHCP

**Actions :** Gestionnaire de serveur → *Ajouter des rôles et fonctionnalités* → rôle
**Serveur DHCP** → installation (aucun redémarrage requis).

Puis, via la notification du Gestionnaire de serveur : **Terminer la configuration DHCP**
→ *Valider*.

**Pourquoi cette étape d'autorisation ?**
Un serveur DHCP Windows doit être explicitement **autorisé dans l'Active Directory** pour
distribuer des adresses. Cette sécurité vise à empêcher qu'une machine non approuvée se mette
à distribuer de fausses configurations réseau sur le domaine — technique d'attaque connue
sous le nom de **rogue DHCP**, qui permet notamment de rediriger le trafic des victimes vers
un équipement contrôlé par l'attaquant (passerelle ou DNS malveillant).

**Vérifications :**
```powershell
Get-Service DHCPServer     # doit être "Running"
Get-DhcpServerInDC         # doit lister srv-dc01.cyberlab.local / 192.168.209.10
```

### Étape 44 — Problème rencontré : console DHCP vide

**Symptôme :** la console MMC s'ouvrait sans afficher de serveur ni de nœud IPv4, alors que
le service DHCP tournait et que le serveur était bien autorisé dans l'AD.

**Fausses pistes écartées :** service arrêté, autorisation AD manquante, rôle mal installé —
toutes infirmées par les commandes de vérification ci-dessus.

**Cause réelle :** la console lancée n'était pas la bonne. Les commandes `dnsmgmt.msc` (DNS)
et `dhcpmgmt.msc` (DHCP) ne diffèrent que de quelques lettres ; la complétion automatique de la
boîte *Exécuter* rappelait la commande DNS précédemment saisie. La console **DNS** était donc
ouverte, où aucun nœud d'étendue DHCP n'existe.

**Solution :** ouvrir la console via **Gestionnaire de serveur → Outils → DHCP**, qui lance
l'outil avec élévation et déjà connecté au serveur local.

**Leçon retenue :** face à un outil qui semble « vide » ou anormal, vérifier d'abord que l'on
se trouve dans le bon outil avant de suspecter la configuration. Sur un serveur, privilégier
le menu *Outils* du Gestionnaire de serveur aux raccourcis `.msc`, dont les noms se
ressemblent.

### Étape 45 — Création de l'étendue DHCP

**Étendue créée :**

| Paramètre | Valeur |
|-----------|--------|
| Nom | `LAN-CyberLab` |
| Plage distribuée | `192.168.209.30` → `192.168.209.199` |
| Masque | `255.255.255.0` (/24) |
| Durée du bail | 8 jours (valeur par défaut) |
| Exclusions | Aucune |
| État | Active |

**Options d'étendue configurées :**

| Option | Valeur | Rôle |
|--------|--------|------|
| 003 — Routeur | `192.168.209.1` | Passerelle par défaut (pfSense) |
| 006 — Serveurs DNS | `192.168.209.10` | Serveur DNS (le DC) |
| 015 — Nom de domaine DNS | `cyberlab.local` | Suffixe DNS des clients |

**Pourquoi aucune exclusion ?** Une exclusion sert à retirer des adresses situées *à
l'intérieur* de la plage distribuée. Les serveurs étant déjà en dehors (sous `.30`), il n'y a
rien à exclure — bénéfice direct d'un plan d'adressage défini en amont.

**Note :** les numéros d'options (003, 006, 015) sont standardisés par le protocole DHCP et
se retrouvent sur tout serveur DHCP, quel que soit le système.

### Étape 46 — Bascule du poste client en DHCP

**Actions sur WIN11-01 :**
1. `ncpa.cpl` → propriétés de la carte Ethernet → *Protocole Internet version 4 (TCP/IPv4)*.
2. Sélection de **Obtenir une adresse IP automatiquement** et **Obtenir les adresses des
   serveurs DNS automatiquement** (le poste était jusque-là en IP fixe `192.168.209.20`).
3. Renouvellement du bail :
```powershell
ipconfig /release
ipconfig /renew
ipconfig /all
```

**Résultat obtenu :**
```
DHCP activé . . . . . . . . . . . : Oui
Adresse IPv4. . . . . . . . . . . : 192.168.209.30
Passerelle par défaut . . . . . . : 192.168.209.1
Serveurs DNS. . . . . . . . . . . : 192.168.209.10
Serveur DHCP. . . . . . . . . . . : 192.168.209.10
```

Aucune de ces valeurs n'a été saisie sur le poste : adresse, passerelle et DNS ont été
transmis automatiquement par le serveur. L'ancienne adresse `.20` est libérée et redevient
disponible pour un futur serveur.

### Étape 47 — Vérification côté serveur

Console DHCP → *IPv4* → *LAN-CyberLab* → **Baux d'adresses** : présence d'un bail actif pour
`WIN11-01` en `192.168.209.30`, avec sa date d'expiration.

**Intérêt de cette vue pour la sécurité :** la table des baux constitue un inventaire en
temps réel des machines présentes sur le réseau. Dans une démarche de supervision, l'apparition
d'un bail pour une machine inconnue est un signal à investiguer.

### Ce que j'ai appris

- Rôle et fonctionnement du **DHCP** : attribution d'adresses par bail, et transmission des
  options réseau (passerelle, DNS, suffixe de domaine) en plus de l'adresse elle-même.
- Construction d'un **plan d'adressage** cohérent : découpage par usage, zone tampon entre
  adresses fixes et dynamiques, réserve pour l'avenir.
- Mécanisme d'**autorisation DHCP dans l'Active Directory** et menace du *rogue DHCP*.
- Signification des **options DHCP standardisées** (003, 006, 015).
- Méthode de **diagnostic** : distinguer le symptôme de la cause racine, et vérifier son
  environnement de travail avant de mettre en cause la configuration.

### Bonnes pratiques retenues

- Définir le plan d'adressage **avant** de déployer le service, et non l'inverse.
- Maintenir les serveurs et équipements réseau hors de la plage dynamique.
- Sur un serveur, ouvrir les consoles via **Gestionnaire de serveur → Outils**.
- Valider chaque déploiement par un test bout en bout : côté client (`ipconfig /all`) **et**
  côté serveur (table des baux).
- Prendre un **snapshot** après chaque étape fonctionnelle validée.

> 📝 Note : ce journal reprend les étapes réalisées avant la refonte du dépôt.
