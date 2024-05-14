---
archetype: "page"
lang: "fr"
title: "Timer"
linktitle: "Timer"
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
Cette fiche vise à présenter une première utilisation des timers sur un microcontrôleur STM32.
À la suite de cette page, un développeur logiciel doit être capable:
- De trouver les informations nécessaires dans une datasheet pour l'utilisation des timers,
- De programmer les différents registres pour différentes utilisations des timers (LED, bouton-poussoir *etc.*),
- D'adapter le fonctionnement d'un timer à l'utilisation d'un capteur externe.
{{% /notice %}}

{{% notice style="warning" title="Version des outils" %}}
Cette fiche utilise la version ***v1.12.0*** du logiciel [***STM32CubeIDE***](https://www.st.com/en/development-tools/stm32cubeide.html#st_description_sec-nav-tab).
Certaines variations au niveau des captures d'écrans peuvent apparaître si vous utilisez des versions différentes.
De même, la carte utilisée est la ***Nucleo-F446RE***.
{{% /notice %}}

## Sommaire
- [Sommaire](#sommaire)
- [Détection d'un obstacle](#détection-dun-obstacle)
  - [Clignotement régulier d'une LED](#clignotement-régulier-dune-led)
  - [Lecture d'un capteur HC-SR04](#lecture-dun-capteur-hc-sr04)
  - [Conception d'une application](#conception-dune-application)
- [Communication avec une télécommande infrarouge](#communication-avec-une-télécommande-infrarouge)
- [Références](#références)


## Détection d'un obstacle

Pour comprendre le fonctionnement des timers et leur intérêt, nous allons étudier l'implémentation d'une application de détection d'obstacle.
À l'aide d'un capteur de distance à ultrason, l'objectif sera de faire clignoter une LED plus ou moins rapidement selon la distance de l'obstacle.
La mise en place de cette application se décomposera en trois parties:
1. L'utilisation du timer TIM2 pour faire clignoter de manière régulière la LED,
2. La lecture de la distance à partir d'un capteur HC-SR04 en utilisant le timer TIM3,
3. La conception finale de l'application.
  
### Clignotement régulier d'une LED

Lors des exercices précédents, nous avons utilisé différentes stratégies pour faire clignoter une LED:
- Inversion lors de l'appui sur un bouton poussoir (scrutation),
- Inversion lors de l'appui sur un bouton poussoir (interruption),
- Boucle d'exécution avec l'ajout d'un délai avant l'inversion.

Si toutes ces solutions étaient fonctionnelles, aucune ne permettait d'avoir une prise en compte précise du temps.
Il n'était donc pas possible de réaliser un clignotement régulier et d'une durée parfaitement maîtrisée.
Pour cela, nous allons donc utiliser les timers mis à disposition au sein de la carte STM32.
Plus partiuclièrement, nous utiliserons ici le timer TIM2.

{{% notice style="orange" title="Question 1.1" icon="bolt" %}}
Avant d'utiliser un timer, il est nécessaire d'activer son signal d'horloge.
Pour cela, mettez à 1 le bit correspondant du registre `RCC_APB1ENR`.
{{% expand title="Documentation ..." %}}
Le registre `RCC_APB1ENR` est détaillé page 148 du *STM32F446 Reference Manual*.
{{% /expand %}}
{{% /notice %}}

{{% notice style="orange" title="Question 1.2" icon="bolt" %}}
Le registre `CR1` du TIM2 permet de configurer son fonctionnement.
Notamment, les champs *CEN*, *DIR*, *CMS* et *ARPE* permettent respectivement d'activer le compteur, de sélectionner la direction du compteur (croissant ou décroissant), l'alignement (aligné sur les fronts ou centré) et de bufferiser le préchargement.
Modifier le registre pour le compteur soit éteint, croissant, aligné sur les fronts et que le préchargement soit bufferisé.
On l'activera seulement une fois la configuration terminée.

{{% expand title="Documentation ..." %}}
Le registre `TIM2_CR1` est détaillé page 558 du *STM32F446 Reference Manual*.
L'utilisation d'un buffer pour le pré-chargement permet de s'assurer que le compteur ne change pas de valeur maximale n'importe quand.
Ainsi, c'est seulement lorsqu'un débordement (*overflow*) aura lieu que la nouvelle valeur sera prise en compte.
{{% /expand %}}
{{% /notice %}}

{{% notice style="orange" title="Question 1.3" icon="bolt" %}}
Par défaut, la génération d'un signal d'interruption lors d'un débordement est désactivé.
Pour l'utiliser, il faut donc mettre à 1 le champ *UIE* du registre `DIER`.
{{% expand title="Documentation ..." %}}
Le registre `TIM2_DIER` est détaillé page 563 du *STM32F446 Reference Manual*.
{{% /expand %}}
{{% /notice %}}

{{% notice style="warning" title="Fréquence des horloges" icon="bolt" %}}
Sur la vue système (double-cliquez le fichier *.ioc* du projet), dans *Clock Configuration*, vérifiez pour la suite la valeur des horloges de votre système.
Notamment, regardez la valeur des horloges contrôlant le processeur (*core*) et le bus APB1.
Pour la suite, on considèrera que la valeur d'horloge pour le microcontrôleur est de 180MHz et celle des timers de 90MHz.
{{% /notice %}}

{{% notice style="orange" title="Question 1.4" icon="bolt" %}}
Dans un premier temps, on souhaite faire clignoter la LED toutes les secondes.
La fréquence de l'horloge du timer étant de 90MHz, il est nécessaire d'attendre 90,000,000 cycles pour qu'une seconde s'écoule.
À partir de ces informations, modifiez les registres `ARR` et `PSC` pour avoir le fonctionnement attendu.
On considèrera que chacun de ces registres ne fait que 16 bits.

{{% expand title="Documentation ..." %}}
Les registres `TIM2_PSC` et `TIM2_ARR` sont détaillés page 573 du *STM32F446 Reference Manual*.
{{% /expand %}}
{{% expand title="Aide supplémentaire ..." %}}
180,000,000 s'écrit sur 27 bits.
Chacun des regitres étant sur 16 bits, il va falloir diviser la fréquence de l'horloge avec `PSC` pour obtenir une valeur inférieur à 2^16 à stocker dans `ARR`. 
{{% /expand %}}
{{% /notice %}}

{{% notice style="orange" title="Question 1.5" icon="bolt" %}}
Le timer TIM2 est associé à l'interruption numéro 28.
De la même manière que sur les [exercices précédents](irq/), activez la gestion de cette interruption.
{{% /notice %}}

{{% notice style="orange" title="Question 1.6" icon="bolt" %}}
Écrivez à présent la routine qui inversera l'état de la LED à chaque fois qu'elle sera appelée.
Pensez également à réinitiliser l'interruption une fois qu'elle a été traitée.
Comme précédemment, le système est pré-configuré pour que la routine d'interruption liée au timer TIM2 soit appelée `TIM2_IRQHandler`.
{{% /notice %}}

{{% notice style="orange" title="Question 1.7" icon="bolt" %}}
Dans votre fonction `main` à présent, ajoutez le code pour initialiser l'état de la LED à 0.
{{% /notice %}}

{{% notice style="orange" title="Question 1.8" icon="bolt" %}}
La configuration du timer est à présent terminée.
Activez le compteur en mettant le bit correspondant du registre `CR1` à 1 puis vérifiez le bon fonctionnement de votre système.
{{% expand title="Aide supplémentaire ..." %}}
En cas d'erreur, utilisez le mode debug de l'IDE pour visualiser l'état des registres du TIM2.
Assurez-vous dans l'ordre:
1. que le registre `CNT`s'incrémente correctement,
2. que les registres de configuration ont les valeurs attendus,
3. que les valeurs des registres `ARR`et `PSC` ont bien été calculées.
{{% /expand %}}
{{% /notice %}}

{{% notice style="orange" title="Question 1.9" icon="bolt" %}}
On souhaite à présent réduire la fréquence de clignotement.
Quel(s) registre(s) faut-il modifier ?
Quel est théoriquement la vitesse de clignotement minimale possible avec une fréquence d'horloge de 180MHz et `ARR`/`PSC` sur 16 bits ?
{{% /notice %}}

{{% notice style="orange" title="Question 1.10" icon="bolt" %}}
Pour notre application finale, nous aurons besoin de trois fonctions:
1. `TIM2_Init` qui configure et initialise l'état du timer au départ,
2. `TIM2_IRQHandler` qui inverse l'état de LED et gère l'interruption,
3. `TIM2_Frequency` qui prend en paramètre une distance en cm et fait clignoter la LED plus ou moins rapidement selon la valeur reçue. Pour information, la distance calculée mesurée peut aller de 2.5cm à 425cm. En-dessous de cet intervalle, on laissera la LED constamment allumé (ou clignotement très rapide). Au-dessus de cet intervalle, on laisse la LED constamment éteine (ou clignotement très lent).

Préparez ces trois fonctions pour la suite du projet.
{{% /notice %}}

### Lecture d'un capteur HC-SR04


### Conception d'une application


## Communication avec une télécommande infrarouge




## Références

Cette partie regroupe les informations sur les {{< ref-cnt-page >}} références utilisées sur cette page.

{{< ref-list-page >}}