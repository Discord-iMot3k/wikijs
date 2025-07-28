---
title: 🖥️ Machines virtuelles
description: Cette page présente les étapes essentielles pour créer, configurer et gérer des machines virtuelles dans un environnement de virtualisation.
published: false
date: 2025-07-28T12:36:30.370Z
tags: proxmox, pve, kvm
editor: markdown
dateCreated: 2025-07-28T12:04:37.155Z
---

# 🖥️ Machines virtuelles
## 📌 Introduction 

La virtualisation est devenue un **pilier incontournable** des infrastructures modernes, qu'elles soient professionnelles ou personnelles (homelabs, plateformes de test, etc...).

Créer une machine virtuelle (VM), c'est simuler un ordinateur entier dans un environnement isolé, que ce soit pour **héberger un service, faire des tests**, ou encore **déployer rapidement des systèmes** sans avoir besoin de matériel physique dédié.
<center>
<img src="/schema-virtualisation.png")>
</center>

*Source:  CoeurDuWeb*

Dans cette section, nous allons voir comment : 

* Créer une VM (depuis une image ISO).
* Configurer ses ressources (CPU, RAM, disque, réseau, ...).
* Gérer ses snapshots (sauvegarde/restauration d'un instant T).
* Monter des iso ou des images disques.

*🎯 Objectif : Comprendre les bases de la gestion des machines virtuelles, et acquérir les bons réflexes pour créer des environnements propres et fonctionnels.*

---

# 🛠️ Création et configuration d'une VM

## 1. Importation d'une image ISO

La première étape lorsque vous souhaitez créer une VM est de vérifier que votre Proxmox contient bien l'*image ISO*$^1$ de l'OS que vous souhaitez créer.

Pour cela, rendez-vous sur votre Proxmox, dans votre disque ayant été configuré pour recevoir les images ISO (**par défaut : local**), puis dans l'onglet **ISO Images**

<center>
<img src="/access_iso_list_pve.png")>
</center>

> Ici, nous pouvons voir que 4 iso ont étés importés : Windows10.iso / Windows_Server_2022.iso / debian-12.9.0-amd64-netinst.iso / virtio-win-0.1.271.iso.
{.is-info}

Pour importer une nouvelle image ISO, deux méthodes s'offrent à vous : "Upload" ou "Download from URL"

### Méthode 1 : Upload (local)

L'upload permet d'envoyer une image ISO depuis votre propre ordinateur vers le serveur.

### Méthode 2 : Download from URL

Le serveur va chercher lui même l'image ISO sur Internet
Il suffit de donner l'adresse de téléchargement, et Proxmox va s'occuper de télécharger l'ISO.

> **Download from URL** est plus rapide et plus stable si le serveur a un bon accès internet.
> **Upload** est utile si l'ISO est en interne (Ex : ISO customisée, serveur sans accès internet).
{.is-success}

---

## 2. 🛠️ Création de la machine virtuelle

Une fois votre image ISO importée, vous pouvez désormais commencer la création de votre machine virtuelle.

Pour cela, cliquez sur l'onglet "Create VM" en haut à droite. La page de création devrait apparaitre. Celle-ci contient 8 onglets (*General / OS / System / Disks / CPU / Memory / Network / Confirm*), tous nécessaire à la création de la VM. Nous allons voir comment paramétrer chaque étape.

> Si vous souhaitez avoir davantage d'option de personnalisation et de paramétrage de votre machine virtuelle, cochez l'onglet "Advanced" situé en bas à droit de la page de paramétrage de votre machine virtuelle. Pas d'inquiétudes, la procédure ci-dessous explique également les paramétrages avancés !
{.is-info}

<center>
<img src="/active_advanced_settings_vmsetup_proxmox.png" width="400">
</center>

### 1. General - Par défaut

**Node :** Noeud Proxmox sur lequel votre machine virtuelle sera installée. Utile dans le cas où vous avez un cluster, et souhaitez changer la machine physique sur lequelle votre VM sera installée.

**VM ID :** Numéro d'identification de votre machine virtuelle. Celui-ci permet d'identifier votre VM.
> Une fois créee, le "VM ID" ne peut être modifiable sans passer par un clonage ou autre méthode. Si vous souhaitez suivre une nomenclature et une certaine logique, réfléchissez aux VM ID avant création de votre machine virtuelle.
{.is-warning}

**Name :** Nom de la votre machine virtuelle. Pourra être modifiée dans les paramètres même après création de la VM.

**Ressource Pool :** Vous permet de placer directement votre VM dans un "pool" si vous en avez.

### 1. General - Advanced

**Start at boot :** Si cochée, la machine virtuelle démarrera automatiquement lors du démarrage / après redémarrage de votre serveur Proxmox.












