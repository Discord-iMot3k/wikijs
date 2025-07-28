---
title: 🖥️ Machines virtuelles
description: Cette page présente les étapes essentielles pour créer, configurer et gérer des machines virtuelles dans un environnement de virtualisation.
published: false
date: 2025-07-28T13:10:26.756Z
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

**Node :** Noeud Proxmox sur lequel votre machine virtuelle sera installée. Utile dans le cas où vous avez un cluster, pour choisir manuellement sur quelle machine hôte la VM sera hébergée.

**VM ID :** Numéro d'identification de votre machine virtuelle. Celui-ci permet à Proxmox de référencer la VM dans son système.
> **Attention :** Une fois créée, le VM ID **ne peut pas être modifiable sans passer par un clonage ou autre méthode**. Si vous appliquez une logique de nommage ou de numérotation, pensez bien à le définir **avant** la création.
{.is-warning}

**Name :** Nom de la votre machine virtuelle. Pourra être modifiée dans les paramètres même après création de la VM.

**Ressource Pool :** Vous permet de placer directement votre VM dans un "pool" si vous en avez. Pratique pour organiser vos VMs par projet, équipe, service, ...

### 1. General - Advanced

**Start at boot :** Si cochée, la machine virtuelle démarrera automatiquement lors du démarrage / après redémarrage de votre serveur Proxmox.

**Start/Shutdown order :** Définit dans quel ordre sera démarrée ou arrêtée votre VM lors du boot/shutdown de votre machine Proxmox.
`any : Aucune priorité particulière`
`Nombre entier : 1, 2, 3, ...` - Plus le chiffre est petit, plus la VM sera démarrée ou arrêtée tôt.

**Startup delay :** Défini un **délai d'attente** (en secondes) avant de démarrer cette VM, après le démarrage de votre machine Proxmox.

**Shutdown timeout :** Indique le temps maximal (en secondes) que Proxmox attendra pour que la VM s'éteigne avant de forcer l'arrêt.

**Tags :** Vous permet de créer / appliquer un tag à votre VM. Pratique pour organiser vos VMs par projet, équipe, service, ...

---

### 2. OS

**Use CD/DVD disc image file (iso) :** Si cochée, vous choisissez de d'utiliser une image .iso stockée sur votre Proxmox.
Lorsque vous choisissez cette option, vous avez la possibilité de choisir le `Storage : {Par défaut : local}` ainsi que l'`ISO Image : {Image ISO précédemment importée`.

**Use physical CD/DVD Drive :** Si cochée, vous permet d'utiliser un disque physique externe (USB, DVD, ...)

**Do not use any media :** N'utiliser aucun média. Utilisé lors de migration de VM, préparation d'une VM sans lancer d'installation, clonage, ...

**Guest OS :**

* Type : Permet de définir la "famille" de l'OS que vous allez installer Linux / Microsoft Windows / Solaris Kernel / Other.

* Version : Permet de choisir la version de la "famille" que vous souhaitez installer.

> Dans le cas où vous souhaitez installer une machine Windows / Windows Server, ajouter les VirtIO Drivers (Procédure à venir).
{.is-warning}

*Exemple de création d'une machine Debian 12*

<center>
<img src="/exemple_os_pve.png">
</center>

---

### 3. System (à compléter)

L'onglet System permet de définir les paramètres matériels virtuels que la VM va utiliser.

**Graphic card :** `Default` convient à la plupart des systèmes et ne nécessite pas de configuration particulières. C'est l'option par défaut, et la plus simple, que nous allons utiliser. 

**Machine :**

**SCSI Controller :** 

**Qemu Agent :** Coché - Permet d'installer l'agent QEMU dans votre OS. Permet à Proxmox d'avoir des informations plus précises sur la VM (IP, état de l'OS, ...).

**Firmware :**

* BIOS : 
* Add TPM : 

---

### 4. Disks - Par défaut



### 4. Disks - Advanced




