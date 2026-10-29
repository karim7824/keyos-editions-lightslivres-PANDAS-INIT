# Chapitre 13 : Gestion des structures multidimensionnelles

L'analyse de données complexes nécessite souvent de manipuler des dimensions supplémentaires au-delà du simple tableau bidimensionnel. Qu'il s'agisse de croiser des informations géographiques, temporelles ou organisationnelles, Pandas propose des outils puissants pour structurer, indexer et remodeler vos données de manière hiérarchique.

Dans ce chapitre :

* Introduction aux Panel Data
* Utilisation du `MultiIndex`
* Sélection et tri sur un `MultiIndex`
* Tableaux croisés : `pivot()`, `pivot_table()` et `crosstab()`
* Remodeler les données : `stack()`, `unstack()` et `melt()`
* Exemple de synthèse : Analyse multidimensionnelle e-commerce
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
