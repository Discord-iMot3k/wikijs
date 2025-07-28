---
title: Types de stockage
description: Ici, vous retrouverez l'ensemble des principales technologies de stockage disponibles sur Proxmox VE
published: true
date: 2025-07-28T09:40:02.703Z
tags: proxmox, pve
editor: markdown
dateCreated: 2025-07-28T09:30:52.838Z
---

# 🗂️ Types de Stockage


## 📌 Introduction

Dans toute infrastructure informatique, la gestion du stockage est un choix essentiel. 
* Quel type de stockage offre les meilleurs performances ?
* Que choisir si j'ai envie de gagner en flexibilité dans la gestion de mes volumes ?
* Comment faire en sorte que mes données restent disponibles et intactes, même en cas de problème matériel ou d'erreur humaine ? 

Sur **Proxmox**, on peut gérer du **stockage local** avec **LVM** ou encore **ZFS**, du **stockage réseau** via **NFS** ou **CIFS**, ou entre mettre en place du **stockage distribué** avec **Ceph** pour les environnements en cluster. Chaque solution a ses **spécificités**, ses **avantages**, mais aussi ses **contraintes**.

Cette page vise à présenter les différentes solutions de stockage, qui vous seront utiles à la fois dans des environnements professionnels, des homelab, ou des plateformes de test.

*🎯 L'objectif : Poser les bases pour comprendre les logiques de chaque système, et vous donner le bon outil selon vos besoins.*

---

# 🧩 Stockage local
### *Stockage directement rattaché à la machine physique virtuelle*

- [LVM *Logical Volume Manager*](/Proxmox/stockage/types-de-stockage/lvm)
- [ZFS *Zettabyte File System*](/Proxmox/stockage/types-de-stockage/zfs)
{.links-list}

---

# 🌐 Stockage réseau
### *Stocakge partagé via un réseau local ou distant*

- [NFS *Network File System*](/Proxmox/stockage/types-de-stockage/nfs)
- [CIFS / SMB *Common Internet File System*](/Proxmox/stockage/types-de-stockage/cifs-smb)
{.links-list}

---

# 🧬 Stockage distribué
### *Stockage répliqué, réparti et plus tolérant aux pannes*

- [CEPH](/Proxmox/stockage/types-de-stockage/ceph)
{.links-list}

