---
title: Proxmox
description: Bienvenue sur le wiki dédié à Proxmox Virtual Environment.   Ce guide s'adresse à ceux qui veulent comprendre, déployer et administrer leur infrastructure Proxmox, du simple serveur standalone au cluster haute dispo.
published: true
date: 2025-08-10T15:44:16.145Z
tags: proxmox, pve, hyperviseur
editor: markdown
dateCreated: 2025-06-23T13:31:44.587Z
---

![logo_proxmox.svg](/proxmox/logo_proxmox.svg){.align-center}

# Vue Générale


- [Introduction *Introduction à Proxmox VE, ses composants, cas d’usage et architecture.*](/Proxmox/Introduction)
- [Fonctionnement général *Hyperviseur, conteneurs, réseau, stockage, etc.*](/Proxmox/vue-generale/fonctionnement)
{.links-list}

---

# Déploiement

- [Pré-requis *Configuration minimale recommandée pour Proxmox VE.*](/Proxmox/deploiement/prerequis)
- [Installation *Installation de PVE depuis l’ISO.*](/Proxmox/deploiement/installation)
- [Configuration post-installation *Paramétrage initial, accès web, updates.*](/Proxmox/deploiement/post-install)
{.links-list}

---

# Virtualisation

- [Machines virtuelles *Création, configuration, snapshots, ISO.*](/Proxmox/virtualisation/kvm)
- [Conteneurs (LXC) *Utilisation de `pct`, gestion de templates, montage volumes.*](/Proxmox/virtualisation/lxc)
{.links-list}

---

# Réseau

- [Interfaces et bridges *vmbr0, NAT, accès par pont, etc.*](/Proxmox/reseau/bridges)
- [VLAN *Configuration des VLAN sur Proxmox.*](/Proxmox/reseau/vlan)
- [SDN *Introduction et déploiement du Software Defined Network.*](/Proxmox/reseau/sdn)
- [Firewall *Règles de filtrage au niveau datacenter, node, VM.*](/Proxmox/reseau/firewall)
{.links-list}

---

# Stockage

- [Types de stockage *ZFS, LVM, Ceph, CIFS, NFS, etc.*](/Proxmox/stockage/types-de-stockage)
- [Ajout de disques *Ajout de disques physiques ou partages réseau.*](/Proxmox/stockage/disques)
- [Gestion des pools ZFS *Création, import, snapshots, replication.*](/Proxmox/stockage/zfs)
{.links-list}

---

# Sauvegarde

- [Backups avec Proxmox VE *Planification, compression, destinations.*](/Proxmox/sauvegarde/backup-pve)
- [Proxmox Backup Server *Intégration et configuration PBS.*](/Proxmox/sauvegarde/pbs)
- [Restauration *Restaurer une VM ou un container à partir d’un backup.*](/Proxmox/sauvegarde/restauration)
{.links-list}

---

# Cluster et Haute Disponibilité

- [Création d’un cluster *Ajout de nœuds, quorum, réseau.*](/Proxmox/cluster/creation)
- [Haute disponibilité *Migration, redondance, services critiques.*](/Proxmox/cluster/ha)
{.links-list}

---

# Sécurité & Accès

- [Utilisateurs et rôles *Gestion des comptes, permissions, ACL.*](/Proxmox/securite/utilisateurs)
- [Authentification *PAM, PVE auth, LDAP, 2FA.*](/Proxmox/securite/authentification)
- [Certificats SSL *HTTPS et certificats personnalisés.*](/Proxmox/securite/certificats)
{.links-list}

---

# Maintenance

- [Mises à jour *Dépôts, enterprise vs no-subscription.*](/Proxmox/maintenance/mises-a-jour)
- [Surveillance *Ressources, charge, journalisation, notifications.*](/Proxmox/maintenance/surveillance)
- [Dépannage *Accès root, redémarrages problématiques, erreurs fréquentes.*](/Proxmox/maintenance/depannage)
{.links-list}

---

# Annexes & Outils

- [Proxmox Mail Gateway *Filtrage des emails, antispam, antivirus.*](/Proxmox/annexes/pmg)
- [Proxmox Backup Server *Documentation dédiée.*](/Proxmox/annexes/pbs)
- [Liens utiles *Forum, wiki officiel, docs communautaires.*](/Proxmox/annexes/liens)
{.links-list}

---