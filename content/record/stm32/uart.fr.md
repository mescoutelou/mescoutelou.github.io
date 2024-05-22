---
archetype: "page"
lang: "fr"
title: "UART"
linktitle: "UART"
categories:
- "Fiches"
tags: 
- "Arm"
- "Langage C"
- "Microcontrôleur"
weight: 4
localref: True
draft: False
---

{{% notice style="info" title="Objectifs" %}}
Cette fiche vise à présenter une première utilise du bus UART sur un microcontrôleur STM32.
À la suite de cette page, un développeur logiciel doit être capable:
- De trouver les informations nécessaires dans une datasheet pour l'utilisation du protocole UART,
- De programmer les différents registres,
- D'envoyer des informations octet par octet vers un ordinateur.
{{% /notice %}}

{{% notice style="warning" title="Version des outils" %}}
Cette fiche utilise la version ***v1.12.0*** du logiciel [***STM32CubeIDE***](https://www.st.com/en/development-tools/stm32cubeide.html#st_description_sec-nav-tab).
Certaines variations au niveau des captures d'écrans peuvent apparaître si vous utilisez des versions différentes.
De même, deux carte ***Nucleo-F446RE*** seront utilisées.
{{% /notice %}}

## Sommaire
- [Sommaire](#sommaire)
- [Transmission/Réception UART](#transmissionréception-uart)
  - [Protocole UART](#protocole-uart)
  - [Envoi d'un octet](#envoi-dun-octet)
  - [Réception d'un octet (STM32)](#réception-dun-octet-stm32)
  - [Réception d'un octet (Ordinateur)](#réception-dun-octet-ordinateur)
- [Références](#références)

## Transmission/Réception UART

Lors de la conception d'un microcontrôleur, il est souvent utile de pouvoir échanger avec un autre système.
Par exemple, c'est le cas lorsque l'on souhaite envoyer des données vers un ordinateur responsable du traitement ou du contrôle.
Un protocole de transmission permet alors de mettre en place une interface commune pour l'échange de ces données.
Selon les applications, il existe de multiples types de protocoles.
Dans le cadre de cet exercice, nous allons nous intéresser au protocole UART.

### Protocole UART

Le protocole UART (pour *Universal Asynchronous Receiver Transmitter*) est un protocol de type asynchrone.
Cela signifie qu'il permet d'envoyer des données entre deux systèmes sans signal d'horloge commun.
Pour cela, la trame intègre des évènements permettant de détecter une émission.
De plus, l'émetteur et le récepteur doivent avoir convenu d'un début commun au préalable.

Le protocole UART est un protocole réputé pour sa simplicité.
Il ne contient que deux signaux, Tx et Rx respectivement dédiés à l'émission et à la réception.
Ainsi, d'un point de vue matériel, son implémentation peut se résumer à 2 *FSMs* chargées de mettre en forme/lire les trames.
Pour ces raisons, des contrôleurs UART sont présents dans la plupart des microcontrôleurs.

{{< fig
  fig="/fig/record/stm32/uart-frame.png"
  alt=""
  width=1000px
  caption="Exemple de trame UART"
  subcaption=""
  key="uart-frame"
>}}

Une trame UART se décompose en 5 parties:
- La valeur *IDLE* qui est la valeur par défaut (1). Tant que le signal reste à cette valeur, cela signifie qu'aucune émission n'est en cours.
- Le bit de *start* (0), qui indique le début d'une trame.
- Les bits de données qui contienne l'information transmise. La taille peut varier de 5 à 9 bits selon les implémentations.
- Le bit de parité, qui permet d'identifier de potentiel erreurs dans la donnée. Pour un bit de parité paire, cela revient à compter le nombre de bits à 1 dans la donnée.
- Le bit de stop (1) qui indique la fin d'une trame. Il y a en général entre 0 et 2 bits de stop.

Le *baud rate* définit le temps alloué à chaque bit.
Un *baud rate* de 9600 signifie que 9600 bits sont transmis par seconde.
Parmi les *baud rate* les plus communs, on a 9600, 19200, 38400, 115200, 230400 *etc.*
Cependant, il est généralement possible de configurer les systèmes pour utiliser n'importe quelle valeur de *baud rate*.
Pour cela, des diviseurs internes permettent alors de modifier l'allure de la trame selon la fréquence du système.

Pour la suite de l'exercice, on considèrera la configuration suivante:
- U(S)ART 2,
- *Baud rate*: 9600,
- *Stop bit*: 1,
- Contrôle de la parité: activé,
- Type de parité: paire (*even*).

### Envoi d'un octet

{{% notice style="orange" title="Question 1" icon="bolt" %}}
Tout d'abord, nous allons nous concentrer sur la partie émission.
Pour cela, utilisez le *Reference Manual* pour configurer l'UART et respecter la configuration ci-dessus.
Validez ensuite votre configuration par le biais d'un oscilloscope en validant la trame visuellement.
{{% expand title="Aide supplémentaire ..." %}}
En plus du mode alternatif dans les GPIO nécessaires, la configuration de l'UART concernera ici principaement 3 registres: `BRR`, `CR1` et `CR2`.
Ensuite, un registre supplémentaire `SR` nous donnera les informations sur l'état de l'UART.
{{% /expand %}}
{{% /notice %}}

### Réception d'un octet (STM32)

{{% notice style="orange" title="Question 2" icon="bolt" %}}
La configuration de la réception est partagée avec celle de l'émission: ainsi, une fois l'UART configuré, il ne reste plus qu'à activer la partie réception et tester si des trames valides ont été reçues.
Modifiez votre code pour maintenant lire en scrutatif les octets reçus.
Pour tester votre système, connectez-le à une seconde carte Nucleo.
{{% /notice %}}

{{% notice style="warning" title="Connecter deux cartes" %}}
Chaque carte Nucleo a une sortie Tx et une entrée Rx.
Pour que les cartes puissent communiquer, il est nécessaire de croiser les connexions Tx <-> Rx.
{{% /notice %}}

{{% notice style="orange" title="Question 3" icon="bolt" %}}
La réception d'une trame peut être très longue à l'échelle d'un système (des centaines ou milliers de cycles d'horloge selon la fréquence et le *baud rate*).
Ainsi, il peut être pratique d'utiliser les interruptions pour ne traiter les octets reçu que lorsque c'est nécessaire.
Pour cela, configurer les interruptions et la routine `USART2_IRQHandler(void)` pour que dès qu'un octet est reçu, l'état de la LED s'inverse et qu'un nouvel octet soit renvoyé.
{{% /notice %}}

### Réception d'un octet (Ordinateur)

{{% notice style="orange" title="Question 4" icon="bolt" %}}
Une communication UART peut également être effectuée par le biais d'un branchement USB.
Ainsi, il est tout à fait possible d'envoyer/recevoir des trames exactement de la même manière avec un PC.
Utilisez le USART2 pour communiquer avec votre ordinateur *via* le câble USB.
Pour visualiser les trames sous Windows, utilisez un script (*e.g.* Python) ou logiciel (*e.g.* IDE Arduino) de votre choix.
{{% /notice %}}

## Références

Cette partie regroupe les informations sur les {{< ref-cnt-page >}} références utilisées sur cette page.

{{< ref-list-page >}}