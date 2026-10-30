# Chapitre 2 : Présentation des Series 

Les `Series` constituent la structure de données unidimensionnelle fondamentale de Pandas, conçue pour stocker et manipuler efficacement des séquences de valeurs. Si elles s'apparentent de prime abord à un tableau unidimensionnel, elles se distinguent radicalement des `ndarray` de NumPy par un concept clé : **l'indexation explicite par étiquettes (labels)**.

Dans ce chapitre :

* Comparaison entre structures ndarray et Series
* Méthodes de création d'une `Series`
* Attributs et méthodes d'inspection d'une `Series`
* Les `Series` temporelles : gestion des dates et du temps
* La conversion de type dans une `Series` (`.astype()`)
* Exemple de synthèse
* Exercices de fin de chapitre

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

Points clés à retenir :
* **L'indexation sémantique et temporelle** permet de manipuler les données par date de manière intuitive.
* **L'alignement automatique et `.reindex()`** évitent les erreurs lors de la manipulation de séries asymétriques ou incomplètes.
* **La vectorisation** permet d'appliquer instantanément des calculs mathématiques globaux sans utiliser de boucles `for`.
* **Le filtrage booléen** isole en une seule ligne les données répondant à un critère métier précis.

```
## Exercices de fin de chapitre

**Exercice 1 :** Gestion d'un catalogue de prix et conversion de types

Vous récupérez un extrait de données textuelles provenant d'un ancien système informatique. Les prix des articles sont stockés sous forme de chaînes de caractères, et vous devez les nettoyer, les convertir pour effectuer des calculs, puis structurer le tout dans une `Series` Pandas dotée d'un index explicite.

Partez des listes de données brutes suivantes en Python :
```python
import pandas as pd

articles = ['Casque Audio', 'Clavier Gamer', 'Souris Sans Fil', 'Tapis de souris']
prix_texte = ['120.50', '85.00', '45.99', '19.99']

```

Créez une `Series` nommée `ser_prix` en utilisant la liste `prix_texte` pour les valeurs et la liste `articles` pour l'index explicite. Affichez son type initial (`dtype`).
À l'aide de la méthode `.astype()`, convertissez cette `Series` en nombres à virgule flottante (`float64`).
Calculez et affichez le prix de chaque article après application d'une augmentation générale de 10 % (multiplication vectorisée par `1.10`).

---

**Exercice 2 :** Analyse temporelle des ventes d'une boutique

Une boutique en ligne souhaite analyser son volume de commandes quotidiennes sur une période de quatre jours au début du mois de novembre 2026. Vous devez manipuler un index temporel et extraire des informations calendaires.

À l'aide de la fonction `pd.date_range()`, générez une plage de 4 jours consécutifs commençant le **1er novembre 2026** (`2026-11-01`).
Créez une `Series` nommée `commandes` contenant les valeurs `[15, 32, 28, 45]` en associant l'index temporel généré à l'étape précédente.
Utilisez le découpage (*slicing*) temporel par texte pour extraire et afficher uniquement les données allant du **2 novembre 2026 au 3 novembre 2026**.
En utilisant l'accesseur temporel `.dt` sur l'index de votre série d'origine, extrayez et affichez le nom complet des jours de la semaine correspondants (`day_name()`).
---
