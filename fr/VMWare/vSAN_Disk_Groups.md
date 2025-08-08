---
title: vSAN Disk Groups
description: 
published: true
date: 2025-08-08T15:54:04.983Z
tags: 
editor: markdown
dateCreated: 2025-08-08T15:52:04.400Z
---

WORK IN PROGRESS

> <b><p style="text-align: center;"> Auteur : Gz - Discord imot3k 
> Tout droit réservé, reproductions et copies interdites sans autorisation </p></b>

# Introduction
Dans cette documentation nous verrons ensemble le concept de "**Disk Groups**", comment améliorer les performances et comment gérer la capacité de stockage dans nos "**Disk Groups**".

# Disk Groups

Voici un schéma représantant un Cluster d'ESXi dont le stockage sur chacun d'entre eux est organisé en groupes de disques (**Disk Groups**).
Nous avons des disques durs qui consituent notre **stockage permanent**.
Pour rappel, comme nous l'avons vu précédemment, ces disques sont également appellés des "**disques de capacité**".
Nous avons aussi des disques SSD qui fournissent un cache de lecture ainsi qu'une mémoire tampon pour l'écriture.
Nous l'avons aussi vu précédemment mais je rappel que ces disques sont appellés "**disques de cache**".

Dans ce type de configuration, qui offre de nombreuses options, nous pouvons avoir **jusqu'à 7 disques de capacité** par "**Disk Groups**".
De plus il est possible d'avoir jusqu'à 5 **Disk Groups** par ESXi.
Il faut savoir que si vous choisissez une configuration composée entièrement de disques SSD, les **disques de capacités** peuvent être également des SSD.
Les **Disk Groups** peuvent donc être constitués de différentes manières.

![img_1793.png](/img_1793.png)

Dans cet exemple en particulier, nous avons un "**Disk Group**" sur chacun de nos ESXi qui est composé de 6 HDD pour la **capacité** et un SSD pour le **cache**.
C'est ce que l'on appelle une **configuration hybride** puisque nous mélangeons les deux types de disques, à savoir des HDD pour la capacité et un SSD pour le cache.

La capacité totale du ou des **Disk Groups** de l'ensemble de nos ESXi est en suite **combinée pour former un Datastore vSAN**.

# Améliorer les performances

Maintenant que nous avons abordé le concept des "**Disk Groups**" permettant de créer un Datastore vSAN, comment pourrions nous améliorer les performances ?
Toujours à partir du schéma ci-dessus, que pourrions nous faire pour améliorer les performances des "**Disk Groups**" ?
Prenons par exemple l'ESXi 1. Imaginons qu'il ne fonctionne pas comme nous le souhaiterions et que la latence du stockage soit plus élevée qu'elle ne devrait l'être.

Reprennons notre schéma mais modifions le un peu.
Nous pouvons voir que sur l'ESXi 1 au lieu d'avoir 6 HDD pour la capacité, comme pour les autres ESXi, nous en avons à présent plus que 3.
Nous avons donc retiré 3 HDD parmi les "**disques de capacité**".
Avant cette manipulation, nous avions un "**Disk Group**" qui contenait 6 HDD de capacité et 1 SSD pour le cache.
A présent nous avons un "**Disk Group**" contenant 3 HDD pour la capacité et 1 SSD pour le cache.





Afin d'améliorer les performances ce que nous pouvons faire c'est tout simplement procéder à la création d'un second "**Disk Group**".
Nous allons donc créer un second "**Disk Group**" qui possèdera son propre SSD pour le cache et dans lequel nous allons intégrer les 3 HDD de capacité précédemment retiré du premier "**Disk Group**".
Cela nous permettra donc d'avoir deux "**Disk Group**" ayant chacun 1 SSD de cache et 3 HDD de capacité.





Mais quel est l'avantage de cette approche ?
En réalisant cette opération ce que nous avons en réalité fait c'est **augmenter le ratio de SSD par rapport aux HDD**. 
Nous avons donc plus de SSD pour le même nombre de HDD, ce qui signifie qu'il fort probable que les données que nous souhaitons lire dans nos VMs soient mises en cache dans le cache de lecture des SSD.
En effet, tout ce que nous pourrons mettre en place pour améliorer le ratio entre le cache et la capacité améliorera forcément les performances.

La taille du cache qui est recommandée dans les Best Practices est de 10% de la taille de la capacité.
Par exemple, si nous avons 1To de capacité alors la taille recommandée pour le cache sera de 100Go (soit 10%).

> Notez cependant que si la taille du cache est supérieur à 10%, les performances en seront encore meilleures.{.is-info}

# Ajout de capacité de stockage

Si nous avons besoin d'**augmenter la capacité de stockage de notre Datastore vSAN**, il nous suffira de créer d'avantage de **Disk Group** sur nos hôtes.
Cela nous permettra d'ajouter des disques de capacité et de cache supplémentaires et ainsi d'avoir un Datastore vSAN plus volumineux.
Pour rappel chaque ESXi peut avoir **jusqu'à 5 "Disk Group**" différents.




Nous pourrions également ajouter plus d'ESXi à notre Cluster vSAN puisque le stockage de chacun des nouveaux ESXi ajoutés pourra être ainsi ajouté au Datastore vSAN.
Il existe donc plusieurs manière d'augmenter la taille du Datastore vSAN.




Cependant dans cet exemple, il y a tout de même quelque chose qui doit nous interpeller et que nous devons éviter de faire.
Si l'on remarque bien, l'ESXi 4 qui a été ajouté possède une capacité de stockage légèrement inférieure à celle des autres ESXi du Cluster.
Cet ESXi possède un seul "**Disk Group**" tandis que les autres en possèdent deux. Cela va à l'encontre de ce que nous voulons faire avec un cluster vSAN.




En règle générale, nous devons maintenir une configuration **la plus identique et la plus cohérente possible dans l'ensemble du Cluster**.
Dans notre exemple les ESXi 1 à 3 possèderont plus de données et plus d'objets de VMs que l'ESXi 4.
Ils effectueront donc plus d'opérations et leur charge de travail sera donc plus élevée.
L'équilibre au sein du Cluster ne sera pas effectué car l'ESXi 4 aura moins de charge de travail que les autres.
L'ensemble des ressources des ESXi 1 à 3 seront plus solicitées que celles de l'ESXi 4.

L'autre problème que nous pouvons évoquer dans cet exemple est le fait que nous ayons 3 ESXi surdimensionnés par rapport au quatrième.
Si l'un d'entre eux devait tomber en panne, l'impact serait très important par rapport à une sitution où toutes les ressources des ESXi seraient de taille égale.
Avec des ressources identiques sur l'ensemble des ESX, il est plus facile de se préparer et de prévoir les pannes.

# vSAN Datastore

Après avoir respecté l'ensemble des éléments évoqués précédemment ainsi que l'ensemble des bonnes pratiques, nous nous retrouvons donc au final avec 4 ESXi ayant chacun 2 "**Disk Group**" composés de 6 HDD pour la capacité et 1 SSD pour le cache.
Tout ce stockage va être rassemblé et combiné afin de créer un **stockage partagé** pour l'ensemble des ESXi du Cluster vSAN.
Ce stockage partagé sera représenté par un seul gros Datastore appelé "**vSANDatastore**" qui représentera toute la capacité de **stockage physique** des ESXi





Tout la capacité du stockage physique ? oui et non ...
En effet **seuls les disques que nous aurons sélectionnés** seront utilisés pour créer le Datastore vSAN.
Nous pourrions très bien choisir de ne pas utiliser l'ensemble des disques physiques et en reserver quelques-uns pour d'autres utilisations.