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

Création d'un tableau unidimensionnel et inspection de sa structure

```python
import numpy as np

# Création d'un tableau 1D à partir d'une liste Python
notes_array = np.array([12, 15, 18, 14])

print("Tableau :", notes_array)
print("Type des données (.dtype) :", notes_array.dtype)  # Affiche int64 (ou int32 selon l'archi)
print("Dimensions (.shape) :", notes_array.shape)        # Affiche (4,) -> 4 éléments sur 1 dimension

```

Création d'une matrice bidimensionnelle avec typage explicite

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
