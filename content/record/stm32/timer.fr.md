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
  - [Clignotement périodique d'une LED](#clignotement-périodique-dune-led)
- [Détection d'un obstacle](#détection-dun-obstacle)
  - [Clignotement automatique de la LED](#clignotement-automatique-de-la-led)
  - [Lecture d'un capteur HC-SR04](#lecture-dun-capteur-hc-sr04)
    - [Configuration du timer TIM3](#configuration-du-timer-tim3)
    - [Canal 1: génération du signal `Trig`](#canal-1-génération-du-signal-trig)
    - [Canaux 3/4: réception du signal `Echo`](#canaux-34-réception-du-signal-echo)
    - [Montage et test](#montage-et-test)
  - [Conception d'une application](#conception-dune-application)
- [Communication avec une télécommande infrarouge](#communication-avec-une-télécommande-infrarouge)
- [Références](#références)


  
### Clignotement périodique d'une LED

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
Modifiez le registre pour que le compteur soit éteint, croissant, aligné sur les fronts et que le préchargement soit bufferisé.
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
90,000,000 s'écrit sur 27 bits.
Chacun des regitres étant sur 16 bits, il va falloir diviser la fréquence de l'horloge avec `PSC` pour obtenir une valeur inférieur à 2^16 à stocker dans `ARR`. 
{{% /expand %}}
{{% /notice %}}

{{% notice style="warning" title="Mise à jour des registres" icon="bolt" %}}
Au départ, il est conseillé dans l'ordre du programme de modifier `TIM2_PSC` et `TIM2_ARR` avant d'activer le buffer avec le bit `ARPE`.
{{% /notice %}}

{{% notice style="orange" title="Question 1.5" icon="bolt" %}}
Le timer TIM2 est associé à l'interruption numéro 28.
De la même manière que sur les [exercices précédents](irq/), activez la gestion de cette interruption.
{{% /notice %}}

{{% notice style="orange" title="Question 1.6" icon="bolt" %}}
Écrivez à présent la routine qui inversera l'état de la LED à chaque fois qu'elle sera appelée.
Pensez également à réinitiliser l'interruption une fois qu'elle a été traitée.
Cela peut être fait en remettant à 0 le bit `UIF` du registre `TIM2_SR`.

Comme précédemment, le système est pré-configuré pour que la routine d'interruption liée au timer TIM2 soit appelée `TIM2_IRQHandler`.
{{% /notice %}}

{{% notice style="orange" title="Question 1.7" icon="bolt" %}}
Dans votre fonction `main` à présent, ajoutez le code pour initialiser l'état de la LED à 0.
{{% /notice %}}

{{% notice style="orange" title="Question 1.8" icon="bolt" %}}
La configuration du timer est à présent terminée.
Juste avant d'activer le compteur, réinitialisez la valeur du compteur `CNT` à la valeur 0
Vérifiez ensuite le bon fonctionnement de votre système.
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

## Détection d'un obstacle

{{< fig
  fig="/fig/record/stm32/hc-sr04.png"
  alt=""
  width=500px
  caption="Capteur de distance par ultrason HC-SR04"
  subcaption=""
  key="hc-sr04"
>}}

Pour comprendre le fonctionnement des timers et leur intérêt, nous allons étudier l'implémentation d'une application de détection d'obstacle.
À l'aide d'un capteur de distance à ultrason, l'objectif sera de faire clignoter une LED plus ou moins rapidement selon la distance de l'obstacle.

{{< fig
  fig="/fig/record/stm32/obstacle-detector.png"
  alt=""
  width=800px
  caption="Détecteur d'obstacle basé sur un capteur HC-SR04 et une LED"
  subcaption=""
  key="obstacle-detector"
>}}

La mise en place de cette application se décomposera en trois parties:
1. L'utilisation du timer TIM2 pour faire clignoter de manière régulière la LED,
2. La lecture de la distance à partir d'un capteur HC-SR04 en utilisant le timer TIM3,
3. La conception finale de l'application.

### Clignotement automatique de la LED

Dans l'exercice ci-dessus, le clignotement s'effectue avec l'intervention d'une interruption.
Cependant, le timer `TIM2` contient tous les éléments pour effectuer ce clignotement de manière autonome, sans intervention du logiciel après configuration.
Dans cette partie, on se propose donc de le reconfigurer pour contrôler la sortie à l'aide du canal 1.
Pour cela, nous allons utiliser le principe de PWM (*Pulse Width Modulation*) des timers.

{{% notice style="orange" title="Question 2.1" icon="bolt" %}}
Reprenez la configuration du `TIM2` effectuée précédemment.
Désactivez l'interruption.
{{% /notice %}}

{{% notice style="orange" title="Question 2.2" icon="bolt" %}}
Tout d'abord, activez l'horloge du port A de GPIO (registre `RCC_AHB1ENR`).
Modifiez ensuite le mode de la pin 5 du GPIOA pour qu'elle utilise le mode de fonction alternative (registre `GPIOA_MODER`).
Choisissez également le monde registre des fonctions alternatives (`GPIOA_AFR`) pour attribuer la valeur 0x1 à la pin 5.
{{% /notice %}}

{{% notice style="orange" title="Question 2.3" icon="bolt" %}}
La configuration du canal 1 s'effectue au sein du registre `TIM2_CCMR1`.
Dans notre cas, on souhaite configurer le canal en tant que sortie, utilisez le mode 1 de PWM et activer le préchargement (*preload*).
{{% /notice %}}

{{% notice style="orange" title="Question 2.4" icon="bolt" %}}
Le canal 1 n'est pas supposé générer d'interruption.
Ainsi, toujours dans le registre `TIM2_DIER`, désactivez l'interruption pour le canal 1.
{{% /notice %}}

{{% notice style="orange" title="Question 2.5" icon="bolt" %}}
L'activation du signal en sortie et la polarité sont configurées au sein du registre `TIM2_CCER`.
Modifiez le registre pour activer la sortie sur l'état haut.
{{% /notice %}}

{{% notice style="orange" title="Question 2.6" icon="bolt" %}}
A l'aide d'un schéma, représentez l'allure attendue du signal de sortie contrôlant la LED en fonction des valeurs de `CNT`, `PSC`, `ARR` et `CCR1`.
{{% /notice %}}

{{% notice style="orange" title="Question 2.7" icon="bolt" %}}
Pour notre application finale, nous aurons besoin de trois fonctions:
1. `TIM2_Init` qui configure et initialise l'état du timer TIM2 au départ,
2. `TIM2_Start` qui lance le fonctionnement du compteur,
3. `TIM2_Frequency` qui prend en paramètre une distance en cm et fait clignoter la LED plus ou moins rapidement selon la valeur reçue. Pour information, la distance calculée mesurée peut aller de 2.5cm à 425cm. En-dessous de cet intervalle, on laissera la LED constamment allumé (ou clignotement très rapide). Au-dessus de cet intervalle, on laisse la LED constamment éteinte (ou clignotement très lent). Une stratégie envisageable est simplement de modifier la valeur du registre `CCR1` selon la distance et ainsi faire varier le rapport cyclique.

Préparez ces trois fonctions pour la suite du projet.
Testez ensuite leur bon fonctionnement.
{{% /notice %}}

### Lecture d'un capteur HC-SR04

Dans cette partie, nous allons développer notre première application intérragissant avec un capteur externe.
Ce dernier est le HC-SR04, un capteur de mesure de distance par ultrasons.
Il permet de détecter des obstacles situés devant lui.
Ce système possède deux signaux:
- Une entrée `Trig` utilisée pour commander la mesure. Chaque demande de mesure est effectuée en mettant le signal `Trig` à l'état haut pour au moins 10µs.
- Une sortie `Echo` qui est la réponse sous forme d'impulsion. Cette impulsion est à l'état haut entre 0.15ms et 25ms. Lorsque cette valeur dépasse 38ms, alors cela signifie qu'aucun obstacle n'a été détecté. On s'assurera qu'une nouvelle mesure n'est commandée que toutes les 60ms.
La distance est ensuite déduite selon le calcul suivant: *distance = (T / 58.8235)* avec la distance en cm et T la durée de l'impulsion en µs.

#### Configuration du timer TIM3

{{% notice style="orange" title="Question 3.1" icon="bolt" %}}
Comme pour le timer `TIM2`, activez l'horloge pour le timer `TIM3` dans le registre `RCC_APB1ENR`.
De même, modifiez le registre pour que le compteur soit éteint, croissant, aligné sur les fronts et que le préchargement soit bufferisé.
{{% /notice %}}

{{% notice style="orange" title="Question 3.2" icon="bolt" %}}
Pour cette application, nous n'avons pas besoin de générer d'interruption lors d'un débordement.
Il est donc nécessaire de désactiver l'interruption correspondante dans le registre `TIM3_DIER`.
{{% /notice %}}

{{% notice style="orange" title="Question 3.3" icon="bolt" %}}
Pour les intéractions avec le capteur HC-SR04, la granularité de temps minimale est de l'orde du µs.
Il est également précisé que le temps minimal entre deux requêtes de mesure doit être d'environ 60 ms.
Configurez les registres `PSC` et `ARR` en conséquences.
{{% /notice %}}

#### Canal 1: génération du signal `Trig`

Pour qu'une mesure soit effectuée, une demande doit avoir été faite sur le signal `Trig`.
Ainsi, détecter dès que possible un obstacle implique d'effectuer de manière régulière des mesures.
Pour cela, nous allons à nouveau utiliser le principe de PWM (*Pulse Width Modulation*) des timers.
La configuration attendue sera  finalement en grande partie similaire à celle pour la LED: seuls la période et le rapport cyclique seront différents.

{{% notice style="orange" title="Question 3.4" icon="bolt" %}}
Pour utiliser l'entrée `Trig` du capteur comme sortie de notre timer, il est nécessaire de la rediriger par le biais des fonctions alternatives des GPIO.
À partir des spécifications de la carte Nucleo et du microcontrôleur, déduisez quelles pins peuvent être utilisées pour relier `Trig` à la sortie du canal 1 du timer TIM3.
{{% /notice %}}

{{% notice style="orange" title="Question 3.5" icon="bolt" %}}
Selon votre choix, activez le port et la clock du port correspondant et utilisez la fonction alternative nécessaire pour rediriger le signal vers le canal 1 du timer TIM3.
{{% /notice %}}

{{% notice style="orange" title="Question 3.6" icon="bolt" %}}
La configuration du canal 1 s'effectue au sein du registre `TIM3_CCMR1`.
Dans notre cas, on souhaite configurer le canal en tant que sortie, utilisez le mode 1 de PWM et activer le préchargement (*preload*).
{{% /notice %}}

{{% notice style="orange" title="Question 3.7" icon="bolt" %}}
Le canal 1 n'est pas supposé générer d'interruption.
Ainsi, toujours dans le registre `TIM3_DIER`, désactivez l'interruption pour le canal 1.
{{% /notice %}}

{{% notice style="orange" title="Question 3.8" icon="bolt" %}}
L'activation du signal en sortie et la polarité sont configurées au sein du registre `TIM3_CCER`.
Modifiez le registre pour activer la sortie sur l'état haut.
{{% /notice %}}

{{% notice style="orange" title="Question 3.9" icon="bolt" %}}
A l'aide d'un schéma, représentez l'allure attendue du signal de sortie `Trig`.
En considérant les valeurs précédemment choisies d'`ARR` et `PSC`, à quelle valeur le signal `Trig` doit être mis à 0 / à 1.
Modifiez le registre `TIM3_CCR1` en fonction de la valeur voulue.
{{% /notice %}}

#### Canaux 3/4: réception du signal `Echo`

{{% notice style="orange" title="Question 3.10" icon="bolt" %}}
Pour utiliser la sortie `Echo` du capteur comme entrée de notre timer, il est nécessaire de la rediriger par le biais des fonctions alternatives des GPIO.
À partir des spécifications de la carte Nucleo et du microcontrôleur, déduisez quels pins peuvent être utilisées pour relier `Echo` à l'entrée des canaux 3 ou 4 du timer TIM3.
{{% /notice %}}

{{% notice style="orange" title="Question 3.11" icon="bolt" %}}
Selon votre choix, activez le port et la clock du port correspondant et utilisez la fonction alternative nécessaire pour rediriger le signal vers les canaux 3 ou 4 du timer TIM3.
{{% /notice %}}

{{% notice style="orange" title="Question 3.12" icon="bolt" %}}
Le timer TIM3 dispose de 4 canaux.
Le canal 1 est utilisé pour l'émission du signal `Trig`.
D'après l'architecture du timer page 521 du manuel d'utilisateur, pourquoi est-il alors nécessaire d'utiliser les canaux 3 et 4 pour faire les détections distinctes des fronts montants et descendants plutôt que les canaux 2 et 3 ?
Expliquez avec un schéma simplifié.
{{% /notice %}}

{{% notice style="orange" title="Question 3.13" icon="bolt" %}}
La configuration du canal 3 s'effectue au sein du registre `TIM3_CCMR2`.
Dans notre cas, on souhaite configurer le canal en tant qu'entrée, sans filtre, sans *prescaler* et avec IC3 connecté à TI3 (voir schéma).
De même, désactivez l'interruption correspondante dans `TIM3_DIER`.
{{% /notice %}}

{{% notice style="orange" title="Question 3.14" icon="bolt" %}}
La capture et la polarité sont configurées au sein du registre `TIM3_CCER`.
Modifiez le registre pour activer la capture sur front montant  du canal 3.
{{% /notice %}}

{{% notice style="orange" title="Question 3.15" icon="bolt" %}}
La configuration du canal 4 s'effectue également au sein du registre `TIM3_CCMR2`.
Dans notre cas, on souhaite configurer le canal en tant qu'entrée, sans filtre, sans *prescaler* et avec IC4 connecté à TI3 (voir schéma).
En revanche, la détecton d'un front descendant implique que la mesure est terminée.
Il est donc nécessaire d'activer l'interruption correspondante dans `TIM3_DIER`.
{{% /notice %}}

{{% notice style="orange" title="Question 3.16" icon="bolt" %}}
La capture et la polarité sont configurées au sein du registre `TIM3_CCER`.
Modifiez le registre pour activer la capture sur front descendant  du canal 4.
{{% /notice %}}

{{% notice style="orange" title="Question 3.17" icon="bolt" %}}
  La routine appelée pour le timer TIM3 s'appelle `TIM3_IRQHandler`.
  Modifiez cette routine pour qu'elle calcule le nombre de cycles écoulés entre les deux fronts et stocke le résultat dans une variable globale.
  Pensez également à réinitialiser l'état du bit d'interruption dans le registre `TIM3_SR`.
{{% /notice %}}

#### Montage et test

Nous avons à présent configué notre timer TIM3 afin qu'il s'interface avec le capteur HC-SR04.
L'objectif va être de tester le bon fonctionnement du système et de préparer les fonctions pour l'application finale.

{{% notice style="orange" title="Question 3.18" icon="bolt" %}}
Selon les pins choisies précédemment, connecté le capteur à votre carte Nucleo.
{{% /notice %}}

{{% notice style="orange" title="Question 3.19" icon="bolt" %}}
Pour notre application finale, nous aurons besoin de trois fonctions:
1. `TIM3_Init` qui configure et initialise l'état du timer TIM3 au départ,
1. `TIM3_Start` qui lance le fonctionnement du timer,
2. `TIM3_Handler` qui est appelée par une interruption pour calculer le temps entre les deux fronts.

Préparez ces trois fonctions pour la suite du projet.
Testez ensuite leur bon fonctionnement.
`TIM3_Init` et `TIM3_Start` doivent être exécutées avant la boucle `while`.
À l'aide du mode *debug*, vérifiez la valeur renvoyée par `TIM3_Handler` après plusieurs appels.
{{% /notice %}}

### Conception d'une application

Nous disposons à présent de tous les éléments pour notre application de détecteur d'obstacle.
Il nous reste à appeler les différentes fonctions dans notre `main` afin de les faire fonctionner ensemble.

{{% notice style="orange" title="Question 4.1" icon="bolt" %}}
Modifiez votre fonction `main` pour qu'elle appelle d'abord les fonctions d'initialisations puis lance les compteurs.
Dans la boucle `while`, ajoutez le code nécessaire au calcul de la distance en cm à partir de la valeur reçue de `TIM3_Handler` et appelez la fonction `TIM2_Frequency`.
{{% /notice %}}

{{% notice style="orange" title="Question 4.2" icon="bolt" %}}
Plusieurs optimisations sont imaginables au niveau du code conçu jusqu'à maintenant:
- Utiliser un seul canal pour la détection des deux fronts,
- N'appeler `TIM2_Frequency` depuis la boucle `while` que lorsque c'est nécessaire,
- Avoir une boucle `while` complètement vide, 
- ...

Proposez et/ou testez une ou plusieurs de ces possibles optimisations.
{{% /notice %}}

## Communication avec une télécommande infrarouge

{{< fig
  fig="/fig/record/stm32/ir-remote.png"
  alt=""
  width=500px
  caption="Télécommande et récepteur infrarouge"
  subcaption=""
  key="ir-remote"
>}}



## Références

Cette partie regroupe les informations sur les {{< ref-cnt-page >}} références utilisées sur cette page.

{{< ref-list-page >}}