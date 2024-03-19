---
archetype: "page"
lang: "fr"
title: "Programmation de microcontrôleur STM32 avec STM32CubeIDE"
linktitle: "STM32CubeIDE"
categories:
- "Fiches"
tags: 
- "ISA"
- "Processeur"
- "RISC-V"
weight: 10
localref: True
draft: False
---

{{% notice style="info" title="Objectifs" %}}
Cette fiche vise à présenter une première utilisation du logiciel STM32CubeIDE pour la programmation de microcontrôleurs STM32.
En la suivant pas-à-pas, un développeur logiciel doit être capable:
- de créer un projet,
- de savoir où ajouter son programme (simple) en langage C,
- de connaître quelles informations sont mises à disposition par l'outil pour le débogage et comprende à quoi elles servent.
{{% /notice %}}

{{% notice style="warning" title="Attention" %}}
Cette fiche utilise la version v1.12.0 du logiciel [***STM32CubeIDE***](https://www.st.com/en/development-tools/stm32cubeide.html#st_description_sec-nav-tab).
Certaines variations au niveau des captures d'écrans peuvent apparaître si vous utilisez des versions différentes.
De même, la carte utilisée est la Nucleo-F446RE.
{{% /notice %}}

## Sommaire
- [Sommaire](#sommaire)
- [Créer un projet](#créer-un-projet)
- [Ajouter du code](#ajouter-du-code)
- [Debug](#debug)
  - [Lancement](#lancement)
  - [Fenêtres](#fenêtres)
  - [Exécution](#exécution)
- [Exemples de programmes](#exemples-de-programmes)
- [Références](#références)

Lors du premiere démarrage, demande le répertoire pour l'espace de travail.
Correspond à l'endroit où l'IDE installera les différents projets: choisir un réprtoire avec des droits d'accès

## Créer un projet

{{< tabs title="Création d'un projet:" >}}
{{% tab title="Étape 0" %}}
{{< fig
  fig="/fig/record/stm32/stm32cubeide_workspace.png"
  alt=""
  width=500px
  caption=""
  subcaption=""
>}}

Tout d'abord, lancez STM32CubeIDE.
Au démarrage, le logiciel vous demandera (si vous ne l'avez jamais utilisé) où est situé l'espace de travail.
Cela correspond à l'endroit pù seront placés vos futurs projets.
Choisissez donc un répertoire qui vous appartient et où vous pourrez récupérer facilement vos données.
Si vous souhaitez ne plus voir cette fenêtre à chaque démarrage de STM32CubeIDE, vous pouvez cocher la case correspondante.
{{% /tab %}}

{{% tab title="Étape 1" %}}
{{< fig
  fig="/fig/record/stm32/stm32cubeide_start.png"
  alt=""
  width=100px
  caption=""
  subcaption=""
>}}

Lors de la première ouverture, aucun projet n'existe.
Pour en créer un, cliquez sur l'icône correspondant.
{{% /tab %}}

{{% tab title="Étape 2" %}}
{{< fig
  fig="/fig/record/stm32/stm32cubeide_target.png"
  alt=""
  width=800px
  caption=""
  subcaption=""
>}}

Sur la prochaine fenêtre, la cible utilisée doit être choisie.
Ici, on considère une carte Nucleo-F446RE.
Pour la trouver, allez dans l'onglet ***Board Selector***.
Dans ***Type***, cochez *Nucleo-64*.
Enfin, dans la liste de cartes proposées, sélectionnez ***NUCLEO-F446RE***.
Cliquez ensuite sur le bouton ***Next>***.
{{% /tab %}}

{{% tab title="Étape 3" %}}
{{< fig
  fig="/fig/record/stm32/stm32cubeide_name.png"
  alt=""
  width=500px
  caption=""
  subcaption=""
>}}

Donnez un nom au projet que vous souhaiter créer.
Laissez les autres valeurs par défaut et cliquez sur ***Next>***.
Deux fenêtres vont s'ouvrir, cliquez à chaque fois sur *Yes*.

{{% /tab %}}

{{% tab title="Étape 4" %}}
{{< fig
  fig="/fig/record/stm32/stm32cubeide_new.png"
  alt=""
  width=800px
  caption=""
  subcaption=""
>}}

Une fois différentes données locales téléchargées, vous arrivez alors sur le fenêtre suivante.
Le projet a donc été créé.
Vou retrouvez sur la gauche la hiérarchie des fichiers du projet et au centre une animation présentant la configuration de votre système.

{{% /tab %}}
{{< /tabs >}}

{{< caption 
  type="Figure" 
  key="new-project"
  main="Création d'un projet STM32 sur STM32CubeIDE."
  sub=""
>}}

Pour créer un projet sur STM32CubeIDE, différentes étapes sont nécessaires. 
Elles sont résumées ici sur la {{< num type="Figure" key="new-project" >}}.

{{% notice style="warning" title="Attention" %}}
Par défaut, laissez les paramètres proposés par défaut lorsque des fenêtres s'ouvrent.
{{% /notice %}}

## Ajouter du code

{{< fig
  fig="/fig/record/stm32/stm32cubeide_files.png"
  alt=""
  width=200px
  caption="Hiérarchie des fichiers dans un projet."
  subcaption=""
>}}

Après la configuration du projet, différents fichiers sont créés à l'intérieur.
Celui qui nous intéresse ici est celui où nous allons pouvoir ajouter du code à exécuter, c'est-à-dire là où se trouve la fonction `main`.
Pour cela, ouvrez le fichier *main.c*.
À l'intérieur, retrouvez la fonction `main` suivante:

```c
int main(void)
{
  /* USER CODE BEGIN 1 */

  /* USER CODE END 1 */

  /* MCU Configuration--------------------------------------------------------*/

  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* USER CODE BEGIN Init */

  /* USER CODE END Init */

  /* Configure the system clock */
  SystemClock_Config();

  /* USER CODE BEGIN SysInit */

  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_USART2_UART_Init();
  /* USER CODE BEGIN 2 */

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
```

Lors de la création du fichier, l'IDE pré-complète et organise la fonction `main`.
Ainsi, en plus de fonctions d'initialisation du système, il ajouter également des commentaires vous indiquants où placer le code utilisateur (`USER CODE`).
Pour développer une application, la partie essentielle de la fonction `main` est la boucle `while(1)`.
Étant infinie, elle permet de garantir que l'exécution restera bloquée à l'intérieur.
Ainsi, une fois l'initialisation terminée, c'est à l'intérieur de cette boucle que l'on viendra ajouter les opérations que l'on souhaitera effectuer aussi longtemps que le microcontrôleur fonctionnera.
Par exemple, pour créer une variable `count` qui s'incrémentera infiniment, ajoutez le code suivant:

```c
int main(void) {
  /* USER CODE BEGIN 2 */
  uint32_t count = 0;
  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1) {
    /* USER CODE END WHILE */
	  count = count + 1;
    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
```

La partie `USER CODE 2` étant avant la boucle, alors celle-ci ne s'exécutera qu'une seule fois.
En revanche, la partie `USER CODE 3` étant dans la boucle, alors elle sera exécuter aussi longtemps que le système restera en marche.
Dans notre cas, on souhaite créer la variable `count` une seule fois au départ: on la place donc dans `USER CODE 2`.
Par la suite, on souhaite l'incrémenter tout le long du fonctionnement donc on place le code correspondant dans `USER CODE 3`.

De la même manière, il sera possible d'exécuter indéfiniment des tâches plus complexes en les plaçant dans cette boucle infinie.
C'est ce qui est fait lorsque l'on souhaite qu'un système réalise une mission bien précise et répétitive dans le temps.

## Debug

Une fois le code modifié, il est nécessaire de s'assurer qu'il réalise la fonctionnalité voulue.
Il est donc nécessaire de passer par une phase de *debug* (ou débogage), où le code est directement exécuté.
STM32CubeIDE met pour cela à disposition une interface dédiée.

{{% notice style="warning" title="Attention" %}}
Pour réaliser le débogage, pensez à connecter votre carte et à vous assurer qu'elle a bien été détectée par le logiciel.
{{% /notice %}}

### Lancement

Pour lancer le mode de débogage, cliquez sur l'icône ***Debug*** (icône en forme d'insecte).
Une fenêtre va s'ouvrir pour configurer la session de débogage.
Laissez les paramètres par défaut et cliquez sur ***OK***.

### Fenêtres

En mode débogage, plusieurs fenêtres permettent d'avoir différentes informations sur le système.
Voici ci-dessous un rapide récapitulatif de ces fenêtres, accessibles par des onglets sur la droite du logiciel.

{{< tabs title="Fenêtres:" >}}
{{% tab title="Variables" %}}
{{< fig
  fig="/fig/record/stm32/stm32cubeide_debug_variables_0.png"
  alt=""
  width=500px
  caption=""
  subcaption=""
>}}

Cette fenêtre liste les différentes variables du programme.
Pour chacune d'elles, sa valeur est également précisée.
{{% /tab %}}

{{% tab title="*Breakpoints*" %}}
{{< fig
  fig="/fig/record/stm32/stm32cubeide_breakpoint_list.png"
  alt=""
  width=500px
  caption=""
  subcaption=""
>}}

Cette fenêtre liste les différents breakpoints (points d'arrêt) existants.
Un point d'arrêt sert durant le débogage à arrêter l'exécution à un endroit précis.
Nous verrons dans la prochaine partie [exécution](#exécution) comment les utiliser.
{{% /tab %}}

{{% tab title="*Registers*" %}}
{{< fig
  fig="/fig/record/stm32/stm32cubeide_debug_registers.png"
  alt=""
  width=500px
  caption=""
  subcaption=""
>}}

Cette fenêtre liste les différents registres internes du processeur.
Pour chacun d'eux, il est possible de connaître la valeur qu'il contient.
{{% /tab %}}

{{% tab title="*SFRs*" %}}
{{< fig
  fig="/fig/record/stm32/stm32cubeide_debug_sfr_0.png"
  alt=""
  width=500px
  caption=""
  subcaption=""
>}}

Cette fenêtre liste les différents registres spéciaux (SFRs pour *Special Function Registers*) du système.
Nous verrons certains d'entre eux lors de l'utilisation des périphériques.
{{% /tab %}}

{{% tab title="*Disassembly*" %}}
{{< fig
  fig="/fig/record/stm32/stm32cubeide_disassembly.png"
  alt=""
  width=500px
  caption=""
  subcaption=""
>}}

Cette fenêtre présente code assembleur correspondant au code C compilé.
C'est ce qu'on appelle le *désassemblé*.
Pour activer cette fenêtre, cliquez sur **Window -> Show View -> Disassembly**.
{{% /tab %}}
{{< /tabs >}}

{{< caption 
  type="Figure" 
  key="debug-windows"
  main="Différentes fenêtres utiles pour le débogage."
  sub=""
>}}

### Exécution

Pour réaliser le débogage d'un code, il est nécessaire de l'exécuter.
Différentes commandes peuvent être utilisées pour cela.

{{< fig
  fig="/fig/record/stm32/stm32cubeide_icons-num.png"
  alt=""
  width=500px
  caption="Liste des icônes utilisables pour l'exécution"
  subcaption=""
  key="exec-icons"
>}}

La {{< num type="Figure" key="exec-icons" >}} reprend les icônes des principales commande.
Voici leurs fonctions:
1. Relancer le mode **Debug**.
2. Lancer l'exécution indéfiniment.
3. Terminer l'exécution en cours et en relancer une nouvelle depuis le début.
4. Poursuivre l'exécution qui a été interrompue.
5. Interrompre l'exécution en cours.
6. Terminer l'exécution en cours.
7. Exécuter la prochaine ligne de code. En cas d'appel de fonction, l'exécution s'arrête au début de la fonction.
8. Exécuter la prochaine ligne de code. En cas d'appel de fonction, celle-ci est entièrement exécutée. 

En utilisant ces différentes commandes, il est donc possible d'avancer dans l'exécution et de voir si une application fonctionne correctement.
Cependant, elles peuvent s'avérer limitées lorsqu'il s'agit d'observer le comportement d'une ligne de code en particulier.
Pour cela, il est possible d'introduire des ***points d'arrêts*** (ou *breakpoints*).

{{< fig
  fig="/fig/record/stm32/stm32cubeide_breakpoint.png"
  alt=""
  width=500px
  caption="Exemple de points d'arrêts dans une exécution."
  subcaption=""
  key="exec-breakpoints"
>}}

Un breakpoint est une directive indiquant au système d'interrompre l'exécution dès qu'il arrive à une certaine ligne du code.
En débogage, il est possible d'introduire des breakpoints en cliquant dans la zone bleue sur la gauche du code, comme sur la {{< num type="Figure" key="exec-breakpoints" >}}.
Ici, deux points d'arrêts sont placés lignes 92 et 100: à chaque fois que l'exécution arrivera à l'une de ces lignes, elle sera interrompue et devra être relancée manuellement.

{{% notice style="orange" title="Exercice" icon="bolt"%}}
## Exemples de programmes

Recopiez et analysez l'exécution des programmes ci-dessous.
Pour chacun d'entre eux, essayez de retrouver pour votre code C:
- le code désassemblé correspondant,
- le rôle de chacune des instructions du code désassemblé,
- l'impact du code sur les registres internes du processeur.

{{< tabs title="Programmes:" >}}
{{% tab title="Incrément" %}}
```c
int main(void) {
  /* USER CODE BEGIN 2 */
  uint32_t count = 0;
  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1) {
    /* USER CODE END WHILE */
	  count = count + 1;
    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
```
{{% /tab %}}

{{% tab title="Fonction" %}}
```c

uint32_t add32 (uint32_t a, uint32_t b) {
  return a + b;
}

int main(void) {
  /* USER CODE BEGIN 2 */
  uint32_t count = 0;
  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1) {
    /* USER CODE END WHILE */
	  count = add32(count, 1);
    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
```
{{% /tab %}}
{{< /tabs >}}
{{% /notice %}}

## Références

Cette partie regroupe les informations sur les {{< ref-cnt-page >}} références utilisées sur cette page.

{{< ref-list-page >}}