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

**Le `ndarray` (NumPy) :**
- Possède uniquement un **index implicite** d'entiers positionnels (de `0` à `N-1`).
- L'accès aux éléments se fait exclusivement par leur position numérique.
- Idéal pour le calcul mathématique pur, mais aveugle au sens sémantique des données (ex: impossible de lier directement une valeur à un nom de client ou à une date sans gérer un tableau parallèle).

**La `Series` (Pandas) :**
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

---

## Méthodes de création d'une `Series`

Les `Series` peuvent être instanciées à partir de diverses structures de données en Python ou NumPy. Selon la source utilisée, Pandas sait interpréter automatiquement les données ou vous permet de spécifier un index sur mesure.

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
---

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
--- 

## Les `Series` temporelles : gestion des dates et du temps

Dans le monde de l'analyse de données (finance, IoT, logs serveurs, météorologie), le temps est une dimension critique. Pandas intègre nativement une gestion poussée des séries temporelles (*Time Series*) grâce à un index spécialisé : le **`DatetimeIndex`**.

Celui-ci permet non seulement de stocker des horodatages de manière structurée, mais aussi d'effectuer des sélections par tranches de dates (slicing textuel) et des rééchantillonnages automatiques.

### Conversion de chaînes de caractères en dates (`pd.to_datetime`)

Le plus souvent, les dates brutes proviennent de fichiers CSV ou de bases de données sous forme de chaînes de caractères (textes). La fonction `pd.to_datetime()` permet de les convertir en objets temporels natifs (`Timestamp`).

```python
import pandas as pd

# Series initiale avec des dates au format texte
dates_texte = pd.Series([120, 145, 130], index=['2026-06-01', '2026-06-02', '2026-06-03'])

# Conversion des étiquettes textuelles en un véritable DatetimeIndex
dates_texte.index = pd.to_datetime(dates_texte.index)

print(dates_texte.index)
# Sortie : DatetimeIndex(['2026-06-01', '2026-06-02', '2026-06-03'], dtype='datetime64[ns]', freq=None)

```

### Génération automatique de séquences temporelles (`pd.date_range`)

Pour initialiser un suivi régulier (journalier, horaire, mensuel), la fonction `pd.date_range()` évite de saisir les dates à la main. Elle génère une suite d'horodatages selon une fréquence définie.

```python
import numpy as np

# Création d'une plage de 5 jours consécutifs à partir d'aujourd'hui
index_jours = pd.date_range(start='2026-10-01', periods=5, freq='D')

# Association avec des données aléatoires ou mesurées
mesures = pd.Series([22.5, 23.0, 19.5, 21.0, 24.2], index=index_jours)
print(mesures)
# 2026-10-01    22.5
# 2026-10-02    23.0
# 2026-10-03    19.5
# 2026-10-04    21.0
# 2026-10-05    24.2
# dtype: float64

```

### Le découpage temporel intelligent (Slicing par date)

Grâce au `DatetimeIndex`, vous pouvez extraire des plages de dates par simple texte, sans formater précisément les objets. Pandas comprend des écritures partielles comme `"2026-10"` pour cibler tout un mois ou `"2026-10-01":"2026-10-03"` pour un intervalle.

```python
# Sélection de toutes les données du 2 au 4 octobre
sous_serie = mesures.loc['2026-10-02':'2026-10-04']
print(sous_serie)
# 2026-10-02    23.0
# 2026-10-03    19.5
# 2026-10-04    21.0
# dtype: float64

```

---

### Extraction d'attributs temporels via l'accesseur `.dt`

Une fois l'index ou les données typés en temps, l'accesseur `.dt` offre une multitude de propriétés pour extraire instantanément des composantes calendaires (année, mois, jour, nom du jour, etc.).

```python
# Extraire le nom du jour de la semaine pour chaque mesure
jours_semaine = mesures.index.day_name()
print(jours_semaine)
# Sortie : Index(['Thursday', 'Friday', 'Saturday', 'Sunday', 'Monday'], dtype='object')

# Extraire uniquement le mois de chaque date
mois = mesures.index.month
print(mois)
# Sortie : Index([10, 10, 10, 10, 10], dtype='int32')

```
--- 

## La conversion de type dans une `Series` (`.astype()`)

Lors de l'importation de données (provenant de fichiers CSV, JSON ou d'API), il arrive fréquemment que les types de données (`dtype`) ne soient pas adaptés aux calculs que vous souhaitez réaliser. Par exemple, des chiffres peuvent être interprétés comme du texte (chaînes de caractères), ou des nombres entiers peuvent nécessiter d'être convertis en nombres à virgule pour intégrer des valeurs manquantes (`NaN`).

La méthode **`.astype()`** est l'outil fondamental de Pandas pour convertir explicitement le type des valeurs d'une `Series`.

### Conversion de texte en nombres entiers (`int`)

Il est très courant que des données numériques lues depuis un fichier texte brut soient typées en `object` (chaîne de caractères). Il faut les convertir pour pouvoir effectuer des calculs arithmétiques.

```python
import pandas as pd

# Series initialement lue comme du texte (dtype: object)
prix_texte = pd.Series(['120', '250', '340'], index=['Article_A', 'Article_B', 'Article_C'])
print("Type initial :", prix_texte.dtype)  # Sortie : object

# Conversion en entiers 64 bits
prix_int = prix_texte.astype('int64')
print("Type après conversion :", prix_int.dtype)  # Sortie : int64
print(prix_int + 10)  # Les calculs arithmétiques sont désormais possibles

```

### Conversion d'entiers en nombres à virgule (`float`) pour intégrer des `NaN`

En Pandas, un tableau d'entiers purs (`int64`) ne peut pas contenir de valeurs manquantes (`NaN`), car celles-ci sont gérées en interne sous forme de `float`. Convertir une série en `float` est donc une étape classique avant d'insérer des trous ou des données incomplètes.

```python
# Series d'entiers
quantites = pd.Series([10, 25, 40], index=['Q1', 'Q2', 'Q3'])

# Conversion en float pour permettre l'accueil futur de valeurs manquantes
quantites_float = quantites.astype('float64')
print(quantites_float)
# Q1    10.0
# Q2    25.0
# Q3    40.0
# dtype: float64

```

### Conversion de nombres en texte (`str`)

Pour des opérations de formatage, d'affichage ou pour concaténer des identifiants, il est parfois nécessaire de ramener des valeurs numériques au format textuel.

```python
# Series d'identifiants numériques
ids = pd.Series([101, 102, 103], index=['Utilisateur_1', 'Utilisateur_2', 'Utilisateur_3'])

# Conversion en chaînes de caractères
ids_texte = ids.astype(str)
print(ids_texte.dtype)  # Sortie : object (ou string)

# Concaténation textuelle facilitée
codes_complets = "ID_" + ids_texte
print(codes_complets)
# Utilisateur_1    ID_101
# Utilisateur_2    ID_102
# Utilisateur_3    ID_103
# dtype: object

```

### Conversion en booléens (`bool`) ou en types catégoriels (`category`)

Pour optimiser l'espace mémoire ou structurer des données binaires (vrai/faux), vous pouvez convertir vos séries vers des types logiques ou catégoriels.

```python
# Series représentant des indicateurs de réussite (1 ou 0)
statuts_bruts = pd.Series([1, 0, 1, 1], index=['Test_1', 'Test_2', 'Test_3', 'Test_4'])

# Conversion en booléens
statuts_bool = statuts_bruts.astype(bool)
print(statuts_bool)
# Test_1     True
# Test_2    False
# Test_3     True
# Test_4     True
# dtype: bool

```
--- 

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

## Extraction et recherche de données

L'une des grandes forces de Pandas réside dans la flexibilité de ses mécanismes d'extraction. Selon que vous souhaitez interroger vos données par leur **position numérique** (du premier au dernier élément) ou par leur **sémantique (leurs étiquettes)**, Pandas propose des outils dédiés et sécurisés : `.iloc`, `.loc`, ainsi que le découpage (`slicing`) classique.

### Comprendre la différence : `.loc` vs `.iloc`

* **`.iloc` (pour *Integer Location*) :**
* Fonctionne exclusivement avec des **positions entières** (de `0` à `N-1`), exactement comme une liste Python classique ou un tableau NumPy.
* Le borne de fin est exclusive lors d'un découpage (slicing).


* **`.loc` (pour *Location*) :**
* Fonctionne exclusivement avec les **étiquettes (labels)** de l'index.
* Le borne de fin est inclusive lors d'un découpage (contrairement aux habitudes de Python), ce qui est très pratique pour cibler explicitement une plage nommée.


---

Imaginons une `Series` représentant la population (en millions) de quelques grandes villes françaises :

```python
import pandas as pd

population = pd.Series([2.1, 1.3, 0.6, 0.5, 0.4], index=['Paris', 'Marseille', 'Lyon', 'Toulouse', 'Nice'])
print(population)
# Paris        2.1
# Marseille    1.3
# Lyon         0.6
# Toulouse     0.5
# Nice         0.4
# dtype: float64

```

Extraction d'un élément unique (par position ou par étiquette)

Vous pouvez récupérer une valeur isolée soit en connaissant son nom exact (`.loc`), soit sa position dans le classement (`.iloc`).

```python
# Accès par étiquette avec .loc
pop_paris = population.loc['Paris']
print("Population de Paris (par label) :", pop_paris)  # Sortie : 2.1

# Accès par position entière avec .iloc (le premier élément à l'indice 0)
premiere_ville = population.iloc[0]
print("Première valeur (par position) :", premiere_ville)  # Sortie : 2.1

```

Le découpage (Slicing) par position avec `.iloc`

Le slicing avec `.iloc` se comporte comme les listes standard en Python : l'indice de début est inclus, mais l'indice de fin est **exclu**.

```python
# Extrait les éléments de la position 1 incluse à 4 exclue (indices 1, 2 et 3)
sous_ensemble_iloc = population.iloc[1:4]
print(sous_ensemble_iloc)
# Marseille    1.3
# Lyon         0.6
# Toulouse     0.5
# dtype: float64

```

Le découpage (Slicing) par étiquette avec `.loc`

Attention à la particularité majeure de `.loc` : **la borne de fin est incluse**. Si vous demandez les villes de 'Paris' à 'Lyon', 'Lyon' apparaîtra bien dans le résultat.

```python
# Extrait toutes les villes de 'Paris' jusqu'à 'Lyon' (incluses)
sous_ensemble_loc = population.loc['Paris':'Lyon']
print(sous_ensemble_loc)
# Paris        2.1
# Marseille    1.3
# Lyon         0.6
# dtype: float64

```

L'accès direct via les crochets `[]` (Shorthand)

Pandas autorise un accès direct par étiquette ou par position via les crochets `[]`. Cependant, pour éviter toute ambiguïté (surtout si l'index est lui-même composé d'entiers), il est **fortement recommandé** d'utiliser explicitement `.loc` ou `.iloc` dans vos codes de production.

```python
# Accès direct par étiquette
print(population['Lyon'])  # Sortie : 0.6

# Accès par tranche (slicing) basée sur les labels (inclusif également)
print(population['Marseille':'Toulouse'])

```

Le filtrage conditionnel (Recherche par critères)

Vous pouvez passer une condition booléenne directement à l'intérieur des crochets pour extraire uniquement les éléments qui répondent à un critère précis.

```python
# Filtrer les villes dont la population est supérieure à 1 million
villes_peuplees = population[population > 1.0]
print(villes_peuplees)
# Paris        2.1
# Marseille    1.3
# dtype: float64

```
--- 

## Travailler et manipuler les index d'une `Series`

L'index d'une `Series` ne se contente pas d'étiqueter les données : c'est un véritable outil de pilotage. Il est possible de le modifier, de le renommer partiellement, de réindexer la série ou d'extraire des sous-ensembles ciblés pour adapter la structure de vos données en cours d'analyse.

Méthodes essentielles pour manipuler les index :

### Modification globale directe de l'index (`.index = ...`)

Vous pouvez remplacer entièrement les étiquettes d'une `Series` en réassignant une nouvelle liste ou un nouveau tableau à son attribut `.index`. Attention toutefois : la taille de la nouvelle liste doit correspondre exactement au nombre d'éléments.

```python
import pandas as pd

# Series initiale avec des indices numériques
ser = pd.Series([10, 20, 30], index=[0, 1, 2])

# Modification directe de l'index
ser.index = ['Alpha', 'Beta', 'Gamma']
print(ser)
# Alpha    10
# Beta     20
# Gamma    30
# dtype: int64

```

### Renommer des étiquettes ciblées (`.rename()`)

Plutôt que de tout réécrire, la méthode `.rename()` permet de modifier uniquement certaines étiquettes spécifiques via un dictionnaire de correspondance, tout en préservant le reste.

```python
# Renommer uniquement 'Beta' en 'Delta'
ser_renommee = ser.rename({'Beta': 'Delta'})
print(ser_renommee)
# Alpha    10
# Delta    20
# Gamma    30
# dtype: int64

```

### Réindexer une série avec `.reindex()`

La méthode `.reindex()` permet d'ajuster une `Series` à une nouvelle liste d'index. Si un nouveau label n'existait pas dans la série d'origine, Pandas y insère automatiquement la valeur manquante `NaN` (ou une valeur par défaut via `fill_value`).

```python
# On définit une nouvelle liste d'index plus large
nouvel_index = ['Alpha', 'Gamma', 'Omega']

ser_reindexee = ser.reindex(nouvel_index, fill_value=0)
print(ser_reindexee)
# Alpha    10
# Gamma    30
# Omega     0  (Valeur par défaut grâce à fill_value)
# dtype: int64

```

### Filtrer ou réordonner par une liste de labels

Vous pouvez passer une liste d'étiquettes directement entre crochets pour extraire un sous-ensemble ou modifier l'ordre d'affichage des éléments d'une `Series`.

```python
# Extraire uniquement certains éléments et inverser leur ordre
subset = ser.loc[['Gamma', 'Alpha']]
print(subset)
# Gamma    30
# Alpha    10
# dtype: int64

```

## Transformation des valeurs d'une `Series`

La transformation consiste à modifier, normaliser ou enrichir les données contenues dans une `Series`. Selon la complexité du traitement à réaliser, Pandas propose plusieurs approches : les **opérations vectorisées** (les plus rapides), l'utilisation de fonctions via **`.apply()`**, ou le remplacement par correspondance via **`.map()`**.

Méthodes essentielles pour transformer vos données :

### La transformation vectorisée (L'approche native et recommandée)

C'est la méthode la plus efficace en termes de performance. Grâce au moteur de NumPy sous-jacent, les opérateurs mathématiques ou les fonctions mathématiques s'appliquent instantanément à l'ensemble de la `Series` sans nécessiter de boucle explicite.

```python
import pandas as pd
import numpy as np

# Series initiale de températures en Celsius
celsius = pd.Series([0, 20, 25, 100], index=['Hiver', 'Printemps', 'Été', 'Ébullition'])

# Conversion vectorisée en Fahrenheit : (C * 9/5) + 32
fahrenheit = (celsius * 9 / 5) + 32
print(fahrenheit)
# Hiver          32.0
# Printemps      68.0
# Été            77.0
# Ébullition    212.0
# dtype: float64

```

### L'application d'une fonction personnalisée avec `.apply()`

Lorsque la transformation demande une logique conditionnelle ou un traitement spécifique qu'une simple formule mathématique ne peut pas exprimer, la méthode `.apply()` permet d'exécuter une fonction sur chaque valeur de la `Series`.

```python
# Utilisation d'une fonction lambda pour classifier les températures
statut = celsius.apply(lambda x: 'Chaud' if x > 22 else 'Frais')
print(statut)
# Hiver         Frais
# Printemps     Frais
# Été           Chaud
# Ébullition    Chaud
# dtype: object

```

### Utiliser une fonction Python nommée avec `.apply()`

Pour des traitements plus complexes, vous pouvez définir une fonction classique en Python et la passer en argument à `.apply()`.

```python
# Définition d'une fonction de normalisation personnalisée
def ajuster_temperature(valeur):
    if valeur < 0:
        return 0
    return valeur + 5  # Ajout d'une marge d'erreur

# Application de la fonction
celsius_ajuste = celsius.apply(ajuster_temperature)
print(celsius_ajuste)
# Hiver           5
# Printemps      25
# Été            30
# Ébullition    105
# dtype: int64

```

### Le mappage par dictionnaire avec `.map()`

La méthode `.map()` est idéale pour transformer des valeurs discrètes (comme des codes, des catégories ou du texte) en vous appuyant sur un dictionnaire de correspondance. Les valeurs absentes du dictionnaire sont automatiquement converties en `NaN`.

```python
# Series de codes climatiques
codes = pd.Series(['H', 'P', 'E'], index=['Ville_A', 'Ville_B', 'Ville_C'])

# Dictionnaire de correspondance (Mapping)
correspondance = {'H': 'Hivernal', 'P': 'Tempéré', 'E': 'Estival'}

# Transformation par correspondance
saisons = codes.map(correspondance)
print(saisons)
# Ville_A    Hivernal
# Ville_B     Tempéré
# Ville_C     Estival
# dtype: object

```

--- 

## Exemple de synthèse 

Pour illustrer la puissance combinée des fonctionnalités de base des `Series`, imaginons le suivi du chiffre d'affaires journalier d'un site e-commerce sur une semaine. Ce script complet montre comment structurer, nettoyer, transformer et analyser ces données de bout en bout :

```python
import pandas as pd
import numpy as np

# Création de données initiales (avec un oubli le 4 octobre pour simuler un trou)
ventes_brutes = pd.Series(
    [150, 220, 180, 310, 290], 
    index=pd.to_datetime(['2026-10-01', '2026-10-02', '2026-10-03', '2026-10-05', '2026-10-06'])
)

print("--- 1. Données brutes initiales ---")
print(ventes_brutes)

# Réindexation pour obtenir une chronologie continue du 1 au 6 octobre
# Les jours manquants (comme le 4) se voient attribuer la valeur 0 grâce à fill_value
plage_jours = pd.date_range(start='2026-10-01', end='2026-10-06', freq='D')
ventes_completes = ventes_brutes.reindex(plage_jours, fill_value=0)

print("\n--- 2. Série continue après réindexation ---")
print(ventes_completes)

# Opération arithmétique (Calcul du CA sachant qu'un article rapporte en moyenne 45 €)
panier_moyen = 45.0
chiffre_affaires = ventes_completes * panier_moyen

print("\n--- 3. Chiffre d'affaires calculé (en €) ---")
print(chiffre_affaires)

# Extraction et filtrage : Sélection des jours où le CA dépasse 10 000 €
# Utilisation de l'accesseur temporel .dt pour afficher le nom du jour en clair
jours_succes = chiffre_affaires[chiffre_affaires > 10000]

print("\n--- 4. Jours à fort chiffre d'affaires (> 10 000 €) ---")
print(jours_succes)

# Inspection globale : Résumé statistique de la semaine
print("\n--- 5. Résumé statistique global du CA ---")
print(chiffre_affaires.describe())

```

* **L'indexation sémantique et temporelle** permet de manipuler les données par date de manière intuitive.
* **L'alignement automatique et `.reindex()**` évitent les erreurs lors de la manipulation de séries asymétriques ou incomplètes.
* **La vectorisation** permet d'appliquer instantanément des calculs mathématiques globaux sans utiliser de boucles `for`.
* **Le filtrage booléen** isole en une seule ligne les données répondant à un critère métier précis.

--- 

## Exercices de fin de chapitre

**Excerices 1:** Extraction d'une période spécifique et transformation ciblée des valeurs. Dans cet exemple, nous disposons d'un suivi journalier du trafic d'un site web. Nous souhaitons **extraire** uniquement la semaine où s'est déroulée une campagne promotionnelle, puis **transformer** ces données en leur appliquant un coefficient de correction lié à une anomalie de comptage.

**Excerices 2:** Extraction de composantes temporelles (.dt) et transformation conditionnelle. Cet exemple illustre comment extraire des informations sémantiques à partir des dates (les jours de la semaine) pour ensuite appliquer une transformation conditionnelle (par exemple, appliquer un tarif week-end ou jour ouvré).
