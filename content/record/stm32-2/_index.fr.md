---
archetype: "page"
lang: "fr"
title: "Programmation sur STM32: Interruptions"
linktitle: "STM32: Interruptions"
categories:
- "Fiches"
tags: 
- "Arm"
- "Langage C"
- "Microcontrôleur"
weight: 12
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
- [Références](#références)
  
## Détection d'un bouton poussoir

[Précédemment](/record/stm32-1), nous avons utilisé un bouton poussoir pour déclencher l'allumage d'une LED.
La détection de l'état du bouton était alors faite de manière scrutative: le programme vérifiait régulièrement l'état du registre correspondant.
À présent, on propose d'utiliser un mécanisme d'interruptions pour déclencher l'allumage de la LED.

{{% notice style="orange" title="Question 2.1" icon="bolt" %}}
Comme précédemment, configurez la pin du bouton poussoir (pin 13 du GPIOC) dans l'état suivant:
- MODE: *Input*,
- RESISTANCE DE RAPPEL: *pull-up*.
{{% /notice %}}

{{% notice style="orange" title="Question 2.2" icon="bolt" %}}
Pour déclencher une interruption depuis un GPIO, il est nécessaire d'activer son utilisant dans le NVIC (*Nested Vectored Interrupt Controller*).
Grâce au register `ISER` du NVIC, activez l'interruption correspondant à la ligne EXTI susceptible de recevoir un changement d'état du bouton-poussoir.
{{% expand title="Aide supplémentaire ..." %}}
Chaque ligne EXTI*x* est susceptible de détecter un évènement sur la pin *x* d'un des ports.
Ainsi, pour la ligne à activer pour le bouton poussoir est celle correspondants aux pins 13: *EXTI15_10*.
{{% /expand %}}
{{% /notice %}}

{{% notice style="orange" title="Question 2.3" icon="bolt" %}}
De base, le signal d'horloge pour certains modules du système est désactivé.
Pour les interruptions depuis un GPIO, il est nécessaire de réactiver le signal d'horloge pour `SYSCFG`.
Pour cela, ajoutez la ligne de code suivant:
```c
RCC->APB2ENR |= RCC_APB2ENR_SYSCFGEN;
```
{{% /notice %}}

{{% notice style="orange" title="Question 2.4" icon="bolt" %}}
Le registre `EXTICR`dans SYSCFG permet de choisir le port de déclenchement sur chaque ligne.
Modifiez le registre `EXTICR` pour que la ligne EXTI13 utilise le port C du bouton poussoir.
{{% /notice %}}

{{% notice style="orange" title="Question 2.5" icon="bolt" %}}
Pour être activée, une ligne d'interruption doit être non-masquée.
Modifiez le registre `IMR` de EXTI en mettant le bit pour EXTI13 dans l'état correspondant. 
{{% /notice %}}

{{% notice style="orange" title="Question 2.6" icon="bolt" %}}
Une interruption peut aussi être déclenchée pour différents types d'évènements, aussi bien sur un front montant qu'un front descendant.
Pour cela, deux registres `RTSR` et `FTSR` sont mis à disposition, respectivement pour activer/désactiver un déclenchement sur front montant ou descendant.
Configurez ces deux registres pour ne lancer une interruption que sur front montant.
{{% /notice %}}

{{% notice style="orange" title="Question 2.7" icon="bolt" %}}
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

{{% notice style="orange" title="Question 2.8" icon="bolt" %}}
Vérifiez à présent que le système fonctionne.
En mode debug, lancez l'exécution du programme et placez un point d'arrêt sur la fonction `EXTI15_10_IRQHandler`.
Que se passe-t-il ? Le fonctionnement est-il correct ?
{{% /notice %}}

{{% notice style="orange" title="Question 2.9" icon="bolt" %}}
Lorsqu'une interruption a été traitée, il est nécessaire de la remettre à 0.
Pour cela, réinitialisez le bit correspondant dans le registre `PR` de EXTI.
{{% /notice %}}

{{% notice style="orange" title="Question 2.10" icon="bolt" %}}
Modifiez le contenu de la fonction pour inverser l'état de la LED à chaque fois que le bouton est pressé.
{{% /notice %}}

## Références

Cette partie regroupe les informations sur les {{< ref-cnt-page >}} références utilisées sur cette page.

{{< ref-list-page >}}