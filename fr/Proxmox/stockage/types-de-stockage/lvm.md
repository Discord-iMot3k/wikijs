---
title: LVM - Local Volume Manager
description: Cette page présente le fonctionnement de LVM, ses cas d'usage, ses avantages, ses limites, et dans quels contextes il peut être utilisé comme solution de stockage local.
published: true
date: 2025-07-28T10:26:48.221Z
tags: proxmox, pve, stockage
editor: markdown
dateCreated: 2025-07-28T10:22:21.500Z
---

# 📄 Présentation Générale

## ⚙️ Difficulté de mise en place : 4/10
*LVM est relativement simple à mettre en place, comparé à ZFS. Il ne nécessite pas de configuration particulière ni de concepts plus complexes comme les pools ou les checksums. Pour un usage courant en stockage local, c'est un bon point d'entrée*

## 🔍 Qu'est ce que LVM

**LVM (Logical Volume Manager)** est un système qui permet de mieux gérer l'espace disque sur un serveur sous Linux.
Au lieu de créer des partitions fixes sur un disque, LVM permet de créer des *volumes logiques$^1$* que l'on peut **agrandir, réduir, ou déplacer** plus facilement.

Avec LVM, on peut par exemple : 

* Ajouter un disque à une machine sans tout reconfigurer.
* Agrandir un volume lorsqu'il est plein.
* Faire des sauvegardes instantanées (*snapshot$^2$*) d'un volume.

C'est une solution pratique pour ceux qui veulent garder le contrôle sur l'organisation de leur stockage.

---

*Volume logique$^1$* : Espace de stockage virtuel crée à partir d'un ou plusieurs disques physiques.

*Snapshot$^2$* : Copie intantanée d'un volume à un moment donné, utile pour les sauvegardes ou les tests.

## ✅ Quand utiliser LVM ?

LVM est particulièrement adapté dans les cas suivants : 

* **Plateformes de test ou de formation légère** : Si tu montes rapidement des VM ou des containers dans un environnement de test, LVM est rapide à mettre en place, et ne demande pas de configuration particulière.

* **Petits serveurs internes ou PC administratifs** : Pour des machines qui n'ont pas besoin d'un *résilience$^1$* avancée ou de fonctionnalités complexes.

* **Environnements à faible ressources** : Contrairement à ZFS, LVM est léger et n'impose pas une consommation importante. Il fonctionne sans souci sur des machines peu puissantes (VM, Raspberry Pi, vieux serveurs, ...)

* **Installations rapides ou automatisées** : LVM est bien intégré aux installateurs automatiques (Ansible, Cloud-init, ...). Il s'utilise facilement dans des scripts sans ajout de dépendances.

* **Besoin de volumes séparés sans complexité** : Pour créer plusieurs volumes logiques bien séparés (`/home`, `/var`, `/srv`, etc).

---

*Résilience$^1$* : Capacité d'un système à fonctionner même en cas de panne ou d'erreur.

## 👍 Avantages

* **Flexible** : On peut agrandir ou réduire un volume sans tout casser
* **Ajout de disques à chaud** : On peut ajouter un disque et l'intégrer à un volume existant sans redémarrer
* **Intégré dans les installateurs Linux** : LVM est souvent proposé par défaut dans les distributions comme Debian, CentOS, Ubuntu.
* **Snapshots** : Permet de créer une "photo" d'un volume à un instant donné, utile pour les sauvegardes d'anciennes versions ou les tests.
* **Léger et peu contraignant** : Utilise peu de ressources système, ne surchage pas la RAM, ni les disques. Convient bien aux enviornnements simples ou peu puissants.

## 👎 Inconvénients

* **Pas de contrôle d'intégrité automatique** : Contrairement à ZFS, LVM ne vérifie pas si les données ont été altérées ou corrompues.
* **Pas de compression native** : Ne permet pas de gagner de la place "automatiquement".
* **Structure parfois complexe** : Avec plusieurs disques, groupes et volumes, la configuration peut devenir difficile à suivre au début.
* **Pas conçu pour la haute disponibiité** : Pas de *redondance$^1$* intégrée en cas de panne matérielle. Pour cela, mieux vaut se tourner vers ZFS ou CEPH.

---

*Redondance$^1$* : Duplication des données ou des composants pour garantir leur disponibilité même en cas de panne.

---

# 🛠️ Mise en place et Configuration de LVM

