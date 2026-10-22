# Chapitre 1 : Création et manipulation de Series

Les `Series` constituent la structure de données unidimensionnelle fondamentale de Pandas, conçue pour stocker et manipuler efficacement des séquences de valeurs. Si elles s'apparentent de prime abord à un tableau unidimensionnel, elles se distinguent radicalement des `ndarray` de NumPy par un concept clé : **l'indexation explicite par étiquettes (labels)**.

Dans ce chapitre :

* Comparaison entre structures ndarray et Series
* Méthodes de création d'une `Series`
* Attributs et méthodes d'inspection d'une `Series`
* Réalisation d'opérations arithmétiques
* Techniques d'extraction et de filtrage
* Gestion et manipulation des index
* Méthodes de transformation

## Comparaison entre structures ndarray et Series

Une Series diffère d'un ndarray (tableau NumPy) principalement par la présence d'un index explicite, permettant d'étiqueter chaque donnée au lieu de se limiter à une position entière. Cette association clé-valeur facilite considérablement la recherche et l'organisation des informations.

Pour bien saisir la spécificité d'une `Series`, il est utile de la comparer au `ndarray` NumPy dont elle hérite la performance et la logique de calcul vectorisé :

* **Le `ndarray` (NumPy) :**
* Possède uniquement un **index implicite** d'entiers positionnels (de `0` à `N-1`).
* L'accès aux éléments se fait exclusivement par leur position numérique.
* Idéal pour le calcul mathématique pur, mais aveugle au sens sémantique des données (ex: impossible de lier directement une valeur à un nom de client ou à une date sans gérer un tableau parallèle).


* **La `Series` (Pandas) :**
* Est composée de **deux tableaux unidimensionnels couplés** : les **valeurs** (`values`) et les **labels** (`index`).
* L'index peut contenir des chaînes de caractères, des dates, ou des entiers personnalisés.
* Offre une **sémantique métier** : chaque valeur est explicitement étiquetée.

```python

import pandas as pd
import numpy as np

# Le ndarray se base sur la position, la Series sur l'index
arr = np.array([10, 20])
ser = pd.Series([10, 20], index=['a', 'b'])

```
**L'accès intuitif par étiquette :** Il devient possible de récupérer une valeur via son libellé (ex: `serie['Paris']`) plutôt que par sa position `serie[2]`, rendant le code plus lisible et moins fragile face aux réorganisations de données.
**L'alignement automatique :** Lors d'opérations arithmétiques entre deux `Series`, Pandas aligne automatiquement les données en fonction de leurs labels et non de leur position dans le tableau, évitant ainsi de nombreuses erreurs de correspondance.
**La gestion des données manquantes :** Si un label est absent lors d'une opération ou d'une fusion, Pandas gère l'incompatibilité en y associant proprement une valeur nulle (`NaN`), là où un `ndarray` provoquerait une erreur de dimension.
 
## Méthodes de création d'une `Series`

Les `Series` peuvent être instanciées à partir de diverses structures de données en Python ou NumPy. Selon la source utilisée, Pandas sait interpréter automatiquement les données ou vous permet de spécifier un index sur mesure.

Voici les 4 principales méthodes de création à retenir :

À partir d'une liste (ou d'un tuple) Python. C'est la méthode la plus simple pour démarrer. Si aucun index n'est explicitement fourni, Pandas génère automatiquement un index positionnel entier allant de `0` à `N-1`.

```python

import pandas as pd

# Création depuis une liste simple
ser_liste = pd.Series([10, 20, 30, 40])
print(ser_liste)
# Sortie :
# 0    10
# 1    20
# 2    30
# 3    40
# dtype: int64

```

À partir d'un tableau NumPy (`ndarray`).Idéal lorsque vous manipulez des données numériques issues de calculs scientifiques. Vous pouvez associer un index explicite (personnalisé) dès la construction.

```python

import numpy as np

# Création depuis un ndarray avec un index textuel explicite
arr = np.array([1.5, 2.3, 3.8])
ser_arr = pd.Series(arr, index=['Janvier', 'Février', 'Mars'])
print(ser_arr)
# Sortie :
# Janvier    1.5
# Février    2.3
# Mars       3.8
# dtype: float64

```

À partir d'un dictionnaire Python.C'est l'une des méthodes les plus élégantes et naturelles : les **clés** du dictionnaire deviennent automatiquement les **labels de l'index**, et les **valeurs** constituent le contenu de la `Series`.

```python

# Création depuis un dictionnaire (les clés deviennent l'index)
capitales = {'France': 'Paris', 'Italie': 'Rome', 'Espagne': 'Madrid'}
ser_dict = pd.Series(capitales)
print(ser_dict)
# Sortie :
# France      Paris
# Italie       Rome
# Espagne    Madrid
# dtype: object

```

À partir d'une valeur scalaire (constante).Si vous devez initialiser une `Series` entière avec une valeur unique, vous passez un scalaire en argument tout en définissant obligatoirement l'index souhaité. La valeur est alors répétée (diffusée) pour chaque label.

```python

# Création par répétition d'une valeur unique
ser_scalaire = pd.Series(25, index=['Lundi', 'Mardi', 'Mercredi', 'Jeudi'])
print(ser_scalaire)
# Sortie :
# Lundi       25
# Mardi       25
# Mercredi    25
# Jeudi       25
# dtype: int64

```
## Attributs et méthodes d'inspection d'une `Series`

Une fois une `Series` créée, il est indispensable de pouvoir interroger sa structure, sa taille et ses métadonnées. Pandas met à disposition un ensemble d'attributs et de méthodes légers (qui ne nécessitent pas de calculs lourds) pour inspecter rapidement l'objet.

### Les principaux attributs structurels

* **`.values`** : Renvoie les données brutes sous la forme d'un tableau NumPy (`ndarray`).
* **`.index`** : Permet de consulter ou de modifier les étiquettes associées aux données.
* **`.shape`** : Renvoie un tuple indiquant les dimensions de la série (pour une `Series`, il s'agit toujours d'un tuple à un seul élément, ex: `(4,)`).
* **`.size`** (ou la fonction standard **`len()`**) : Indique le nombre total d'éléments contenus dans la série.
* **`.dtype`** : Précise le type de données stockées (ex: `int64`, `float64`, `object`).

### Les méthodes d'information et de résumé

* **`.head(n)` / `.tail(n)**` : Permettent d'afficher respectivement les $n$ premières ou $n$ dernières valeurs (très utile pour les grands jeux de données).
* **`.describe()`** : Génère un résumé statistique complet (compte, moyenne, écart-type, minimum, quartiles, maximum) pour les séries numériques.

```python
import pandas as pd
import numpy as np

# Création d'une Series de test
ser = pd.Series([10, 25, 30, 45, 60], index=['a', 'b', 'c', 'd', 'e'])

# Inspection de la structure et des dimensions
print("Forme (shape) :", ser.shape)     # Sortie : (5,)
print("Taille (size)  :", ser.size)      # Sortie : 5
print("Type de données:", ser.dtype)     # Sortie : int64

# Accès aux composants fondamentaux
print("Valeurs brutes :", ser.values)   # Sortie : [10 25 30 45 60]
print("Étiquettes     :", ser.index)     # Sortie : Index(['a', 'b', 'c', 'd', 'e'], dtype='object')

# Résumé statistique rapide
print("\nRésumé statistique :")
print(ser.describe())

```

## Opérations arithmétiques et alignement automatique

Contrairement aux tableaux NumPy ou aux listes Python classiques qui s'appuient uniquement sur la position des éléments, Pandas effectue toutes ses opérations arithmétiques **en s'appuyant sur les index (les étiquettes)**.

Cette fonctionnalité majeure, appelée **alignement automatique**, garantit que les calculs sont effectués entre les bonnes données, même si les séries n'ont pas le même ordre ou ne partagent pas exactement les mêmes clés.

### Tableau récapitulatif des opérations arithmétiques

| Opération | Opérateur | Méthode Pandas équivalente | Description |
| --- | --- | --- | --- |
| **Addition** | `+` | `.add()` | Additionne les éléments correspondants |
| **Soustraction** | `-` | `.sub()` | Soustrait les éléments correspondants |
| **Multiplication** | `*` | `.mul()` | Multiplie les éléments correspondants |
| **Division** | `/` | `.div()` | Divise les éléments correspondants |
| **Puissance** | `**` | `.pow()` | Élève à la puissance |

---

### Opération avec un scalaire (Broadcasting)

Lorsqu'on applique une opération mathématique entre une `Series` et une valeur unique (un scalaire), cette valeur est « diffusée » à l'ensemble des éléments de la série.

```python
import pandas as pd

# Series initiale de prix
prix = pd.Series([100, 200, 300], index=['A', 'B', 'C'])

# Ajout d'une taxe fixe de 20 sur tous les éléments
prix_avec_taxe = prix + 20
print(prix_avec_taxe)
# A    120
# B    220
# C    320
# dtype: int64

```

### Addition de deux `Series` aux index parfaitement identiques

Si deux `Series` possèdent exactement les mêmes index, l'opération s'effectue élément par élément de manière classique.

```python
qte_magasin_1 = pd.Series([10, 5, 8], index=['Pommes', 'Poires', 'Bananes'])
qte_magasin_2 = pd.Series([5, 2, 3], index=['Pommes', 'Poires', 'Bananes'])

# Somme des stocks des deux magasins
stock_total = qte_magasin_1 + qte_magasin_2
print(stock_total)
# Pommes     15
# Poires      7
# Bananes    11
# dtype: int64

```

### Alignement automatique sur des index partiels ou décalés

C'est la grande force de Pandas. Si les index diffèrent, Pandas aligne les étiquettes communes et attribue automatiquement la valeur manquante **`NaN`** (Not a Number) là où une correspondance est introuvable.

```python
ventes_janvier = pd.Series([100, 150], index=['Paris', 'Lyon'])
ventes_fevrier = pd.Series([120, 200, 90], index=['Lyon', 'Marseille', 'Paris'])

# Addition de deux séries aux index asymétriques
total_ventes = ventes_janvier + ventes_fevrier
print(total_ventes)
# Lyon         270.0  (150 + 120)
# Marseille    NaN    (Absent en janvier)
# Paris        190.0  (100 + 90)
# dtype: float64

```

### Multiplication et propagation des types flottants

Les opérations arithmétiques transforment souvent le type des données (par exemple, des entiers `int64` passent en nombres à virgule `float64` dès qu'un `NaN` ou une division intervient).

```python
notes = pd.Series([12, 15, 18], index=['Maths', 'Physique', 'Chimie'])
coefficient = pd.Series([2, 3, 1.5], index=['Maths', 'Physique', 'Chimie'])

# Calcul des notes pondérées
notes_ponderees = notes * coefficient
print(notes_ponderees)
# Maths       24.0
# Physique    45.0
# Chimie      27.0
# dtype: float64

```

### Gestion des valeurs manquantes avec les méthodes dédiées (`fill_value`)

Pour éviter l'apparition de `NaN` lors d'une opération sur des index non concordants (comme dans l'exemple 3), il est possible d'utiliser la méthode explicite associée (ex: `.add()`) en spécifiant un paramètre de remplacement `fill_value`.

```python
stock_A = pd.Series([10, 20], index=['Clavier', 'Souris'])
stock_B = pd.Series([15, 30], index=['Souris', 'Ecran'])

# Addition en remplaçant les valeurs absentes par 0 avant le calcul
stock_global = stock_A.add(stock_B, fill_value=0)
print(stock_global)
# Clavier    10.0  (10 + 0 pour l'écran absent dans B)
# Ecran      30.0  (0 + 30 pour l'écran absent dans A)
# Souris     35.0  (20 + 15)
# dtype: float64

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
