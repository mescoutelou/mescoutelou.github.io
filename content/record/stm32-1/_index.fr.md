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
- [Détection d'un bouton poussoir](#détection-dun-bouton-poussoir)
- [Développement d'une surcouche logicielle](#développement-dune-surcouche-logicielle)
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

{{% notice style="orange" title="Question 1.1" icon="bolt" %}}
Pour modifier les registres d'un GPIO, il est nécessaire de savoir dans quel(s) registre(s) et partie(s) de registre(s) il est nécessaire d'écrire.
En utilisant le [manuel de référence](https://www.st.com/resource/en/reference_manual/rm0390-stm32f446xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf) (Chapitre 7 *General Purpose I/Os*), repérez les valeurs de registres nécessaires pour configurer le GPIO du port A ainsi:
- MODE: *General purpose output mode*,
- TYPE: *push-pull*,
- DATA: 1.
  
On laissera pour l'instant les autres paramètres inchangés.
{{% /notice %}}

{{% notice style="orange" title="Question 1.2" icon="bolt" %}}
Chaque structure dispose d'un champ dédié à chaque registre du port: `MODER`,`OTYPER`,`IDR` ...
Un pointeur de structure `GPIOA` existe pour le PORTA.
On peut accéder à la valeur de chaque registre du port en utilisant l'opérateur `->`: `GPIOA->MODER`.
En utilisant ce mécanisme, initialisez les registres nécessaires pour obtenir la configuration de la Question 1.1 et ainsi allumer la LED verte.
{{% /notice %}}

{{% notice style="orange" title="Question 1.3" icon="bolt" %}}
Dans le cas où cela ne serait pas le cas, modifiez votre code de la Question 1.2 pour maintenant utiliser des masques binaires afin de ne modifier que les bits qui correspondent à la pin de la LED.
{{% /notice %}}

{{% notice style="orange" title="Question 1.4" icon="bolt" %}}
Copiez votre code précédent à la suite et modifiez-le maintenant pour éteindre la LED.
Combien de registres doivent être modifiés ?
{{% /notice %}}

{{% notice style="orange" title="Question 1.5" icon="bolt" %}}
Généralement, allumer/éteindre une LED est une opération réalisée par le biais d'une fonction.
Cela permet de réutiliser le même morceau de code à chaque fois que l'on en a besoin.
Dans votre projet, ajoutez à présent deux fonctions `gpio_set_pin` et `gpio_reset_pin` vous permettant respectivement d'allumer ou éteindre la LED.
{{% /notice %}}

{{% notice style="orange" title="Question 1.6" icon="bolt" %}}
On se propose maintenant de modifier la fonction `main` pour faire clignoter notre LED à une vitesse visible.
Pour cela, l'idée va être d'appeler à tour de rôle les fonctions respectives au sein de la boucle `while(1)`.
Pour créer l'effet de clignotement, ajouter entre les appels de chaque fonction des boucles pour insérer du délai dans le temps d'exécution.
Le microcontrôleur tournant à plusieurs dizaines de MHz, pensez à ajouter suffisamment de tours de boucles entre les appels pour que le délai soit suffisant.
{{% /notice %}}

## Détection d'un bouton poussoir

Faire clignoter une LED implique d'utiliser un GPIO en tant que sortie.
Cependant, un GPIO peut aussi également être utilisé en tant qu'entrée pour lire une valeur externe.
C'est nécessaire par exemple si l'on souhaite connaître l'état d'un bouton poussoir (pressé ou non).
Ce dernier est le sujet de cette seconde partie.

La carte Nucleo-F446RE dispose d'un bouton poussoir directement intégré sur la carte.
Celui-ci est connecté à la pin 2 de la carte.
Cela correspond au GPIO 13 du port C. 

Lorsque le bouton est pressé, l'entrée est connectée à la masse (valeur logique de 0).
Sinon, l'entrée est considérée comme flottante (haute impédance).
Une résistance de rappel vers *VDD* est alors nécessaire pour détecter ce cas.

{{% notice style="warning" title="Bouton-poussoir" %}}
Les cartes disposent généralement de plusieurs bouton-poussoirs, dont un pour le *reset* (redémarrage à zéro du système).
Sur la ***Nucleo-F446RE***, le bouton bleu est un bouton utilisateur standard (utilisable pour n'importe quel application).
Le bouton noir est lui un bouton de *reset*.
{{% /notice %}}

{{% notice style="orange" title="Question 2.1" icon="bolt" %}}
En utilisant le [manuel de référence](https://www.st.com/resource/en/reference_manual/rm0390-stm32f446xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf) (Chapitre 7 *General Purpose I/Os*), repérez les valeurs de registres nécessaires pour configurer le GPIO du port C ainsi:
- MODE: *Input*,
- RESISTANCE DE RAPPEL: *pull-up*.
  
On laissera pour l'instant les autres paramètres inchangés.
{{% /notice %}}

{{% notice style="orange" title="Question 2.2" icon="bolt" %}}
Initialisez les registres nécessaires pour obtenir la configuration de la Question 2.1 avant de lire l'état du bouton poussoir.
{{% /notice %}}

{{% notice style="orange" title="Question 2.3" icon="bolt" %}}
À partir du registre *IDR*, lisez l'état du bouton poussoir et placez le dans une variable.
Traitez le résultat pour n'avoir que deux valeurs de variables possibles: 0 ou 1.
Utilisez le mode *debug* pour vérifier que la variable prend bien la bonne valeur selon que vous poussez le bouton ou non.
{{% expand title="Aide supplémentaire ..." %}}
Pour traiter la valeur lue dans le registre et isoler le bit utile, réutilisez le principe du masquage binaire.
Pour cela, appliquez un masque qui remettra à 0 tous les bits non-utilisés.
Ensuite, pensez à décaler vers la droite la valeur pour passer le bit utile en position 0.
{{% /expand %}}
{{% /notice %}}

{{% notice style="orange" title="Question 2.4" icon="bolt" %}}
À partir du résultat de la Question 2.3, modifiez la fonction `main` pour qu'elle allume la LED lorsque le bouton est pressé.
{{% /notice %}}

{{% notice style="orange" title="Question 2.5" icon="bolt" %}}
Modifiez la fonction `main` pour que l'état de la LED soit inversé à chaque nouvelle pression du bouton-poussoir.
{{% expand title="Aide supplémentaire ..." %}}
Pour détecter que le bouton poussoir a été pressé une nouvelle fois, il est nécessaire de détecter qu'il a été relâché entre temps.
Pour cela, le plus simple est de stocker à chaque fois l'état du bouton dans une variable.
Si le bouton n'était pas pressé la fois précédente alors qu'à présent c'est le cas, alors cela signifie que c'est une nouvelle pression.
{{% /expand %}}
{{% /notice %}}

{{% notice style="orange" title="Question 2.6" icon="bolt" %}}
Modifiez à nouveau votre programme pour cette fois faire en sorte que la LED reste indéfiniment allumée après un nombre N de pressions du bouton.
{{% expand title="Aide supplémentaire ..." %}}
Par rapport à la Question 2.5, une nouvelle information est maintenant nécessaire: le nombre de pressions du bouton.
Ainsi, à chaque fois, qu'une pression est détectée, il faudra incrémentée une variable puis l'utiliser pour décider de garder la LED à 1 ou pas.
{{% /expand %}}
{{% /notice %}}

## Développement d'une surcouche logicielle

L'utilisation de registres par le logiciel est directement dépendant du matériel.
Ainsi, si l'adresse d'un des ports est modifiée, il est alors nécessaire de réadapter l'ensemble du code qui est dépend.
Généralement, une surcouche logicielle est donc mise à disposition avec des fonctions qui s'occupent du paraméter les registres en fonction des besoins.
De ce fait, si la plateforme matérielle change, alors seules ces fonctions seront à adapter pour que l'application finale continue de fonctionner.

Sur la plateforme Arduino, des fonctions sont par exemple mises à disposition pour configurer les entrées/sorties.
L'objectif de cette partie es simplement de reproduire cette fonctionnalité direcetement avec du langage C.

{{% notice style="orange" title="Question 3.1" icon="bolt" %}}
En utilisant le [manuel de référence](https://www.st.com/resource/en/reference_manual/rm0390-stm32f446xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf) (Chapitre 2 "Memory and bus architecture*), repérez les adresses de base des différents ports.
Fondamentalement, quelle est donc la seule différence entre GPIOA, GPIOB, GPIOC *etc.* ?
{{% /notice %}}

{{% notice style="orange" title="Question 3.2" icon="bolt" %}}
À partir de ces informations, concevez une fonction `gpio_mode` responsable de changer le mode d'une pin.
Elle devra prendre en paramètre:
1. Le GPIO à modifier,
2. Le numéro du pin,
3. Le mode utilisé.

Dans notre cas, on définira les modes suivants:
1. OUTPUT (sortie + *push/pull*),
2. INPUT (sortie, pas de *pull-up/pull-down*),
3. INPUT_PULLUP (entrée + *pull-up*),
4. INPUT_PULLDOWN (entrée + *pull-down*).
   
{{% expand title="Aide supplémentaire ..." %}}
Voici l'en-tête de la fonction attendue:
```c
void gpio_mode(GPIO_TypeDef *GPIO, uint8_t pin, uint8_t mode);
```

Les différentes valeurs des modes peuvent ensuite être fixées par des macros:
```c
#define OUTPUT          0
#define INPUT           1
#define INPUT_PULLUP    2
#define INPUT_PULLDOWN  3
```
{{% /expand %}}
{{% /notice %}}

{{% notice style="orange" title="Question 3.3" icon="bolt" %}}
Sur le même modèle, créez une fonction `gpio_output_set` qui met la valeur de sortie d'un GPIO à 1.
{{% /notice %}}

{{% notice style="orange" title="Question 3.4" icon="bolt" %}}
De la même manière, créez une fonction `gpio_output_reset` qui met la valeur de sortie d'un GPIO à 0.
{{% /notice %}}

{{% notice style="orange" title="Question 3.5" icon="bolt" %}}
Créez une fonction `gpio_input` renvoyant 0 ou 1 selon la valeur d'entrée d'un GPIO.
{{% /notice %}}

{{% notice style="orange" title="Question 3.5" icon="bolt" %}}
Reprenez la question 2.6 et utilisez à présent les nouvelles fonctions développées `gpio_mode`, `gpio_output_set`, `gpio_output_reset` et `gpio_input`
{{% /notice %}}

## Références

Cette partie regroupe les informations sur les {{< ref-cnt-page >}} références utilisées sur cette page.

{{< ref-list-page >}}