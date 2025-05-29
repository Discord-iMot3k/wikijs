---
title: Les concepts des réseaux virtuels
description: 
published: true
date: 2025-05-29T09:59:30.216Z
tags: 
editor: markdown
dateCreated: 2025-05-29T09:59:30.216Z
---

> <b><p style="text-align: center;"> Auteur : Gz - Discord imot3k 
> Tout droit réservé, reproductions et copies interdites sans autorisation </p></b>

Dans cette partie nous allons voir quelques concepts sur les réseaux virtuels et voir comment nos machines virtuelles peuvent avoir accès aux ressources, que ce soit des ressources virtuelles disponibles sur le même hôte ESXi ou des ressources directement connectées sur le réseau physique.

## Le trafic réseau des VMs

Pour commencer, posons-nous la question suivante : "**Comment les machines virtuelles gèrent l’envoi et la réception de données à travers le réseau ?**"

Une machine virtuelle fonctionne exactement de la même manière qu'une machine physique.
Tout comme n'importe quelle autre machine qui serait connectée sur un réseau, elle possède donc elle aussi un adaptateur réseau cependant dans son cas il sera évidemment virtuel lui aussi.
L'adaptateur réseau virtuel d'une VM est appelé **vNIC**.

![1.png](/1.png)

Dans cet exemple l'OS Windows qui est installé dans la VM n'aura aucun moyen de savoir qu'il s'agit ici d'une VM et d'un adaptateur réseau virtuel.
Tout ce que Windows verra c'est un adaptateur réseau.

Windows va donc envoyer des packets à la carte réseau et cette carte réseau, comme n'importe quelle autre carte réseau sera connecté à un switch.
Dans notre cas il s'agira d'un switch virtuel sur lequel seront définis et créés un ou plusieurs "**groupes de ports**". 
Ce groupe de ports, appelé "**Port Group**", sera **connecté au vNIC de la VM** et permettra de définir différentes configurations telles que des vlans, règles de sécurité, etc.

![2.png](/2.png)

Bien que notre hôte ESXi possède des cartes réseau physiques le trafic de nos VMs n'est pas nécessairement obligé de passer par ces cartes.
Si nous avons plusieurs VMs connectées au même **Port Group**, elles seront capables de communiquer entre elles sans avoir besoin de passer par le réseau physique car le trafic sera géré par le switch virtuel de l'hôte ESXi.

![3.png](/3.png)

Cependant si les VMs ont besoin de communiquer avec des machines se trouvant sur le réseau physique, que ce soit des serveurs ou tout simplement pour accéder à Internet, alors le trafic devra passer par les cartes réseau physiques de l'ESXi.
Les cartes réseau physiques d'un ESXi sont appelées "**vmnic**".
Elles sont elles aussi connectées à un vSwitch (virtual switch) mais également à un switch physique, permettant au trafic de passer sur le réseau physique.

![4.png](/4.png)

Un **vmnic** est donc ce que l'on appelle "**une liaison montante**" ou "**uplink**" pour un vSwitch et va permettre de lui fournir une connectivité avec le réseau physique.
De la même manière, les **Port Group** disponibles sur les vSwitches ne sont utilisés **que pour le trafic réseau des VMs et rien d'autre**.

## Les VMkernel Ports

Pour tout autre type de trafic réseau que celui des VMS, nous utiliserons ce que l'on appelle des "**VMkernel Ports**".
Bien qu'il s'agisse également de ports définis et créés sur un vSwitch, **aucun PC et aucun serveurs ne pourront s'y connecter**.
Leur utilisation est totalement différente de celles des Ports Groups.

![5.png](/5.png)

Ils sont utilisés par les ESXi et vCenter pour communiquer entre eux mais également pour tout ce qui ne concerne pas le trafic réseau des VMs.
Les "**VMkernel Ports**" sont des ports utilisés et dédiés à des tâches telles que:

- vMotion
- Le management
- IP Storage
- L'accès aux baies de stockage
- etc.

## Vlans et Trunk

Les vSwitch et les hôtes ESXi supportent à la fois les VLANs et les port Trunk.
Pour comprendre le fonctionnement des VLANs et des ports Trunk prenons un exemple.

La VM en haut de l'image est connectée sur un **Port Group** d'un vSwitch.
**Ce Port Group s'est vu assigner un VLAN** correspondant à la Prod et dont **l'ID est 10**.

La VM en bas de l'image est connectée sur **un autre Port Group du même vSwitch** que la première VM.
**Le Port Group sur lequel cette VM est connectée s'est quant à lui vu assigner un autre VLAN** correspondant à la section Dev de l'entreprise et dont **l'ID est 20**.

![6.png](/6.png)

Ainsi lorsque le trafic réseau entre dans le vSwitch **depuis la machine virtuelle en haut de l'image**, il passera obligatoirement par le **Port Group** identifié par le **VLAN 10**.

Cela signifie que si cette VM essaye de communiquer avec la VM en bas de l'image, le trafic réseau devra alors passer par le réseau physique à travers un switch physique puis atteindre un routeur permettant d'acheminer le trafic vers différents VLANs dont le VLAN 20 correspondant au VLAN identifié sur le Port Group du vSwitch sur lequel la VM en bas de l'image est connectée.
Si tel est le cas alors le trafic pourra passer d'une VM à l'autre et les deux VMs pourront communiquer. Dans le cas contraire, sans effectuer le routage d'un VLAN à l'autre, aucune communication ne sera possible.

Pour résumer, c'est ainsi que la segmentation par VLANs fonctionne avec un vSwitch :
Chaque VMs se connecte à un Port Group sur lequel est défini un VLAN.
Puis le trafic réseau passera par un switch physique sur lequel un port **Trunk** sera défini, lui permettant de gérer le trafic de plusieurs VLANs.
De cette façon le switch physique peut voir depuis quel vSwitch et depuis quel VLAN appartient le trafic réseau d'une VM lorsqu'il lui arrive.
Pour finir, le switch physique pourra acheminer ce trafic vers un routeur qui redirigera le trafic vers la machine ciblée.

![7.png](/7.png)

Il est donc très important de comprendre que les hôtes ESXi possèdent des **vSwitch** reliés à leurs cartes réseau, appelées **vmnic**, et que sur ces vSwitch nous avons configuré des **Port Group** qui seront eux même reliés aux **vNIC** des VMs.
Les **Port Group** permettant de fournir des VLANs et des options de sécurité sur l'ensemble des VMs qui y sont connectées par exemple.

## Points importants

Les machines virtuelles se connectent aux ressources disponibles sur le réseau en utilisant un "**vSphere Standard Switch**"
- **vNIC** (Virtual Network Interface Cards) fourni la connectivité réseau aux VMs.
- Les **Port Group** sont utilisés **UNIQUEMENT** pour le trafic réseau des VMs.
- Les **Port Group** permettent d'assigner des VLANs et des règles de sécurité.
- Les ports **VMKernel** sont des ports sur les vSwitch dédiés **UNIQUEMENT à l'ensemble du trafic réseau qui ne concerne PAS les VMs**, tels que le Management, le vMotion, le stockage, etc.

