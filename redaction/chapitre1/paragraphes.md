# Chapitre 1 : Création et manipulation de Series

Les Series constituent la structure fondamentale de données unidimensionnelles dans Pandas, permettant de stocker et d'organiser efficacement des informations. Elles sont particulièrement utiles pour gérer des séquences de données étiquetées tout en offrant une grande souplesse pour le calcul et la transformation.

Dans ce chapitre :

* Comparaison entre structures ndarray et Series
* Méthodes de création de Series
* Exploration des propriétés et méthodes essentielles
* Réalisation d'opérations arithmétiques
* Techniques d'extraction et de filtrage
* Gestion et manipulation des index
* Méthodes de transformation

## ndarray vs Series

Une Series diffère d'un ndarray (tableau NumPy) principalement par la présence d'un index explicite, permettant d'étiqueter chaque donnée au lieu de se limiter à une position entière. Cette association clé-valeur facilite considérablement la recherche et l'organisation des informations.

```python
import pandas as pd
import numpy as np

# Le ndarray se base sur la position, la Series sur l'index
arr = np.array([10, 20])
ser = pd.Series([10, 20], index=['a', 'b'])

```

## Series depuis un ndarray, liste

Vous pouvez initialiser une Series à partir de divers formats de données comme des listes Python ou des tableaux NumPy (ndarrays). Si aucun index n'est précisé, Pandas en génère un par défaut commençant à zéro.

```python
# Création depuis une liste
ser_liste = pd.Series([1, 2, 3])

# Création depuis un ndarray avec index explicite
ser_arr = pd.Series(np.array([10, 20]), index=['x', 'y'])

```

## Propriétés et méthodes d'une Series

Les Series possèdent des attributs intégrés tels que `.values` pour accéder aux données brutes et `.index` pour consulter les étiquettes. De nombreuses méthodes permettent d'obtenir rapidement des informations descriptives sur la série.

```python
# Accès aux données et aux index
donnees = ser_arr.values
index = ser_arr.index

# Méthode pour obtenir un résumé statistique simple
resume = ser_arr.describe()

```

## Opérations arithmétiques

Pandas aligne automatiquement les données en fonction de leurs index lors des calculs arithmétiques. Cette fonctionnalité permet d'effectuer des opérations entre deux Series sans se soucier de l'ordre des éléments.

```python
# Addition automatique selon les index communs
resultat = ser_arr + 5

# Opération entre deux Series
totaux = ser_arr + ser_arr

```

## Extraction et recherche

L'extraction de données s'effectue par découpage (slicing) classique, ou via `.loc` (basé sur les étiquettes) et `.iloc` (basé sur la position entière). Ces outils garantissent un accès précis, que vous connaissiez le nom de l'index ou sa position dans la série.

```python
# Sélection par index (label) et position
valeur_loc = ser_arr.loc['x']
valeur_iloc = ser_arr.iloc[0]

```

## Travailler sur les index

Les index peuvent être modifiés, réorganisés ou utilisés pour filtrer la Series afin d'extraire des sous-ensembles spécifiques. Manipuler les index permet d'adapter la structure des données à vos besoins d'analyse.

```python
# Renommer ou modifier l'index
ser_arr.index = ['nouveau_x', 'nouveau_y']

# Filtrage par index
subset = ser_arr[['nouveau_x']]

```

## Transformation

La transformation consiste à appliquer des fonctions sur les valeurs d'une Series pour modifier ou normaliser les données. Ces méthodes permettent d'effectuer des traitements complexes sur l'ensemble de la série de manière concise.

```python
# Application d'une fonction simple sur chaque valeur
ser_transformee = ser_arr.apply(lambda x: x * 2)

```

### Synthèse

Les Series offrent un cadre flexible pour la manipulation de données grâce à leur indexation personnalisable, leur alignement automatique lors des calculs et leurs méthodes puissantes d'extraction et de transformation.

### Exercices

1. Créez une Series à partir d'une liste de 4 nombres et affichez son index ainsi que ses valeurs.
2. À partir de cette même Series, utilisez `.loc` pour modifier la valeur correspondant au troisième élément de votre index.