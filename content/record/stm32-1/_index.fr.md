---
archetype: "page"
lang: "fr"
title: "Programmation sur STM32: GPIO"
linktitle: "STM32: GPIO"
categories:
- "Fiches"
tags: 
- "Arm"
- "Langage C"
- "Microcontrôleur"
weight: 11
localref: True
draft: False
---

{{% notice style="info" title="Objectifs" %}}
Cette fiche vise à présenter une première utilisation des GPIO sur un microcontrôleur STM32.
À la suite de cette page, un développeur logiciel doit être capable:
- De trouver les informations nécessaires dans une datasheet pour l'utilisation des GPIO,
- De programmer les différents registres pour différentes utilisations des GPIO (LED, bouton-poussoir *etc.*)
{{% /notice %}}

{{% notice style="warning" title="Version des outils" %}}
Cette fiche utilise la version ***v1.12.0*** du logiciel [***STM32CubeIDE***](https://www.st.com/en/development-tools/stm32cubeide.html#st_description_sec-nav-tab).
Certaines variations au niveau des captures d'écrans peuvent apparaître si vous utilisez des versions différentes.
De même, la carte utilisée est la ***Nucleo-F446RE***.
{{% /notice %}}

## Sommaire
- [Sommaire](#sommaire)
- [Clignotement d'une LED](#clignotement-dune-led)
- [Références](#références)
  
## Clignotement d'une LED

Pour la première partie, on se propose de concevoir un programme faisant clignoter la LED d'une carte.
Notamment, la carte Nucleo-F446RE dispose d'une LED directement intégrée sur la carte.
Celle-ci est connectée à la pin 21 de la carte.
Cela correspond au GPIO 5 du port A. 

Les différentes informations dont nous aurons besoin par la suite sont disponibles dans deux documents.
Le [manuel utilisateur des cartes Nucleo-64](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf) (dont fait partie la Nucleo-F446RE) regroupe les différentes informations propres à la carte (caractéristiques de la carte, branchements existants *etc.*).
Le [manuel de référence des microcontrôleurs STM32 F446XX](https://www.st.com/resource/en/reference_manual/rm0390-stm32f446xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf) donne lui toutes les informations propres au microcontrôleur lui-même (bus mémoires, mémoires, registres et périphériques *etc.*).

Du point de vue d'un microcontrôleur, allumer/éteindre une LED est similaire: cela revient à configurer différents registres en mémoire pour forcer une valeur en sortie à 1/0.
Les prochaines questions s'intéressent donc à la manière de configurer un GPIO en tant que sortie.

{{% notice style="orange" title="Question 1" icon="bolt"%}}
Pour modifier les registres d'un GPIO, il est nécessaire de savoir dans quel(s) registre(s) et partie(s) de registre(s) il est nécessaire d'écrire.
En utilisant le [manuel de référence](https://www.st.com/resource/en/reference_manual/rm0390-stm32f446xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf) (Chapitre 7 *General Purpose I/Os*), repérez les valeurs de registres nécessaires pour configurer le GPIO du port A ainsi:
- MODE: *General purpose output mode*,
- TYPE: *push-pull*,
- DATA: 1.
  
On laissera pour l'instant les autres paramètres inchangés.
{{% /notice %}}

{{% notice style="orange" title="Question 2" icon="bolt"%}}
Chaque structure dispose d'un champ dédié à chaque registre du port: `MODER`,`OTYPER`,`IDR` ...
Un pointeur de structure `GPIOA` existe pour le PORTA.
On peut accéder à la valeur de chaque registre du port en utilisant l'opérateur `->`: `GPIOA->MODER`.
En utilisant ce mécanisme, initialisez les registres nécessaires pour obtenir la configuration de la Question 1 et ainsi allumer la LED verte.
{{% /notice %}}

{{% notice style="orange" title="Question 3" icon="bolt"%}}
Dans le cas où cela ne serait pas le cas, modifiez votre code de la question 2 pour maintenant utiliser des masques binaires afin de ne modifier que les bits qui correspondent à la pin de la LED.
{{% /notice %}}

{{% notice style="orange" title="Question 4" icon="bolt"%}}
Copiez votre code précédent à la suite et modifiez-le maintenant pour éteindre la LED.
Combien de registres doivent être modifiés ?
{{% /notice %}}

{{% notice style="orange" title="Question 5" icon="bolt"%}}
Généralement, allumer/éteindre une LED est une opération réalisée par le biais d'une fonction.
Cela permet de réutiliser le même morceau de code à chaque fois que l'on en a besoin.
Dans votre projet, ajoutez à présent deux fonctions `gpio_set_pin` et `gpio_reset_pin` vous permettant respectivement d'allumer ou éteindre la LED.
{{% /notice %}}

{{% notice style="orange" title="Question 6" icon="bolt"%}}
On se propose maintenant de modifier la fonction `main` pour faire clignoter notre LED à une vitesse visible.
Pour cela, l'idée va être d'appeler à tour de rôle les fonctions respectives au sein de la boucle `while(1)`.
Pour créer l'effet de clignotement, ajouter entre les appels de chaque fonction des boucles pour insérer du délai dans le temps d'exécution.
Le microcontrôleur tournant à plusieurs dizaines de MHz, pensez à ajouter suffisamment de tours de boucles entre les appels pour que le délai soit suffisant.
{{% /notice %}}

## Références

Cette partie regroupe les informations sur les {{< ref-cnt-page >}} références utilisées sur cette page.

{{< ref-list-page >}}