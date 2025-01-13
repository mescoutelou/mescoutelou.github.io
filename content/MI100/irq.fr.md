---
archetype: "page"
lang: "fr"
title: "Requête d'interruptions"
linktitle: "Interruptions"
categories:
- "Fiches"
tags: 
- "Arm"
- "Langage C"
- "Microcontrôleur"
weight: 3
localref: True
draft: False
---

{{% notice style="info" title="Objectifs" %}}
Cette fiche vise à présenter une première utilisation des interruptions sur un microcontrôleur STM32.
À la suite de cette page, un développeur logiciel doit être capable:
- De trouver les informations nécessaires dans une datasheet pour l'utilisation des interruptions,
- De programmer les différents registres pour l'utilisation d'une interruption,
- De comprendre le fonctionnement pour différentes sources d'interruption.
{{% /notice %}}

{{% notice style="warning" title="Version des outils" %}}
Cette fiche utilise la version ***v1.12.0*** du logiciel [***STM32CubeIDE***](https://www.st.com/en/development-tools/stm32cubeide.html#st_description_sec-nav-tab).
Certaines variations au niveau des captures d'écrans peuvent apparaître si vous utilisez des versions différentes.
De même, la carte utilisée est la ***Nucleo-F446RE***.
{{% /notice %}}

## Sommaire
- [Sommaire](#sommaire)
- [Détection d'un bouton poussoir](#détection-dun-bouton-poussoir)
- [Communication *main*/ *handler*](#communication-main-handler)
- [Source d'interruption logicielle](#source-dinterruption-logicielle)
- [Références](#références)
  
## Détection d'un bouton poussoir

[Précédemment](gpio/), nous avons utilisé un bouton poussoir pour déclencher l'allumage d'une LED.
La détection de l'état du bouton était alors faite de manière scrutative: le programme vérifiait régulièrement l'état du registre correspondant.
À présent, on propose d'utiliser un mécanisme d'interruptions pour déclencher l'allumage de la LED.

{{% notice style="orange" title="Question 1.1" icon="bolt" %}}
Comme précédemment, configurez la pin du bouton poussoir (pin 13 du GPIOC) dans l'état suivant:
- MODE: *Input*,
- RESISTANCE DE RAPPEL: *pull-up*.
{{% /notice %}}

{{% notice style="orange" title="Question 1.2" icon="bolt" %}}
Pour déclencher une interruption depuis un GPIO, il est nécessaire d'activer son utilisation dans le NVIC (*Nested Vectored Interrupt Controller*).
Grâce au register `ISER` du NVIC, activez l'interruption correspondant à la ligne EXTI susceptible de recevoir un changement d'état du bouton-poussoir.
{{% expand title="Aide supplémentaire ..." %}}
Chaque ligne EXTI*x* est susceptible de détecter un évènement sur la pin *x* d'un des ports.
Ainsi, pour la ligne à activer pour le bouton poussoir est celle correspondants aux pins 13: *EXTI15_10*.
{{% /expand %}}
{{% /notice %}}

{{% notice style="orange" title="Question 1.3" icon="bolt" %}}
De base, le signal d'horloge pour certains modules du système est désactivé.
Pour les interruptions depuis un GPIO, il est nécessaire de réactiver le signal d'horloge pour `SYSCFG`.
Pour cela, ajoutez la ligne de code suivante:
```c
RCC->APB2ENR |= RCC_APB2ENR_SYSCFGEN;
```
{{% /notice %}}

{{% notice style="orange" title="Question 1.4" icon="bolt" %}}
Le registre `EXTICR`dans SYSCFG permet de choisir le port de déclenchement sur chaque ligne.
Modifiez le registre `EXTICR` pour que la ligne EXTI13 utilise le port C du bouton poussoir.
{{% /notice %}}

{{% notice style="orange" title="Question 1.5" icon="bolt" %}}
Pour être activée, une ligne d'interruption doit être non-masquée.
Modifiez le registre `IMR` de EXTI en mettant le bit pour EXTI13 dans l'état correspondant. 
{{% /notice %}}

{{% notice style="orange" title="Question 1.6" icon="bolt" %}}
Une interruption peut être déclenchée pour différents types d'évènements, aussi bien sur un front montant qu'un front descendant.
Pour cela, deux registres `RTSR` et `FTSR` sont mis à disposition, respectivement pour activer/désactiver un déclenchement sur front montant ou descendant.
Configurez ces deux registres pour ne lancer une interruption que lorsque le bouton est pressé.
{{% /notice %}}

{{% notice style="orange" title="Question 1.7" icon="bolt" %}}
Toutes les étapes précédentes permettent d'activer et paramétrer l'activation d'une interruption sur la ligne EXTI13 depuis la pin 13 du GPIOC.
Il faut maintenant créer la fonction qui sera appelée lorsque cette interruption interviendra.
Pour cela, créez une fonction:
```c
void EXTI15_10_IRQHandler(void) {
  // Contenu
}
```
Le système est conçu pour automatiquement rediriger les interruptions correspondantes vers cette ligne à l'aide du vecteur d'adresses du NVIC.
{{% /notice %}}

{{% notice style="orange" title="Question 1.8" icon="bolt" %}}
Vérifiez à présent que le système fonctionne.
En mode debug, lancez l'exécution du programme et placez un point d'arrêt sur la fonction `EXTI15_10_IRQHandler`.
Que se passe-t-il ? Le fonctionnement est-il correct ?
{{% /notice %}}

{{% notice style="orange" title="Question 1.9" icon="bolt" %}}
Lorsqu'une interruption a été traitée, il est nécessaire de la remettre à 0.
Pour cela, utilisez le bit correspondant dans le registre `PR` de EXTI.
{{% /notice %}}

{{% notice style="orange" title="Question 1.10" icon="bolt" %}}
Modifiez le contenu de la fonction pour inverser l'état de la LED à chaque fois que le bouton est pressé.
{{% /notice %}}

{{% notice style="orange" title="Question 1.11" icon="bolt" %}}
Modifiez le paramétrage de l'interruption pour qu'elle ne s'active que lorsque le bouton est relâché.
{{% /notice %}}

## Communication *main*/ *handler*

On aimerait à présent concevoir un système où la fonction `main` fait clignoter la LED à un rythme variable.
Ce rythme pourra ensuite être modifié selon le nombre de pressions effectuées sur le bouton poussoir: plus le bouton est pressé un grand nombre de fois, plus le clignotement accélère ou inversement.
Pour cela, nous allons devoir faire des échanges d'informations entre les deux fonctions.

{{% notice style="orange" title="Question 2.1" icon="bolt" %}}
Modifiez votre fonction `main` pour qu'elle fasse clignoter la LED selon la valeur d'une variable `tempo`.
{{% /notice %}}

{{% notice style="orange" title="Question 2.2" icon="bolt" %}}
Modifiez votre routine d'interruption appelée par le bouton poussoir pour qu'elle diminue la variable `tempo` d'une valeur N à chaque exécution.
{{% expand title="Aide supplémentaire ..." %}}
Les variables locales sont internes à chaque fonction.
Les variables globales sont accessibles depuis n'importe quelle fonction.
{{% /expand %}}
{{% /notice %}}

{{% notice style="orange" title="Question 2.3" icon="bolt" %}}
Modifiez votre routine d'interruption appelée par le bouton poussoir pour qu'elle diminue la variable `tempo` d'une valeur N jusqu'à O.
Si la valeur `tempo` est déjà à `0`, alors la variable `tempo` doit être augmentée d'une valeur N jusqu'à une valeur maximale libre.
{{% expand title="Aide supplémentaire ..." %}}
Pour savoir le sens utilisé (augmentation ou diminution de `tempo`), vous pouvez utiliser une nouvelle variable pour stocker l'information.
{{% /expand %}}
{{% /notice %}}

## Source d'interruption logicielle
Les lignes d'interruptions EXTI peuvent être activées par un évènement extérieur (GPIO) ou alors depuis le logiciel en écrivant dans le registre `SWIER` d'EXTI.

{{% notice style="orange" title="Question 3.1" icon="bolt" %}}
De la même manière que pour le bouton poussoir avec la ligne EXTI13, paramétrez une routine qui s'active lorsqu'une interruption arrive sur la ligne EXTI4.
Générez ensuite cette exception depuis le `main` avec le registre `SWIER` d'EXTI.
{{% /notice %}}

{{% notice style="orange" title="Question 3.2" icon="bolt" %}}
En utilisant le registre de priorité `IPR` du NVIC, modifiez la priorité des interruptions venant de EXTI4 et EXTI13 pour que la seconde puisse pré-emptée la première.
En effectuant une exécution en mode debug, validez ensuite le fonctionnement du système.
{{% /notice %}}

## Références

Cette partie regroupe les informations sur les {{< ref-cnt-page >}} références utilisées sur cette page.

{{< ref-list-page >}}