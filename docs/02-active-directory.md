# 02 — Active Directory

> Détail de la mise en place d'AD DS, DNS, DHCP, OU, utilisateurs et groupes.
> Le déroulé chronologique se trouve dans [`journal.md`](journal.md).

## État actuel
- Forêt / domaine : `cyberlab.local` (NetBIOS `CYBERLAB`).
- Rôles : AD DS, DNS opérationnels sur `SRV-DC01` (192.168.209.10).
- OU : Administration, Serveurs, Postes, Utilisateurs (IT/RH/Compta/Direction),
  Groupes, Comptes privilégiés.
- Groupes de sécurité globaux : `GG_IT`, `GG_RH`, `GG_COMPTA`, `GG_DIRECTION`.
- 4 comptes utilisateurs créés et affectés à leurs groupes.

## À venir
- Rôle DHCP (étendue + réservations).
- Synchronisation du temps (PDC comme source autoritaire) — critique pour Kerberos.
- Modèle de tiering administratif (voir [`03-hardening-gpo.md`](03-hardening-gpo.md)).

_Section à compléter._
