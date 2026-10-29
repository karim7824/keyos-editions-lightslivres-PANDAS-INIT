# Chapitre 9 : Jointures et fusion de données

La fusion de données est une opération fondamentale pour consolider des informations provenant de sources multiples au sein d'un même projet. Ce chapitre détaille comment combiner vos structures de données pour obtenir une vision unifiée et complète.

Dans ce chapitre :

* Concaténation de structures
* `pd.merge()` : Le couteau suisse des jointures (Basé sur des colonnes ou des clés)
* La méthode `.join()` : Le spécialiste des index
* Exemple de synthèse
* Exercices de fin de chapitre

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
