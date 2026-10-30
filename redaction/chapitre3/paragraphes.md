# Chapitre 3 : Manipuler les Series

Dans ce chapitre :

* Opérations arithmétiques et alignement automatique
* Extraction et recherche de données
* Travailler avec les index d'une `Series`
* Transformation des valeurs d'une `Series`
* Exemple de synthèse
* Exercices de fin de chapitre

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

**`.iloc` (pour *Integer Location*) :**
* Fonctionne exclusivement avec des **positions entières** (de `0` à `N-1`), exactement comme une liste Python classique ou un tableau NumPy.
* Le borne de fin est exclusive lors d'un découpage (slicing).

**`.loc` (pour *Location*) :**
* Fonctionne exclusivement avec les **étiquettes (labels)** de l'index.
* Le borne de fin est inclusive lors d'un découpage (contrairement aux habitudes de Python), ce qui est très pratique pour cibler explicitement une plage nommée.

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

## Exercices de fin de chapitre

**Excerice 1 :** Extraction d'une période spécifique et transformation ciblée des valeurs. 

Dans cet exemple, nous disposons d'un suivi journalier du trafic d'un site web. Nous souhaitons **extraire** uniquement la semaine où s'est déroulée une campagne promotionnelle, puis **transformer** ces données en leur appliquant un coefficient de correction lié à une anomalie de comptage.

**Excerice 2 :** Extraction de composantes temporelles (.dt) et transformation conditionnelle. 

Cet exemple illustre comment extraire des informations sémantiques à partir des dates (les jours de la semaine) pour ensuite appliquer une transformation conditionnelle (par exemple, appliquer un tarif week-end ou jour ouvré).
