---
title: Introduction à Proxmox VE
description: Proxmox VE (Virtual Environment) est une solution libre et open-source de virtualisation basée sur Debian. Elle permet d’héberger et de gérer à la fois des machines virtuelles et des conteneurs Linux, avec une interface web centralisée.
published: true
date: 2025-08-10T15:44:55.044Z
tags: 
editor: markdown
dateCreated: 2025-06-23T18:32:08.824Z
---

![logo_proxmox.svg](/proxmox/logo_proxmox.svg)

## Objectif de Proxmox VE

Proxmox est conçu pour :
- Simplifier le **déploiement et l’administration** d’infrastructures virtualisées,
- Offrir une solution complète pour le **cloud privé**, le **homelab** ou l’**hébergement professionnel**,
- Réduire les coûts grâce à des outils puissants et libres.



## Fonctionnalités principales

- Hyperviseur de type 1 basé sur **KVM**.
- Gestion des conteneurs **LXC**.
- Interface web complète et réactive.
- **Snapshots**, **clonage**, **live migration**, etc.
- Intégration **ZFS**, **LVM**, **Ceph**, **iSCSI**, **NFS**, etc.
- **Firewall** intégré et gestion fine des permissions.
- Système de **sauvegarde intégré** + support natif de **Proxmox Backup Server**.
- Fonctionnalités de **cluster** et **haute disponibilité (HA)**.



## Composants associés

- **Proxmox VE** : hyperviseur principal.
- **Proxmox Backup Server (PBS)** : solution de sauvegarde pour VM, CT et hôtes Proxmox.
- **Proxmox Mail Gateway (PMG)** : filtre antispam/antivirus pour serveurs mail.



## Cas d’usage courants

- Homelab complet avec supervision, sécurité, stockage, routage...
- Infrastructure mutualisée (TPE, PME, ESN, clients VPS).
- Environnement de test ou préproduction virtualisé.
- Remplacement de solutions propriétaires (VMware, Hyper-V).


## Licence

Proxmox VE est publié sous licence **AGPLv3**.  
Une souscription commerciale est disponible pour le support et les dépôts "enterprise", mais **toutes les fonctionnalités restent accessibles sans licence payante**.