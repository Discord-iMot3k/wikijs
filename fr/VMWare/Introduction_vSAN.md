---
title: Introduction au vSAN
description: 
published: true
date: 2025-08-08T15:15:59.626Z
tags: 
editor: markdown
dateCreated: 2025-08-08T14:31:14.649Z
---

work in progress

> <b><p style="text-align: center;"> Auteur : Gz - Discord imot3k 
> Tout droit réservé, reproductions et copies interdites sans autorisation </p></b>


# Introduction

Dans cette documentation, nous allons voir ensemble quelques éléments de l'architecture de base de vSAN mais pour cela nous allons commencer par l'élément le plus basique : le Cluster d'hôtes.

# Cluster d'hôtes

Un Cluster n'est simplement rien d'autre qu'un regroupement **logique** d'hôtes ESXi.
Un tel regroupement présente de nombreux avantages mais nous n'entrerons pas dans le détail ici.
Je peux cependant indiquer quelques exemples particulièrement intéressants.

Imaginons que nous ayons un groupe d'hôtes ESXi et que nous voulions permettre à nos machines virtuelles de basculer automatiquement d'un ESXi à un autre en cas de défaillance de l'ESXi qui les héberge. Nous aurons alors besoin d'un Cluster d'hôtes permettant ce qu'on appelle la haute disponibilité (**High Availability ou HA** en angais).
En effet, pour mettre en place ce mécanisme nous auront **obligatoirement** besoin d'un Cluster d'hôtes, sans lui les VMs ne basculeront pas d'un ESXi à un autre automatiquement.

Nous aurons également besoin d'un Cluster pour utiliser le **DRS (Distributed Resource Scheduler)**.
C'est précisément à l'aide du **DRS** que nous pourrons avoir des VMs qui seront automatiquement basculées d'un ESXi à un autre dans un but d'équilibrage de charge grâce au **vMotion**.

Une autre fonctionnalité nécessitant **obligatoirement** un Cluster d'ESXi est l'utilisation de vSAN. C'est d'ailleurs celle-ci fonctionnalité qui va nous intéresser ici et dont nous allons parler.
Pour l'utiliser, la toute première étape consistera donc à créer un Cluster d'hôtes.

![img_1774.png](/img_1774.png)

# vSAN Network

Créer un Cluster d'hôtes n'est pas le seul prérequis pour utiliser vSAN.
Il y a quelques conditions préalables à remplir : 

- Avoir la bonne version de vSphere
- Avoir la bonne version de vCenter
- Avoir du matériel physique compatible
- Avoir configuré au moins un VMKernel Port sur chacun des ESXi

Prenons par exemple le schéma ci-dessous.

![img_1775.png](/img_1775.png)

Sur chacun de ces ESXi nous pouvons voir que nous avons plusieurs éléments à disposition.
Commençons par nous concentrer sur "ESX 1".
Cet ESXi possède deux VMNICs, pour rappel il s'agit de port Ethernet **physique** sur l'ESXi.
Il possède donc deux adapatateurs Ethernet physiques et pour notre exemple nous allons supposer qu'il s'agit de ports 10Gbs et que chacun de ces ports soit connecté à un switch différent.

![img_1776.png](/img_1776.png)

En regardant le schéma nous pouvons dire la même chose pour "les ESXi 2" et "ESXi 3"
Ils sont configurés de manière identique.
Ils ont donc tous deux ports Ethernet physiques de 10gbs (vmnic0 et vmnic1) connectés sur deux switches différents.

![img_1777.png](/img_1777.png)

Sur chacun de ces ESXi nous avons créé un VMKernel Port et nous l'avons configuré de manière à ce qu'il soit utilisé pour le trafic vSAN.

Pour résumer il s'agit d'un port que nous avons créé, auquel nous avons configuré une adresse IP et qui sera utilisé pour l'ensemble du trafic réseau qui concernera vSAN.
Par exemple si une machine virtuelle a besoin de transmettre du trafic lié à vSAN d'un hôte à l'autre alors ce port sera utilisé.
Je vous invite toutefois à consulter la documentation.

Nous devons donc **obligatoirement** avoir cette interface réseau pour que vSAN puisse fonctionner correctement.

![img_1778.png](/img_1778.png)

Je tiens à préciser que la manière dont le réseau a été mis en place sur le schéma fait parti de ce que l'on appelle "les bonnes pratiques". J'entends par là que le réseau lui même est redondé via l'utilisation de deux switches différents qui sont reliés à chacun des ESXi.
Si l'un des switches tombe en panne, le réseau fonctionnera toujours grâce au second switch.

![img_1779.png](/img_1779.png)

De plus, ici, il s'agit de switches dédiés au vSAN, rien d'autres n'est connecté dessus.
Seuls les ESXi y ont accès.

# Storage of VM Objects

Petit rappel, dans le cas d'une utilisation de vSAN une VM est décomposée en une série d'objets qui sont stockés localement sur le **stockage physique des ESXi d'un même Cluster**.
Nous allons à présent voir comment sont stockées les objets des machines virtuelles et comment les VMKernel Ports entrent en jeu.
Reprenons notre schéma mais ajoutons une VM appellée "**VM1**", stockée sur le vSAN.

![img_1780.png](/img_1780.png)

Lorsque cette VM effetuera des actions de lecture ou d'écriture, les commandes SCSI qu'elle enverra seront poussées **sur le réseau physique au travers du VMKernel Port** vers l'hôte de destination.

![img_1781.png](/img_1781.png)

Nous pouvons voir ici que le VMDK principal de notre VM est stockée sur l'ESXi 2 mais que l'ESXi 3 en possède également une copie. Cette copie est en réalité un miroir, parfaitement identique, juste au cas où le VMDk principal se trouverai sur un hôté défaillant.

![img_1782.png](/img_1782.png)

Le VMKernel Port utilisé pour vSAN est donc là pour **gérer l'ensemble du trafic** qui va devoir circuler sur le réseau vSAN.
Le Compute de la machine virtuelle est géré par un hôte tandis que son disque virtuel se trouve sur un autre hôte.
C'est pourquoi lorsque la VM voudra effectuer des commandes SCSI vers sur disque virtuel, elle utilisera un VMKernel Port dédié au vSAN pour envoyer ce trafic sur le réseau.

# Read and Write Buffer
## Read

Avec un peu de chance, ce qui arrivera c'est que la majorité des opérations de lecture se fera depuis le **cache** mis en place dans vSAN.
Regardons le schéma suivant pour comprendre.
Ce que nous pouvons voir ici est ce que l'on appelle une **configuration hybride**.
Mais qu'est ce que ça veut dire concrètement ?

![img_1783.png](/img_1783.png)

Sur chacun de nos ESXi nous disposons de périphériques de stockage traditionnels, autrement dit des HDD.
Dans le cas d'un vSAN, ces disques sont appelés "**disques de capacité**" (qu'on appelle ausso des disques de capa pour aller plus vite). Ils sont représentés dans le schéma en rose.
Nous avons également des SSD, appelés "**disques de cache**", qui sont beaucoup plus rapide que des HDD.

![img_1784.png](/img_1784.png)

Nous avons alors sur chacun de nos hôtes deux types de disques. Un pour la capacité et un pour le cache.
Les "**disques de capacité**" sont présents pour stocker un grand nombre et un maximum de données possibles tandis que les "**disques de cache**" eux ne sont là que pour du cache.

Voyons ensemble ce qui se passe lorsque notre machine virtuelle VM1 veut lire des données sur son disque virtuel.
Le VMKernel Port va être utilisé pour transmettre la demande de lecture sur le réseau physique permettant ainsi d'atteindre l'hôte de destination **où se trouve le VMDK actif**.
Lorsque la trame arrivera sur l'hôte de destination elle passera également par le VMKernel Port dédié au vSAN puis sera acheminée sur un des "**disques de cache**".

![img_1789.gif](/img_1789.gif)

La lecture de la donnée se fera ainsi **très rapidement** depuis le SSD.
L'objectif des "**disques de cache**" est de **stocker les données les plus fréquemment lues** afin de pouvoir y accéder très rapidement.
70% de la capacité de stockage des "**disques de cache**" est consacrée au cache de lecture.
Une copie de toutes les données les plus fréquemment lues se trouve donc sur le ou "**disques de cache**" mais également sur les "**disques de capacité**" (HDD).
Ainsi la plus part du temps lorsque l'on cherche uniquement à lire des données, elles sont généralement lues depuis les SSD de cache permettant un accès très rapide.

Si les données ne sont pas présentes sur les SSD, alors on appelle ça un "**cache miss**".
Dans ce cas de figure la lecture des données se fera depuis les "**disques de capacité**" mais de manière plus lente.

![img_1790.gif](/img_1790.gif)

De ce fait dans ce type de configuration que l'on appelle **hybride** puisqu'elle est composée d'un mixe de HDD et de SSD, l'utilisation des "**disques de capacité**" (HDD) en lecture sera toujours plus lente que celle des "**disques de cache**" (SSD).

## Write

Jusqu'à présent nous avons parlé de lecture mais qu'en est-il des écritures ?
Que se passe-t-il sinotre machine virtuelle doit écrire des données sur le disque ?

La première chose à prendre en compte est le fait que notre VM possède plusieurs copies de son ou ses VMDK.
Elle possède une copie sur ESXi 2 mais pour prévenir toute défaillance de cet ESXi elle en possède également une copie qui est mise en miroir sur ESXi 3.
De cette manière si notre ESXi 2 tombe en panne, les données de la machine virtuelle VM1 ne seront pas perdues.

![img_1787.png](/img_1787.png)

Lorsque VM1 va émettre des trames en écritures sur son ou ses disques, ce qui va se passer c'est que la commande SCSI pour écrire sur le ou les disques sera alors envoyée **à chacun des ESXi qui en possède une copie**.
Autrement dit dans notre cas VM1 enverra ses commandes sur ESXi 2 et ESXi 3.
Elle sera donc "**mirrored**".
Si vous êtes familier avec le Raid, c'est très similaire à la façon dont les écritures sont mises en miroir dessus.

De cette manière les deux ESXi possèdent toujours une version à jour du ou des VMDK, juste au cas où l'un d'entre eux tomberait en panne.

![img_1788.png](/img_1788.png)

La seconde chose à prendre en compte, c'est la destination des commandes d'écriture.
Si vous regardez bien le gif au dessus, les commandes atteignent en premier les disques SSD.
C'est ce que l'on appelle le "**Write Buffer**".
En effet, à chaque fois qu'une VM hébergée sur un vSAN a besoin d'écrire des données, ces écritures se feront dans le "**Write Buffer**" qui se trouve sur les SSD de cache.
30% des disques de cache est donc dédiée au "**Write Buffer**".
Cela permettra également d'effectuer des écritres rapidement.
Du point de vue de la VM, une fois que les commandes d'écritures sont arrivées sur le "**Write Buffer**" c'est fini l'écriture est faite.
Ensuite, au niveau du back-end, les données sont écrites depuis le "**Write Buffer**" vers les "**disques de capacité**".

De cette manière nos machines virtuelles ont toujours l'impression d'écrire sur un disque SSD.
Les vitesses d'écriture sont toujours très rapides et, après coup, vSAN s'occupe de l'écriture depuis les SSD vers les HDD.




# Review

Pour résumer : 

- vSAN **ne peut être activé que** sur un Cluster d'ESXi.
- Les ESXi d'un Cluster vSAN **doivent tous avoir un VMKernel Port dédiée au trafic vSAN**.
- Le VMKernel Port dédiée au trafic vSAN fera transiter **l'ensemble du trafic réseau** pour le vSAN
- Les objets de nos VMs sont **"découpés" (striped) et mis en miroir** sur les ESXi pour assurer une continuité de service en cas de panne d'un hôte.
- Un **cache de lecture et un cache d'écriture** est utilisé pour améliorer les performances.
- Le **back-end va se charger de transférer les données** depuis le cache d'écriture vers les disques de capacité.




