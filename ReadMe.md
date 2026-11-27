# ML Azure

Azure est une infrastructure cloud développée par Microsoft, proposant divers services.

Dans le cadre de ce projet, nous utilisons principalement le service **Azure Machine Learning**, mais nous recourons également à d’autres services pour des tâches annexes.

Ce rapport vous guidera chronologiquement à travers la création d’un modèle de **régression linéaire** dans Azure, destiné à prédire la consommation d’électricité future à une date spécifique.
  
## Données d’entraînement

Nos données d’entraînement ont la forme **(145362, 2)**, avec deux colonnes : **Datetime** et **PJME_MW**, correspondant à la quantité d’électricité consommée à cette date.

## Avant Azure

Avant de travailler sur Azure, nous avons créé un notebook, servant de brouillon pour le pipeline Azure ML.

## Azure

### Compte de stockage

La première étape sur Azure a consisté à créer un **compte de stockage**, en activant l’option de hiérarchie, afin de disposer d’une chronologie traçable pour nos données selon la structure suivante :
  
```

Année/
├─ Mois/
│ ├─ Jour/
│ │ ├─ data.csv

```

Au sein du **compte de stockage**, nous créons deux conteneurs : l’un pour les données sources et l’autre pour les données cibles, où notre pipeline **Data Factory** les placera.

### Data Factory
**Data Factory** est tout simplement un service destiné à faciliter la création de pipelines de données. Dans le cadre de ce projet, nous l’utilisons de manière assez simple : il s’agit simplement de copier les données du conteneur source vers le conteneur cible, en les plaçant dans le chemin d’accès approprié conformément à la structure spécifiée ci-dessus. Pour ce faire, nous utilisons un script personnalisé afin de créer un chemin d’accès sur mesure en fonction de la date indiquée dans le « Linker » du conteneur cible, qui est un outil permettant de relier les conteneurs ** compteurs du **compte de stockage** aux pipelines **Data Factory**.

##### Exemple de création d’un Linker :
![[Screenshot 2026-11-26 at 10-03-46 dawrf - Azure Data Factory.png]]

### Azure Machine Learning
Le service principal de notre projet, **Azure Machine Learning**, est un outil permettant de créer des pipelines spécifiquement destinés aux algorithmes d’apprentissage automatique. Il couvre le prétraitement des données, l’ingénierie des caractéristiques, l’entraînement du modèle et son évaluation.

Pour créer ce pipeline, nous utilisons d’autres fonctionnalités du service **Machine Learning**.
#### Data store
Le **Data store** est similaire au Linker de **Data Factory**. Il pointe vers un conteneur d’un **compte de stockage**.

#### Data asset
La **Data asset** s’apparente à un **Data store**, mais elle pointe vers un fichier spécifique situé dans le conteneur vers lequel pointe le **Data store** créé.

#### Cluster de calcul
**Le cluster de calcul** est la machine sur laquelle le pipeline s'exécute. Dans le cadre de ce projet, nous avons utilisé un cluster présentant les caractéristiques suivantes :

```
- **Région :** Europe du Nord.
- Type : CPU.
- Machine virtuelle : `StandardDS11v2`.
- Ressources : 2 vCPU et 14 Go de RAM.
- Nombre minimal de nœuds : 0.
- Nombre maximal de nœuds : 1.
- Mise à l’échelle automatique : activée.

```

#### Designer
**Designer** est l’endroit où nous créons le pipeline proprement dit.
Nous avons utilisé des modules déjà créés par Microsoft pour le concevoir, car nous n’avons pas besoin d’une solution trop puissante.

Nous utilisons tout d’abord la **ressource de données** que nous avons créée et qui pointe vers notre fichier `data.csv`.

Ensuite, nous la relions à d’autres modules, en nous basant sur l’ébauche que nous avons rédigée dans le notebook. Nous avons notamment utilisé des modules `Exécuter un script Python`, pour le débogage et l’ingénierie des caractéristiques.

##### Exemple de module « Execute Python Script » :
![[Screenshot 2026-11-26 at 17-25-50 Authoring - Azure Machine Learning.png]]

Nous avons construit le pipeline nœud par nœud. Nous avons choisi la `régression linéaire` comme modèle. Nous l’avons entraîné, évalué, puis enregistré.

Pour exécuter le pipeline, nous avons choisi un **cluster de calcul** sur lequel l’exécuter.

##### Exemple de sélection d’un cluster de calcul :
  ![[Screenshot 2026-11-26 at 18-15-27 Authoring - Azure Machine Learning 1.png]]

##### Pipeline final
![[Screenshot 2026-11-26 at 20-33-21 Authoring - Azure Machine Learning.png]]

##### Métriques du modèle :
![[Screenshot 2026-11-26 at 20-07-50 Pipeline-Created-on-11-26-2026 - Azure Machine Learning.png]]