---
title: 🖥️ Machines virtuelles
description: Cette page présente les étapes essentielles pour créer, configurer et gérer des machines virtuelles dans un environnement de virtualisation.
published: false
date: 2025-07-28T12:04:37.155Z
tags: proxmox, pve, kvm
editor: markdown
dateCreated: 2025-07-28T12:04:37.155Z
---

# 🖥️ Machines virtuelles
## 📌 Introduction 

La virtualisation est devenue un **pilier incontournable** des infrastructures modernes, qu'elles soient professionnelles ou personnelles (homelabs, plateformes de test, etc...).

Créer une machine virtuelle (VM), c'est simuler un ordinateur entier dans un environnement isolé, que ce soit pour **héberger un service, faire des tests**, ou encore **déployer rapidement des systèmes** sans avoir besoin de matériel physique dédié.

![schema-virtualisation.png](/schema-virtualisation.png)
*Source:  CoeurDuWeb*

Dans cette section, nous allons voir comment : 

* Créer une VM (depuis une image ISO).
* Configurer ses ressources (CPU, RAM, disque, réseau, ...).
* Gérer ses snapshots (sauvegarde/restauration d'un instant T).
* Monter des iso ou des images disques.

*🎯 Objectif : Comprendre les bases de la gestion des machines virtuelles, et acquérir les bons réflexes pour créer des environnements propres et fonctionnels.*