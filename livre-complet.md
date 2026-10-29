# Chapitre 1 : Rappel NumPy 

Avant d'explorer la puissance des structures de Pandas (`Series` et `DataFrame`), il est indispensable de maîtriser **NumPy** (*Numerical Python*). Pandas n'est pas une bibliothèque indépendante : elle est entièrement construite sur le moteur mathématique de NumPy. Comprendre comment NumPy stocke, manipule et calcule les données en mémoire, c'est comprendre ce qui se passe "sous le capot" de Pandas.

Dans ce chapitre :

* Pourquoi NumPy pour comprendre Pandas
* Le `ndarray` : création, `shape`, `dtype`
* Indexation et slicing
* Opérations vectorisées et broadcasting
* Fonctions mathématiques et statistiques de base (`sum`, `mean`, paramètre `axis`)
* Valeurs manquantes : `np.nan` et ses pièges
* Exemple de synthèse
* Exercices de fin de chapitre

---

## Pourquoi NumPy pour comprendre Pandas ?

En Python, les listes standard sont très souples (elles peuvent contenir du texte, des nombres et des objets de types différents), mais elles souffrent de deux handicaps majeurs en calcul scientifique :

**La lenteur d'exécution** : Python doit vérifier le type de chaque élément à chaque opération.
**La dispersion en mémoire** : Les éléments d'une liste sont des pointeurs disséminés un peu partout en mémoire RAM, ce qui empêche d'optimiser le cache du processeur.

**NumPy** introduit le **`ndarray`** (tableau multidimensionnel homogène) :

* **Un typage unique** : Tous les éléments du tableau ont exactement le même type de données (`int64`, `float64`, etc.), ce qui permet un stockage contigu en mémoire.
* **La vectorisation** : Les calculs sont délégués à des routines écrites en C/C++, s'affranchissant totalement des boucles `for` en Python.
* **Le socle de Pandas** : Une `Series` ou une colonne de `DataFrame` n'est rien d'autre qu'un `ndarray` enrichi d'un index sémantique (des étiquettes).

---

## Le `ndarray` : création, `shape`, `dtype`

Le `ndarray` est l'objet central de NumPy. Ses propriétés fondamentales sont le type des données (`dtype`) et les dimensions (`shape`).

### Exemple 1 : Création d'un tableau unidimensionnel et inspection de sa structure

```python
import numpy as np

# Création d'un tableau 1D à partir d'une liste Python
notes_array = np.array([12, 15, 18, 14])

print("Tableau :", notes_array)
print("Type des données (.dtype) :", notes_array.dtype)  # Affiche int64 (ou int32 selon l'archi)
print("Dimensions (.shape) :", notes_array.shape)        # Affiche (4,) -> 4 éléments sur 1 dimension

```

### Création d'une matrice bidimensionnelle avec typage explicite

Pour représenter un tableau de données tabulaires (lignes et colonnes), on utilise un tableau 2D en forçant un type précis de données (par exemple, des nombres à virgule `float64`).

```python
# Création d'une matrice 2D (3 lignes, 2 colonnes) avec un type explicite
matrice_prix = np.array([
    [50.5, 10],
    [25.0, 42],
    [80.0, 5]
], dtype=np.float64)

print(matrice_prix)
print("Type forcé (.dtype) :", matrice_prix.dtype)   # Affiche float64
print("Forme de la matrice (.shape) :", matrice_prix.shape) # Affiche (3, 2) -> 3 lignes, 2 colonnes

```

---

## Indexation et slicing

L'indexation permet d'accéder à des éléments précis, tandis que le *slicing* (découpage) permet d'extraire des sous-ensembles de données sans dupliquer les tableaux en mémoire (les tableaux partagent la même zone mémoire par vue).

### Slicing sur un tableau unidimensionnel

```python
import numpy as np

temperatures = np.array([18, 20, 22, 25, 27, 24, 21])

# Extraire les 3 premiers jours (indices 0 à 2 exclus la fin)
print("Début de semaine :", temperatures[0:3])

# Extraire un élément sur deux (pas de 2)
print("Un jour sur deux :", temperatures[::2])

```

### Indexation et slicing sur un tableau bidimensionnel (Lignes et Colonnes)

Sur un tableau 2D, la syntaxe suit le format `[ligne, colonne]`.

```python
# Matrice représentant [Prix_HT, Stock] pour 4 produits
catalogue = np.array([
    [50.0, 15],
    [25.0, 42],
    [80.0, 8],
    [200.0, 3]
])

# Récupérer le prix du 3ème produit (ligne d'indice 2, colonne 0)
print("Prix du 3ème produit :", catalogue[2, 0])

# Extraire toutes les lignes mais uniquement la colonne des stocks (colonne 1)
print("Tous les stocks :", catalogue[:, 1])

```

---

## Opérations vectorisées et broadcasting

Les opérations vectorisées appliquent une formule mathématique à l'ensemble du tableau instantanément. Le **broadcasting** (diffusion) permet quant à lui d'effectuer des calculs entre des tableaux de dimensions différentes (par exemple, ajouter une constante à une matrice ou additionner un vecteur à une matrice).

### Opération vectorisée pure

```python
import numpy as np

prix_ht = np.array([50.0, 25.0, 80.0])

# Application d'une taxe de 20% sur tout le tableau en une seule opération (pas de boucle)
prix_ttc = prix_ht * 1.20
print("Prix TTC vectorisés :", prix_ttc)

```

### Le Broadcasting (Diffusion)

```python
# Matrice de prix (3 produits, 2 fournisseurs)
tarifs = np.array([
    [100, 105],
    [200, 210],
    [300, 315]
])

# Un vecteur de frais de port fixes par produit (3 valeurs)
frais_port = np.array([5, 10, 15])

# Grâce au broadcasting, NumPy "étire" automatiquement le vecteur pour l'additionner ligne par ligne
# Attention : les dimensions doivent être compatibles
tarifs_avec_port = tarifs + frais_port[:, np.newaxis] 
print("Tarifs avec frais de port :\n", tarifs_avec_port)

```

---

## Fonctions mathématiques et statistiques de base (`sum`, `mean`, paramètre `axis`)

NumPy intègre des fonctions de réduction puissantes. Le paramètre crucial à maîtriser est **`axis`**, qui détermine la direction de l'opération :

* **`axis=0`** : L'opération s'effectue **verticalement** (en descendant le long des lignes pour chaque colonne).
* **`axis=1`** : L'opération s'effectue **horizontalement** (en se déplaçant de gauche à droite à travers les colonnes pour chaque ligne).

### Fonctions statistiques globales sur un tableau 1D

```python
import numpy as np

notes = np.array([12, 15, 10, 18, 14])

print("Somme totale :", np.sum(notes))
print("Moyenne générale :", np.mean(notes))
print("Valeur maximale :", np.max(notes))

```

### Utilisation du paramètre `axis` sur une matrice 2D

```python
# Matrice de notes (3 élèves, 3 matières : Maths, Physique, Info)
notes_classe = np.array([
    [12, 14, 16],
    [10, 11, 12],
    [15, 17, 18]
])

# Moyenne par matière (en descendant les lignes, donc axis=0)
moyenne_matieres = np.mean(notes_classe, axis=0)
print("Moyenne par matière (Maths, Physique, Info) :", moyenne_matieres)

# Moyenne générale de chaque élève (de gauche à droite, donc axis=1)
moyenne_eleves = np.mean(notes_classe, axis=1)
print("Moyenne de chaque élève :", moyenne_eleves)

```

---

## Valeurs manquantes : `np.nan` et ses pièges

En analyse de données, l'absence d'information est courante. NumPy représente les valeurs manquantes ou non valides par la constante spéciale **`np.nan`** (Not a Number).

### La propagation de `np.nan` dans les calculs

Le piège majeur de `np.nan` est qu'il est "contagieux" : toute opération arithmétique impliquant un `np.nan` retourne automatiquement `nan`.

```python
import numpy as np

mesures = np.array([10.5, 12.0, np.nan, 14.2])

# Le calcul de la somme classique échoue à cause du nan
print("Somme naïve :", np.sum(mesures))  # Affiche nan

# Solution : utiliser les fonctions dédiées ignorant les NaN (ex: np.nansum)
print("Somme propre (sans nan) :", np.nansum(mesures))
print("Moyenne propre :", np.nanmean(mesures))

```

### Détection et filtrage des valeurs manquantes

```python
# Identifier où se trouvent les NaN dans un tableau
masque_nan = np.isnan(mesures)
print("Masque booléen des NaN :", masque_nan)

# Filtrer le tableau pour ne garder que les valeurs valides
donnees_valides = mesures[~np.isnan(mesures)]
print("Données nettoyées :", donnees_valides)

```

---

## Exemple de synthèse

Ce script complet combine la création d'un tableau structuré, le broadcasting, les calculs statistiques par axe (`axis`), et la gestion sécurisée des valeurs manquantes (`np.nan`).

```python
import numpy as np

# Données brutes de ventes (4 produits, 3 trimestres) avec un oubli (np.nan)
ventes_brutes = np.array([
    [1200, 1500, np.nan],
    [800,  950,  1100],
    [3000, 3200, 3400],
    [450,  500,  550]
])

print("--- Matrice des ventes brutes ---")
print(ventes_brutes)

# Correction des valeurs manquantes : remplacement des NaN par 0 pour les calculs
ventes_nettoyees = np.nan_to_num(ventes_brutes, nan=0.0)

# Application d'une prime de fin d'année de 50€ (Broadcasting sur chaque trimestre)
prime = np.array([50, 50, 50])
ventes_primees = ventes_nettoyees + prime

print("\n--- Ventes après application de la prime ---")
print(ventes_primees)

# Synthèse par axe : Chiffre d'affaires total par produit (axis=1)
ca_par_produit = np.sum(ventes_primees, axis=1)
print("\n--- CA total par produit ---")
print(ca_par_produit)

# Synthèse globale : Vente moyenne par trimestre (axis=0, en ignorant les erreurs potentielles)
moyenne_trimestre = np.mean(ventes_primees, axis=0)
print("\n--- Vente moyenne par trimestre ---")
print(moyenne_trimestre)

```

---

## Exercices de fin de chapitre

**Exercice 1 :** Manipulation de matrices et slicing

Créez un tableau NumPy unidimensionnel contenant les entiers de 10 à 50 (inclus).
Extrayez-en une tranche (slicing) contenant uniquement les nombres pairs situés entre les indices 10 et 30.

**Exercice 2 :** Statistiques par axe et gestion des données manquantes

Vous disposez du tableau de températures suivant relevé dans 3 villes (lignes) sur 4 jours (colonnes) :

```python
temperatures = np.array([
    [21.5, 22.0, np.nan, 24.1],
    [15.0, 16.2, 17.0, 16.5],
    [28.0, np.nan, 30.2, 29.5]
])

```

Calculez la température moyenne pour chaque jour (en effectuant l'opération sur les lignes, `axis=0`) en gérant proprement les valeurs manquantes (`np.nan`).
Identifiez la ville (l'indice de la ligne) qui enregistre la température maximale globale sur l'ensemble de la période.

# Chapitre 2 : Présentation des Series 

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

À partir d'un dictionnaire Python. C'est l'une des méthodes les plus élégantes et naturelles : les **clés** du dictionnaire deviennent automatiquement les **labels de l'index**, et les **valeurs** constituent le contenu de la `Series`.

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

À partir d'une valeur scalaire (constante). Si vous devez initialiser une `Series` entière avec une valeur unique, vous passez un scalaire en argument tout en définissant obligatoirement l'index souhaité. La valeur est alors répétée (diffusée) pour chaque label.

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
## Exemple de synthèse

## Exercices de fin de chapitre

**Exercice 1 :** 
**Exercice 2 :** 
---
 
# Chapitre 3 : Manipuler les Series

Dans ce chapitre :

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

Pour éviter l'apparition de `NaN` lors d'une opération sur des index non concordants , il est possible d'utiliser la méthode explicite associée (ex: `.add()`) en spécifiant un paramètre de remplacement `fill_value`.

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

## Travailler avec les index d'une `Series`

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

print("--- Données brutes initiales ---")
print(ventes_brutes)

# Réindexation pour obtenir une chronologie continue du 1 au 6 octobre
# Les jours manquants (comme le 4) se voient attribuer la valeur 0 grâce à fill_value
plage_jours = pd.date_range(start='2026-10-01', end='2026-10-06', freq='D')
ventes_completes = ventes_brutes.reindex(plage_jours, fill_value=0)

print("\n--- Série continue après réindexation ---")
print(ventes_completes)

# Opération arithmétique (Calcul du CA sachant qu'un article rapporte en moyenne 45 €)
panier_moyen = 45.0
chiffre_affaires = ventes_completes * panier_moyen

print("\n--- Chiffre d'affaires calculé (en €) ---")
print(chiffre_affaires)

# Extraction et filtrage : Sélection des jours où le CA dépasse 10 000 €
# Utilisation de l'accesseur temporel .dt pour afficher le nom du jour en clair
jours_succes = chiffre_affaires[chiffre_affaires > 10000]

print("\n--- Jours à fort chiffre d'affaires (> 10 000 €) ---")
print(jours_succes)

# Inspection globale : Résumé statistique de la semaine
print("\n--- Résumé statistique global du CA ---")
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


# Chapitre 4 : Présentation un DataFrame

Le DataFrame est la structure de données pivot de Pandas, permettant de manipuler des tableaux bidimensionnels de manière flexible et efficace. Ce chapitre détaille comment structurer, créer et extraire des données au sein de cet environnement tabulaire.

Dans ce chapitre :

* Comparaison entre ndarray et DataFrame
* Organisation interne et propriétés
* Création à partir de listes, tuples et dictionnaires
* Importation depuis CSV et Excel
* Manipulation des colonnes, lignes et index
* Techniques d'extraction de données

## Series vs DataFrame (et comparaison avec NumPy)

La différence fondamentale entre les structures de Pandas (`Series` et `DataFrame`) et les tableaux NumPy (`ndarray`) réside dans l'**indexation**. Alors qu'un `ndarray` se base uniquement sur des positions entières implicites, les objets Pandas intègrent des étiquettes explicites :

* **`Series`** : Tableau unidimensionnel étiqueté pouvant contenir n'importe quel type de données.
* **`DataFrame`** : Structure tabulaire bidimensionnelle composée de lignes et de colonnes étiquetées (comparable à une table SQL).

Cette structure permet un **accès sémantique** aux données, facilitant grandement leur interprétation et leur manipulation.

---

### Création d'une `Series` avec un index personnalisé

Contrairement à un tableau simple, une `Series` associe chaque valeur à une étiquette explicite.

```python
import pandas as pd

# Création d'une Series représentant des notes par matière
notes = pd.Series([15, 18, 12], index=['Maths', 'Physique', 'Informatique'])
print(notes)

```

### Création d'un `DataFrame` (structure 2D)

Le `DataFrame` étend ce concept à deux dimensions en combinant un index de lignes et des étiquettes de colonnes.

```python
import pandas as pd

# Un DataFrame associe des lignes étiquetées à des colonnes nommées
data = {
    'Nom': ['Alice', 'Bob', 'Charlie'],
    'Âge': [25, 30, 35],
    'Ville': ['Paris', 'Lyon', 'Marseille']
}
df = pd.DataFrame(data)
print(df)

```

### Accès sémantique aux données (`.loc`)

Grâce aux étiquettes, il est possible d'interroger les données par leur nom plutôt que par leur position numérique, ce qui rend le code plus lisible.

```python
# Accès direct à une valeur dans la Series via son étiquette textuelle
print(notes['Physique'])  # Affiche : 18

# Accès à une colonne entière du DataFrame par son nom
print(df['Ville'])

```

### Comparaison directe avec un `ndarray` NumPy

Cette comparaison illustre la limite d'un tableau NumPy brut face à un `DataFrame` lorsque les données nécessitent un contexte sémantique.

```python
import numpy as np
import pandas as pd

# NumPy : indexation uniquement par position entière (pas de nom de colonne)
arr = np.array([[10, 20], [30, 40]])
print("NumPy (sans contexte) :")
print(arr)

# DataFrame Pandas : ajout d'un index et de noms de colonnes pour l'interprétation
df_comparaison = pd.DataFrame(arr, index=['Ligne_1', 'Ligne_2'], columns=['Col_A', 'Col_B'])
print("\nPandas DataFrame (avec étiquettes sémantiques) :")
print(df_comparaison)

```

---

## Organisation interne et propriétés d'un DataFrame

Un `DataFrame` possède une architecture interne rigoureuse : il peut être visualisé comme un dictionnaire ordonné ou un conteneur de **`Series`** partageant un **index commun**. Techniquement, chaque colonne d'un `DataFrame` est donc une `Series` indépendante.

Ses propriétés principales permettent d'inspecter rapidement sa structure, ses métadonnées et ses types de données sans avoir à parcourir tout le tableau :

* **`.shape`** : Renvoie un tuple indiquant le nombre de lignes et de colonnes `(lignes, colonnes)`.
* **`.index`** : Retourne les étiquettes des lignes.
* **`.columns`** : Retourne les noms des colonnes.
* **`.dtypes`** : Indique le type de données (entier, texte, flottant, etc.) pour chaque colonne.
* **`.values`** (ou `.to_numpy()`) : Permet d'extraire les données sous forme de tableau NumPy brut.

---

### Vérifier l'organisation interne (une colonne = une Series)

L'extraction d'une colonne d'un `DataFrame` renvoie bien un objet de type `Series`.

```python
import pandas as pd

data = {
    'Produit': ['Clavier', 'Souris', 'Écran'],
    'Prix': [45.0, 25.0, 199.99],
    'Stock': [120, 85, 14]
}
df = pd.DataFrame(data, index=['P1', 'P2', 'P3'])

# Vérification du type d'une colonne individuelle
colonne_prix = df['Prix']
print(type(colonne_prix))  # Affiche : <class 'pandas.core.series.Series'>

```

### Explorer les propriétés structurelles (`.shape`, `.index`, `.columns`, `.dtypes`)

Ces attributs sont indispensables lors de l'exploration initiale d'un jeu de données pour en comprendre la taille et la nature.

```python
# Dimensions du DataFrame (3 lignes, 3 colonnes)
print("Dimensions (.shape) :", df.shape)

# Identifiants des lignes
print("Index (.index) :", df.index)

# Noms des colonnes
print("Colonnes (.columns) :", df.columns)

# Types de données par colonne
print("\nTypes des données (.dtypes) :\n", df.dtypes)

```

### Les axes d'un DataFrame

L'argument `axis` indique la direction d'une opération : `axis=0` (ou `'index'`) agit verticalement le long des lignes, tandis que `axis=1` (ou `'columns'`) agit horizontalement à travers les colonnes.

L'axe 0 (axis=0 ou 'index') représente les lignes (vertical). Les opérations descendent le long des lignes.
L'axe 1 (axis=1 ou 'columns') représente les colonnes (horizontal). Les opérations se déplacent de gauche à droite à travers les colonnes.

`df.mean(axis=0)` : Calcule la moyenne de **chaque colonne** (en descendant les lignes).
`df.sum(axis=1)` : Calcule la somme des valeurs de **chaque ligne** (de gauche à droite).
`df.drop('ma_colonne', axis=1)` : Supprime **une colonne** entière du DataFrame.
`df.dropna(axis=0)` : Supprime les **lignes** qui contiennent au moins une valeur manquante.

Utilisation de `.axes`
```python
import pandas as pd

df = pd.DataFrame({
    'Maths': [10, 15, 20],
    'Physique': [12, 14, 16]
}, index=['Élève_A', 'Élève_B', 'Élève_C'])

# Afficher les deux axes du DataFrame
print(df.axes)

```

df.axes[0] correspond à l'axe vertical (les lignes).
df.axes[1] correspond à l'axe horizontal (les colonnes).

```python
[Index(['Élève_A', 'Élève_B', 'Élève_C'], dtype='object'), 
 Index(['Maths', 'Physique'], dtype='object')]

```

Calculer la moyenne par colonne (`axis=0`).C'est le comportement par défaut de Pandas. On calcule la moyenne de chaque matière pour l'ensemble des élèves en descendant les lignes.

```python
print(df.mean(axis=0))
# Résultat :
# Maths      15.0
# Physique   14.0
# dtype: float64

```

Calculer la moyenne par ligne (`axis=1`). Si l'on souhaite calculer la moyenne générale de chaque élève (en additionnant ses notes de gauche à droite) :

```python
print(df.mean(axis=1))
# Résultat :
# 0    11.0
# 1    14.5
# 2    18.0
# dtype: float64

```

```python
# Supprimer une colonne (on se déplace horizontalement sur les colonnes)
df = df.drop(columns='Physique') # Équivalent à df.drop('Physique', axis=1)

# Supprimer une ligne par son index (on se déplace verticalement sur les lignes)
df = df.drop(index=0)            # Équivalent à df.drop(0, axis=0)

```

### Accéder aux valeurs brutes (`.values`)

Si vous avez besoin de convertir ou de manipuler les données pures sans les étiquettes Pandas (par exemple pour les passer à un modèle de Machine Learning avec Scikit-Learn), la propriété `.values` extrait un tableau NumPy (`ndarray`).

```python
# Extraction des données sous forme de tableau NumPy bidimensionnel
matrice_brute = df.values
print(matrice_brute)
print(type(matrice_brute))  # Affiche : <class 'numpy.ndarray'>

```
---
## Gestion des données invalides (`NaN`)

Les valeurs manquantes, représentées par `NaN` (Not a Number), sont fréquentes dans les jeux de données réels. Pandas propose des méthodes robustes pour identifier, supprimer ou remplacer ces valeurs afin d'éviter des erreurs lors de vos calculs.

```python
import pandas as pd
import numpy as np

# Création d'un DataFrame avec des valeurs manquantes
df = pd.DataFrame({'A': [1, np.nan, 3], 'B': [4, 5, np.nan]})

# Supprimer les lignes contenant au moins un NaN
df_clean = df.dropna()

# Remplacer les NaN par une valeur spécifique (ex: 0)
df_filled = df.fillna(0)

```
---

## Créations multiples : listes, tuples et dictionnaires

La création d'un `DataFrame` est extrêmement flexible et s'adapte à la structure initiale de vos données. Selon que vous partez de tableaux bruts, de flux JSON (API) ou de variables isolées, plusieurs approches s'offrent à vous :

* **Liste de listes ou de tuples** : Idéal pour transformer une matrice numérique ou un ensemble de lignes brutes en tableau étiqueté.
* **Dictionnaire de listes (ou de tableaux)** : La méthode la plus intuitive et courante, où chaque clé du dictionnaire devient automatiquement le nom d'une colonne.
* **Liste de dictionnaires** : Parfait pour traiter des données orientées lignes (comme des fichiers JSON ou des réponses d'API), où chaque dictionnaire représente une ligne.

---

### Création à partir d'une liste de listes ou de tuples

Cette approche nécessite de spécifier explicitement les noms des colonnes (`columns`) et éventuellement l'index des lignes (`index`).

```python
import pandas as pd

# Données sous forme de liste de tuples (ou listes imbriquées)
donnees_tuples = [
    ('Paris', 2140000),
    ('Lyon', 513275),
    ('Marseille', 861635)
]

# Création avec définition explicite des colonnes et de l'index
df_tuples = pd.DataFrame(donnees_tuples, columns=['Ville', 'Population'], index=['Île-de-France', 'Auvergne-Rhône-Alpes', 'PACA'])
print(df_tuples)

```

### Création à partir d'un dictionnaire de listes (orientation colonnes)

C'est la méthode standard la plus lisible. Chaque clé associe un nom de colonne à une séquence de valeurs de même longueur.

```python
import pandas as pd

# Chaque clé du dictionnaire correspond à une colonne du DataFrame
data_dict = {
    'Produit': ['Ordinateur', 'Tablette', 'Smartphone'],
    'Prix': [1200, 450, 800],
    'En_Stock': [True, False, True]
}

df_dict = pd.DataFrame(data_dict)
print(df_dict)

```

### Création à partir d'une liste de dictionnaires (orientation lignes)

Si vos données proviennent d'une source au format JSON (comme une API web), chaque élément de la liste représente une ligne complète. Les clés manquantes sont automatiquement remplies par des valeurs manquantes (`NaN`).

```python
import pandas as pd

# Liste de dictionnaires (chaque dictionnaire = une ligne)
utilisateurs = [
    {'ID': 1, 'Nom': 'Alice', 'Role': 'Admin'},
    {'ID': 2, 'Nom': 'Bob', 'Role': 'Utilisateur'},
    {'ID': 3, 'Nom': 'Charlie'}  # La clé 'Role' est absente ici
]

df_lignes = pd.DataFrame(utilisateurs)
print(df_lignes)

```

### Combinaison de `Series` indépendantes

Vous pouvez également assembler des objets `Series` préexistants dans un dictionnaire pour former un `DataFrame` cohérent. Pandas alignera automatiquement les données selon leurs index.

```python
import pandas as pd

# Création de deux Series distinctes
titres = pd.Series(['Livre A', 'Livre B', 'Livre C'])
ventes = pd.Series([150, 320, 95])

# Assemblage dans un DataFrame via un dictionnaire
df_series = pd.DataFrame({
    'Ouvrage': titres,
    'Exemplaires_Vendus': ventes
})

print(df_series)

```
---

## Chargement de fichiers externes : CSV et Excel

Pandas propose des fonctions hautement performantes pour importer directement des données tabulaires stockées dans des fichiers (`pd.read_csv` et `pd.read_excel`). Ces fonctions automatisent de nombreuses tâches complexes, telles que :

* La reconnaissance automatique ou manuelle des en-têtes de colonnes.
* La détection des types de données (entiers, flottants, chaînes de caractères, dates).
* La gestion des valeurs manquantes (`NaN`).

Pour être pleinement efficace, ces fonctions acceptent de nombreux paramètres de configuration (comme le séparateur de champs, l'encodage des caractères ou le choix des feuilles de calcul).

---

### Importation de base d'un fichier CSV

C'est le cas le plus simple. Par défaut, `pd.read_csv` considère que les colonnes sont séparées par une virgule (`,`) et que la première ligne contient les en-têtes.

```python
import pandas as pd

# Chargement d'un fichier CSV standard
# df_csv = pd.read_csv('clients.csv')

# Simulation avec un chemin fictif ou URL directe
url = "https://raw.githubusercontent.com/datasciencedojo/datasets/master/titanic.csv"
df = pd.read_csv(url)

# Dimensions globales
print(f"Dimensions (lignes, colonnes) : {df.shape}")
print(f"Nombre total de cellules : {df.size}")
print(f"Nombre de dimensions : {df.ndim}") # Renvoie 2 pour un DataFrame

# Structure des index et des colonnes
print(f"Type d'index : {type(df.index)}")
print(f"Plage des index : {df.index.min()} à {df.index.max()}")
print(f"Liste des colonnes : {df.columns.tolist()}")

# Dictionnaire des types de données (très utile pour du code programmatique)
print(df.dtypes.to_dict())

# Vérifier la structure et les types réels
df.info(memory_usage='deep')

# Obtenir un premier résumé statistique global
print(df.describe(include='all'))

# Aperçu des 5 premières lignes
print(dfc.head())

```

### CSV avec paramètres avancés (séparateur, encodage, index)

En Europe, les fichiers CSV utilisent souvent le point-virgule (`;`) comme séparateur et des accents nécessitant un encodage spécifique (`utf-8` ou `latin-1`). Vous pouvez aussi définir directement une colonne comme index.

```python
import pandas as pd

# Importation d'un CSV personnalisé :
# - sep=';' : délimiteur de colonnes point-virgule
# - encoding='utf-8' : gestion correcte des caractères accentués
# - index_col='ID' : utilise la colonne 'ID' comme index du DataFrame
# df_avance = pd.read_csv('ventes_france.csv', sep=';', encoding='utf-8', index_col='ID')

```

### Importation basique d'un fichier Excel

La fonction `pd.read_excel` permet de charger des classeurs Excel (`.xlsx` ou `.xls`). Par défaut, elle importe la **première feuille** du fichier.

```python
import pandas as pd

# Chargement de la première feuille d'un fichier Excel
# df = pd.read_excel('budget_annuel.xlsx')

```

### Excel avancé (choix de la feuille et des lignes à ignorer)

Les fichiers Excel contiennent souvent des en-têtes publicitaires, des titres ou plusieurs onglets. Vous pouvez cibler précisément la zone utile grâce à `sheet_name` et `skiprows`.

```python
import pandas as pd

# Importation ciblée d'un fichier Excel :
# - sheet_name='2026_Q1' : sélectionne une feuille spécifique par son nom
# - skiprows=2 : ignore les 2 premières lignes (ex: titres du tableau)
# - usecols="A:D" : limite l'importation aux colonnes de A à D
# df_Q1 = pd.read_excel('entreprise.xlsx', sheet_name='2026_Q1', skiprows=2, usecols="A:D")

```
---
## Exemple de synthèse

## Exercices de fin de chapitre

**Exercice 1 :** 
**Exercice 2 :** 
---

# Chapitre 5 : Manipuler un DataFrame

Dans ce chapitre :

## Travailler sur les colonnes : sélection, calcul et nettoyage

Manipuler les colonnes est l'une des tâches les plus fréquentes en science des données. Cela permet d'enrichir un jeu de données avec de nouvelles variables, de nettoyer les structures en éliminant le superflu ou de clarifier les étiquettes.

Grâce à la **vectorisation** de Pandas
, ces opérations s'exécutent de manière extrêmement rapide sur de grands volumes de données, évitant ainsi l'utilisation de boucles `for` lentes et inefficaces.

---

### Ajout d'une colonne calculée (vectorisation)

Vous pouvez créer de nouvelles variables en appliquant des opérations mathématiques directes entre des colonnes existantes.

```python
import pandas as pd
import numpy as np

# Création d'un DataFrame de base
df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2]
})

# Calcul vectorisé du prix TTC (TVA à 20%) et du total par ligne
df['Prix_TTC'] = df['Prix_HT'] * 1.20
df['Total'] = df['Prix_TTC'] * df['Quantite']

print(df)

```

Ajoute une colonne arbitraire avec des valeurs invalides `np.nan`
```python
df["X"] = np.nan

```

Le paramètre `axis` est aussi obligatoire lorsqu'on supprime des éléments avec `drop()` :

```python
# Supprimer une colonne (on se déplace horizontalement sur les colonnes)
df = df.drop(columns='Physique') # Équivalent à df.drop('Physique', axis=1)

# Supprimer une ligne par son index (on se déplace verticalement sur les lignes)
df = df.drop(index=0)            # Équivalent à df.drop(0, axis=0)

```

### Renommer des colonnes (`.rename`)

Pour rendre un jeu de données plus explicite ou harmoniser des noms de colonnes, la méthode `.rename()` est idéale.

```python
# Renommage ciblé via un dictionnaire de correspondance
df_renomme = df.rename(columns={
    'Prix_HT': 'Prix_Hors_Taxe',
    'Quantite': 'Qte_Stock'
})

print(df_renomme.columns)

```

### Supprimer des colonnes (`drop`)

Lorsqu'une variable n'est plus utile ou contient trop de valeurs manquantes, vous pouvez la supprimer du `DataFrame`.

```python
# Suppression d'une ou plusieurs colonnes avec la méthode drop
# axis=1 ou columns=... indique qu'il s'agit d'une colonne
df_reduit = df.drop(columns=['Total'])

print(df_reduit.columns)

```

### Sélectionner et réordonner un sous-ensemble de colonnes

Il est souvent nécessaire de restreindre le `DataFrame` à un sous-ensemble de variables précises et de modifier leur ordre d'affichage.

```python
# Passer une liste de noms de colonnes permet de filtrer et réordonner simultanément
df_ordonne = df[['Produit', 'Quantite', 'Prix_HT', 'Prix_TTC', 'Total']]

print(df_ordonne.head())

```
---

## Travailler sur les lignes : filtrage, ajout et suppression

La manipulation des lignes permet de sélectionner, d'ajouter ou de supprimer des enregistrements spécifiques au sein d'un `DataFrame`.

Le filtrage repose généralement sur des **conditions booléennes** (ou masques booléen) : Pandas évalue chaque ligne selon un critère pour ne conserver que celles qui remplissent la condition (`True`). Cette approche évite également les boucles itératives inefficaces grâce à la vectorisation.

---

### Filtrage simple avec une condition booléenne

Vous pouvez isoler un sous-ensemble de lignes en testant la valeur d'une colonne par rapport à un seuil ou une chaîne de caractères.

```python
import pandas as pd

# Création d'un DataFrame de référence
df = pd.DataFrame({
    'Nom': ['Alice', 'Bob', 'Charlie', 'David'],
    'Score': [75, 90, 82, 60],
    'Ville': ['Paris', 'Lyon', 'Marseille', 'Paris']
})

# Filtrage des lignes où le score est strictement supérieur à 80
selection = df[df['Score'] > 80]
print(selection)

```

### Filtrage avancé avec conditions multiples

Il est possible de combiner plusieurs critères en utilisant les opérateurs logiques adaptés : `&` (ET), `|` (OU) et `~` (NON). *Attention : chaque condition doit impérativement être entourée de parenthèses.*

```python
# Sélection des étudiants de Paris dont le score est supérieur à 70
filtre_multiple = df[(df['Score'] > 70) & (df['Ville'] == 'Paris')]
print(filtre_multiple)

```

### Ajouter de nouvelles lignes (`.loc`)

Vous pouvez insérer de nouveaux enregistrements à la volée en ciblant un index inexistant via la propriété `.loc`.

```python
# Ajout d'une ligne à un index spécifique (ex: index 4)
df.loc[4] = ['Eve', 95, 'Bordeaux']

print(df)

```

### Supprimer des lignes (`drop`)

Si certains enregistrements ne sont plus nécessaires ou s'avèrent erronés, vous pouvez les supprimer en ciblant leurs index.

```python
# Suppression de lignes par leurs indices (ex: supprimer les lignes d'index 0 et 3)
df_nettoye = df.drop(index=[0, 3])

print(df_nettoye)

```
---

## Travailler sur les labels et index

Contrairement aux tableaux NumPy, un `DataFrame` de Pandas ne se limite pas à des positions numériques entières : il s'appuie sur un **index** qui joue le rôle de clé d'identification unique (analogue à une clé primaire en SQL).

Manipuler les étiquettes et l'index permet d'organiser vos données selon un **identifiant métier** plutôt qu'une simple position (ex: nom d'un produit, identifiant client, date). Cela offre plusieurs avantages majeurs :

* **Alignement automatique** des données lors des calculs ou des fusions (jointures).
* **Recherche et accès sémantique** ultra-rapides et précis.
* **Flexibilité** pour restructurer la table à la volée.

---

### Définir une colonne existante comme index (`set_index`)

Vous pouvez transformer n'importe quelle colonne de votre `DataFrame` en index principal pour identifier vos lignes de manière signifiante.

```python
import pandas as pd

# DataFrame initial avec un index numérique par défaut (0, 1, 2)
df = pd.DataFrame({
    'Code_Emp': ['EMP01', 'EMP02', 'EMP03'],
    'Nom': ['Alice', 'Bob', 'Charlie'],
    'Departement': ['RH', 'IT', 'Finance']
})

# Définition de la colonne 'Code_Emp' comme nouvel index
df_indexe = df.set_index('Code_Emp')
print(df_indexe)

```

### Réinitialiser l'index (`reset_index`)

Si vous avez besoin de replacer l'index actuel dans une colonne standard du tableau (par exemple pour l'exporter vers un fichier CSV classique), la méthode `.reset_index()` est indispensable.

```python
# Retour en arrière : l'index redevient une colonne normale et un index numérique par défaut est recréé
df_reinitialise = df_indexe.reset_index()
print(df_reinitialise)

```

### Renommer des étiquettes d'index (`rename`)

Tout comme pour les colonnes, vous pouvez modifier dynamiquement certaines étiquettes d'index si elles comportent des erreurs ou nécessitent une mise à jour.

```python
# Renommage d'une étiquette d'index spécifique
df_renomme = df_indexe.rename(index={'EMP01': 'EMP_ADMIN_01'})
print(df_renomme)

```

### Sélectionner des données par étiquette avec `.loc`

Une fois l'index configuré, l'attribut `.loc` vous permet d'interroger directement les lignes en utilisant leurs étiquettes textuelles (ce qui est beaucoup plus robuste qu'un numéro de ligne qui pourrait changer).

```python
# Recherche d'une ligne spécifique grâce à son étiquette d'index métier
employe_specifique = df_indexe.loc['EMP02']
print(employe_specifique)

```
---

## Extraction de données : `.loc` vs `.iloc`

L'extraction de sous-ensembles est une opération quotidienne en science des données. Pandas propose deux attributs principaux pour cibler précisément les cellules, lignes ou colonnes d'un `DataFrame` :

* **`.loc` (Location)** : Indexation basée sur les **étiquettes** (les noms des lignes et des colonnes). *Note importante : les bornes de découpage (slicing) sont inclusives.*
* **`.iloc` (Integer Location)** : Indexation basée strictement sur les **positions numériques entières** (de 0 à n-1), indépendamment des étiquettes. *Note importante : les bornes de découpage excluent la fin, comme les listes Python classiques.*

Ces outils permettent d'isoler des valeurs uniques, des tranches (slices) ou des combinaisons complexes de lignes et de colonnes en une seule commande.

---

### Extraction par étiquette avec `.loc`

Si votre `DataFrame` possède des étiquettes claires (par exemple un nom ou un identifiant), `.loc` permet d'accéder directement à une ligne, une colonne ou une cellule précise.

```python
import pandas as pd

# Création d'un DataFrame avec un index textuel
df = pd.DataFrame({
    'Score': [85, 90, 78],
    'Ville': ['Paris', 'Lyon', 'Marseille']
}, index=['Alice', 'Bob', 'Charlie'])

# Extraction d'une cellule spécifique (ligne 'Alice', colonne 'Score')
valeur = df.loc['Alice', 'Score']
print("Score d'Alice :", valeur)

# Extraction d'une ligne entière par son étiquette
ligne_bob = df.loc['Bob']
print("\nToutes les infos de Bob :\n", ligne_bob)

```

### Extraction par position avec `.iloc`

Lorsque vous ne connaissez pas les étiquettes ou que vous souhaitez automatiser un parcours par position numérique (ex: récupérer la première ligne), `.iloc` est l'outil idéal.

```python
# Extraction de la première ligne (index positionnel 0)
premiere_ligne = df.iloc[0]
print("Première ligne (position 0) :\n", premiere_ligne)

# Extraction d'une valeur précise : 2ème ligne (index 1), 1ère colonne (index 0) -> 'Lyon'
valeur_pos = df.iloc[1, 0]
print("\nValeur à la position [1, 0] :", valeur_pos)

```

### Découpage bidimensionnel (Slicing) de tranches

Les deux méthodes permettent de sélectionner simultanément des plages de lignes et de colonnes en utilisant le format `[lignes, colonnes]`.

```python
# Avec .loc : les bornes textuelles sont INCLUSIVES ('Alice' jusqu'à 'Bob' inclus)
tranche_loc = df.loc['Alice':'Bob', 'Score':'Ville']
print("Slicing avec .loc (inclusif) :\n", tranche_loc)

# Avec .iloc : les bornes numériques excluent la fin (de la ligne 0 à 1 exclue, soit seulement la ligne 0)
tranche_iloc = df.iloc[0:2, 0:1]
print("\nSlicing avec .iloc (exclusif en fin) :\n", tranche_iloc)

```

### Extraction conditionnelle combinée avec `.loc`

Il est très fréquent de combiner un filtre booléen sur les lignes avec `.loc` pour modifier ou extraire uniquement certaines colonnes de manière ciblée.

```python
# Sélectionner la colonne 'Ville' uniquement pour les étudiants dont le Score > 80
villes_bons_scores = df.loc[df['Score'] > 80, 'Ville']
print("Villes des étudiants avec un score > 80 :\n", villes_bons_scores)

```

---

## Exemple de synthèse

Gestion et analyse d'un catalogue de produits. Ce script de synthèse met en œuvre un flux complet de manipulation de données avec Pandas, combinant toutes les techniques vues précédemment.

```python
import pandas as pd

# CRÉATION DU DATAFRAME (Depuis un dictionnaire de listes)
data = {
    'Reference': ['PROD_01', 'PROD_02', 'PROD_03', 'PROD_04'],
    'Nom': ['Clavier', 'Souris', 'Écran', 'Casque'],
    'Prix_HT': [45.0, 25.0, 199.99, 80.0],
    'Stock': [120, 85, 14, 0],
    'Categorie': ['Accessoire', 'Accessoire', 'Moniteur', 'Audio']
}
df = pd.DataFrame(data)
print("--- DataFrame Initial ---")
print(df)

# EXPLORATION DES PROPRIÉTÉS STRUCTURALES
print("\n--- Propriétés clés ---")
print("Dimensions (.shape) :", df.shape)
print("Colonnes (.columns) :", df.columns.tolist())

# MANIPULATION DES COLONNES (Vectorisation)
# Ajout d'une colonne calculée pour le prix TTC (20% de TVA) et la valeur totale du stock
df['Prix_TTC'] = df['Prix_HT'] * 1.20
df['Valeur_Stock'] = df['Prix_TTC'] * df['Stock']

# GESTION DE L'INDEX
# Utilisation de la colonne 'Reference' comme index métier unique
df = df.set_index('Reference')
print("\n--- DataFrame avec Index Métier ---")
print(df.head())

# FILTRAGE DES LIGNES (Conditions multiples)
# Sélection des produits en stock (Stock > 0) de la catégorie 'Accessoire'
produits_dispos = df[(df['Stock'] > 0) & (df['Categorie'] == 'Accessoire')]
print("\n--- Accessoires disponibles en stock ---")
print(produits_dispos[['Nom', 'Prix_TTC', 'Stock']])

# EXTRACTION DE DONNÉES CIBLÉES (.loc et .iloc)
# Extraction par étiquette textuelle (.loc)
prix_ecran = df.loc['PROD_03', 'Prix_TTC']
print(f"\nPrix TTC de l'Écran via .loc : {prix_ecran} €")

# Extraction par position numérique (.iloc) pour la première ligne
premier_produit = df.iloc[0]
print("\nPremier produit (position 0) :\n", premier_produit[['Nom', 'Prix_TTC']])

```

* **Création & Structure** : Privilégiez les dictionnaires pour la lisibilité et contrôlez toujours vos dimensions avec `.shape`.
* **Performance** : Effectuez vos calculs sur les colonnes de manière globale (vectorisation) plutôt qu'avec des boucles.
* **Sémantique** : Utilisez `.set_index()` pour remplacer les numéros de lignes par des identifiants métiers significatifs.
* **Requêtes** : Utilisez `.loc` lorsque vous connaissez les étiquettes et `.iloc` lorsque vous travaillez sur des positions numériques.

--- 

## Exercices de fin de chapitre

**Exercice 1** Importez un petit fichier CSV de votre choix et affichez les noms des colonnes à l'aide de l'attribut approprié.

**Exercice 2** Créez un DataFrame à partir d'un dictionnaire, puis utilisez `.iloc` pour extraire uniquement la première colonne de toutes les lignes.


# Chapitre 6 : Opérations sur les données

Apprendre à manipuler et transformer vos données est indispensable pour extraire des informations pertinentes. Ce chapitre vous permet de maîtriser le tri, l'itération et les fonctions avancées de transformation pour nettoyer et enrichir vos jeux de données.

Dans ce chapitre :

* Opération de Tri
* Itération sur les lignes et modification
* Utilisation des fonctions `apply` pour les transformations
* Utilisation des `pipe` pour les transformations
* Recherches et critères de filtrage

## Opération de Tri

Le tri permet de réorganiser vos données selon les valeurs d'une ou plusieurs colonnes pour mieux visualiser les extrêmes ou les tendances. Vous pouvez trier par ordre croissant ou décroissant très simplement.

Tri simple par ordre décroissant (Trouver les meilleures ventes).Pour identifier immédiatement les produits qui génèrent le plus de chiffre d'affaires, on trie la colonne `Total` du plus grand au plus petit (`ascending=False`).

```python

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2],
    'Prix_TTC': [60.0, 30.0, 96.0],
    'Total': [180.0, 300.0, 192.0]
})


# Trier par chiffre d'affaires décroissant
df_tri_desc = df.sort_values(by='Total', ascending=False)
print(df_tri_desc)

```

Tri simple par ordre croissant (Trouver les prix les plus bas).Pour classer les articles du moins cher au plus cher, on utilise le tri croissant par défaut (`ascending=True`).

```python
# Trier par prix unitaire HT croissant
df_tri_asc = df.sort_values(by='Prix_HT', ascending=True)
print(df_tri_asc)

```

Tri multicritères (Hiérarchiser selon plusieurs colonnes). Il est possible de trier selon plusieurs critères en passant une liste à `by` et à `ascending`. Par exemple, trier d'abord par `Quantite` décroissante, puis par `Prix_HT` croissant en cas d'égalité des quantités.

```python
# Tri sur deux colonnes avec des directions différentes
df_multi_tri = df.sort_values(
    by=['Quantite', 'Prix_HT'], 
    ascending=[False, True]
)
print(df_multi_tri)

```

## Itération sur les lignes et modification

Bien que l'itération manuelle soit moins performante que les méthodes vectorisées, elle permet de modifier des lignes complexes individuellement. On utilise généralement `iterrows()` pour parcourir les lignes une par une.

```python
# Itération sur les lignes pour modifier une valeur at pour la celle
for index, row in df.iterrows():
    df.at[index, 'Score'] += 1

```

Pour la lecture ou l'écriture d'une cellule isolée, df.at contourne la surcharge de sélection de Pandas et offre de meilleures performances en termes de temps d'exécution (très utile dans des boucles ou des scripts d'optimisation).

## Utilisation des fonctions `apply`

La méthode `apply` permet d'appliquer une fonction personnalisée à chaque ligne ou chaque colonne d'un DataFrame. C'est un outil très puissant pour réaliser des transformations complexes non supportées nativement par Pandas.

Appliquer une transformation sur une colonne (Série). Pour transformer ou formater les valeurs d'une colonne spécifique (par exemple, mettre tous les noms de produits en majuscules) :

```python
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2],
    'Prix_TTC': [60.0, 30.0, 96.0],
    'Total': [180.0, 300.0, 192.0]
})

# Appliquer une fonction lambda pour mettre en majuscules la colonne 'Produit'
df['Produit_Maj'] = df['Produit'].apply(lambda x: x.upper())

print(df[['Produit', 'Produit_Maj']])

```

---

Appliquer une fonction ligne par ligne (`axis=1`). Lorsque l'on a besoin de croiser plusieurs colonnes d'une **même ligne**, on utilise `axis=1`. Par exemple, pour générer une phrase de synthèse descriptive pour chaque produit :

```python
# Combiner les valeurs de plusieurs colonnes par ligne
df['Resume'] = df.apply(
    lambda row: f"Le {row['Produit']} (qté: {row['Quantite']}) génère un total de {row['Total']}€", 
    axis=1
)

print(df['Resume'])

```

## Utilisation des `pipe`

La méthode `pipe` permet d'enchaîner plusieurs fonctions de transformation de manière lisible et structurée. Cette approche facilite la maintenance du code en isolant chaque étape de traitement dans des fonctions séparées.

Lorsque les transformations se succèdent, `pipe` permet de lier des fonctions personnalisées de manière séquentielle et lisible :

```python
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2],
    'Total': [180.0, 300.0, 192.0]
})

# Première fonction : appliquer une remise promotionnelle
def appliquer_remise(data, taux):
    data['Prix_Promo'] = data['Prix_HT'] * (1 - taux)
    return data

# Seconde fonction : filtrer les produits au-delà d'un seuil de chiffre d'affaires
def filtrer_par_seuil(data, seuil):
    return data[data['Total'] >= seuil].copy()

# Enchaînement propre et lisible avec pipe
df_traite = (
    df.pipe(appliquer_remise, taux=0.10)
      .pipe(filtrer_par_seuil, seuil=190.0)
)

print(df_traite[['Produit', 'Prix_HT', 'Prix_Promo', 'Total']])

```

La méthode .copy() permet de créer une copie indépendante d'un DataFrame (ou d'une Série) en mémoire.

## Recherches et critères de filtrage

Le filtrage consiste à extraire uniquement les lignes qui répondent à une ou plusieurs conditions logiques précises. Vous pouvez combiner plusieurs critères avec des opérateurs booléens pour un ciblage très granulaire.


### Exemple 1 : 

Isoler un catalogue avant une modification de prix et créer une grille tarifaire promotionnelle à partir du catalogue principal sans modifier les prix de base d'origine :

```python
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2],
    'Total': [180.0, 300.0, 192.0]
})

# Création d'une copie indépendante pour les soldes
df_soldes = df.copy()

# Modification sur la copie uniquement (baisse de 10% sur le Prix_HT)
df_soldes['Prix_HT'] = df_soldes['Prix_HT'] * 0.9

print("--- Original inchangé ---")
print(df[['Produit', 'Prix_HT']])

print("\n--- Copie modifiée (Soldes) ---")
print(df_soldes[['Produit', 'Prix_HT']])

```

---

Filtrer et modifier un sous-ensemble (Éviter le `SettingWithCopyWarning`). Lorsque tu isoles des lignes spécifiques (par exemple, les produits dont la quantité vendue est supérieure à 2), l'utilisation conjointe du filtrage et de `.copy()` garantit que tu peux ajouter une nouvelle colonne sans déclencher d'avertissement ni corrompre la mémoire :

```python
# Extraction sécurisée des produits à forte rotation
df_populaires = df[df['Quantite'] > 2].copy()

# Ajout d'une colonne de statut sans risque
df_populaires['Alerte_Stock'] = 'Réassort prioritaire'

print(df_populaires)

```

---

Conserver un historique de référence dans un pipeline de transformation. Dans un enchaînement de traitements, il est fréquent de dupliquer un DataFrame à une étape clé pour pouvoir comparer l'état initial et l'état final :

```python
# Sauvegarde de l'état initial (Baseline)
df_reference = df.copy()

# Traitement sur le DataFrame principal
df['Chiffre_Affaires_Estime'] = df['Prix_HT'] * df['Quantite']

# Comparaison directe entre la référence et le résultat traité
comparaison = pd.DataFrame({
    'Produit': df['Produit'],
    'Total_Ref': df_reference['Total'],
    'Total_Calcule': df['Chiffre_Affaires_Estime']
})

print(comparaison)

```
## Exemple de synthèse

En analyse de données, la synthèse consiste à agréger les indicateurs clés (totaux, moyennes, extrêmes) pour obtenir une vue macroscopique en une seule instruction.

Générer un rapport de synthèse global

```python
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2],
    'Total': [180.0, 300.0, 192.0]
})

# Utilisation de la méthode .agg() pour combiner plusieurs fonctions d'agrégation
synthese_chiffres = df.agg({
    'Prix_HT': ['mean', 'min', 'max'],
    'Quantite': 'sum',
    'Total': 'sum'
})

print("--- Indicateurs agrégés ---")
print(synthese_chiffres)

# Extraction d'un résumé textuel des performances commerciales
chiffre_affaires_global = df['Total'].sum()
quantite_totale = df['Quantite'].sum()
produit_phare = df.loc[df['Total'].idxmax(), 'Produit']

print("\n--- SYNTHÈSE COMMERCIALE ---")
print(f"Chiffre d'affaires total : {chiffre_affaires_global} €")
print(f"Volume total d'articles vendus : {quantite_totale} unités")
print(f"Produit le plus performant (en CA) : {produit_phare}")

```

* **`.agg()`** permet d'appliquer des fonctions statistiques différentes par colonne en une seule passe (par exemple, calculer la moyenne et les extrêmes sur les prix, mais la somme sur les quantités et les totaux).
* **`idxmax()`** permet de récupérer l'index de la ligne ayant la valeur maximale dans une colonne donnée, ce qui est idéal pour identifier dynamiquement le meilleur produit sans avoir à trier tout le DataFrame.

## Exercices de fin de chapitre


** Exercice 1** : Diagnostic et Audit d'un Dataset de Ventes

Tu récupères un fichier CSV contenant des données de ventes brutes d'une entreprise e-commerce (`ventes_techno.csv`). Avant toute analyse marketing, tu dois mener un audit rigoureux de la structure et de la santé des données.

```python
import pandas as pd
import io

csv_data = """id_vente,produit,categorie,prix_unitaire,quantite,date_vente
101,Clavier RGB,Accessoire,45.5,4,2026-05-10
102,Écran 27,Moniteur,299.0,2,2026-05-11
103,Souris Sans Fil,Accessoire,25.0,NaN,2026-05-11
104,PC Portable Pro,Ordinateur,1200.0,1,2026-05-12
105,Casque Audio,Accessoire,60.0,3,2026-05-13
"""
df_ventes = pd.read_csv(io.StringIO(csv_data))

```

* Importation et typage : Charge ce jeu de données en veillant à convertir la colonne `date_vente` directement en objets temporels (`datetime`) et à spécifier les colonnes catégorielles (`produit`, `categorie`).
* Inspection structurelle : Affiche les dimensions du DataFrame, la liste des axes (`.axes` ou `.columns`/`.index`), ainsi que le diagnostic complet de l'empreinte mémoire avec `info(memory_usage='deep')`.
* Audit des anomalies : Identifie le nombre de valeurs manquantes par colonne.
* Statistiques descriptives : Affiche un résumé statistique global (incluant les variables textuelles/catégorielles) pour évaluer les fourchettes de prix et les volumes.

---

**Exercice 2 :** Pipeline de Transformation et Synthèse Commerciale

À partir du catalogue de produits consolidé, tu dois appliquer une série de transformations sécurisées, calculer des métriques personnalisées ligne par ligne, et extraire un rapport de synthèse décisionnel.


```python
import pandas as pd

df_catalogue = pd.DataFrame({
    'Reference': ['SKU-01', 'SKU-02', 'SKU-03', 'SKU-04'],
    'Produit': ['Clavier', 'Souris', 'Casque', 'Ecran'],
    'Prix_HT': [50.0, 25.0, 80.0, 300.0],
    'Stock': [15, 42, 8, 5]
})

```

* Isolation sécurisée (`.copy()`) : Crée une copie indépendante nommée `df_promo` pour préparer une opération promotionnelle sans risquer de corrompre le catalogue d'origine.
* Transformation par Pipeline (`.pipe()`) : Écris une fonction `appliquer_taxe(df, tva)` qui ajoute une colonne `Prix_TTC` (calculée avec une TVA de 20%), puis enchaîne-la via `.pipe()`.
* Traitement ligne par ligne (`.apply(axis=1)`) : À l'aide de `.apply()` et `axis=1`, génère une colonne `Valeur_Stock_HT` représentant la valorisation financière totale du stock pour chaque produit (`Prix_HT * Stock`).
* Synthèse globale (`.agg()`) : Utilise `.agg()` pour obtenir simultanément le prix moyen, minimum et maximum des produits HT, ainsi que la somme totale de la valeur du stock de l'entreprise.


# Chapitre 7 : Filtrage et Mappage dans les DataFrames Pandas

En programmation Python classique, on utilise souvent les fonctions intégrées **`filter()`** et **`map()`** sur des listes ou des itérables. Sous Pandas, ces concepts existent mais ont été entièrement repensés pour exploiter la vectorisation et offrir des performances de haut niveau sur des millions de lignes.

| Opération | Fonction / Outil Pandas | Cas d'usage principal |
| --- | --- | --- |
| **Filtrer des lignes** | `df[condition]` ou `df.query()` | Extraire un sous-ensemble de données selon des critères chiffrés ou textuels. |
| **Filtrer des colonnes** | `df.filter(like=...)` | Sélectionner rapidement des colonnes par motif ou nom. |
| **Mapper des valeurs** | `serie.map(dictionnaire)` | Remplacer des codes par des libellés (tables de correspondance). |
| **Appliquer une fonction** | `serie.apply()` / `df.map()` | Exécuter une logique de calcul personnalisée sur les données. |

---

## Le Filtrage de données : Bien plus qu'un simple `filter()`

En Pandas, il existe deux manières distinctes de "filtrer" : le **filtrage conditionnel de lignes** (le plus fréquent) et le **filtrage de colonnes/index par nom**.

## Le filtrage conditionnel de lignes (Indexation booléenne)

Pour extraire des lignes selon un critère métier (ex: *tous les produits dont le prix dépasse 50€*), on n'utilise pas la fonction `filter()`, mais l'**indexation booléenne**, qui crée un masque de `True`/`False`.

```python
import pandas as pd

# Données d'exemple (notre catalogue)
df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0],
    'Stock': [10, 40, 5, 0]
})

# Filtrer les produits dont le prix est supérieur à 50
produits_chers = df[df['Prix_HT'] > 50]
print("--- Produits chers (> 50) ---")
print(produits_chers)

# Combiner plusieurs conditions avec les opérateurs logiques (& pour ET, | ou OU)
produits_disponibles_chers = df[(df['Prix_HT'] > 50) & (df['Stock'] > 0)]

```

*Alternative élégante :* La méthode **`.query()`** permet d'écrire les conditions sous forme de chaîne de caractères textuelle, très lisible pour des requêtes complexes :

```python
resultat = df.query("Prix_HT > 50 and Stock > 0")

```

## La méthode `.filter()` : Sélectionner des colonnes ou des index par nom

À l'inverse, la méthode `.filter()` de Pandas sert à **filtrer les étiquettes** (les noms des colonnes ou de l'index), et non les valeurs des lignes.

```python
# Ne conserver que les colonnes dont le nom contient "Prix"
df_filtre_cols = df.filter(like='Prix')

# Sélectionner des colonnes spécifiques par leur nom exact
df_selection = df.filter(items=['Produit', 'Stock'])

```

---

## Le Mappage (`.map()`) : Transformer et traduire des valeurs

La méthode **`.map()`** s'applique à une **Série** (une seule colonne). Elle est idéale pour remplacer des valeurs, traduire des codes numériques en libellés textuels, ou appliquer un dictionnaire de correspondance.

## Traduire des codes catégories

Imaginons une colonne contenant des identifiants numériques de catégories que l'on souhaite transformer en texte lisible :

```python
import pandas as pd

df_ventes = pd.DataFrame({
    'ID_Produit': [1, 2, 3],
    'Code_Cat': [10, 20, 10]
})

# Dictionnaire de correspondance (Mapping)
correspondance_cat = {
    10: 'Accessoire Informatique',
    20: 'Matériel Audio'
}

# Appliquer le map via un dictionnaire
df_ventes['Nom_Cat'] = df_ventes['Code_Cat'].map(correspondance_cat)

print(df_ventes)

```

> **Attention aux valeurs absentes :** Si `.map()` rencontre une valeur qui n'existe pas dans le dictionnaire de correspondance, elle la remplace par `NaN` (valeur manquante). Pour éviter cela, on peut passer une fonction ou utiliser `.replace()`.

---

## Aller plus loin : `.apply()` et `.applymap()` (ou `.map()` global)

Quand le mapping par dictionnaire ne suffit plus et qu'on souhaite appliquer une fonction personnalisée (ou une fonction lambda) sur tout un DataFrame ou une colonne :

* **Sur une Série (`colonne.apply()`)** : Applique une fonction ligne par ligne sur une colonne.
* **Sur tout le DataFrame (`df.map()**` - anciennement `applymap` dans les anciennes versions de Pandas) : Applique une fonction cellule par cellule sur l'ensemble de la table.

```python
# Appliquer une fonction lambda pour appliquer une remise de 10% sur tous les prix
df['Prix_Remise'] = df['Prix_HT'].apply(lambda x: x * 0.9)

print(df[['Produit', 'Prix_HT', 'Prix_Remise']])

```

---
## Exemple de synthèse

Le nettoyage et l'enrichissement du catalogue e-commerce. Imaginons que nous récupérons un fichier de données brut comportant des codes de catégories chiffrés et des prix bruts. Notre objectif est de :

**Filtrer** les articles disponibles et dont le prix dépasse un certain seuil.
**Traduire** les codes de catégories en libellés explicites grâce à un dictionnaire de mapping.
**Appliquer** une fonction de calcul personnalisée pour simuler un nouveau tarif promotionnel.

```python
import pandas as pd

# Données brutes issues de l'importation
df_brut = pd.DataFrame({
    'Reference': ['SKU-01', 'SKU-02', 'SKU-03', 'SKU-04', 'SKU-05'],
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam', 'Tapis'],
    'Code_Cat': [10, 20, 30, 20, 10],
    'Prix_Brut': [60.0, 25.0, 120.0, 85.0, 15.0],
    'En_Stock': [True, False, True, True, True]
})

print("--- Données brutes d'origine ---")
print(df_brut)

# ==========================================
# APPLICATION DU FILTRAGE ET DU MAPPING
# ==========================================

# Filtrer uniquement les produits en stock ET dont le prix dépasse 20€
# (Utilisation d'un masque booléen combiné)
df_filtre = df_brut[(df_brut['En_Stock'] == True) & (df_brut['Prix_Brut'] > 20)].copy()

# Mappage des codes de catégories vers des libellés métier explicites
dictionnaire_categories = {
    10: 'Accessoires',
    20: 'Périphériques',
    30: 'Audio & Son'
}
df_filtre['Categorie'] = df_filtre['Code_Cat'].map(dictionnaire_categories)

# Appliquer une fonction personnalisée (.apply) : Remise de 15% sur les prix
df_filtre['Prix_Promo'] = df_filtre['Prix_Brut'].apply(lambda x: round(x * 0.85, 2))

# ==========================================
# SÉLECTION FINALE (Propreté du rapport)
# ==========================================
# On isole uniquement les colonnes pertinentes pour l'analyse
df_synthese = df_filtre[['Reference', 'Produit', 'Categorie', 'Prix_Brut', 'Prix_Promo']]

print("\n--- Tableau de synthèse final (Filtré, Mappé et Calculé) ---")
print(df_synthese)

```

---

### Les clés de la réussite à rappeler à votre lecteur :

**Le filtrage se fait sur les lignes** via des conditions booléennes (`df[...]` ou `.query()`), tandis que `.filter()` est réservé aux étiquettes de colonnes ou d'index.

**Le `.map()` est l'outil roi** pour enrichir une colonne à partir d'un dictionnaire de correspondance, à condition de bien anticiper la gestion des valeurs inconnues (`NaN`).

**Le `.apply()` offre une flexibilité totale** pour injecter du code Python personnalisé (comme des fonctions `lambda`) lorsque les opérations vectorisées de base ne suffisent plus.

## Exercices de fin de chapitre

**Exercice 1:** Analyse temporelle et extraction des pics d'activité
Manipuler les séries temporelles, convertir des chaînes de caractères en objets `datetime`, extraire des composantes temporelles et réaliser une agrégation croisée.

Vous disposez d'un journal de transactions brutes sous forme de texte. La direction souhaite identifier quels jours de la semaine génèrent le plus de chiffre d'affaires.

Créez un DataFrame contenant ces données brutes :
```python
import pandas as pd

df_logs = pd.DataFrame({
    'Horodatage': ['2026-10-01 09:30:00', '2026-10-02 14:15:00', '2026-10-03 18:45:00', 
                   '2026-10-04 10:00:00', '2026-10-05 16:20:00', '2026-10-06 11:10:00'],
    'Montant_EUR': [120.5, 45.0, 300.0, 85.0, 210.0, 95.0]
})

```
Convertissez la colonne `Horodatage` en un type `datetime` exploitable.
Extrayez le **nom du jour de la semaine** (ex: *Monday*, *Tuesday*...) dans une nouvelle colonne nommée `Jour_Semaine`.
Réalisez un regroupement (`groupby`) par `Jour_Semaine` pour calculer la somme totale du chiffre d'affaires (`Montant_EUR`) et le nombre de transactions par jour.


**Exercice 2:**  Enrichissement relationnel, Mapping et Filtrage
Croiser deux tables hétérogènes avec `pd.merge()`, traduire des codes via `.map()` et appliquer un filtre conditionnel multicritère.

Vous gérez deux sources de données distinctes dans votre boutique : un historique de ventes contenant des identifiants produits, et un catalogue général fournissant les détails et les catégories.

Partez des deux DataFrames suivants :
```python
# Table des ventes
df_ventes = pd.DataFrame({
    'ID_Produit': [101, 102, 101, 103, 102],
    'Quantite': [2, 1, 5, 3, 2]
})

# Table du catalogue produits
df_catalogue = pd.DataFrame({
    'ID_Produit': [101, 102, 103, 104],
    'Nom': ['Clavier', 'Souris', 'Casque', 'Webcam'],
    'Code_Cat': [1, 2, 3, 2],
    'Prix_Unitaire': [50.0, 25.0, 80.0, 70.0]
})

```


Réalisez une jointure (`pd.merge()`) de type **`left`** en prenant `df_ventes` comme table principale pour y intégrer les informations du catalogue.
À l'aide d'un dictionnaire de correspondance, créez une colonne `Categorie` en mappant les `Code_Cat` selon la règle suivante :
* `1` : *Bureautique*
* `2` : *Accessoire*
* `3` : *Audio*

Calculez le montant total de chaque ligne (`Quantite * Prix_Unitaire`) dans une nouvelle colonne `CA_Ligne`.
Filtrez enfin le DataFrame final pour n'afficher que les lignes dont le `CA_Ligne` dépasse 100 €.

# Chapitre 8: Nettoyage et Préparation des Données

Le nettoyage des données (*Data Cleaning*) est souvent l'étape la plus chronophage et la plus cruciale de tout projet en science des données. Un modèle d'apprentissage ou une analyse statistique ne vaut que par la qualité des données qui l'alimentent. Les fichiers bruts contiennent fréquemment des valeurs manquantes, des doublons, des fautes de frappe textuelles ou des valeurs aberrantes.

Ce chapitre vous présente les outils et méthodes indispensables sous Pandas pour transformer un jeu de données brut et chaotique en un tableau propre, structuré et prêt pour l'analyse.

Dans ce chapitre :

* Détecter les données invalides (`NaN`, `isna()`, `notna()`)
* Traiter les valeurs manquantes : `fillna()`, `dropna()`, `interpolate()`
* Détecter et supprimer les doublons : `duplicated()`, `drop_duplicates()`
* Repérer les valeurs aberrantes
* Nettoyer les chaînes de caractères avec l'accesseur `.str`
* Le type `category` : gain de mémoire et de performance
* Exemple de synthèse
* Exercices de fin de chapitre

---

## Détecter les données invalides (`NaN`, `isna()`, `notna()`)

Avant de corriger des anomalies, il faut savoir les localiser. Dans Pandas, les valeurs manquantes ou non définies sont représentées par l'indicateur universel **`NaN`** (Not a Number).

### Détection unitaire avec `isna()` et `notna()` sur une Série

Les méthodes `.isna()` (ou son alias `.isnull()`) et `.notna()` renvoient un masque booléen indiquant respectivement la présence ou l'absence de données valides.

```python
import pandas as pd
import numpy as np

# Série contenant des valeurs valides et des NaN
prix = pd.Series([50.0, np.nan, 80.0, 25.0, np.nan])

# Détecter où se trouvent les valeurs manquantes
print("Masque des valeurs manquantes (isna) :")
print(prix.isna())

# Identifier les valeurs valides
print("\nMasque des valeurs valides (notna) :")
print(prix.notna())

```

### Audit global des valeurs manquantes dans un DataFrame

Pour évaluer la santé d'un grand tableau, on combine souvent `.isna()` avec `.sum()` afin d'obtenir le nombre exact d'anomalies par colonne.

```python
import pandas as pd
import numpy as np

# DataFrame de test e-commerce
df_catalogue = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', np.nan],
    'Prix_HT': [50.0, 25.0, np.nan, 70.0],
    'Stock': [10, np.nan, 5, 0]
})

# Compter le nombre de NaN par colonne
anomalies_par_colonne = df_catalogue.isna().sum()
print("Nombre de valeurs manquantes par colonne :\n", anomalies_par_colonne)

```

---

## Traiter les valeurs manquantes : `fillna()`, `dropna()`, `interpolate()`

Une fois les anomalies détectées, trois grandes stratégies s'offrent à vous selon le contexte métier : **supprimer** les lignes/colonnes corrompues, les **remplacer** par une valeur par défaut ou une statistique, ou les **estimer** par interpolation.

### Supprimer (`dropna`) ou Remplacer (`fillna`)

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam'],
    'Prix': [50.0, np.nan, 80.0, 70.0],
    'Stock': [10, 5, np.nan, 20]
})

# Supprimer toute ligne contenant au moins un NaN
df_sans_nan = df.dropna()
print("--- Après dropna() ---")
print(df_sans_nan)

# Remplacer les NaN par une valeur fixe ou par la moyenne de la colonne
df_rempli_fixe = df.fillna({'Prix': df['Prix'].mean(), 'Stock': 0})
print("\n--- Après fillna() avec des valeurs de substitution ---")
print(df_rempli_fixe)

```

### L'interpolation pour les séries temporelles (`interpolate`)

Sur des séries chronologiques, remplacer par une moyenne globale brise la tendance temporelle. L'interpolation linéaire permet d'estimer la valeur manquante en fonction de ses voisines.

```python
import pandas as pd
import numpy as np

# Suivi de température journalier avec un trou (NaN)
dates = pd.date_range(start='2026-10-01', periods=5, freq='D')
temperatures = pd.Series([20.0, 22.0, np.nan, 24.0, 25.0], index=dates)

# Interpolation linéaire pour combler le trou de manière fluide
temperatures_propres = temperatures.interpolate()

print(temperatures_propres)

```

---

## Détecter et supprimer les doublons : `duplicated()`, `drop_duplicates()`

Les doublons faussent les analyses statistiques et les agrégations. Pandas permet de les identifier facilement et de les éliminer.

### Repérer les lignes dupliquées

```python
import pandas as pd

df_clients = pd.DataFrame({
    'ID_Client': [101, 102, 101, 103, 102],
    'Nom': ['Alice', 'Bob', 'Alice', 'Charlie', 'Bob'],
    'Ville': ['Paris', 'Lyon', 'Paris', 'Marseille', 'Lyon']
})

# Détecter les lignes strictement identiques aux précédentes
masque_doublons = df_clients.duplicated()
print("Lignes dupliquées (booléens) :\n", masque_doublons)

# Afficher uniquement les lignes concernées
print("\nLignes en double :\n", df_clients[masque_doublons])

```

### Nettoyer avec `drop_duplicates()`

On peut purger les doublons en conservant soit la première occurrence (`keep='first'`), soit la dernière (`keep='last'`), ou en ciblant une colonne clé (`subset`).

```python
# Supprimer les doublons stricts en gardant la première occurrence
df_nettoye = df_clients.drop_duplicates(keep='first')
print("Après suppression des doublons stricts :\n", df_nettoye)

# Supprimer en se basant uniquement sur l'ID_Client
df_clients_uniques = df_clients.drop_duplicates(subset=['ID_Client'])
print("\nAprès nettoyage basé sur l'ID_Client :\n", df_clients_uniques)

```

---

## Repérer les valeurs aberrantes (Outliers)

Une valeur aberrante est une donnée qui s'écarte anormalement de la tendance générale (par exemple, un prix négatif ou une quantité de 10 000 articles vendus par erreur). Elles se détectent souvent via des seuils logiques ou des indicateurs statistiques (comme les quantiles).

### Filtrage par seuil logique métier

```python
import pandas as pd

df_ventes = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Erreur_Prix'],
    'Prix_HT': [50.0, 25.0, 80.0, -999.0] # -999 est une valeur aberrante évidente
})

# Filtrer pour ne garder que les prix logiques (positifs et réalistes)
df_valide = df_ventes[(df_ventes['Prix_HT'] > 0) & (df_ventes['Prix_HT'] < 5000)]
print(df_valide)

```

### Utilisation des quantiles pour éliminer les valeurs extrêmes

```python
import pandas as pd
import numpy as np

np.random.seed(42)
# Génération d'un jeu de données avec une valeur aberrante extrême
salaires = pd.Series([3000, 3200, 3100, 2900, 3050, 150000]) 

# Calcul des bornes interquartiles (ex: exclure les 5% les plus hauts et les 5% les plus bas)
seuil_bas = salaires.quantile(0.05)
seuil_haut = salaires.quantile(0.95)

# Filtrer les outliers
salaires_filtres = salaires[(salaires >= seuil_bas) & (salaires <= seuil_haut)]
print("Salaires nettoyés des valeurs extrêmes :\n", salaires_filtres)

```

---

## Nettoyer les chaînes de caractères avec l'accesseur `.str`

Les données textuelles issues du monde réel sont souvent hétérogènes (espaces superflus, majuscules/minuscules mélangées, fautes de saisie). L'accesseur **`.str`** de Pandas permet d'appliquer des méthodes de traitement de texte de manière vectorisée sur toute une colonne.

### Uniformisation de texte (`strip`, `lower`)

```python
import pandas as pd

df_utilisateurs = pd.DataFrame({
    'Email': ['  ALICE@gmail.com ', 'Bob@Gmail.COM', ' charlie@yahoo.fr'],
    'Ville': ['Paris ', 'LYON', 'marseille']
})

# Nettoyer les espaces superflus et uniformiser en minuscules
df_utilisateurs['Email_Clean'] = df_utilisateurs['Email'].str.strip().str.lower()
df_utilisateurs['Ville_Clean'] = df_utilisateurs['Ville'].str.strip().str.capitalize()

print(df_utilisateurs[['Email_Clean', 'Ville_Clean']])

```

### Remplacement de motifs textuels (`str.replace`)

```python
# Données contenant des formats de prix hétérogènes (avec le symbole €)
df_factures = pd.DataFrame({
    'Montant': ['150 €', '200 €', '1250 €']
})

# Supprimer le symbole euro et convertir la colonne en type numérique
df_factures['Montant_Numerique'] = (
    df_factures['Montant']
    .str.replace('€', '', regex=False)
    .str.strip()
    .astype(int)
)

print(df_factures)
print("Type de la nouvelle colonne :", df_factures['Montant_Numerique'].dtype)

```

---

## Le type `category` : gain de mémoire et de performance

Par défaut, les colonnes textuelles répétitives (comme le nom d'un pays, un code postal ou une catégorie de produit) sont stockées en mémoire sous forme d'objets chaînes de caractères (`object`), ce qui est très gourmand en RAM.

Le type de données **`category`** stocke en interne des entiers légers (des pointeurs) tout en affichant les libellés textuels correspondants.

### Conversion d'une colonne en type catégoriel

```python
import pandas as pd

df_produits = pd.DataFrame({
    'Nom': ['Clavier', 'Souris', 'Casque', 'Webcam', 'Tapis'],
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio', 'Accessoire']
})

print("Type initial de la colonne Categorie :", df_produits['Categorie'].dtype)

# Conversion explicite en catégorie
df_produits['Categorie'] = df_produits['Categorie'].astype('category')

print("Type après conversion :", df_produits['Categorie'].dtype)

```

### Mesure de l'optimisation mémoire

```python
import pandas as pd
import numpy as np

# Création d'un grand DataFrame avec une colonne textuelle très répétitive
taille = 100000
df_grand = pd.DataFrame({
    'Region': np.random.choice(['Île-de-France', 'Auvergne-Rhône-Alpes', 'PACA', 'Grand Est'], size=taille),
    'Valeur': np.random.randn(taille)
})

# Audit de la mémoire avant conversion
print("--- Empreinte mémoire initiale (object) ---")
df_grand.info(memory_usage='deep')

# Conversion de la colonne textuelle en catégorie
df_grand['Region'] = df_grand['Region'].astype('category')

# Audit de la mémoire après conversion
print("\n--- Empreinte mémoire optimisée (category) ---")
df_grand.info(memory_usage='deep')

```

---

## Exemple de synthèse :

Ce script réunit l'ensemble des bonnes pratiques abordées dans ce chapitre pour transformer un jeu de données brut e-commerce en un tableau propre et optimisé.

```python
import pandas as pd
import numpy as np

# Données brutes et chaotiques
df_brut = pd.DataFrame({
    'Reference': [' sku-01 ', 'SKU-02', 'sku-01', 'SKU-03', 'sku-04'],
    'Produit': ['Clavier', 'Souris', 'Clavier', 'Casque', 'Webcam'],
    'Categorie': [' Accessoire ', 'Accessoire', 'Accessoire', 'Audio', 'Audio'],
    'Prix_HT': [50.0, 25.0, 50.0, -10.0, np.nan], # Contient un doublon, une valeur aberrante et un NaN
    'Stock': [10, 15, 10, np.nan, 25]
})

print("--- Données brutes d'origine ---")
print(df_brut)

# ==========================================
# PIPELINE DE NETTOYAGE ÉTAPE PAR ÉTAPE
# ==========================================

# A. Supprimer les doublons stricts basés sur les références
df_propre = df_brut.drop_duplicates(subset=['Reference']).copy()

# B. Nettoyer les chaînes de caractères (minuscules et suppression des espaces)
df_propre['Reference'] = df_propre['Reference'].str.strip().str.upper()
df_propre['Categorie'] = df_propre['Categorie'].str.strip().str.capitalize()

# C. Traiter les valeurs aberrantes et les valeurs manquantes
# Remplacement des prix aberrants (< 0) et des NaN par la médiane de la colonne
mediane_prix = df_propre.loc[df_propre['Prix_HT'] > 0, 'Prix_HT'].median()
df_propre['Prix_HT'] = df_propre['Prix_HT'].mask(df_propre['Prix_HT'] <= 0, np.nan)
df_propre['Prix_HT'] = df_propre['Prix_HT'].fillna(mediane_prix)

# Remplacer les stocks manquants par 0
df_propre['Stock'] = df_propre['Stock'].fillna(0)

# D. Optimiser la mémoire en convertissant la catégorie en type 'category'
df_propre['Categorie'] = df_propre['Categorie'].astype('category')

print("\n--- Données nettoyées et optimisées ---")
print(df_propre)
print("\nTypes des colonnes :")
print(df_propre.dtypes)

```

---

## Exercices de fin de chapitre

**Exercice 1 :** Détection et correction des anomalies de texte et de valeurs manquantes

Vous disposez du DataFrame suivant simulant des inscrits à une newsletter :

```python
import pandas as pd
import numpy as np

df_inscrits = pd.DataFrame({
    'Nom': [' Alice ', 'BOB', 'charlie', ' Alice ', np.nan],
    'Age': [25, np.nan, 30, 25, 45],
    'Score': [85, 90, np.nan, 85, 120] # 120 est une valeur aberrante (score max = 100)
})

```

Supprimez les lignes strictement dupliquées.
Nettoyez la colonne `Nom` pour supprimer les espaces superflus et homogénéiser la casse (première lettre en majuscule).
Remplacez les âges manquants par la médiane des âges, et corrigez le score aberrant (> 100) en y insérant une valeur manquante (`np.nan`) avant de combler les scores par la moyenne.

**Exercice 2 :** Optimisation mémoire et traitement de chaînes

Créez un DataFrame contenant 10 000 lignes avec deux colonnes :

* `Departement` : contenant aléatoirement les valeurs `'75'`, `'69'`, `'13'` et `'31'`.
* `Montant` : des chaînes de caractères représentant des prix textuels (ex: `"150.50 €"`).

Nettoyez la colonne `Montant` pour la convertir en un type numérique (`float64`).
Convertissez la colonne `Departement` en type `category`.
Vérifiez l'empreinte mémoire du DataFrame optimisé à l'aide de `.info()`.


# Chapitre 9 : Gestion des dates et séries temporelles

La gestion des séries temporelles est un pilier fondamental en analyse de données. Pour manipuler des historiques de ventes, des logs ou des relevés, la fonction clé de Pandas est **`pd.date_range()`**. Elle permet de générer des séquences de dates régulières en spécifiant un point de départ, une fin (ou un nombre de périodes), et une fréquence (`freq`).

Dans ce chapitre :

* Utilisation des modèles de temps (`Timestamp`, `Datetime`, `Timedelta`, `Period`)
* Manipulation de plages temporelles avec `period_range`

## Les 4 modèles fondamentaux du temps dans Pandas

### `Timestamp` : Un instant précis (Le point dans le temps)

Un `Timestamp` représente une date et une heure exactes (l'équivalent optimisé du `datetime` de Python). C'est l'élément de base qui compose les index temporels (`DatetimeIndex`).

```python
import pandas as pd

# Créer un instant précis
instant = pd.Timestamp('2026-10-28 14:30:00')

print(instant)
print(type(instant))
# Extraction d'attributs clés
print(f"Année : {instant.year}, Mois : {instant.month}, Jour de la semaine : {instant.day_name()}")

```

---

### `Datetime` (et `to_datetime`) : La conversion et l'indexation

En pratique, les dates proviennent souvent de fichiers CSV sous forme de texte (chaînes de caractères). La fonction `pd.to_datetime()` permet de les convertir proprement en objets temporels exploitables.

```python
# Série de dates brutes (texte)
dates_texte = ['2026-10-01', '2026-10-02', '2026-10-03']

# Conversion en objets Datetime de Pandas
dates_dt = pd.to_datetime(dates_texte)

print(dates_dt)
# Résultat : DatetimeIndex(['2026-10-01', '2026-10-02', '2026-10-03'], dtype='datetime64[ns]', freq=None)

```

---

### `Timedelta` : Une durée ou un écart temporel

Un `Timedelta` représente une durée (par exemple : 5 jours, 3 heures, 45 secondes). Il permet d'effectuer des opérations arithmétiques directement sur les dates (ajouter ou soustraire du temps).

```python
# Définir une date de référence
date_achat = pd.Timestamp('2026-10-01')

# Créer un décalage (Timedelta) de 10 jours et 4 heures
delai = pd.Timedelta(days=10, hours=4)

# Calculer la date de livraison estimée
date_livraison = date_achat + delai

print(f"Date d'achat : {date_achat}")
print(f"Date de livraison estimée : {date_livraison}")

```

---

### `Period` : Une période ou un intervalle temporel fixe

Contrairement au `Timestamp` qui cible une *milliseconde* précise, le `Period` représente une **durée calendaire entière** (un mois complet, un trimestre, une année). C'est particulièrement utile pour la comptabilité et l'agrégation macro-économique.

```python
# Définir une période correspondant au mois d'octobre 2026
periode_mois = pd.Period('2026-10', freq='M')

print(periode_mois)
print(f"Début de la période : {periode_mois.start_time}")
print(f"Fin de la période : {periode_mois.end_time}")

# Avancer d'une période
mois_suivant = periode_mois + 1
print(f"Mois suivant : {mois_suivant}")

```

**En résumé pour ton lecteur :**

* **`Timestamp`** $\rightarrow$ Une date/heure isolée (ex: *le 28/10/2026 à 14h30*).
* **`Datetime`** $\rightarrow$ Le type de données global pour manipuler ces instants.
* **`Timedelta`** $\rightarrow$ Un intervalle de temps mathématique (ex: *+ 7 jours*).
* **`Period`** $\rightarrow$ Une tranche calendaire fixe (ex: *tout le mois d'octobre 2026*).
Créer des intervalles de dates par **jour** et par **mois**, accompagnés d'exemples pratiques.

---

## Manipulation de plages temporelles avec `period_range`

### intervalle de jours
Générer un intervalle de jours (`freq='D'`). Pour créer une séquence quotidienne (par exemple, pour simuler un historique de ventes sur une semaine) :

```python
import pandas as pd

# Créer une série de 7 jours consécutifs à partir d'aujourd'hui
jours = pd.date_range(start='2026-10-01', periods=7, freq='D')

print(jours)
# Résultat : DatetimeIndex(['2026-10-01', '2026-10-02', ..., '2026-10-07'], dtype='datetime64[ns]', freq='D')

```

---
### intervalle de mois

Générer un intervalle de mois (`freq='MS'` ou `'M'`). Pour planifier des bilans ou analyser des données mensuelles, on utilise la fréquence mensuelle.

* `'MS'` (Month Start) place la date au **premier jour** du mois.
* `'M'` place la date au **dernier jour** du mois.

```python
# Créer les dates de début de mois pour l'année 2026 (12 mois)
mois_debut = pd.date_range(start='2026-01-01', periods=12, freq='MS')

print(mois_debut)
# Résultat : DatetimeIndex(['2026-01-01', '2026-02-01', '2026-03-01', ..., '2026-12-01'], freq='MS')

```

---

## Combiner `pd.date_range()` avec un dictionnaire de données

Créer un DataFrame temporel pour nos produits. On peut combiner `pd.date_range()` avec un dictionnaire de données pour structurer un DataFrame prêt à l'analyse temporelle (par exemple, suivre les ventes quotidiennes d'un Clavier) :

```python
import numpy as np

# Générer un index de 5 jours consécutifs
dates = pd.date_range(start='2026-10-01', periods=5, freq='D')

# Associer ces dates à un DataFrame de ventes
df_ventes_journalieres = pd.DataFrame({
    'Date': dates,
    'Produit': ['Clavier', 'Clavier', 'Clavier', 'Clavier', 'Clavier'],
    'Quantite_Vendue': [3, 5, 2, 8, 4]
})

# Transformer la colonne 'Date' en index temporel (très utile pour Pandas)
df_ventes_journalieres = df_ventes_journalieres.set_index('Date')

print(df_ventes_journalieres)

```

## Modèles de temps

Pandas offre une gestion précise du temps grâce à quatre types d'objets principaux : `Timestamp` pour un instant précis, `Datetime` pour les types de données temporelles, `Timedelta` pour les durées, et `Period` pour représenter un intervalle de temps défini. Ces outils permettent de manipuler des calendriers complexes avec une grande rigueur syntaxique.

```python
import pandas as pd

# Création d'un instant précis (Timestamp)
instant = pd.Timestamp('2026-07-06 18:00:00')

# Calcul d'une durée (Timedelta)
duree = pd.Timedelta(days=5)
nouveau_temps = instant + duree

```

## Manipulation de plages temporelles

La fonction `period_range` permet de générer des séquences régulières de périodes temporelles sur un intervalle donné. Cette fonctionnalité est particulièrement utile pour créer des index basés sur des fréquences spécifiques, comme des mois ou des trimestres.

```python
# Génération d'une plage de périodes mensuelles
plage = pd.period_range(start='2026-01', periods=3, freq='M')

# Utilisation dans une Série
s = pd.Series([10, 20, 30], index=plage)

```
## Extraire le mois, le jour et l'heure

Pour extraire des composantes spécifiques (année, mois, jour, heure, minute) à partir d'une colonne de type date (`datetime`) dans Pandas, on utilise l'accesseur **`.dt`**.

Les principaux attributs de l'accesseur `.dt` :

| Attribut | Ce qu'il extrait | Exemple de résultat |
| --- | --- | --- |
| `.dt.year` | L'année | `2026` |
| `.dt.month` | Le numéro du mois (1-12) | `10` |
| `.dt.day` | Le numéro du jour du mois (1-31) | `28` |
| `.dt.hour` | L'heure (0-23) | `14` |
| `.dt.minute` | La minute (0-59) | `30` |
| `.dt.day_name()` | Le nom complet du jour (ex: *Monday*) | `'Wednesday'` |
| `.dt.month_name()` | Le nom complet du mois (ex: *October*) | `'October'` |

```python
import pandas as pd

# Création d'un DataFrame avec des dates et heures précises
df_logs = pd.DataFrame({
    'Timestamp_Brut': ['2026-10-28 08:30:00', '2026-10-29 14:15:45', '2026-10-30 19:00:30'],
    'Valeur': [100, 250, 180]
})

# Conversion indispensable en objet datetime
df_logs['Date_Heure'] = pd.to_datetime(df_logs['Timestamp_Brut'])

# Extraction du mois, du jour et de l'heure via l'accesseur .dt
df_logs['Mois'] = df_logs['Date_Heure'].dt.month
df_logs['Jour'] = df_logs['Date_Heure'].dt.day
df_logs['Heure'] = df_logs['Date_Heure'].dt.hour
df_logs['Jour_Semaine'] = df_logs['Date_Heure'].dt.day_name() # Nom du jour en anglais par défaut

print(df_logs[['Date_Heure', 'Mois', 'Jour', 'Heure', 'Jour_Semaine']])

```
---

## Exemple de synthèse

Rapport de synthèse des ventes par heure de la journée.Imaginons un journal de transactions e-commerce avec des horodatages précis. Nous voulons savoir à quelles heures la structure réalise le plus de chiffre d'affaires.

```python
import pandas as pd

# Données de ventes brutes avec horodatage
df_transactions = pd.DataFrame({
    'Horodatage': [
        '2026-10-01 09:15:00', '2026-10-01 10:30:00', '2026-10-01 14:10:00',
        '2026-10-02 09:45:00', '2026-10-02 14:20:00', '2026-10-02 18:05:00',
        '2026-10-03 10:00:00', '2026-10-03 14:50:00', '2026-10-03 18:30:00'
    ],
    'Montant': [50.0, 25.0, 120.0, 50.0, 80.0, 250.0, 25.0, 75.0, 100.0]
})

# Conversion indispensable en objet datetime
df_transactions['Horodatage'] = pd.to_datetime(df_transactions['Horodatage'])

# Extraction des composantes temporelles via l'accesseur .dt
df_transactions['Heure'] = df_transactions['Horodatage'].dt.hour
df_transactions['Jour'] = df_transactions['Horodatage'].dt.day_name()

# Synthèse : Grouper par heure pour analyser les créneaux les plus rentables
synthese_horaire = df_transactions.groupby('Heure').agg(
    Nombre_Commandes=('Montant', 'count'),  # Volume de transactions
    Chiffre_Affaires=('Montant', 'sum'),    # CA total cumulé
    Panier_Moyen=('Montant', 'mean')        # Montant moyen par commande
).reset_index()

print(synthese_horaire)

```

**La chaîne logique :** On convertit d'abord la colonne brute en `datetime`, on extrait les informations fines (`.dt.hour`, `.dt.day_name()`), puis on structure l'analyse macroscopique avec `.groupby().agg()`.
**La puissance du multi-indicateur :** En une seule passe, `.agg()` permet d'isoler à la fois le volume, la somme et la moyenne par tranche horaire, fournissant un tableau de bord directement exploitable par les équipes métiers.

## Exercices de fin de chapitre 

**Exercice 1 :** Créez un `Timestamp` correspondant à la date d'aujourd'hui et ajoutez-lui un `Timedelta` de 48 heures.

**Exercice 2:** Utilisez `period_range` pour générer une liste de 6 périodes trimestrielles commençant au premier trimestre 2026.


# Chapitre 10 : Statistiques et agrégations

L'analyse de données repose sur la capacité à synthétiser de vastes ensembles d'informations en indicateurs clés. Ce chapitre vous présente les outils pour calculer des statistiques descriptives et organiser vos données par catégories afin d'en extraire toute la valeur.

Dans ce chapitre :

* Statistiques descriptives
* Agrégations par catégories avec `groupby`

## Statistiques descriptives

Les statistiques descriptives permettent d'obtenir une vue d'ensemble rapide de la distribution et de la tendance centrale de vos données. Pandas propose des méthodes intégrées pour calculer instantanément des mesures comme la moyenne (`mean`), la somme (`sum`), le minimum (`min`), le maximum (`max`) ou l'écart-type (`std`) sur une série ou un DataFrame.

```python
import pandas as pd

# Création d'un jeu de données de scores
df = pd.DataFrame({'Scores': [10, 20, 30, 40, 50]})

# Calculs statistiques classiques
moyenne = df['Scores'].mean()  # Retourne 30.0
ecart_type = df['Scores'].std() # Mesure la dispersion
maxi = df['Scores'].max()      # Retourne 50

```

## Agrégations par catégories (`groupby`)

Le groupement de données est une étape fondamentale pour analyser des sous-ensembles selon des catégories spécifiques. La méthode `groupby` segmente vos données en groupes, vous permettant d'appliquer des calculs statistiques indépendamment pour chaque catégorie identifiée.

La méthode **`groupby()`** est l'équivalent sous Pandas du fameux `GROUP BY en SQL`. Elle repose sur le concept puissant du **Split-Apply-Combine** (Diviser - Appliquer - Combiner) : elle segmente le DataFrame en sous-groupes selon une ou plusieurs catégories, applique une fonction d'agrégation sur chacun d'eux, puis rassemble le tout dans un nouveau tableau synthétique.

```python
# Données avec une colonne catégorielle
data = {'Equipe': ['A', 'B', 'A', 'B'], 'Scores': [10, 20, 30, 40]}
df = pd.DataFrame(data)

# Calcul de la moyenne par équipe
resultat = df.groupby('Equipe')['Scores'].mean()
# Le résultat montre la moyenne pour A (20) et B (30)

```

```python
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam', 'Tapis'],
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio', 'Accessoire'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0, 15.0],
    'Quantite': [3, 10, 2, 5, 20],
    'Total': [150.0, 250.0, 160.0, 350.0, 300.0]
})

```

Calculer une métrique simple par catégorie. Pour obtenir rapidement le chiffre d'affaires total généré par chaque famille de produits :

```python
# Grouper par catégorie et sommer la colonne 'Total'
ca_par_categorie = df.groupby('Categorie')['Total'].sum()

print(ca_par_categorie)

```

* **Ce qui se passe :** Pandas isole les lignes par `Categorie` ('Accessoire' d'un côté, 'Audio' de l'autre), additionne les valeurs de la colonne `Total` pour chaque groupe, et renvoie une Série indexée par les catégories.

---

Appliquer plusieurs fonctions d'agrégation (`.agg()`). Tout comme nous l'avons vu pour l'ensemble du DataFrame, `.agg()` s'associe parfaitement à `groupby` pour croiser plusieurs indicateurs par groupe :

```python
# Appliquer des fonctions différentes selon les colonnes pour chaque catégorie
synthese_categorie = df.groupby('Categorie').agg({
    'Prix_HT': 'mean',        # Prix moyen par catégorie
    'Quantite': 'sum',        # Volume total d'articles vendus
    'Total': ['sum', 'max']   # CA total et CA maximal atteint
})

print(synthese_categorie)

```

* **Ce qui se passe :** Pandas génère un tableau croisé dynamique à colonnes multiples (MultiIndex) permettant d'auditer finement chaque rayon.

---

L'agrégation nommée pour des rapports propres. Pour éviter d'obtenir des noms de colonnes complexes ou imbriqués lors d'agrégations multiples, la syntaxe moderne de **l'agrégation nommée** permet de renommer directement les colonnes à la volée :

```python
# Création d'un rapport synthétique avec des noms de colonnes explicites
rapport_propre = df.groupby('Categorie').agg(
    Prix_Moyen=('Prix_HT', 'mean'),
    Volume_Ventes=('Quantite', 'sum'),
    Chiffre_Affaires=('Total', 'sum')
).reset_index()

print(rapport_propre)

```

* **Ce qui se passe :** L'utilisation de `.reset_index()` transforme l'index catégoriel en colonne classique, ce qui rend le DataFrame immédiatement exploitable pour un export ou la génération de graphiques.

## Agrégation avec plusieurs fonctions de réduction (`.agg`) 

La méthode **`.agg()`** (abréviation d'*aggregate*, ou son alias `.aggregate()`) est l'outil de synthèse par excellence sous Pandas. Elle permet d'appliquer une ou plusieurs fonctions de réduction (moyenne, somme, minimum, maximum, etc.) sur un DataFrame ou une Série, de manière globale ou personnalisée par colonne.

Appliquer une fonction unique à toutes les colonnes numériques

```python
# Calcule la somme de toutes les colonnes numériques
print(df.agg('sum'))

```

Appliquer plusieurs fonctions à la fois. En passant une liste de fonctions, Pandas retourne un tableau récapitulatif structuré :

```python
# Calcule à la fois la moyenne et l'écart-type sur les colonnes numériques
print(df.agg(['mean', 'std']))

```

Appliquer des fonctions différentes selon les colonnes (Le plus puissant). En passant un **dictionnaire**, tu peux spécifier exactement quel indicateur calculer pour chaque colonne, ce qui est particulièrement utile pour un rapport de synthèse :

```python
synthese = df.agg({
    'Prix_HT': ['mean', 'min', 'max'], # Moyenne et extrêmes pour les prix
    'Quantite': 'sum',               # Volume total vendu
    'Total': ['sum', 'mean']         # CA total et panier moyen global
})

print(synthese)

```

Contrairement aux méthodes directes comme `df['Prix_HT'].mean()` qui s'exécutent colonne par colonne séparément, `.agg()` centralise la logique de calcul et permet d'obtenir une **matrice de synthèse multi-métriques** en une seule ligne de code propre et lisible.

## `.agg` vs `.groupby`

C'est la syntaxe standard en analyse de données pour obtenir des rapports structurés et multi-indicateurs par catégorie :
* groupby() : Le découpeur (Structure)
* agg() (ou aggregate) : Le calculateur (Synthèse)

```python 
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam'],
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0],
    'Quantite': [3, 10, 2, 5]
})

# GROUPBY crée les groupes ('Accessoire', 'Audio')
# AGG calcule les métriques spécifiques pour chaque groupe
rapport = df.groupby('Categorie').agg(
    Prix_Moyen=('Prix_HT', 'mean'),
    Volume_Total=('Quantite', 'sum')
)

print(rapport)
```

## Exemple de synthèse

Rapport de synthèse commercial par catégorie
Reprenons notre catalogue enrichi avec une colonne 'Categorie'

```python
import pandas as pd

# Données de départ
df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam', 'Tapis'],
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio', 'Accessoire'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0, 15.0],
    'Quantite': [3, 10, 2, 5, 20],
    'Total': [150.0, 250.0, 160.0, 350.0, 300.0]
})

# Combinaison de groupby et agg (avec l'agrégation nommée)
synthese_par_categorie = df.groupby('Categorie').agg(
    Nombre_References=('Produit', 'count'),    # Nombre de produits distincts dans la catégorie
    Prix_Moyen_HT=('Prix_HT', 'mean'),          # Prix moyen unitaire
    Volume_Ventes=('Quantite', 'sum'),          # Nombre total d'articles vendus
    Chiffre_Affaires_Total=('Total', 'sum')     # CA total généré
).reset_index()

print(synthese_par_categorie)
```

## Exercices de fin de chapitre

**Exercice 1 :** Rapport de synthèse décisionnel par catégorie (`groupby` et `agg`)

En tant qu'analyste de données, la direction te demande de produire un tableau de bord synthétique pour analyser les performances des ventes par grande catégorie de produits.

```python
import pandas as pd

df_catalogue = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam', 'Tapis', 'Écran'],
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio', 'Accessoire', 'Moniteur'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0, 15.0, 250.0],
    'Quantite_Vendue': [12, 45, 8, 15, 30, 5],
    'Chiffre_Affaires': [600.0, 1125.0, 640.0, 1050.0, 450.0, 1250.0]
})

```

À l'aide d'un enchaînement `groupby()` et `agg()`, construis un DataFrame de synthèse qui regroupe les données par `Categorie` et calcule pour chacune d'elles :

* Le nombre total de références distinctes (`count` sur la colonne `Produit`).
* Le prix unitaire moyen HT (`mean` sur la colonne `Prix_HT`), renommé explicitement `Prix_Moyen`.
* Le volume total d'articles vendus (`sum` sur la colonne `Quantite_Vendue`), renommé `Volume_Total`.
* Le chiffre d'affaires cumulé (`sum` sur la colonne `Chiffre_Affaires`), renommé `CA_Total`.
* Transforme l'index de groupe en colonne classique pour obtenir un tableau propre et plat.

---

**Exercice 2 :** Pipeline de transformation sécurisée et enrichissement (`copy`, `pipe`, `apply`)
Préparer une simulation commerciale : appliquer une politique de remise promotionnelle ciblée et générer des libellés textuels automatisés pour chaque ligne, sans altérer le jeu de données initial.

```python
import pandas as pd

df_stocks = pd.DataFrame({
    'Reference': ['REF-01', 'REF-02', 'REF-03'],
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_Actuel': [50.0, 25.0, 80.0],
    'Stock_Restant': [10, 25, 4]
})

```

* Isolation sécurisée :** Créer une copie indépendante du DataFrame nommée `df_simulation` pour travailler sans risque de modification de l'original.
* Pipeline de remise :** Écrire une fonction personnalisée `appliquer_promo(df, taux)` qui calcule un nouveau prix `Prix_Promo` en appliquant le taux de réduction fourni (par exemple 15%), puis intègre cette fonction dans un appel `.pipe()`.
* Traitement ligne par ligne :** À l'aide de `.apply()` avec `axis=1`, ajoute une colonne `Alerte_Reappro` qui retourne la chaîne de caractères `"Prioritaire"` si le stock est strictement inférieur à 5, et `"Normal"` dans le cas contraire.


# Chapitre 11 : Jointures et fusion de données

La fusion de données est une opération fondamentale pour consolider des informations provenant de sources multiples au sein d'un même projet. Ce chapitre détaille comment combiner vos structures de données pour obtenir une vision unifiée et complète.

Dans ce chapitre :

* Concaténation de structures de données (`concat`)
* Jointures complexes (`merge`, `join`)

## Concaténation de structures

La fonction `concat` permet d'assembler des DataFrames ou des Séries les uns aux autres, soit verticalement (en ajoutant des lignes), soit horizontalement (en ajoutant des colonnes). C'est l'outil idéal pour regrouper des jeux de données de structure identique.

Le principe de base de `pd.concat()`. Contrairement aux jointures complexes (`pd.merge`), `pd.concat()` réalise un assemblage **structurel** basé sur les axes :

* **`axis=0`** (Vertical) : On ajoute des **lignes** (les DataFrames ont les mêmes colonnes mais des périodes ou des sources différentes).
* **`axis=1`** (Horizontal) : On ajoute des **colonnes** (les DataFrames partagent le même index mais apportent des informations différentes).

---

### Concaténation verticale (`axis=0`) : Empiler des lignes

C'est le cas le plus fréquent, par exemple pour fusionner les ventes de janvier et les ventes de février dans un seul grand historique.

```python
import pandas as pd

# Premier DataFrame : Catalogue A
df_partie1 = pd.DataFrame({
    'Produit': ['Clavier', 'Souris'],
    'Prix_HT': [50.0, 25.0]
})

# Deuxième DataFrame : Catalogue B
df_partie2 = pd.DataFrame({
    'Produit': ['Casque', 'Webcam'],
    'Prix_HT': [80.0, 70.0]
})

# Assemblage vertical (par défaut axis=0)
catalogue_global = pd.concat([df_partie1, df_partie2])

print(catalogue_global)

```

### Le piège de l'index et la bonne pratique. 

Par défaut, `pd.concat()` conserve les index d'origine de chaque DataFrame (ce qui peut donner des index du style `0, 1, 0, 1`). Pour réinitialiser proprement l'index du tableau final, on utilise l'argument **`ignore_index=True`** :

```python
# Assemblage propre avec réindexation séquentielle
catalogue_propre = pd.concat([df_partie1, df_partie2], ignore_index=True)
print(catalogue_propre)

```

---

### Concaténation horizontale (`axis=1`) : Juxtaposer des colonnes

Plus rare, cette opération permet d'associer deux tableaux qui partagent exactement le même index (par exemple, un tableau de prix d'un côté et un tableau de stocks de l'autre).

```python
# Tableau 1 : Les prix
df_prix = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0]
}, index=['A', 'B', 'C'])

# Tableau 2 : Les stocks (avec le même index)
df_stocks = pd.DataFrame({
    'Quantite': [10, 25, 4]
}, index=['A', 'B', 'C'])

# Assemblage horizontal (axis=1)
df_complet = pd.concat([df_prix, df_stocks], axis=1)

print(df_complet)

```

---

* **`pd.concat([df1, df2], axis=0)`** $\rightarrow$ **Vertical (V)** : On empile les lignes. Idéal pour concaténer des fichiers mensuels ou des historiques. Pense à `ignore_index=True` pour nettoyer l'index.
* **`pd.concat([df1, df2], axis=1)`** $\rightarrow$ **Horizontal (H)** : On met les tableaux côte à côte en se basant sur l'index commun.



---

## `pd.merge()` : Le couteau suisse des jointures (Basé sur des colonnes ou des clés)

Les jointures sont indispensables en analyse de données dès que l'information est répartie dans plusieurs tables distinctes (par exemple, une table de produits et une table de stocks, ou une table de clients et une table de commandes).

En Pandas, on distingue deux outils principaux : **`pd.merge()`** (le plus puissant, inspiré du `JOIN` en SQL) et **`.join()`** (plus spécialisé, basé sur les index).

| Fonction / Méthode | Objectif principal | Critère de fusion | Cas d'usage type |
| --- | --- | --- | --- |
| **`pd.merge()`** | Croiser des tables relationnelles | Des **colonnes** communes (`on`, `left_on`, `right_on`) | Relier des ventes à un catalogue produits via une clé étrangère. |
| **`.join()`** | Fusion rapide de tables indexées | Les **index** des DataFrames | Assembler deux tableaux partageant exactement la même granularité d'index (ex: chronologique ou géographique). |

`pd.merge()` permet de croiser deux DataFrames en se basant sur une ou plusieurs colonnes communes (les **clés de jointure**).

### Les 4 grands types de jointures (`how`) :

* **`'inner'`** (Par défaut) : Ne conserve que les lignes dont la clé est présente dans **les deux** tables (intersection).
* **`'left'`** : Conserve toutes les lignes de la table de **gauche**, même si aucune correspondance n'existe dans la droite (les valeurs manquantes deviennent des `NaN`).
* **`'right'`** : Conserve toutes les lignes de la table de **droite**.
* **`'outer'`** : Conserve toutes les lignes des **deux** tables (union complète).

### Enrichir un catalogue avec les stocks

```python
import pandas as pd

# Table 1 : Le catalogue des produits
df_produits = pd.DataFrame({
    'ID_Produit': [1, 2, 3],
    'Nom_Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0]
})

# Table 2 : Les stocks disponibles (gérés par un autre service)
df_stocks = pd.DataFrame({
    'ID_Produit': [1, 2, 4], # Note l'ID 4 qui n'existe pas dans les produits, et le 3 absent ici
    'Quantite': [15, 42, 10]
})

# Jointure interne (inner join) : uniquement les produits présents dans les deux tables
df_interne = pd.merge(df_produits, df_stocks, on='ID_Produit', how='inner')
print("--- Jointure Inner ---")
print(df_interne)

# Jointure à gauche (left join) : conserve tout le catalogue, même sans stock renseigné
df_gauche = pd.merge(df_produits, df_stocks, on='ID_Produit', how='left')
print("\n--- Jointure Left ---")
print(df_gauche)

```

---

## La méthode `.join()` : Le spécialiste des index

Alors que `pd.merge()` cherche des colonnes, la méthode `.join()` est conçue pour fusionner des DataFrames **directement via leurs index**. C'est une écriture plus courte et élégante lorsque les données sont déjà indexées par la même clé (par exemple, un identifiant client ou une date).

### Fusionner par l'index

```python
# Table 1 : Prix indexés par la référence produit
df_prix = pd.DataFrame({
    'Prix_HT': [50.0, 25.0, 80.0]
}, index=['SKU-01', 'SKU-02', 'SKU-03'])

# Table 2 : Stocks indexés par la même référence produit
df_stocks = pd.DataFrame({
    'Quantite': [15, 42, 8]
}, index=['SKU-01', 'SKU-02', 'SKU-03'])

# Utilisation directe de .join() sur l'index commun
df_catalogue_complet = df_prix.join(df_stocks)

print(df_catalogue_complet)

```

---

## Exemple de synthèse

### Le Scénario Global : La Cuisine des Données

Imaginons que tu gères les données d'une boutique informatique. Tes sources sont éparpillées :

Les ventes de janvier et de février proviennent de fichiers distincts $\rightarrow$ **On les empile (`concat`).**
Le catalogue des produits doit être enrichi avec les coordonnées des fournisseurs via une référence commune $\rightarrow$ **On les croise (`merge`).**
Les tableaux de prix et de stocks partagent exactement la même référence produit en index $\rightarrow$ **On les fusionne (`join`).**

---

### `pd.concat()` : Empiler les historiques (Vertical)

* **Objectif :** Réunir des tables qui ont la **même structure** (mêmes colonnes) mais des périodes différentes.

```python
import pandas as pd

# Ventes de janvier
df_janvier = pd.DataFrame({
    'Produit': ['Clavier', 'Souris'],
    'Quantite': [10, 25]
})

# Ventes de février
df_fevrier = pd.DataFrame({
    'Produit': ['Clavier', 'Souris'],
    'Quantite': [15, 30]
})

# Assemblage vertical avec réindexation propre
historique_ventes = pd.concat([df_janvier, df_fevrier], ignore_index=True)
print("--- CONCAT (Empilement) ---")
print(historique_ventes)

```

---

### `pd.merge()` : Relier des tables par colonnes (Relationnel)

* **Objectif :** Croiser des informations hétérogènes grâce à une **clé commune** (ex: ID Produit ou Code Fournisseur).

```python
# Le catalogue des produits
df_catalogue = pd.DataFrame({
    'ID_Produit': [1, 2],
    'Nom': ['Clavier', 'Souris'],
    'ID_Fournisseur': [101, 102]
})

# La table des fournisseurs
df_fournisseurs = pd.DataFrame({
    'ID_Fournisseur': [101, 103], # Note l'ID 103 absent du catalogue
    'Nom_Fournisseur': ['TechGlobal', 'AudioPro']
})

# Jointure externe (outer) pour ne rien perdre
bilan_fournisseurs = pd.merge(df_catalogue, df_fournisseurs, on='ID_Fournisseur', how='outer')
print("\n--- MERGE (Jointure par clé) ---")
print(bilan_fournisseurs)

```

---

### `.join()` : Fusionner des tables indexées

* **Objectif :** Assembler rapidement deux tables qui partagent **exactement le même index** (sans avoir à spécifier de colonnes de jointure).

```python
# Tableau 1 : Les prix indexés par SKU
df_prix = pd.DataFrame({
    'Prix_HT': [50.0, 25.0]
}, index=['SKU-01', 'SKU-02'])

# Tableau 2 : Les stocks indexés par le même SKU
df_stocks = pd.DataFrame({
    'Stock': [10, 45]
}, index=['SKU-01', 'SKU-02'])

# Fusion directe par l'index
catalogue_complet = df_prix.join(df_stocks)
print("\n--- JOIN (Fusion par index) ---")
print(catalogue_complet)

```

---

### Bilan synthétique pour le manuel

| Méthode | Axe de travail | Critère d'association | Cas d'usage type |
| --- | --- | --- | --- |
| **`pd.concat()`** | Principalement **Vertical** (`axis=0`) | Aucun (empilement pur) | Fusionner des historiques de fichiers mensuels. |
| **`pd.merge()`** | **Colonnes** | Une clé commune (`on`, `left_on`) | Relier des commandes à des clients ou des produits. |
| **`.join()`** | **Index** | Les index des DataFrames | Assembler des métriques partageant la même granularité indicielle. |


## Exercices de fin de chapitre

**Exercice 1 :** Créez deux petits DataFrames avec une colonne identique et utilisez `merge` pour les fusionner.

**Exercice 2 :** Utilisez `pd.concat` pour ajouter une ligne supplémentaire à un DataFrame existant en créant un second DataFrame contenant uniquement cette ligne.


# Chapitre 12 :  Persistance et Formats 

La persistance des données sous Pandas consiste à faire le pont entre la mémoire vive (le DataFrame en RAM) et le stockage sur disque (fichiers physiques). Qu'il s'agisse d'ingérer des données brutes issues de systèmes tiers ou d'exporter des rapports finalisés, Pandas propose des fonctions unifiées et performantes basées sur les préfixes **`read_`** et **`to_`**.

Dans ce chapitre :

* Lecture et écriture de fichiers CSV
* Persistance des données via le format Pickle

## Le format CSV (*Comma-Separated Values*) : Le standard tabulaire

Le CSV est le format universel pour les données tabulaires. Léger et lisible, il est géré nativement par la quasi-totalité des outils logiciels.

### Lecture d'un fichier CSV (`pd.read_csv`)

La fonction `pd.read_csv()` est la plus utilisée en science des données. Elle intègre de nombreux paramètres pour s'adapter à la structure du fichier source :

```python
import pandas as pd

# Lecture d'un fichier CSV avec gestion du séparateur et de l'encodage
df_ventes = pd.read_csv(
    'ventes_2026.csv',
    sep=',',               # Séparateur de colonnes (virgule, point-virgule ';', tabulation '\t')
    encoding='utf-8',      # Encodage des caractères (utf-8, latin-1 pour les accents français)
    index_col=0,           # Utiliser la première colonne comme index du DataFrame
    na_values=['NA', 'NULL', '?'] # Définition personnalisée des valeurs manquantes
)

print(df_ventes.head())

```

### Sauvegarde d'un DataFrame (`df.to_csv`)

Pour exporter un DataFrame vers un fichier CSV, on utilise la méthode `.to_csv()` :

```python
# Exportation propre sans inclure l'index technique de Pandas
df_ventes.to_csv(
    'export_ventes_nettoye.csv',
    index=False,           # Évite d'enregistrer l'index (souvent inutile)
    sep=';',               # Privilégier le point-virgule pour Excel en version française
    encoding='utf-8-sig'   # 'utf-8-sig' garantit l'affichage correct des accents sous Excel
)

```

---

## Le format JSON (*JavaScript Object Notation*) : Le roi du Web

Le format JSON est omniprésent dans les architectures web et les API REST. Contrairement au CSV, il gère des structures **hiérarchiques** (imbrications d'objets et de listes).

### Lecture d'un fichier JSON (`pd.read_json`)

Pandas analyse automatiquement la structure JSON pour la convertir sous forme tabulaire :

```python
# Chargement d'un fichier JSON structuré en enregistrements
df_produits = pd.read_json('catalogue.json', orient='records')

print(df_produits.head())

```

### Sauvegarde au format JSON (`df.to_json`)

L'argument **`orient`** est crucial lors de l'exportation JSON pour définir la structure géométrique du fichier final :

```python
# Exportation au format liste d'enregistrements (idéal pour le web/API)
df_produits.to_json(
    'catalogue_export.json',
    orient='records',      # Structure chaque ligne comme un objet JSON distinct
    force_ascii=False,     # Conserve les caractères accentués en clair (non échappés)
    indent=4               # Indentation pour rendre le fichier lisible par un humain
)

```

---

## Le format XML (*Extensible Markup Language*) : Les données semi-structurées

Bien que plus lourd que le JSON, le format XML reste la référence dans les environnements d'entreprise, bancaires ou administratifs (factures électroniques, flux RSS, configurations).

#### Lecture d'un fichier XML (`pd.read_xml`)

Pandas intègre nativement un moteur d'analyse XML (nécessitant les bibliothèques `lxml` ou `html5lib` en arrière-plan) :

```python
# Lecture d'un fichier XML en ciblant la balise répétitive (xpath)
df_factures = pd.read_xml(
    'factures.xml',
    xpath='.//facture'     # XPath ciblant chaque nœud <facture> à transformer en ligne
)

print(df_factures.head())

```

### Sauvegarde au format XML (`df.to_xml`)

Pour sérialiser un DataFrame en XML :

```python
# Exportation en XML avec personnalisation des balises racines
df_factures.to_xml(
    'factures_export.xml',
    root_name='Entreprise', # Nom de la balise racine globale
    row_name='Facture',     # Nom de la balise représentant chaque ligne
    index=False,
    encoding='utf-8'
)

```

---

### Synthèse des méthodes de persistance

| Format | Fonction de lecture | Méthode d'écriture | Cas d'usage principal | 
| --- | --- | --- | --- |
| **CSV** | `pd.read_csv()` | `df.to_csv()` | Tableaux plats, exports Excel, volumétrie simple. | 
| **JSON** | `pd.read_json()` | `df.to_json()` | Échanges avec des API web, données imbriquées. | 
| **XML** | `pd.read_xml()` | `df.to_xml()` | Systèmes legacy, facturation électronique, flux RSS. | 

## Persistance des données via le format Pickle

Le format Pickle permet de sérialiser des objets Python, ce qui signifie que vous pouvez sauvegarder un DataFrame exactement tel qu'il est, en conservant les types de données complexes et la structure des index. Cette méthode est idéale pour stocker des données temporaires entre deux sessions de travail sans perte d'information. Le format pickle est propre à Python

```python
# Sauvegarde au format binaire Pickle
df.to_pickle('donnees.pkl')

# Chargement du fichier Pickle
df_recharge = pd.read_pickle('donnees.pkl')

```

## Exemple de synthèse
Le cycle complet de persistance du catalogue.Ce script démontre comment sauvegarder un même DataFrame sous les trois grands formats standards du marché en respectant les bonnes pratiques (gestion des accents, suppression des index superflus, formats structurés), puis à le recharger.

```python
import pandas as pd

# Données initiales (notre catalogue fil rouge)
df_catalogue = pd.DataFrame({
    'ID_Produit': [101, 102, 103],
    'Produit': ['Clavier Mécanique', 'Souris Ergonomique', 'Casque Audio'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Disponible': [True, True, False]
})

print("--- DataFrame d'origine ---")
print(df_catalogue)

# ==========================================
# EXPORTATION MULTI-FORMATS
# ==========================================

# A. Format CSV (Idéal pour la bureautique et Excel en France)
# -> index=False évite d'exporter la colonne d'index technique (0, 1, 2)
# -> sep=';' et utf-8-sig garantissent un affichage parfait sous Excel
df_catalogue.to_csv('catalogue.csv', index=False, sep=;', encoding='utf-8-sig')

# B. Format JSON (Idéal pour le Web, les API et NoSQL)
# -> orient='records' structure chaque ligne comme un objet indépendant
df_catalogue.to_json('catalogue.json', orient='records', force_ascii=False, indent=4)

# C. Format XML (Idéal pour les échanges d'entreprise et facturation)
# -> root_name et row_name personnalisent les balises racines et de lignes
df_catalogue.to_xml('catalogue.xml', root_name='Boutique', row_name='Article', index=False)

print("\n[✔] Fichiers CSV, JSON et XML exportés avec succès sur le disque.")

# ==========================================
# RELECTURE ET VÉRIFICATION (EXEMPLE CSV)
# ==========================================
df_recharge = pd.read_csv('catalogue.csv', sep=';', encoding='utf-8-sig')

print("\n--- Données rechargées depuis le fichier CSV ---")
print(df_recharge)

```

---

### Les règles d'or à retenir pour votre manuel :

**Toujours utiliser `index=False**` lors des exports (`to_csv`, `to_xml`, etc.) sauf si l'index possède un sens métier avéré (comme une date ou une clé primaire explicite).
**Soigner l'encodage (`utf-8-sig` ou `utf-8`)** pour ne jamais corrompre les caractères accentués lors du passage de Python vers des fichiers tiers.
**Choisir le bon format selon l'écosystème cible** : CSV pour l'analyse tabulaire classique, JSON pour le développement web/API, et XML pour les flux d'entreprise stricts.

## Exercices de fin de chapitre

**Exercice 1:** Créez un DataFrame, sauvegardez-le en format CSV, puis rechargez-le dans une nouvelle variable pour vérifier l'intégrité des données.
**Exercice 2:** Expérimentez la sauvegarde d'un DataFrame contenant des types de données variés (nombres, textes, dates) au format Pickle et confirmez que les types sont conservés après le rechargement.


# Chapitre 13 : Gestion des structures multidimensionnelles

L'analyse de données complexes nécessite souvent de manipuler des dimensions supplémentaires au-delà du simple tableau bidimensionnel. Qu'il s'agisse de croiser des informations géographiques, temporelles ou organisationnelles, Pandas propose des outils puissants pour structurer, indexer et remodeler vos données de manière hiérarchique.

Dans ce chapitre :

* Introduction aux Panel Data
* Utilisation du `MultiIndex`
* Sélection et tri sur un `MultiIndex`
* Tableaux croisés : `pivot()`, `pivot_table()` et `crosstab()`
* Remodeler les données : `stack()`, `unstack()` et `melt()`
* Exemple de synthèse
* Exercices de fin de chapitre

---

## Introduction aux Panel Data

Les **Panel Data** (ou données de panel) combinent des données longitudinales (temporelles) et transversales (cross-sectional). Elles permettent de suivre plusieurs entités (par exemple, des magasins, des pays ou des produits) sur plusieurs périodes de temps (jours, mois, années).

Sous Pandas, ces structures multidimensionnelles sont généralement gérées à l'aide d'un index hiérarchique (`MultiIndex`) ou par un remodelage tabulaire adapté.

### Représentation conceptuelle d'un jeu de données de panel

```python
import pandas as pd

# Données de ventes pour 2 magasins sur 2 trimestres
data = {
    'Chiffre_Affaires': [12000, 15000, 18000, 21000]
}

# Création d'un index à deux niveaux : (Magasin, Trimestre)
index_panel = pd.MultiIndex.from_tuples([
    ('Magasin_Paris', 'Q1'),
    ('Magasin_Paris', 'Q2'),
    ('Magasin_Lyon', 'Q1'),
    ('Magasin_Lyon', 'Q2')
], names=['Magasin', 'Trimestre'])

df_panel = pd.DataFrame(data, index=index_panel)
print(df_panel)

```

### Transformation d'un tableau large en format panel long

Souvent, les données brutes arrivent sous forme "large" (une colonne par période). Il est nécessaire de les structurer en format panel exploitable.

```python
import pandas as pd

# Données larges initiales
df_large = pd.DataFrame({
    'Magasin': ['Paris', 'Lyon'],
    '2025': [100000, 120000],
    '2026': [110000, 135000]
})

# Passage au format long (Panel Data) via la méthode melt
df_panel_long = df_large.melt(
    id_vars=['Magasin'], 
    var_name='Annee', 
    value_name='CA'
).set_index(['Magasin', 'Annee'])

print(df_panel_long)

```

---

## Utilisation du `MultiIndex`

Le **`MultiIndex`** (ou index hiérarchique) permet d'associer plusieurs index à une même ligne ou à une même colonne. Il transforme un DataFrame bidimensionnel classique en une structure capable de représenter 3 dimensions ou plus.

### Création d'un MultiIndex à partir de listes de tuples

```python
import pandas as pd
import numpy as np

# Définition de tuples hiérarchiques (Rayon, Produit)
index_hierarchique = pd.MultiIndex.from_tuples([
    ('Accessoire', 'Clavier'),
    ('Accessoire', 'Souris'),
    ('Audio', 'Casque'),
    ('Audio', 'Microphone')
], names=['Rayon', 'Produit'])

df_multilignes = pd.DataFrame({
    'Prix_HT': [50.0, 25.0, 80.0, 60.0],
    'Stock': [120, 200, 45, 30]
}, index=index_hierarchique)

print(df_multilignes)

```

### Création d'un MultiIndex par produit cartésien (`from_product`)

Pour croiser toutes les combinaisons possibles entre des listes de valeurs (ex: des villes et des mois), la fonction `pd.MultiIndex.from_product()` automatise la génération.

```python
villes = ['Paris', 'Lyon']
mois = ['Janvier', 'Février']

# Génère toutes les combinaisons croisées Ville x Mois
index_produit = pd.MultiIndex.from_product([villes, mois], names=['Ville', 'Mois'])

df_meteo = pd.DataFrame({
    'Temperature_Moy': [5.2, 6.1, 4.8, 5.5]
}, index=index_produit)

print(df_meteo)

```

---

## Sélection et tri sur un `MultiIndex`

Manipuler un `MultiIndex` exige des méthodes de sélection adaptées pour interroger les différents niveaux de la hiérarchie. L'utilisation de l'objet **`pd.IndexSlice`** est indispensable pour réaliser des découpages (*slicing*) complexes.

### Sélection par niveau avec `.loc` et `IndexSlice`

```python
import pandas as pd

# Reprise de notre DataFrame hiérarchique (Rayon, Produit)
index_hierarchique = pd.MultiIndex.from_tuples([
    ('Accessoire', 'Clavier'),
    ('Accessoire', 'Souris'),
    ('Audio', 'Casque'),
    ('Audio', 'Microphone')
], names=['Rayon', 'Produit'])

df = pd.DataFrame({
    'Prix_HT': [50.0, 25.0, 80.0, 60.0]
}, index=index_hierarchique)

# A. Sélectionner tout le contenu du rayon 'Audio'
print("--- Rayon Audio ---")
print(df.loc['Audio'])

# B. Sélectionner un produit précis en passant un tuple
print("\n--- Prix du Casque ---")
print(df.loc[('Audio', 'Casque'), 'Prix_HT'])

```

### Tri d'un MultiIndex avec `.sort_index()`

Un `MultiIndex` non trié peut bloquer certaines opérations de sélection par tranche. Le tri hiérarchique s'effectue via `.sort_index()`.

```python
# Création d'un index volontairement désordonné
index_desordonne = pd.MultiIndex.from_tuples([
    ('Audio', 'Casque'),
    ('Accessoire', 'Clavier'),
    ('Audio', 'Microphone'),
    ('Accessoire', 'Souris')
], names=['Rayon', 'Produit'])

df_vrac = pd.DataFrame({'Stock': [10, 20, 30, 40]}, index=index_desordonne)

# Tri hiérarchique par ordre alphabétique des index
df_trie = df_vrac.sort_index()
print(df_trie)

```

---

## Tableaux croisés : `pivot()`, `pivot_table()` et `crosstab()`

Pour réorganiser des tableaux de données plates en matrices de synthèse croisées, Pandas propose trois fonctions majeures adaptées à des cas d'usage précis.

* **`pivot()`** : Réorganise les données sans fonction d'agrégation (les doublons provoquent une erreur).
* **`pivot_table()`** : Un tableau croisé dynamique puissant intégrant des fonctions d'agrégation (gestion des doublons).
* **`crosstab()`** : Calcule une table de contingence (fréquence de croisement entre variables catégorielles).

### Utilisation de `pivot_table()` pour synthétiser des ventes

```python
import pandas as pd

df_ventes = pd.DataFrame({
    'Mois': ['Jan', 'Jan', 'Fév', 'Fév'],
    'Magasin': ['Paris', 'Lyon', 'Paris', 'Lyon'],
    'CA': [1000, 1500, 1200, 1600]
})

# Création d'une table croisée : Mois en lignes, Magasins en colonnes, somme du CA
table_croisee = df_ventes.pivot_table(
    index='Mois', 
    columns='Magasin', 
    values='CA', 
    aggfunc='sum'
)

print(table_croisee)

```

### Utilisation de `pd.crosstab()` pour compter des fréquences

```python
# Analyser la répartition des types de clients par canal de commande
df_commandes = pd.DataFrame({
    'Canal': ['Web', 'Boutique', 'Web', 'Web', 'Boutique'],
    'Type_Client': ['VIP', 'Standard', 'Standard', 'VIP', 'VIP']
})

matrice_frequence = pd.crosstab(df_commandes['Canal'], df_commandes['Type_Client'])
print(matrice_frequence)

```

---

## Remodeler les données : `stack()`, `unstack()` et `melt()`

Le remodelage consiste à faire basculer un DataFrame de colonnes à lignes (`stack`), de lignes à colonnes (`unstack`), ou de le "fondre" pour le aplatir (`melt`).

* **`stack()`** : Pivote les colonnes vers les lignes (augmente le nombre de lignes, réduit le nombre de colonnes).
* **`unstack()`** : Pivote les lignes vers les colonnes (génère un MultiIndex sur les colonnes).
* **`melt()`** : Transforme des colonnes larges en paires attribut-valeur (format long).

### Utilisation croisée de `stack()` et `unstack()`

```python
import pandas as pd

# DataFrame avec des mois en colonnes
df_large = pd.DataFrame({
    'Jan': [100, 200],
    'Fev': [150, 220]
}, index=['Produit_A', 'Produit_B'])

# Transformer les colonnes de mois en lignes avec stack()
df_empile = df_large.stack()
print("--- Après stack() ---")
print(df_empile)

# Revenir en arrière avec unstack()
df_retrousse = df_empile.unstack()
print("\n--- Après unstack() ---")
print(df_retrousse)

```

### Utilisation de `melt()` pour aplatir un tableau

```python
# Tableau de scores par trimestre
df_trimestres = pd.DataFrame({
    'Eleve': ['Alice', 'Bob'],
    'Q1': [14, 12],
    'Q2': [16, 15]
})

# Fusionner les colonnes de trimestres en lignes uniques
df_fondu = df_trimestres.melt(
    id_vars=['Eleve'], 
    var_name='Trimestre', 
    value_name='Note'
)

print(df_fondu)

```

---

## Exemple de synthèse : Analyse multidimensionnelle e-commerce

Ce script complet met en œuvre le chaînage des opérations : création de données de panel, structuration via un `MultiIndex`, pivotement de table et extraction analytique.

```python
import pandas as pd
import numpy as np

# Données brutes de ventes multi-critères
df_brut = pd.DataFrame({
    'Region': ['Nord', 'Nord', 'Sud', 'Sud', 'Nord', 'Sud'],
    'Tranche_Horaire': ['Matin', 'Soir', 'Matin', 'Soir', 'Matin', 'Soir'],
    'Categorie': ['Accessoire', 'Audio', 'Accessoire', 'Audio', 'Audio', 'Accessoire'],
    'CA': [1200, 800, 950, 1400, 1100, 750]
})

print("--- Données brutes ---")
print(df_brut)

# Création d'une table croisée dynamique multidimensionnelle (Pivot Table)
# Croisement : Région et Tranche horaire en lignes, Catégorie en colonnes
table_synthese = df_brut.pivot_table(
    index=['Region', 'Tranche_Horaire'],
    columns='Categorie',
    values='CA',
    aggfunc='sum',
    fill_value=0
)

print("\n--- Table croisée dynamique (MultiIndex en lignes) ---")
print(table_synthese)

# Remodeler la structure avec stack() pour analyse séquentielle
df_empile = table_synthese.stack().reset_index(name='Chiffre_Affaires_Total')

print("\n--- Structure aplatie (Format Long / Panel) ---")
print(df_empile.head())

```

---

## Exercices de fin de chapitre

**Exercice 1 :** Manipulation d'un MultiIndex et sélection

Créez un DataFrame comportant un `MultiIndex` à deux niveaux : `"Ville"` (`"Paris"`, `"Marseille"`) et `"Mois"` (`"Janvier"`, `"Février"`), contenant une colonne `"Consommation_Elec_kWh"`.
Utilisez l'attribut `.loc` pour extraire l'ensemble des données relatives à la ville de `"Paris"`.

**Exercice 2 :** Utilisation de `pivot_table` et de `melt`

À partir du jeu de données suivant :

```python
import pandas as pd
df_donnees = pd.DataFrame({
    'Equipe': ['Dev', 'Dev', 'Ops', 'Ops'],
    'Projet': ['Alpha', 'Beta', 'Alpha', 'Beta'],
    'Budget': [50000, 30000, 20000, 45000]
})

```

Transformez ce tableau à l'aide de `pivot_table` pour obtenir les équipes en lignes, les projets en colonnes, et les budgets en valeurs.
Appliquez ensuite un `melt()` sur le résultat obtenu pour le ramener sous un format long.

# Chapitre 14: Rappel Pyplot – Les fondations graphiques de Python

Bien que Pandas propose une surcouche intuitive avec `df.plot()` pour prototyper rapidement, l'écosystème graphique de Python repose en profondeur sur **Matplotlib** et son module d'interface haut niveau : **Pyplot** (`plt`). Maîtriser Pyplot est indispensable pour reprendre le contrôle total de la mise en page, structurer des figures complexes et produire des graphiques de qualité professionnelle.

Dans ce chapitre :

* Architecture de Matplotlib : `Figure` et `Axes`
* Tracer un graphique simple : `plt.plot()`
* Titres, étiquettes, légendes et styles
* Plusieurs graphiques dans une figure : `plt.subplots()`
* Enregistrer une figure
* Exemple de synthèse
* Exercices de fin de chapitre

---

## Architecture de Matplotlib : `Figure` et `Axes`

Pour construire un graphique avec Matplotlib, il faut comprendre sa hiérarchie visuelle, reposant sur deux objets fondamentaux :

* **La `Figure**` : C'est la fenêtre globale ou la "feuille de dessin" qui contient l'ensemble des éléments graphiques (titres globaux, légende générale, sous-graphiques).
* **Les `Axes**` : Ce sont les zones de traçage proprement dites (le système d'axes $X$ et $Y$ où les courbes et barres sont dessinées). Une `Figure` peut contenir un ou plusieurs `Axes`.

### Création explicite d'une Figure et d'un unique objet Axes

```python
import matplotlib.pyplot as plt

# Création explicite de la figure (fig) et de la zone de tracé (ax)
fig, ax = plt.subplots(figsize=(6, 3))

# Tracé d'une ligne simple sur l'objet ax
ax.plot([1, 2, 3, 4], [10, 20, 25, 30])

plt.show()

```

### Manipulation des propriétés de l'objet Axes

L'objet `ax` centralise toutes les commandes de modification des axes, ce qui rend le code plus modulaire et lisible que l'utilisation de fonctions globales.

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots()

# Utilisation de la méthode .set() pour configurer l'axe en une seule ligne
ax.set(
    title="Évolution des performances",
    xlabel="Temps (en heures)",
    ylabel="Score technique"
)

ax.plot([0, 1, 2], [5, 12, 18])
plt.show()

```

---

## Tracer un graphique simple : `plt.plot()`

La fonction **`plt.plot()`** est la fonction universelle de Matplotlib pour tracer des graphiques linéaires ou des nuages de points reliés par des lignes. Elle accepte en arguments des listes ou des tableaux NumPy pour les coordonnées $X$ et $Y$.

### Tracé linéaire basique à partir de listes Python

```python
import matplotlib.pyplot as plt

# Coordonnées des abscisses (X) et des ordonnées (Y)
jours = ['Lun', 'Mar', 'Mer', 'Jeu', 'Ven']
visiteurs = [120, 250, 180, 300, 420]

# Tracé du graphique linéaire
plt.plot(jours, visiteurs)
plt.show()

```

### Personnalisation directe des traits et des marqueurs dans `plt.plot()`

Il est possible de modifier l'apparence des courbes directement via les arguments de la fonction `plt.plot()` (couleurs, épaisseur, style de ligne et symboles de sommets).

```python
import numpy as np

# Génération d'une onde sinusoïdale avec NumPy
x = np.linspace(0, 10, 50)
y = np.sin(x)

# Tracé avec une ligne en tirets ('.--'), de couleur verte, avec des marqueurs ronds ('o')
plt.plot(x, y, color='forestgreen', linestyle='--', linewidth=2, marker='o', markersize=4)
plt.show()

```

---

## Titres, étiquettes, légendes et styles

Un graphique professionnel doit être immédiatement compréhensible hors de son contexte. Cela nécessite l'ajout rigoureux de titres, d'étiquettes sur les axes (`xlabel`, `ylabel`), de légendes pour identifier les courbes, et éventuellement d'une grille de lecture.

### Ajout de titres, libellés d'axes et de légendes

```python
import matplotlib.pyplot as plt

mois = ['Jan', 'Fév', 'Mar', 'Avr']
ca_2025 = [100, 120, 115, 130]
ca_2026 = [110, 135, 140, 155]

# Traçage des deux courbes avec des étiquettes (label)
plt.plot(mois, ca_2025, label='Année 2025', color='gray', linestyle=':')
plt.plot(mois, ca_2026, label='Année 2026', color='teal', linewidth=2)

# Ajout des éléments textuels et de la légende
plt.title("Comparatif annuel du Chiffre d'Affaires")
plt.xlabel("Mois de l'année")
plt.ylabel("Montant (k€)")
plt.legend()       # Affiche la légende en se basant sur les arguments 'label'
plt.grid(True, alpha=0.3) # Ajout d'une grille légère

plt.show()

```

### Application d'un style global prédéfini

Matplotlib propose des thèmes visuels intégrés pour modifier instantanément l'esthétique de vos figures sans tout reconfigurer à la main.

```python
import matplotlib.pyplot as plt

# Activer un style visuel moderne (ex: 'seaborn-v0_8-whitegrid' ou 'ggplot')
plt.style.use('ggplot')

x = [1, 2, 3, 4]
y = [10, 25, 15, 30]

plt.plot(x, y, marker='s')
plt.title("Graphique avec le style 'ggplot'")
plt.xlabel("Index")
plt.ylabel("Valeur")
plt.show()

```

---

## Plusieurs graphiques dans une figure : `plt.subplots()`

Lorsque l'on souhaite comparer plusieurs métriques côte à côte sans surcharger un unique repère, on utilise un système de grille de sous-graphiques (*subplots*) via `plt.subplots(nrows, ncols)`.

### Grille de 1 ligne et 2 colonnes

```python
import matplotlib.pyplot as plt

# Création d'une figure contenant 2 sous-graphiques côte à côte (1 ligne, 2 colonnes)
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(10, 4))

# Premier graphique (à gauche) : Les ventes
ax1.plot([1, 2, 3], [10, 20, 15], color='blue', marker='o')
ax1.set_title("Volume des Ventes")
ax1.set_xlabel("Semaine")

# Second graphique (à droite) : Les profits
ax2.plot([1, 2, 3], [2, 8, 5], color='orange', marker='s')
ax2.set_title("Bénéfices nets")
ax2.set_xlabel("Semaine")

# Ajustement automatique de l'espacement entre les graphiques
plt.tight_layout()
plt.show()

```

### Grille matricielle de 2x2 sous-graphiques

```python
import matplotlib.pyplot as plt
import numpy as np

# Création d'une grille 2x2 (retourne une matrice d'axes 2x2)
fig, axes = plt.subplots(2, 2, figsize=(8, 6))

# Accès aux sous-graphiques par leurs indices de ligne et colonne
axes[0, 0].plot([1, 2], [3, 4], color='red')
axes[0, 0].set_title("Graphique A")

axes[0, 1].plot([1, 2], [4, 1], color='green')
axes[0, 1].set_title("Graphique B")

axes[1, 0].plot([1, 2], [2, 5], color='purple')
axes[1, 0].set_title("Graphique C")

axes[1, 1].plot([1, 2], [1, 1], color='brown')
axes[1, 1].set_title("Graphique D")

plt.tight_layout()
plt.show()

```

---

## Enregistrer une figure

Une fois le graphique mis en forme, il est souvent nécessaire de l'exporter sous forme de fichier image (PNG, JPG) ou de format vectoriel haute qualité (PDF, SVG) pour l'insérer dans un rapport ou un e-book. On utilise pour cela la méthode **`plt.savefig()`**.

### Exportation standard au format PNG haute résolution

```python
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]
y = [10, 20, 25, 30]

plt.plot(x, y, marker='o')
plt.title("Rapport de performance")
plt.xlabel("Mois")
plt.ylabel("Score")

# Sauvegarde de la figure avec une résolution élevée (dpi=300) et un fond propre
plt.savefig("rapport_performance.png", dpi=300, bbox_inches='tight')
plt.close() # Ferme la figure pour libérer la mémoire

```

### Exportation au format vectoriel SVG pour l'édition web ou print

```python
import matplotlib.pyplot as plt

# Traçage d'une figure simple
plt.figure(figsize=(5, 3))
plt.plot([1, 2, 3], [5, 10, 15], color='crimson')
plt.title("Graphique vectoriel")

# Export en SVG (les lignes restent nettes peu importe le zoom)
plt.savefig("graphique_vectoriel.svg", format='svg')
plt.close()

```

---

## Exemple de synthèse

Ce script complet rassemble l'ensemble des notions de Pyplot abordées dans ce chapitre : création d'une figure structurée, génération d'une grille multi-axes (`plt.subplots`), personnalisation des styles, ajout de légendes et exportation finale sur disque.

```python
import matplotlib.pyplot as plt
import numpy as np

# Préparation des données de test
semaines = ['S1', 'S2', 'S3', 'S4', 'S5']
trafic_web = [1200, 1500, 1350, 1800, 2100]
taux_conversion = [2.1, 2.4, 2.3, 2.8, 3.2]

# Création d'une figure avec 2 sous-graphiques verticaux (2 lignes, 1 colonne)
fig, (ax_trafic, ax_conv) = plt.subplots(2, 1, figsize=(8, 6), sharex=True)

# Premier graphique : Évolution du trafic web (Ligne bleue avec marqueurs)
ax_trafic.plot(semaines, trafic_web, color='royalblue', marker='o', linewidth=2, label='Visiteurs uniques')
ax_trafic.set_title("Tableau de bord marketing - Suivi mensuel", fontsize=14, fontweight='bold')
ax_trafic.set_ylabel("Nombre de visiteurs")
ax_trafic.legend(loc='upper left')
ax_trafic.grid(True, linestyle='--', alpha=0.5)

# Second graphique : Évolution du taux de conversion (Ligne orange avec carrés)
ax_conv.plot(semaines, taux_conversion, color='darkorange', marker='s', linewidth=2, label='Taux de conversion (%)')
ax_conv.set_xlabel("Semaines de l'année")
ax_conv.set_ylabel("Pourcentage (%)")
ax_conv.legend(loc='upper left')
ax_conv.grid(True, linestyle='--', alpha=0.5)

# Mise en page globale et exportation propre
plt.tight_layout()
plt.savefig("tableau_de_bord_marketing.png", dpi=300, bbox_inches='tight')

# Affichage à l'écran
plt.show()

```

---

## Exercices de fin de chapitre

**Exercice 1 :** Création d'une figure multi-axes personnalisée

Créez un script Python qui génère une figure de taille `(8, 4)` contenant deux graphiques côte à côte (1 ligne, 2 colonnes) à l'aide de `plt.subplots()`.
Sur le premier sous-graphique, tracez une fonction carrée ($y = x^2$ pour $x$ allant de 0 à 5) avec une ligne rouge en pointillés.
Sur le second sous-graphique, tracez une fonction racine carrée ($y = \sqrt{x}$) avec une ligne bleue continue, puis enregistrez l'ensemble sous le nom `exercices_maths.png` à 300 DPI.

**Exercice 2 :** Habillage et superposition de courbes

À l'aide de `plt.plot()`, tracez sur un même repère deux courbes représentant l'évolution hypothétique des ventes de deux produits ("Produit A" et "Produit B") sur 4 trimestres.
Personnalisez le graphique en ajoutant un titre explicite, des libellés sur les axes $X$ et $Y$, une légende positionnée en haut à droite, ainsi qu'une grille de fond en pointillés.

# Chapitre 15 : Visualisation de données avec Pandas (`df.plot()`)

L'analyse de données complexes nécessite souvent de manipuler des dimensions supplémentaires au-delà du simple tableau bidimensionnel. Ce chapitre vous présente les Panel Data et l'utilisation des MultiIndex pour structurer vos informations de manière hiérarchique et précise.

Dans ce chapitre :

* Introduction aux Panel Data
* Utilisation du `MultiIndex`

## Visualisation avec Matplotlib, Pyplot et Seaborn

Bien que la méthode `df.plot()` suffise amplement pour l'exploration rapide et la génération de graphiques simples, l'écosystème Python de la visualisation repose sur trois piliers complémentaires. Pour concevoir des rapports de production avancés ou des analyses statistiques poussées, il est indispensable de savoir sortir de Pandas pour exploiter directement **Matplotlib**, son module **Pyplot**, et la bibliothèque statistique **Seaborn**.

---

### La hiérarchie des outils de visualisation

Pour comprendre comment ces outils s'articulent, il faut voir Pandas comme une surcouche de simplification :

**Pandas (`df.plot()`)** : Intégré nativement, idéal pour prototyper et visualiser directement depuis un DataFrame. Il délague le travail d'affichage à Matplotlib en arrière-plan.
**Matplotlib / Pyplot (`plt`)** : Le moteur graphique de référence en Python. Il offre un contrôle absolu sur chaque pixel, chaque axe, chaque annotation et la structure globale de la figure.
**Seaborn (`sns`)** : Une bibliothèque construite *au-dessus* de Matplotlib, spécialisée dans la **visualisation statistique**. Elle gère nativement les regroupements complexes, les distributions et propose des designs modernes par défaut.

---

### Matplotlib et Pyplot (`plt`) : Le contrôle total sur la figure

Quand les options de Pandas montrent leurs limites (par exemple pour superposer des graphiques de types différents, ajouter des annotations textuelles complexes ou gérer des figures multi-axes sophistiquées), on fait appel directement à `matplotlib.pyplot`.

```python
import pandas as pd
import matplotlib.pyplot as plt

# Données d'exemple
Mois = ['Jan', 'Fév', 'Mar', 'Avr', 'Mai']
CA_2025 = [10000, 12000, 11000, 13500, 15000]
CA_2026 = [12500, 14000, 13800, 16000, 18500]

# Création explicite d'une figure et d'une zone de dessin (axes)
fig, ax = plt.subplots(figsize=(8, 4))

# Tracé de deux courbes superposées avec personnalisation fine
ax.plot(Mois, CA_2025, label='2025', color='gray', linestyle='--', marker='o')
ax.plot(Mois, CA_2026, label='2026', color='teal', linewidth=2.5, marker='s')

# Habillage professionnel via l'objet ax
ax.set(title="Comparatif du Chiffre d'Affaires N-1 vs N",
       xlabel="Mois",
       ylabel="Euros (€)")
ax.legend()
ax.grid(True, alpha=0.3)

plt.show()

```

---

### Seaborn (`sns`) : L'expert en statistiques et esthétique

Seaborn brille particulièrement lorsqu'il s'agit d'analyser des distributions statistiques ou de croiser plusieurs variables catégorielles sans avoir à manipuler laborieusement les index de Pandas. Il intègre nativement des thèmes visuels élégants et gère les regroupements complexes en un seul argument (`hue`, `col`, etc.).

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Application d'un thème Seaborn moderne
sns.set_theme(style="whitegrid")

# Données de test (répartition des prix par catégorie)
df_produits = pd.DataFrame({
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio', 'Moniteur', 'Moniteur', 'Accessoire', 'Audio'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0, 250.0, 300.0, 40.0, 90.0],
    'Canal': ['Web', 'Boutique', 'Web', 'Boutique', 'Web', 'Web', 'Boutique', 'Boutique']
})

# Génération d'un diagramme en boîte (boxplot) avec Seaborn
# Il gère automatiquement la répartition statistique (médiane, quartiles, valeurs aberrantes)
plt.figure(figsize=(8, 4))
sns.boxplot(
    data=df_produits, 
    x='Categorie', 
    y='Prix_HT', 
    hue='Canal',          # Regroupement secondaire instantané
    palette='Set2'
)

plt.title("Distribution des prix par catégorie et canal de vente")
plt.show()

```

---

### Synthèse : Quel outil choisir pour votre projet ?

| Outil | Niveau d'abstraction | Force principale | Cas d'usage idéal |
| --- | --- | --- | --- |
| **Pandas (`df.plot()`)** | Élevé (Intégré) | Rapidité d'exécution | Exploration de données rapide, graphiques simples intégrés à un script de nettoyage. |
| **Matplotlib (`plt`)** | Bas / Intermédiaire | Contrôle chirurgical | Rapports de production sur-mesure, annotations complexes, multi-graphiques avancés. |
| **Seaborn (`sns`)** | Intermédiaire (Statistique) | Esthétique moderne et inférence | Analyse exploratoire statistique, distributions, corrélations et graphiques multi-variables. |

## Le catalogue des représentations graphiques 

* **Le graphique linéaire (`kind='line'`)** : Suivi des tendances et des séries temporelles (ex: évolution du chiffre d'affaires au fil des mois).
* **Le diagramme en barres (`kind='bar'` / `barh`)** : Comparaison de catégories (ex: volume des ventes par catégorie de produits).
* **L'histogramme (`kind='hist'`)** : Analyse de la distribution statistique d'une variable numérique (ex: répartition des prix unitaires).
* **Le nuage de points (`kind='scatter'`)** : Croisement de deux variables pour identifier des corrélations (ex: Prix vs Quantité vendue).
* **Le diagramme circulaire (`kind='pie'`)** : Représentation des parts de marché (avec modération).

---

### Le DataFrame de référence pour la visualisation

```python
import pandas as pd
import matplotlib.pyplot as plt

# Création du DataFrame de test
df_visualisation = pd.DataFrame({
    'Date': pd.date_range(start='2026-10-01', periods=10, freq='D'),
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam', 'Tapis', 'Écran', 'Clavier', 'Souris', 'Casque', 'Webcam'],
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio', 'Accessoire', 'Moniteur', 'Accessoire', 'Accessoire', 'Audio', 'Audio'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0, 15.0, 250.0, 50.0, 25.0, 80.0, 70.0],
    'Quantite': [5, 12, 3, 8, 15, 2, 7, 10, 4, 6]
})

# Calcul d'une colonne dérivée pour l'analyse
df_visualisation['Chiffre_Affaires'] = df_visualisation['Prix_HT'] * df_visualisation['Quantite']

print(df_visualisation)

```

---

### Diagramme en barres (`kind='bar'`) : Comparer par catégorie

Idéal pour visualiser le chiffre d'affaires cumulé par grande famille de produits en couplant `groupby` et `plot()` :

```python
# Agréger le CA par catégorie et générer un graphique en barres
df_visualisation.groupby('Categorie')['Chiffre_Affaires'].sum().plot(
    kind='bar',
    color='skyblue',
    title="Chiffre d'Affaires par Catégorie",
    rot=0 # Garde les libellés horizontaux
)
plt.ylabel('Euros (€)')
plt.show()

```

### Graphique linéaire (`kind='line'`) : Suivre une série temporelle

Parfait pour observer l'évolution des ventes au fil des jours :

```python
# Indexer par date pour un rendu temporel propre
df_temporel = df_visualisation.set_index('Date')

# Tracer l'évolution de la quantité vendue par jour
df_temporel['Quantite'].plot(
    kind='line',
    marker='o',
    color='green',
    title="Évolution journalière des volumes vendus",
    figsize=(10, 4)
)
plt.ylabel('Quantité')
plt.show()

```

### Nuage de points (`kind='scatter'`) : Analyser une corrélation

Utile pour étudier la relation entre le prix unitaire et les quantités achetées :

```python
# Croiser le prix et la quantité
df_visualisation.plot(
    kind='scatter',
    x='Prix_HT',
    y='Quantite',
    color='purple',
    title="Relation Prix / Quantité vendue"
)
plt.show()

```
---
## Habillage, personnalisation et multi-graphiques

Un graphique brut généré par défaut est utile pour l'exploration rapide, mais un rapport professionnel exige un habillage rigoureux : des axes lisibles, des titres explicites, des couleurs adaptées et une mise en page soignée. Puisque Pandas s'appuie directement sur **Matplotlib** sous le capot, il est possible de personnaliser finement chaque rendu.

---

### Personnalisation des axes et des titres

Il est indispensable de nommer clairement les axes et d'ajouter un titre pour que le graphique soit compréhensible hors de son contexte. Ces paramètres peuvent être passés directement dans la méthode `.plot()` ou complétés via les fonctions de `matplotlib.pyplot`.

```python
import pandas as pd
import matplotlib.pyplot as plt

# Données d'exemple (ventes par jour)
df_ventes = pd.DataFrame({
    'Jour': ['Lun', 'Mar', 'Mer', 'Jeu', 'Ven'],
    'CA': [1200, 1500, 950, 1800, 2200]
}).set_index('Jour')

# Génération du graphique avec habillage intégré
df_ventes.plot(
    kind='bar',
    title="Chiffre d'affaires journalier",  # Titre principal
    rot=0                                  # Garde les libellés de l'axe X horizontaux
)

# Personnalisation explicite des axes et de la légende
plt.xlabel("Jours de la semaine")
plt.ylabel("Montant en Euros (€)")
plt.legend(["CA Global"])                  # Affichage et nom de la légende
plt.show()

```
---

### Mise en forme avancée : Dimensions, couleurs et styles

Pour adapter la lisibilité d'un graphique (notamment pour l'intégrer dans un support de cours ou un rapport PDF), on joue sur trois leviers majeurs :

* **`figsize=(largeur, hauteur)`** : Ajuste les proportions physiques de la figure en pouces.
* **`colormap`** : Applique une palette de couleurs cohérente (ex: `'viridis'`, `'plasma'`, `'tab10'`).
* **`linewidth` / `linestyle**` : Modifie l'épaisseur et le style des lignes pour les courbes.

```python
# Traçage d'une courbe stylisée
df_ventes.plot(
    kind='line',
    figsize=(9, 4),           # Figure large et moins haute
    color='teal',             # Couleur principale
    linewidth=3,              # Ligne épaisse
    marker='o',               # Ajout de points sur les sommets
    markersize=8,
    title="Suivi de la performance hebdomadaire"
)

plt.grid(True, linestyle='--', alpha=0.6) # Ajout d'une grille en pointillés légers
plt.ylabel("Euros (€)")
plt.show()

```

---

### Multi-subplots : Organiser plusieurs graphiques sur une grille

Lorsque l'on souhaite analyser plusieurs métriques en parallèle sans saturer un seul graphique, l'argument **`subplots=True`** ordonne à Pandas de créer un sous-graphique indépendant pour chaque colonne numérique. L'argument **`layout`** permet d'organiser ces graphiques sous forme de grille (lignes, colonnes).

```python
# DataFrame avec plusieurs métriques numériques
df_multimetriques = pd.DataFrame({
    'Date': pd.date_range(start='2026-10-01', periods=4, freq='D'),
    'Visiteurs': [150, 200, 180, 250],
    'Ventes': [12, 18, 15, 22],
    'Panier_Moyen': [50.0, 55.0, 48.0, 52.0]
}).set_index('Date')

# Création d'une grille de graphiques (2 lignes, 2 colonnes)
df_multimetriques.plot(
    kind='line',
    subplots=True,            # Active les graphiques multiples
    layout=(2, 2),            # Grille de 2x2
    figsize=(10, 6),          # Dimensions globales de la figure
    sharex=True,              # Partage l'axe des abscisses (dates)
    marker='.'
)

plt.suptitle("Tableau de bord multi-métriques", fontsize=14, fontweight='bold')
plt.tight_layout()            # Évite le chevauchement des sous-graphiques
plt.show()

```

* **Les libellés (`xlabel`, `ylabel`, `title`)** garantissent l'autonomie sémantique du graphique.
* **`figsize` et `colormap**` assurent l'harmonie visuelle et professionnelle de vos rapports.
* **`subplots=True` couplé à `layout**` transforme un DataFrame multi-colonnes en un tableau de bord analytique structuré en une seule ligne de code.

---
## Exemple de synthèse

## Exercices de fin de chapitre

**Exercice 1 :** Créez un DataFrame avec un `MultiIndex` comportant deux niveaux : "Ville" et "Mois", et remplissez-le avec des données de température fictives.
**Exercice 2 :** Utilisez `.loc` pour extraire toutes les données relatives à une seule ville de votre DataFrame hiérarchisé.
---

# Chapitre 16 : Étude de cas

Analyse complète d'un jeu de données (Jeux Olympiques de Paris 2024) : https://github.com/rythamsaini/Paris-Olympic-Games-2024

Cette étude de cas finale rassemble l'ensemble des compétences et des bonnes pratiques abordées dans ce manuel. De l'importation d'un fichier Excel brut jusqu'à la restitution visuelle des résultats, nous allons mener une analyse complète de données en conditions réelles, en prenant pour support un jeu de données simulant les résultats des **Jeux Olympiques de Paris 2024**.

Dans ce chapitre :

* Présentation du jeu de données et des questions métier
* Chargement et inspection
* Nettoyage et préparation
* Agrégations et croisements
* Visualisation et restitution des résultats
* Exemple de synthèse
* Exercices de fin de chapitre

---

## Présentation du jeu de données et des questions métier

Pour mener une analyse pertinente, il est indispensable de définir clairement le contexte et les objectifs avant d'écrire la moindre ligne de code.

Nous disposons d'un fichier Excel nommé `paris_2024_resultats.xlsx` contenant les performances des athlètes et des délégations. Le classeur est structuré autour de plusieurs informations clés : le nom de l'athlète, son pays d'origine, la discipline sportive, le type de médaille obtenue (Or, Argent, Bronze) et la date de l'épreuve.

La direction d'un institut d'analyse sportive souhaite obtenir des réponses précises aux questions suivantes :

**Quelles sont les nations qui dominent le tableau général des médailles ?**
**Quelle est la répartition des médailles (Or, Argent, Bronze) par grande discipline sportive ?**
**Existe-t-il des temps forts temporels dans l'attribution des médailles au fil des jours de compétition ?**

---

## Chargement et inspection

La première étape consiste à charger le fichier Excel en mémoire à l'aide de `pd.read_excel()` et à auditer sa structure pour identifier d'éventuelles anomalies.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Chargement de la feuille principale du fichier Excel
df_brut = pd.read_excel('paris_2024_resultats.xlsx', sheet_name='Performances')

# Inspection de la structure globale
print("--- Dimensions du DataFrame ---")
print(df_brut.shape)

print("\n--- Aperçu des premières lignes ---")
print(df_brut.head())

print("\n--- Diagnostic technique (.info) ---")
df_brut.info(memory_usage='deep')

```

---

## Nettoyage et préparation

Les données brutes comportent souvent des imperfections : espaces superflus dans les noms de pays, valeurs manquantes ou types de données mal interprétés. Nous allons nettoyer ce jeu de données.

```python
# A. Suppression des doublons stricts éventuels
df_propre = df_brut.drop_duplicates().copy()

# Nettoyage des chaînes de caractères (pays et disciplines)
df_propre['Pays'] = df_propre['Pays'].str.strip().str.title()
df_propre['Discipline'] = df_propre['Discipline'].str.strip()

# Gestion des valeurs manquantes (ex: absence de médaille remplacée par 'Aucune')
df_propre['Type_Medaille'] = df_propre['Type_Medaille'].fillna('Aucune')

# Conversion des colonnes catégorielles pour optimiser la mémoire
df_propre['Pays'] = df_propre['Pays'].astype('category')
df_propre['Discipline'] = df_propre['Discipline'].astype('category')

# Conversion de la date de l'épreuve au format datetime
df_propre['Date_Epreuve'] = pd.to_datetime(df_propre['Date_Epreuve'])

print("--- Données après nettoyage ---")
print(df_propre.head())

```

---

## Agrégations et croisements

Une fois le jeu de données assaini, nous pouvons répondre aux questions métier en mobilisant les outils d'agrégation (`groupby`, `agg`) et de croisement (`pivot_table`).

### Le tableau des médailles par pays

Pour identifier les nations les plus performantes, nous filtrons les médailles attribuées (Or, Argent, Bronze) et regroupons les données par pays.

```python
# Filtrer uniquement les lignes ayant rapporté une médaille
df_medailles = df_propre[df_propre['Type_Medaille'].isin(['Or', 'Argent', 'Bronze'])]

# Création d'un tableau croisé dynamique pour compter les médailles par type et par pays
tableau_medailles = df_medailles.pivot_table(
    index='Pays',
    columns='Type_Medaille',
    values='Athlete',
    aggfunc='count',
    fill_value=0
).reindex(columns=['Or', 'Argent', 'Bronze']) # Réordonner les colonnes logiquement

# Calcul du total des médailles par pays et tri décroissant
tableau_medailles['Total'] = tableau_medailles.sum(axis=1)
tableau_medailles = tableau_medailles.sort_values(by=['Or', 'Argent', 'Bronze', 'Total'], ascending=False)

print("--- Top 5 du Tableau des Médailles ---")
print(tableau_medailles.head(5))

```

---

## Visualisation et restitution des résultats

Pour valoriser les résultats auprès des équipes métiers, nous traduisons ce tableau de synthèse en un graphique visuel clair et soigné à l'aide de l'écosystème Matplotlib/Pandas.

```python
# Sélectionner le Top 5 des pays pour la visualisation
top_5_pays = tableau_medailles.head(5)[['Or', 'Argent', 'Bronze']]

# Génération d'un diagramme en barres empilées
ax = top_5_pays.plot(
    kind='bar',
    stacked=True,
    color=['gold', 'silver', 'peru'],
    figsize=(9, 5)
)

# Habillage professionnel du graphique
plt.title("Top 5 des Nations - Jeux Olympiques de Paris 2024", fontsize=14, fontweight='bold')
plt.xlabel("Pays", fontsize=11)
plt.ylabel("Nombre de Médailles", fontsize=11)
plt.xticks(rotation=0)
plt.legend(title="Type de Médaille")
plt.grid(axis='y', linestyle='--', alpha=0.6)

# Sauvegarde de la figure haute résolution
plt.savefig("top_5_medailles_paris2024.png", dpi=300, bbox_inches='tight')
plt.show()

```

---

## Exemple de synthèse 

Le script ci-dessous résume le pipeline complet de l'analyse, de l'importation à la génération du rapport chiffré.

```python
import pandas as pd
import matplotlib.pyplot as plt

def executer_etude_cas(chemin_fichier):
    # 1. Chargement
    df = pd.read_excel(chemin_fichier, sheet_name='Performances')
    
    # 2. Nettoyage
    df = df.drop_duplicates().copy()
    df['Pays'] = df['Pays'].str.strip().str.title()
    df['Type_Medaille'] = df['Type_Medaille'].fillna('Aucune')
    
    # 3. Traitement analytique (Top 3 pays par nombre total de médailles d'or)
    podiums = df[df['Type_Medaille'] == 'Or']
    compte_or = podiums.groupby('Pays')['Type_Medaille'].count().reset_index(name='Total_Or')
    top_or = compte_or.sort_values(by='Total_Or', ascending=False).head(3)
    
    print("--- Résumé de l'analyse : Top 3 Or ---")
    print(top_or)
    
    return top_or

# Simulation d'appel (nécessite un fichier valide)
# executer_etude_cas('paris_2024_resultats.xlsx')

```

* **Méthodologie rigoureuse** : Toujours suivre les étapes séquentielles (Inspection $\rightarrow$ Nettoyage $\rightarrow$ Transformation $\rightarrow$ Restitution).
* **Sécurité des types** : L'utilisation de `.astype('category')` et du typage date sécurise les calculs futurs et allège l'empreinte mémoire.
* **Clarté de la restitution** : Un graphique propre doté d'un habillage explicite (titres, axes, légende) transforme des chiffres bruts en un outil d'aide à la décision percutant.

---

## Exercices de fin de chapitre

**Exercice 1 :** Analyse par discipline sportive

À l'aide du DataFrame nettoyé `df_propre` :

Réalisez un groupement (`groupby`) sur la colonne `Discipline` pour compter le nombre total d'épreuves disputées.
Identifiez la discipline qui a mobilisé le plus grand nombre d'athlètes uniques.

**Exercice 2 :** Suivi temporel des performances

Convertissez la colonne `Date_Epreuve` en index temporel (`DatetimeIndex`).
Utilisez un regroupement par date (`resample` ou `groupby` sur la date) pour tracer l'évolution quotidienne du nombre total de médailles décernées au cours de la quinzaine olympique.

# Chapitre 17 : Cheatsheet NumPy / Pandas / Matplotlib

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

---

## 1. NumPy : le socle

| Besoin | Code |
| --- | --- |
| Créer un tableau | `np.array([1, 2, 3], dtype=np.float64)` |
| Forme / type | `a.shape` · `a.dtype` · `a.ndim` |
| Séquence | `np.arange(0, 10, 2)` · `np.linspace(0, 1, 50)` |
| Slicing 1D | `a[0:3]` · `a[::2]` (fin exclue) |
| Slicing 2D | `m[2, 0]` · `m[:, 1]` · `m[1, :]` |
| Vectorisation | `a * 1.2` (aucune boucle `for`) |
| Broadcasting | `m + v[:, np.newaxis]` |
| Réduction | `np.sum(m, axis=0)` · `np.mean(m, axis=1)` |
| NaN | `np.nan` · `np.isnan(a)` · `a[~np.isnan(a)]` |
| Sans NaN | `np.nansum(a)` · `np.nanmean(a)` · `np.nan_to_num(a, nan=0)` |

> **`axis=0`** ↓ descend les lignes (résultat **par colonne**)  
> **`axis=1`** → traverse les colonnes (résultat **par ligne**)  
> ⚠️ `nan` est contagieux : `np.sum([1, np.nan]) → nan`

---

## 2. Series

Une `Series` = **valeurs** (ndarray) + **index** (labels).

```python
pd.Series([10, 20], index=['a', 'b'])      # liste + index
pd.Series({'FR': 'Paris', 'IT': 'Rome'})   # dict -> clés = index
pd.Series(25, index=['L', 'M', 'Me'])      # scalaire diffusé
```

| Inspection | Rôle |
| --- | --- |
| `.values` · `.index` | données brutes · labels |
| `.shape` · `.size` · `.dtype` | `(n,)` · nb d'éléments · type |
| `.head(n)` · `.tail(n)` | n premières / dernières valeurs |
| `.describe()` | résumé statistique |

### Sélection

| Méthode | Base | Fin du slice |
| --- | --- | --- |
| `s.loc['a':'c']` | **labels** | **incluse** |
| `s.iloc[0:3]` | **positions** | exclue |
| `s[s > 1]` | masque booléen | — |
| `s.loc[['c', 'a']]` | liste de labels (ordre libre) | — |

### Arithmétique et alignement

```python
s1 + s2                      # aligné sur les LABELS, NaN si absent
s1.add(s2, fill_value=0)     # remplace l'absent par 0 avant calcul
# méthodes : .add .sub .mul .div .pow
```

### Index

```python
s.index = ['A', 'B', 'C']              # remplacement global (même taille)
s.rename({'B': 'Delta'})               # renommage ciblé
s.reindex(['A', 'C', 'Z'], fill_value=0)  # réaligne, crée les manquants
```

### Transformer

```python
s * 9/5 + 32                                  # vectorisé (le plus rapide)
s.apply(lambda x: 'Chaud' if x > 22 else 'Frais')  # logique perso
s.map({'H': 'Hivernal', 'P': 'Tempéré'})       # dict ; absent -> NaN
s.astype('int64')  # 'float64' | str | bool | 'category'
```

---

## 3. DataFrame : création et structure

```python
pd.DataFrame({'Nom': ['A', 'B'], 'Age': [25, 30]})          # dict de listes
pd.DataFrame([('Paris', 2.1), ('Lyon', 0.5)], columns=['Ville', 'Pop'])  # tuples
pd.DataFrame([{'ID': 1, 'N': 'A'}, {'ID': 2}])             # liste de dicts (NaN auto)
pd.DataFrame({'X': ser1, 'Y': ser2})                        # Series alignées
```

| Attribut | Rôle |
| --- | --- |
| `.shape` · `.size` · `.ndim` | `(lignes, colonnes)` · nb cellules · 2 |
| `.index` · `.columns` · `.axes` | labels lignes · colonnes · les deux |
| `.dtypes` | type par colonne |
| `.values` / `.to_numpy()` | retour en ndarray |
| `.info(memory_usage='deep')` | types, non-null, mémoire |
| `.describe(include='all')` | stats (num + texte) |
| `.head()` · `.tail()` | aperçu |

> Une colonne `df['col']` est une **Series**. Axes : `axis=0` lignes, `axis=1` colonnes.

### Import

```python
pd.read_csv('f.csv', sep=';', encoding='utf-8', index_col='ID',
            na_values=['NA', '?'], parse_dates=['date'])
pd.read_excel('f.xlsx', sheet_name='Q1', skiprows=2, usecols='A:D')
```

---

## 4. Colonnes

```python
df['TTC'] = df['HT'] * 1.2             # nouvelle colonne calculée
df['X'] = np.nan                       # colonne vide
df[['Produit', 'TTC']]                 # sélection + ordre
df.rename(columns={'HT': 'Prix_HT'})   # renommer
df.drop(columns=['Total'])             # supprimer (= axis=1)
df.filter(like='Prix')                 # colonnes dont le NOM contient 'Prix'
df.filter(items=['A', 'B'])            # colonnes par nom exact
```

## 5. Lignes et filtrage

```python
df[df['Score'] > 80]                              # masque booléen
df[(df['Score'] > 70) & (df['Ville'] == 'Paris')]  # ET  (parenthèses obligatoires)
df[(df['A'] > 1) | (df['B'] < 5)]                  # OU
df[~(df['A'] > 1)]                                 # NON
df[df['Ville'].isin(['Paris', 'Lyon'])]            # appartenance
df.query("Prix > 50 and Stock > 0")                # syntaxe texte
df.loc[4] = ['Eve', 95, 'Bordeaux']                # ajouter une ligne
df.drop(index=[0, 3])                              # supprimer des lignes
```

## 6. Index, `.loc` et `.iloc`

```python
df.set_index('Code')       # colonne -> index
df.reset_index()           # index -> colonne
df.rename(index={'E1': 'X1'})
```

| Besoin | `.loc` (labels, fin incluse) | `.iloc` (positions, fin exclue) |
| --- | --- | --- |
| Une cellule | `df.loc['Alice', 'Score']` | `df.iloc[1, 0]` |
| Une ligne | `df.loc['Bob']` | `df.iloc[0]` |
| Tranche | `df.loc['A':'B', 'Score':'Ville']` | `df.iloc[0:2, 0:1]` |
| Filtre + colonne | `df.loc[df['Score'] > 80, 'Ville']` | — |

---

## 7. Tri, itération, apply, pipe

```python
df.sort_values('Total', ascending=False)
df.sort_values(['Qte', 'Prix'], ascending=[False, True])   # multi-critères
df.sort_index()

df.at[idx, 'Score'] += 1               # accès cellule rapide (dans une boucle)
for i, row in df.iterrows(): ...       # lent : préférer la vectorisation

df['X'].apply(lambda x: x.upper())                       # par valeur
df.apply(lambda r: f"{r['A']}-{r['B']}", axis=1)         # par ligne
df.pipe(f1, taux=0.1).pipe(f2, seuil=190)                # chaîner des fonctions

df2 = df.copy()                        # copie indépendante
df[df['Q'] > 2].copy()                 # évite le SettingWithCopyWarning
```

| Outil | S'applique à | Usage |
| --- | --- | --- |
| `.map(dict)` | Series | codes → libellés (absent = NaN) |
| `.apply(f)` | Series / DataFrame | fonction perso (`axis=1` = par ligne) |
| `.pipe(f)` | DataFrame | pipeline lisible |
| `df.map(f)` | DataFrame | cellule par cellule (ex-`applymap`) |

---

## 8. Nettoyage

### Valeurs manquantes

```python
df.isna().sum()                    # nb de NaN par colonne
df.notna()                         # masque inverse
df.dropna()                        # supprime les lignes avec NaN
df.fillna({'Prix': df['Prix'].mean(), 'Stock': 0})
s.interpolate()                    # estimation linéaire (séries temporelles)
s.mask(s <= 0, np.nan)             # remplace selon condition
```

### Doublons et aberrants

```python
df.duplicated()                                  # masque des doublons
df.drop_duplicates(subset=['ID'], keep='first')  # keep: 'first' | 'last'
df[(df['Prix'] > 0) & (df['Prix'] < 5000)]       # seuil métier
s.quantile(0.05) ; s.quantile(0.95)              # bornes par quantiles
```

### Texte avec `.str`

```python
df['E'].str.strip().str.lower()
df['V'].str.strip().str.capitalize()    # .upper() .title() .contains() .split()
df['M'].str.replace('€', '', regex=False).str.strip().astype(int)
```

### Type `category`

```python
df['Cat'] = df['Cat'].astype('category')   # très gros gain mémoire si répétitif
```

---

## 9. Dates et temps

| Objet | Rôle | Exemple |
| --- | --- | --- |
| `Timestamp` | un instant | `pd.Timestamp('2026-10-28 14:30')` |
| `Timedelta` | une durée | `pd.Timedelta(days=10, hours=4)` |
| `Period` | une période calendaire | `pd.Period('2026-10', freq='M')` |
| `DatetimeIndex` | index de dates | `pd.date_range(...)` |

```python
pd.to_datetime(df['d'])                                  # texte -> datetime
pd.date_range('2026-10-01', periods=7, freq='D')         # jours
pd.date_range('2026-01-01', periods=12, freq='MS')       # 1er du mois
pd.period_range('2026-01', periods=6, freq='Q')          # trimestres
s.loc['2026-10-02':'2026-10-04']                         # slicing par date
s.loc['2026-10']                                         # tout un mois
```

| Accesseur `.dt` | Donne |
| --- | --- |
| `.dt.year` `.dt.month` `.dt.day` | année, mois, jour |
| `.dt.hour` `.dt.minute` | heure, minute |
| `.dt.day_name()` `.dt.month_name()` | noms en toutes lettres |

> Sur un `DatetimeIndex` : `df.index.month`, `df.index.day_name()` (sans `.dt`).  
> Fréquences : `D` jour · `MS` début de mois · `M` fin de mois (`ME` en pandas ≥ 2.2) · `Q` trimestre · `Y` année.

---

## 10. Statistiques et `groupby`

```python
df['x'].mean()  # .sum() .min() .max() .std() .median() .count() .idxmax()
df.loc[df['Total'].idxmax(), 'Produit']     # ligne du maximum

df.agg('sum')                                # une fonction
df.agg(['mean', 'std'])                      # plusieurs
df.agg({'Prix': ['mean', 'max'], 'Qte': 'sum'})   # par colonne
```

### Split → Apply → Combine

```python
df.groupby('Cat')['Total'].sum()

df.groupby('Cat').agg(                       # agrégation nommée
    Prix_Moyen=('Prix_HT', 'mean'),
    Volume=('Quantite', 'sum'),
    CA=('Total', 'sum'),
).reset_index()                              # index -> colonne
```

---

## 11. Fusion de données

| Outil | Lie par | Cas d'usage |
| --- | --- | --- |
| `pd.concat([a, b], ignore_index=True)` | position (axis=0 lignes) | empiler des fichiers mensuels |
| `pd.concat([a, b], axis=1)` | index commun | juxtaposer des colonnes |
| `pd.merge(a, b, on='ID', how='left')` | **colonnes** clés | relier ventes ↔ catalogue |
| `a.join(b)` | **index** | tables déjà indexées pareil |

| `how=` | Garde |
| --- | --- |
| `inner` (défaut) | clés présentes des deux côtés |
| `left` / `right` | toutes les lignes de gauche / droite |
| `outer` | toutes les lignes des deux tables |

Clés de noms différents : `pd.merge(a, b, left_on='x', right_on='y')`

---

## 12. Structures multidimensionnelles

```python
pd.MultiIndex.from_tuples([('Paris', 'Q1'), ('Paris', 'Q2')], names=['Ville', 'Trim'])
pd.MultiIndex.from_product([villes, mois], names=['Ville', 'Mois'])

df.loc['Paris']                       # un niveau
df.loc[('Paris', 'Q1'), 'CA']         # tuple pour plusieurs niveaux
df.loc[pd.IndexSlice[:, 'Q1'], :]     # tranche complexe
df.sort_index()                       # tri requis avant slicing
```

| Fonction | Rôle |
| --- | --- |
| `df.pivot(index, columns, values)` | pivote **sans agrégation** (doublons = erreur) |
| `df.pivot_table(index, columns, values, aggfunc='sum', fill_value=0)` | pivote **avec agrégation** |
| `pd.crosstab(df['A'], df['B'])` | table de fréquences |
| `df.stack()` | colonnes → lignes |
| `df.unstack()` | lignes → colonnes |
| `df.melt(id_vars=['X'], var_name='V', value_name='Val')` | large → long |

---

## 13. Export / persistance

| Format | Lecture | Écriture |
| --- | --- | --- |
| CSV | `pd.read_csv()` | `df.to_csv('f.csv', index=False, sep=';', encoding='utf-8-sig')` |
| JSON | `pd.read_json(orient='records')` | `df.to_json('f.json', orient='records', force_ascii=False, indent=4)` |
| XML | `pd.read_xml(xpath='.//item')` | `df.to_xml('f.xml', root_name='R', row_name='L', index=False)` |
| Excel | `pd.read_excel()` | `df.to_excel('f.xlsx', index=False)` |
| Pickle | `pd.read_pickle()` | `df.to_pickle('f.pkl')` (types conservés, propre à Python) |

> ✅ `index=False` sauf index porteur de sens · `utf-8-sig` pour Excel FR

---

## 14. Visualisation

### Pandas `df.plot()` (prototypage rapide)

| `kind=` | Usage |
| --- | --- |
| `line` | tendance, séries temporelles |
| `bar` / `barh` | comparer des catégories |
| `hist` | distribution d'une variable |
| `scatter` (`x=`, `y=`) | corrélation entre 2 variables |
| `pie` | parts (avec modération) |

```python
df.groupby('Cat')['CA'].sum().plot(kind='bar', color='skyblue', rot=0, title='CA')
df.plot(kind='line', subplots=True, layout=(2, 2), figsize=(10, 6), sharex=True)
```

Options : `figsize=(l, h)` · `color` · `colormap` · `linewidth` · `marker` · `rot` · `title` · `stacked=True`

### Matplotlib (contrôle total)

```python
fig, ax = plt.subplots(figsize=(8, 4))              # Figure + Axes
ax.plot(x, y, label='2026', color='teal', linestyle='--', marker='o', linewidth=2)
ax.set(title='Titre', xlabel='X', ylabel='Y')
ax.legend(loc='upper right'); ax.grid(True, alpha=0.3)

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(10, 4))   # grille 1×2
fig, axes = plt.subplots(2, 2)                          # axes[0, 1]...
plt.tight_layout()
plt.savefig('fig.png', dpi=300, bbox_inches='tight')    # PNG / .svg / .pdf
plt.show(); plt.close()
plt.style.use('ggplot')
```

### Seaborn (statistiques)

```python
import seaborn as sns
sns.set_theme(style='whitegrid')
sns.boxplot(data=df, x='Cat', y='Prix', hue='Canal', palette='Set2')
```

| Outil | Force |
| --- | --- |
| `df.plot()` | rapide, intégré |
| Matplotlib | contrôle chirurgical |
| Seaborn | stats + esthétique, `hue` |

---

## 15. Pipeline type d'analyse

```
Charger  →  Inspecter  →  Nettoyer  →  Transformer  →  Agréger  →  Visualiser
read_*      shape/info     dropna       colonnes       groupby     plot / savefig
            describe       drop_dup     map/apply      pivot_table
                           .str/astype  merge/concat
```

```python
df = pd.read_excel('f.xlsx').drop_duplicates().copy()
df['Pays'] = df['Pays'].str.strip().str.title().astype('category')
df['Date'] = pd.to_datetime(df['Date'])
top = df.groupby('Pays').agg(Total=('Medaille', 'count')).nlargest(5, 'Total')
top.plot(kind='bar'); plt.savefig('top5.png', dpi=300, bbox_inches='tight')
```

---

## ⚠️ Pièges à retenir

| Piège | Réflexe |
| --- | --- |
| `.loc[a:b]` inclut `b`, `.iloc[a:b]` l'exclut | choisir l'outil selon labels / positions |
| `&` `|` `~` sans parenthèses | `(cond1) & (cond2)` |
| `SettingWithCopyWarning` | `.copy()` après un filtre |
| `NaN` contagieux (NumPy) | `nansum`, `nanmean` · Pandas ignore NaN par défaut |
| `int64` ne supporte pas `NaN` | `.astype('float64')` d'abord |
| `.map(dict)` renvoie NaN si clé inconnue | prévoir un `fillna` ou `.replace()` |
| `concat` garde les anciens index | `ignore_index=True` |
| Boucles `iterrows` lentes | vectoriser, sinon `.apply` |
| Accents illisibles sous Excel | `encoding='utf-8-sig'`, `sep=';'` |
| `df.sort_values` / `drop` / `fillna` ne modifient pas sur place | réassigner : `df = df.drop(...)` |