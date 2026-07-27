# 📸 Captures d'écran — CyberLab

Captures illustrant chaque étape de la construction du laboratoire, classées par phase.
Chaque image est référencée depuis [`../docs/journal.md`](../docs/journal.md).

## Convention de nommage


| Préfixe | Phase |
|---------|-------|
| `p1-` | Préparation du laboratoire, Active Directory, poste client |
| `p2-` | Pare-feu pfSense |
| `p3-` | Service DHCP |

---

## Phase 1 — Préparation du laboratoire

### Création de la machine virtuelle SRV-DC01

![Sélection de l'ISO Windows Server 2022](p1-vm-iso-selection.jpg)

*Assistant de création de VM — sélection de l'ISO Windows Server 2022*

![Dimensionnement du disque virtuel](p1-vm-disque.jpg)

*Dimensionnement du disque virtuel — 80 Go alloués au contrôleur de domaine*

![Choix de l'édition Windows Server](p1-vm-edition.jpg)

*Édition Windows Server 2022 Standard et définition du compte Administrateur*

![Personnalisation matérielle de la VM](p1-vm-materiel.jpg)

*Personnalisation matérielle — 6 Go de RAM, 2 vCPU, carte réseau raccordée à VMnet2 (réseau isolé du laboratoire)*

![VM SRV-DC01 prête au démarrage](p1-vm-prete.jpg)

*Machine virtuelle SRV-DC01 créée et prête au premier démarrage*

### Installation et configuration initiale

![Premier démarrage du serveur](p1-serveur-demarrage-1.jpg)

*Premier démarrage : Gestionnaire de serveur avant toute configuration*

![État initial du serveur](p1-serveur-etat-initial.jpg)

*État initial : nom généré automatiquement (WIN-J3T47SLTB5F), groupe de travail WORKGROUP et adresse IP en DHCP — trois points à corriger avant la promotion en contrôleur de domaine*

### Promotion en contrôleur de domaine

![Installation du rôle AD DS](p1-installation-adds.jpg)

*Installation du rôle Services AD DS sur SRV-DC01*

![Configuration du déploiement AD DS](p1-config-deploiement.jpg)

*Assistant de configuration AD DS : choix du type de déploiement (option retenue : « Ajouter une nouvelle forêt »)*

![Options du contrôleur de domaine](p1-option-dc.jpg)

*Options du contrôleur de domaine : niveau fonctionnel Windows Server 2016, serveur DNS et catalogue global activés, mot de passe DSRM défini*

### Validation du poste client

![Validation de la connectivité depuis WIN11-01](p1-validation-connectivite.jpg)

*Depuis WIN11-01 : ping du contrôleur de domaine par adresse IP puis par nom — la résolution DNS est fonctionnelle*
 
---
 
## Phase 2 — Pare-feu pfSense
 
![Interfaces pfSense](p2-pfsense-console-interfaces.jpg)

*Menu console pfSense : interfaces WAN (em0) et LAN (em1) correctement assignées et adressées*
 
![Passerelle sur le contrôleur de domaine](p2-dc-passerelle.jpg)

*SRV-DC01 : passerelle 192.168.209.1 ajoutée, le serveur DNS restant le DC lui-même*
 
![Redirecteur DNS](p2-dns-redirecteur.jpg)

*Console DNS : redirecteur vers pfSense pour centraliser les requêtes sortantes*
 
![Résolution DNS Internet](p2-nslookup-internet.jpg)

*Validation de la résolution de noms Internet depuis le contrôleur de domaine*

## Phase 3 — Service DHCP

![Propriétés de l'étendue DHCP](p3-dhcp-etendue-proprietes.jpg)

*Propriétés de l'étendue LAN-CyberLab : plage 192.168.209.30 à 192.168.209.199, bail de 8 jours*

![Configuration IP reçue par DHCP](p3-win11-ipconfig.jpg)

*WIN11-01 : adresse, passerelle, serveur DNS et suffixe de domaine reçus automatiquement — aucune valeur saisie manuellement sur le poste*

![Bail DHCP actif côté serveur](p3-dhcp-bail-actif.jpg)

*Console DHCP : bail actif pour WIN11-01 en 192.168.209.30, confirmant l'attribution depuis le serveur*

---
 
## Phase 4 — Serveur Linux (Ubuntu Server)
 
### Installation d'Ubuntu Server
 
![Choix de la langue d'installation](p4-installation-ubuntuserver.jpg)

*Assistant d'installation Ubuntu Server 26.04 LTS — sélection de la langue (anglais, convention serveur)*
 
![Type d'installation](p4-type-installation.jpg)

*Choix de l'édition « Ubuntu Server » (version complète, non minimisée)*
 
![Configuration du stockage guidée](p4-storage-config.jpg)

*Stockage : disque entier avec LVM activé, sans chiffrement LUKS*
 
![Résumé du partitionnement](p4-storage-ok.jpg)

*Partitionnement final — /boot séparé et racine / sur volume LVM, espace entièrement alloué*
 
### Configuration réseau
 
![Configuration réseau à l'installation](P4-network-config.jpg)

*Interface ens33 en DHCP pendant l'installation (adresse 192.168.209.31) — l'IP fixe sera définie ensuite*
 
![Configuration réseau initiale (DHCP)](p4-fichier-config-reseau.jpg)

*Fichier Netplan d'origine généré par l'installateur — interface ens33 en DHCP*
 
![Passage en IP fixe via Netplan](p4-passage-ipfixe.jpg)

*Fichier Netplan modifié — IP fixe 192.168.209.20, passerelle 192.168.209.1, DNS 192.168.209.10*
 
![Vérification de l'adresse IP](p4-ip-a.jpg)

*Commande ip a — l'interface ens33 dispose de son adresse sur le réseau du laboratoire*
 
![Test de connectivité](p4-ping-test.jpg)

*Validation réseau — accès Internet (8.8.8.8) et résolution DNS (google.com) fonctionnels*
 
### Accès et administration
 
![Premier démarrage du serveur](p4-srv-wazuh-firstlog.jpg)

*Premier démarrage de SRV-WAZUH01 — écran de connexion et informations système*
 
![Connexion SSH établie](p4-ssh-connexion-reussie.jpg)

*Connexion SSH réussie vers SRV-WAZUH01 — administration à distance opérationnelle*

## Règles appliquées

- **Anonymisation** — aucune donnée personnelle, adresse IP publique ou information sensible n'apparaît sur les captures.
- **Cadrage** — les captures sont recadrées sur l'information utile.
- **Légende systématique** — chaque image est accompagnée d'une description de ce qu'elle démontre.
