---
title: vSphere Standard Switches
description: 
published: true
date: 2025-05-30T21:23:40.492Z
tags: 
editor: markdown
dateCreated: 2025-05-30T21:20:55.247Z
---

> <b><p style="text-align: center;"> Auteur : Gz - Discord imot3k 
> Tout droit réservé, reproductions et copies interdites sans autorisation </p></b>

Dans cette partie nous allons voir certaines fonctions des "**vSphere Standard Switches**".
Nous allons principalement voir la façon dont le "**nic teaming**" fonctionne et comment nous pouvons configurer les "**vmnic**" de nos ESXi pour tolérer les problèmes réseau.

## vmnic Link State Failure

Les **vmnic** sont les cartes réseau physiques des hôtes ESXi et chacun des **vmnic** ne peut être attribué qu'à **un seul vSwitch**.
Cela implique donc que les **vSwitch ne peuvent PAS partager** une carte réseau physique d'un ESXi.

Reprenons un exemple.
Sur cette Image nous voyons un vSwitch relié à 3 **adaptateurs physiques de notre ESXi (ou VMNICs)** et le réseau est dans un état stable. Les 3 VMNICs sont connectés et la lumière verte habituelle est visible sur chacun d'entre eux.

![8.png](/8.png)

Le trafic de la VM passe actuellement par le premier vmnic.


![9.png](/9.png)

Admettons que quelque chose se passe et que le câble soit débranché ou même directement coupé.
A présent, la connectivité depuis ce vmnic n'est plus possible et la lumière verte qui indique un état stable n'est plus présente.

![10.png](/10.png)

A ce stade, la connexion a été **physiquement interrompu**.
Ce genre de problème peut être facilement détecté par un ESXi et si cela devait arriver alors l'ESXi redirigerai simplement le trafic de la VM vers une autre de ses cartes réseau physiques (vmnic) pour assurer la connectivité.

![11.png](/11.png)

## Upstream Device Failure

Parlons à présent d'un problème un peu plus compliqué à l'aide de l'exemple suivant.

![12.png](/12.png)

Dans cet exemple, toutes les cartes réseau physiques de l'ESXi (vmnic) sont fonctionnelles.
Le trafic réseau de notre VM passe par le 3ème vmnic puis passe à travers les deux Switches physiques pour arriver sur Internet.

![13.png](/13.png)

Imaginons à nouveau qu'il arrive un problème sur un câble mais cette fois-ci sur celui qui assure la connectibité entre les deux Switches physiques.
Ce problème est différent du premier car l'état du **vmnic** par lequel passe le trafic réseau de la VM reste inchangé : **la carte réseau est UP**.
La VM n'a pourtant plus accès à Internet.

![14.png](/14.png)

Si l'on regarde attentivement notre schéma, le trafic de notre VM passe par un Switch qui est totalement isolé. Il n'a plus aucune connectivité avec un autre **Switch physique** et par conscéquant n'a plus non plus de connectivité avec Internet.
Dans ce cas présent, **le vmnic 3 est dans une impasse** et n'a plus aucun accès au réseau physique.

C'est un ce moment là que va intervenir un mécanisme appelé "**Beacon probing**".
Il s'agit d'un **mécanisme de failover réseau**.
Chacune des cartes réseau physiques faisant parties du "**Nic Teaming**" va envoyer aux **autres cartes réseau physiques**, **par l'intermédiaire du réseau physique**, des paquets appelés "**Beacon probes**" afin de vérifier que chacune d'entre elles fonctionne correctement.

Dans notre schéma, les deux premiers VMNICs sont capables de comminiquer entre eux.
Cependant le 3ème envoie des "**Beacon probes**" que **les autres VMNICs ne parviennent pas à voir** dû au "**Upstream Device Failure**".
Ses paquets restent "bloqués" dans le Switch physique qui ne possède pas de liaison permettant la communication avec les autres VMNICs.

![15.png](/15.png)

Lorsque l'ESXi va réaliser qu'il ne parvient pas à recevoir des paquets "**Beacon probes**" en provenance du 3ème vmnic, il va simplement supprimer ce vmnic du service et **le trafic de la VM sera redirigé vers un autre vmnic**.

![16.png](/16.png)

## Nic Teaming options

Parlons à présent de certaines options de **Nic teaming** (association de cartes réseau).
Dans le schéma suivant, nous avons plusieurs **VMNICs**" connectés à un vSwitch.

![17.png](/17.png)

Nous voulons faire en sorte que le trafic réseau de nos machines virtuelles puisse utiliser toutes les cartes réseau physiques de notre ESXi (**VMNICs**).
Nous allons donc devoir configurer une méthode de **Nic Teaming** pour permettre à ces adaptateurs physiques d'équilibrer la charge du trafic réseau des VMs (**load balancing**).

### Nic Teaming by Originating Port ID

La première méthode pour effectuer ce **Load Balancing** s'appelle "**Nic Teaming by Originating Port ID**".
Le fonctionnement est assez simple. Chaque machine virtuelle sera connectée à un port virtuel sur le vSwitch et **en fonction de l'ID de port virtuel** auquel la machine virtuelle sera connectée, tout son trafic réseau passera par un **vmnic** spécifique.

![18.png](/18.png)

Dans cette configuration, chacune des VMs est essentiellement "attachée" à une carte réseau physique spécifique et tout le trafic réseau de ces VMs passera par cette carte réseau physique.
C'est ainsi que le "**Nic Teaming by Originating Port ID**" répartit la charge de travail sur l'ensemble des cartes réseau physiques d'un ESXi

> Dans cette configuration il est particulièrement important de comprendre qu'il ne faut **surtout PAS** configurer le switch physique avec du "Port Channel" ou du "LACP" par exemple. Le switch physique doit voir 4 ports et donc 4 connexions complètement indépendantes les unes des autres.
{.is-warning}

### Nic Teaming by Source MAC Hash

L'utilisation de "**load balancing**" à l'aide de "**Source MAC Hash**" est assez similaire à la méthode précédente cependant au lieu de se baser sur l'ID d'un port virtuel du vSwitch, **nous allons utiliser les adresses MAC des VMs**.

Imaginons que la VM1 possède une adresse MAC **unique** dans notre environnement vSphere et que cette adresse MAC soit appellée "MAC1".
**Sur la base de cette adresse MAC unique**, cette VM sera alors associée à un **vmnic** spécifique.
Il en est de même pour les VM2 et VM3, sur la base de leur adresse MAC elles seront également associées à un vmnic spécifique.

L'intégralité du trafic réseau de ces VMs qui devra quitter l'hôte ESXi passera alors par les vmnic associés.

![19.png](/19.png)

> **Attention** dans cette configuration il est également impératif de ne **surtout PAS** configurer le switch physique avec du "LACP", "Port Channel" ou "EtherChannel" par exemple.{.is-warning}

### Nic Teaming by IP Hash

la dernière méthode de **Nic Teaming** supportée par un "**Standard Virtual Switch**" est appelée "**IP Hash**"
Dans le schéma suivant, nous pouvons voir que nous avons une machine virtuelle avec une adresse IP.

![20.png](/20.png)

Lorsque cette machine virtuelle va envoyer du trafic réseau vers une destination précise possédént une adresse IP unique, ce trafic va passer par une carte réseau physique qui sera **sélectionnée** en fonction **non seulement de l'adresse IP source mais également sur l'adresse IP de destination**.

Donc si cette même VM envoie du trafic réseau vers **une autre destination** précise possédant une adresse IP unique **différente de la première destination**, le trafic réseau passera alors par **une carte réseau physique différente**.

Cette méthode est différentes des précédentes car la VM de notre schéma peut utiliser différents VMNICs pour acheminer son trafic réseau su le réseau physique.

![21.png](/21.png)

> **Attention** contrairement aux méthodes précédentes, ici nous devons configurer correctement le switch physique en y paramètrant du "LACP" ou du "Port Channel" par exemple afin de combiner les cartes réseau ensemble.{.is-warning}

## Traffic Shaping

Une autre fonctionnalité intéressante prise en charge sur le "**vSphere Stantard Switch**" est ce qu'on appelle le "**Traffic Shaping**".
Le "**Traffic Shaping**" permet d'appliquer des paramètres afin de définir une **vitesse de bande passante maximale**, une **vitesse de bande passante moyenne** et ce qu'on appelle "**Burst Size**", ce qui correspond en réalité à une **taille maximale de données autorisées à être envoyées en utilisant la vitesse de bande passante maximale**.

Par exemple, imaginons que nous avons un groupe de VMs connectées à un Port Group sur un vSwitch.
Il peut arriver parfois que nous effectuons des tests de nouvelles applications sur ces VMs ou d'autres choses nécessitant beaucoup de ressources et notament de la bande passante.
Cela entrainerai alors une tendance pour ces VMS à acaparer la bande passante du vSwitch au détrimant d'autres VMs sur d'autres Port Groups qui ne pourraient pas avoir la bande passante dont elles ont besoin.

Ce que nous pouvons faire pour éviter ce problème c'est utiliser le "**Traffic Shaping**" sur un Port Group pour indiquer que chaque VM qui y sera connectée aura une bande passante maximale de 100Mbs.

![22.png](/22.png)

Et que chaque VM de ce Port Group aura une bande passante moyenne (**Average bandwidtch**) de 50Mbs qu'elle ne pourra pas dépésser en temps normal.

![23.png](/23.png)

Cependant si nous avons défini une vitesse de bande passante moyenne que les VMs ne peuvent pas dépasser pourquoi avoir défini une vitesse maximale ?

Imaginons que ces VMs ont besoin d'envoyer certains fichiers assez volumineux.
Pendant une courte période, ces VMs pourront utiliser la vitesse maximale autorisée (**Peak bandwidtch**) correspondant à 100Mbs jusqu'au moment où elles auront attend la taille définie par le "**Burst Size**". 
Une fois cette taille de données envoyées atteinte, elles seront forcées d'utiliser à nouveau une bande passante de 50Mbs.

Par exemple, nous avons défini la vitesse moyenne à 50Mbs, la vitesse maxiale à 100Mbs et le "**Burst Size**" à 500Mb.
Nos VMs vont utiliser la bande passante à une vitesse de 50Mbs mais lorsque l'une d'entre elles aura besoin d'envoyer un fichier volumineux, elle pourra alors utiliser la bande passante à une vitesse de 100Mbs jusqu'à avoir atteint un envoie de données de 500Mb. Une fois ce quota atteint elle redescendra **automatiquement** à une vitesse moyenne inférieure ou éggale à 50Mbs jusqu'à ce qu'elle soit de nouveau autorisée à utiliser la vitesse de 100Mbs.

> La configuration de la vitesse moyenne, maximale et de la taille du Burst Size se fait au niveau du switch. En aucun cas au niveau des VMs. Cela veut donc dire que chaque VM qui sera connectée sur un Port Group sur lequel est appliqué le "Traffic Shaping" se vera alors appliquer cette stratégie de restriction de bande passante selon la configuration définie.{.is-warning}

## Security Settings

Il est possible de configurer des règles de sécurités qui peuvent être configurées au **niveau des vSwitch et/ou des Port Group**.
Les règles de sécurité sur un vSwitch sont considérées comme "**Global Settings**" et s'appliquerons à l'ensemble des Port Group créés sur ce vSwitch.
Cependant si nous éditons les règles de sécurité sur un Port Group alors ces règles remplacerons celles définie au niveau du vSwitch.
Les règles positionnées sur un Port Group d'un vSwitch sont toujours prioritaires par rapport à celles positionnées sur un vSwitch.

### Forged Transmits

Un des paramètres que l'on peut modifier s'appelle "**Forged Transmits**".
Prenons une machine virtuelle pour laquelle nous allons avoir besoin de modifier son adresse MAC, comme par exemple une machine physique qui a été virtualisée et sur laquelle une application dont la licence est basée sur l'adresse MAC est installée.
Pour éviter tout problème de licence nous allons avoir besoin de garder l'adresse MAC que nous avions lorsqu'il s'agissait d'une machine physique.

![24.png](/24.png)

C'est un bon exemple d'utilisation pour faire du **MAC Spoofing**.
Dans ce cas lorsque cette VM va générer du trafic, c'est l'adresse MAC que nous avons spécifié dans le système d'exploitation de la VM qui sera utilisée.

Malheureusement le vSwitch ne va pas aimer cette manipulation puisqu'il s'attend à ce que le trafic sur ce port provienne de l'adresse MAC du **vNIC** de la VM.

Le fait de générer du trafic avec une adresse MAC différente de celle du vNIC de la VM s'appelle du "**Forged Transmits**". 
Cette manière de faire du **MAC Spoofing** ne concerne que le trafic **sortant (outbound)** de la VM et non celui entrant dans la VM.

Il s'agit d'une option que l'on peut **choisir d'activer ou de désactiver**.
Si cette option est activée alors le trafic sera accepté et pourra passer par le vSwitch mais si cette option est désactivée, le trafic sera rejeté par le vSwitch et la communication ne se fera pas.

> Par défaur, les vSwitches sont configurés pour rejeter ce type de trafic.{.is-warning}

![25.png](/25.png)

### MAC Address Changes

"**MAC Address Changes**" est à peu prêt le même genre de configuration que précédement mais il concerne le trafic entrant (**inbound**) et à destination de notre VM. 

Imaginons que le trafic réseau soit à destination de notre VM et que l'adresse MAC de destination soit une adresse MAC que nous ayons configuré dans le système d'exploitation de la VM et qu'elle ne corresponde pas à l'adresse MAC du vNIC de la VM.
Pour que le trafic puisse attendre sa destination il faut alors configurer le "**MAC Address Changes**" sur "autorisé".

![26.png](/26.png)

> Par défaur, les vSwitches sont configurés pour rejeter ce type de trafic.{.is-warning}

### Promiscuous Mode

Le dernier paramètre de sécurité qui peut être configuré est le "**Promiscuous Mode**".
Ce mode permet de "sniffer" **tout le trafic** sur un vSwitch.
Evidement ce n'est pas recommandé de laisser activé ce paramètre tout le temps mais plutot à la demande lorsqu'on en a besoin car cela représenterai un risque de sécurité.

De ce fait les bonnes pratiques recommandent d'activer cette option lorsqu'on en a besoin puis de la désactiver.

> Comme pour les configuration précédentes, par défaur les vSwitches sont configurés pour rejeter ce type de trafic.{.is-warning}

## Multiple TCP/IP Stacks

vSphere6 a introduit la notion de "**Multiple TCP/IP Stacks**" qui est également présent dans l'environnement vSphere7.
Les "**TCP/IP Stacks**" ont pour but de fournir un DNS et une passerelle par défaut.

La **Default Gayteway (passerelle par défaut)** est utilisée lorsque le trafic est à destination d'un autre réseau que celui de la VM. Disons par exemple que l'on se connecte sur une VM puis que l'on souhaite aller sur n'importe quel site Web. Ce trafic est à destination d'Internet et nécessite donc de cibler dans un premier temps un périphérique capable de faire du routage vers un reseau distant. Ce périphérique sera la passerelle par défaut.

Il n'est pas rare en entreprise d'avoir plusieurs réseaux utilisés par plusieurs VMs et dans ce cas nous allons avoir besoin d'utiliser plusieurs passerelles par défaut dans notre configuration réseau. C'est là qu'interviennent les "**TCP/IP Stacks**" puisqu'elles nous permettent d'en configurer.

### Default TCP/IP Stacks

![27.png](/27.png)

la "**Default TCP/IP Stacks**" est utilisée pour **le management et tout autre trafic par défaut présent dans un hôte ESXi**.
Il est biensur possible de la configurer afin de définir son utilisation.

![28.png](/28.png)

### vMotion TCP/IP Stacks

Nous pourrions également avoir besoin de créer d'autres **TCP/IP Stacks** selon des besoins bien définis. **Ces stacks sont totalement séparées les unes des autres y compris de celle par défaut**.

Prenons par exemple le **vMotion**.
Créer une Stack dédiée au vMotion très pratique si l'on a besoin d'envoyer tout le trafic de vMotion sur un réseau particulier pour éviter de saturer celui de la prod par exemple ou si l'on a besoin d'effectuer du vMotion sur une longue distance.

Donc en créant une TCP/IP Stack dédiée à vMotion, nous pouvons rediriger tout le trafic du vMotion vers un autre passerelle par défaut et un autre serveur DNS.

![29.png](/29.png)

### Provisioning TCP/IP Stacks

De la même manière que pour le vMotion, nous pouvons également créer une Stack dédiée au **Provisioning** qui permettrai par exemple d'effectuer des migrations à froid, le cloning, les snapshots en utilisant un réseau dédié pour cela. 

![30.png](/30.png)

### Custom TCP/IP Stacks

Nous pouvons également définir des "**TCP/IP Stacks" personnalisées** pour y définir quel type de trafic sera redirigé sur le réseau défini dans cette Stack.

## Points importants

- Un vSwitch peut être protégé contre les pannes de carte réseau physique à l'aide de la "**détection de l'état des liens**" ou des "**Beacon probes**"
- **NIC Teaming** permet de faire du **load balancing** de manière différentes selon la configuration
	- **Nic Teaming by Originating Port ID** : Les VMs sont associées à un vmnic en fonction de l'ID du port virtuel sur lequel elle sont connectées.
  - **Nic Teaming by Source MAC Hash** : Les VMs sont associées à un vmnic en fonction de leur adresse MAC.
  - **Nic Teaming by IP Hash** : Basé sur l'IP source et l'IP de destination, permettant aux VMs d'utiliser plusieurs VMNICs.
- **Traffic Shaping** : Configuré sur un Port Group et permet d'effectuer un contrôle de la bande passantes des VMs qui y sont connectées.
- **Multiple TCP/IP Stacks** : Plusieurs passerelles par défaut peuvent être configurées et utilisées pour différents types de trafics réseau

