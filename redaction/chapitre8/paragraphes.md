# Chapitre 8: Nettoyage et Préparation des Données

Le nettoyage des données (*Data Cleaning*) est souvent l'étape la plus chronophage et la plus cruciale de tout projet en science des données. Un modèle d'apprentissage ou une analyse statistique ne vaut que par la qualité des données qui l'alimentent. Les fichiers bruts contiennent fréquemment des valeurs manquantes, des doublons, des fautes de frappe textuelles ou des valeurs aberrantes.

Ce chapitre vous présente les outils et méthodes indispensables sous Pandas pour transformer un jeu de données brut et chaotique en un tableau propre, structuré et prêt pour l'analyse.

Dans ce chapitre :

* Détecter les données invalides (`NaN`, `isna()`, `notna()`)
* Traiter les valeurs manquantes : `fillna()`, `dropna()`, `interpolate()`
* Détecter et supprimer les doublons : `duplicated()`, `drop_duplicates()`
* Repérer les valeurs aberrantes (Outliers)
* Nettoyer les chaînes de caractères avec l'accesseur `.str`
* Le type `category` : gain de mémoire et de performance
* Exemple de synthèse :
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
