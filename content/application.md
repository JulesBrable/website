---
title: "Appliquer les concepts étudiés à un projet de data science"
date: 2022-03-03
author: "Romain Avouac et Lino Galiana"
draft: false
# layout options: single, single-sidebar
layout: single
---



L'objectif de cette mise en application est d'**illustrer les différentes étapes qui séparent la phase de développement d'un projet de celle de la mise en production**. Elle permettra de mettre en pratique les différents concepts présentés tout au long du cours.

Nous nous plaçons dans une situation initiale correspondant à la fin de la phase de développement d'un projet de data science. On a un notebook un peu monolithique, qui réalise les étapes classiques d'un *pipeline* de *machine learning* :
- import de données
- statistiques descriptives et visualisations
- *feature engineering*
- entraînement d'un modèle
- évaluation du modèle

**L'objectif est d'améliorer le projet de manière incrémentale jusqu'à pouvoir le mettre en production, en le valorisant sous une forme adaptée.** 

{{% box status="warning" title="Warning" icon="fa fa-exclamation-triangle" %}}
Il est important de bien lire les consignes et d'y aller progressivement.
Certaines étapes peuvent être rapides, d'autres plus fastidieuses ;
certaines être assez guidées, d'autres vous laisser plus de liberté.
Si vous n'effectuez pas une étape, vous risquez de ne pas pouvoir passer à
l'étape suivante qui en dépend.

Bien que l'exercice soit applicable sur toute configuration bien faite, nous 
recommandons de privilégier l'utilisation du [SSP Cloud](https://datalab.sspcloud.fr/home), où tous les 
outils nécessaires sont pré-installés et pré-configurés. 
{{% /box %}}




# Partie 1 : application des bonnes pratiques

Cette première partie vise à **rendre le projet conforme aux bonnes pratiques** présentées dans le cours. Elle fait intervenir les notions suivantes : 
- utilisation du **terminal**
- **qualité du code**
- **architecture de projets**
- **contrôle de version** avec Git
- **travail collaboratif** avec Git et GitHub

Le plan de la partie est le suivant :

0. :zero: Forker le dépôt et créer une branche de travail.
1. :one: S'assurer que le notebook s'exécute correctement.
2. :two: Modularisation : mise en fonctions et mise en module
3. :three: Utiliser un `main` script
4. :four:  Appliquer un *linter* au code
5. :five: Adopter une structure standardisée de package
6. :six: Exporter l'environnement Conda pour favoriser la portabilité du projet.
7. :seven: Mettre les données dans son bucket personnel sur le stockage MinIO du SSP Cloud et adapter la fonction d'import de données. Supprimer les fichiers `train.csv` et `test.csv` du dépôt Git.
8. :eight: Nettoyer le projet Git d'éventuels fichiers/dossiers indésirables (ex : les dossiers __pycache__) et ajouter le [fichier .gitignore adapté à Python](https://github.com/github/gitignore/blob/main/Python.gitignore) à la racine du projet. Ajouter le dossier `data/` au `.gitignore` pour éviter tout versioning de données.
9. :nine: Ouvrir une *pull request* sur le dépôt du projet.



## Etape 0: forker le dépôt d'exemple et créer une branche de travail

- Ouvrir un service `vscode` sur le [SSP Cloud](https://datalab.sspcloud.fr/home). Vous pouvez aller
dans la page `My Services` et cliquer sur `New service`. Sinon, vous
pouvez lancer le service en cliquant directement [ici](https://datalab.sspcloud.fr/launcher/inseefrlab-helm-charts-datascience/vscode?autoLaunch=false).

- Générer un jeton d'accès (*token*) sur GitHub afin de permettre l'authentification en ligne de commande à votre compte. La procédure est décrite [ici](https://docs.sspcloud.fr/onyxia-guide/controle-de-version#creer-un-jeton-dacces-token). Garder le jeton généré de côté.

- Forker le dépôt `Github` <i class="fab fa-github"></i> https://github.com/avouacr/ensae-reproductibilite-projet

- Clôner __votre__ dépôt `Github` <i class="fab fa-github"></i> en utilisant le
terminal depuis `Visual Studio` (`Terminal > New Terminal`) :

```shell
$ git clone https://<TOKEN>@github.com/avouacr/ensae-reproductibilite-projet.git
```

où `<TOKEN>` est à remplacer par le jeton que vous avez généré précédemment.

- Se placer avec le terminal dans le dossier en question : 

```shell
$ cd ensae-reproductibilite-projet
```

- Créez une branche `nettoyage` :

```shell
$ git checkout -b nettoyage
Switched to a new branch 'nettoyage'
```

## Etape 1 : s'assurer que le notebook s'exécute correctement

La première étape est simple, mais souvent oubliée : **vérifier que le code fonctionne correctement**. 

- Ouvrir dans VSCode le notebook `titanic.ipynb`, et choisir comme kernel `basesspcloud`
- Exécuter le notebook en entier pour vérifier s'il fonctionne
- Corriger l'erreur qui empêche la bonne exécution.

Il est maintenant temps de *commit* les changements effectués avec Git :

```shell
$ git add titanic.ipynb
$ git commit -m "Corrige l'erreur qui empêchait l'exécution"
$ git push
```

Essayez de *commit* vos changements à chaque étape de l'exercice, c'est une bonne habitude à prendre.

## Etape 2 : Modularisation - mise en fonctions et mise en module

Nous allons **mettre en fonctions les parties importantes de l'analyse, et les mettre dans un module afin de pouvoir les importer directement depuis le notebook**. En reformattant le code présent dans le notebook :

- créer une fonction qui importe les données d'entraînement (`train.csv`) et de test (`test.csv`) et renvoie des `DataFrames` pandas
- créer une (ou plusieurs) fonction(s) pour réaliser les étapes de *feature engineering*
- créer une fonction qui réalise le *split train/test* de validation
- créer une fonction qui entraîne et évalue un classifieur `RandomForest`, et qui prend en paramètre le nombre d'arbres (`n_estimators`). La fonction doit imprimer à la fin la performance obtenue et la matrice de confusion.
- mettre ces fonctions dans un module `functions.py`
- importer les fonctions via le module dans le notebook et vérifier que l'on retrouve bien les différents résultats en utilisant les fonctions.

{{% box status="warning" title="Warning" icon="fa fa-exclamation-triangle" %}}
Attention à bien **spécifier les dépendances** (packages à importer) dans le module pour que les fonctions puissent faire leur travail indépendamment du notebook !
{{% /box %}}

## Etape 3 : utiliser un `main` script

Fini le temps de l'expérimentation : on va maintenant essayer de se passer complètement du notebook. Pour cela, on va utiliser un `main` script, c'est à dire un script qui reproduit l'analyse en important et en exécutant les différentes fonctions dans l'ordre attendu.

- créer un script `main.py` (convention de nommage pour les `main` scripts en Python)
- importer les fonctions nécessaires à partir du module `functions.py`. Ne pas faire d' `import *`, ce n'est pas une bonne pratique ! Appeler les fonctions une par une en les séparant par des virgules
- programmer leur exécution dans l'ordre attendu dans le script
- vérifier que tout fonctionne bien en exécutant le `main` script à partir de l'exécutable Python :

```shell
$ python main.py
```

Si tout a correctement fonctionné, la performance du `RandomForest` et la matrice de confusion devraient s'afficher dans la console.

## Etape 4 : appliquer un *linter* au code

On va maintenant améliorer la qualité de notre code en appliquant les standards communautaires. Pour cela, on va utiliser le *linter* classique `PyLint`. 

Pour appliquer le linter à un script `.py`, la syntaxe à entrer dans le terminal est la suivante : 
```shell
$ pylint mon_script.py
```
Le linter renvoie alors une série d'irrégularités, en précisant à chaque fois la ligne de l'erreur et le message d'erreur associé (ex : mauvaise identation). Il renvoie finalement une note sur 10, qui estime la qualité du code à l'aune des standards communautaires (PEP8 et PEP257).

- appliquer une première fois le linter, respectivement aux scripts `functions.py` et `main.py`. Noter les notes obtenues.
- à partir des codes d'erreur, modifier le code pour résoudre les différents problèmes un par un
- viser une note minimale de 9/10 pour `main.py` et 6/10 pour `functions.py`.

{{% box status="warning" title="Warning" icon="fa fa-exclamation-triangle" %}}
N'hésitez pas à taper un code d'erreur sur un moteur de recherche pour obtenir plus d'informations si jamais le message n'est pas clair !
{{% /box %}}

## Etape 5 : adopter une structure standardisée de package

S'inspirer du template de projet [cookiecutter datascience](https://drivendata.github.io/cookiecutter-data-science/) pour construire une structure de package.

# Partie 2 : construction d'un projet portable et reproductible

# Partie 3 : mise en production