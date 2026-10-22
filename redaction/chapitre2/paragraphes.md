# Chapitre 2 : Création et manipulation de DataFrames

Le DataFrame est la structure de données pivot de Pandas, permettant de manipuler des tableaux bidimensionnels de manière flexible et efficace. Ce chapitre détaille comment structurer, créer et extraire des données au sein de cet environnement tabulaire.

Dans ce chapitre :

* Comparaison entre ndarray et DataFrame
* Organisation interne et propriétés
* Création à partir de listes, tuples et dictionnaires
* Importation depuis CSV et Excel
* Manipulation des colonnes, lignes et index
* Techniques d'extraction de données

## ndarray vs DataFrame

La différence fondamentale réside dans l'indexation : alors qu'un `ndarray` (NumPy) se base uniquement sur des positions entières, un `DataFrame` associe un index aux lignes et des étiquettes (noms) aux colonnes. Cette structure permet un accès sémantique aux données, facilitant leur interprétation.

```python
import pandas as pd
import numpy as np

# Un ndarray requiert la connaissance de l'index entier
arr = np.array([[1, 2], [3, 4]])

# Le DataFrame permet de nommer les colonnes pour une lisibilité accrue
df = pd.DataFrame(arr, columns=['A', 'B'])

```

## Organisation interne et propriétés

Un DataFrame est organisé en un ensemble de colonnes, chacune étant techniquement une `Series`. Ses propriétés principales incluent `.shape` (dimensions), `.columns` (noms des colonnes) et `.index` (identifiants des lignes), qui définissent sa structure et ses métadonnées.

```python
# Affichage des propriétés structurelles
print(df.shape)    # Retourne le tuple (lignes, colonnes)
print(df.columns)  # Liste des noms de colonnes

```

## Créations multiples : listes, tuples, dictionnaires

La création d'un DataFrame est flexible et s'adapte à la structure initiale de vos données. Vous pouvez transformer des listes ou des tuples en colonnes, tandis que les dictionnaires sont idéaux pour définir explicitement le nom des colonnes.

```python
# Création depuis des listes/tuples
df_liste = pd.DataFrame([[1, 2], [3, 4]], columns=['Col1', 'Col2'])

# Création depuis un dictionnaire (plus lisible)
df_dict = pd.DataFrame({'Nom': ['Alice', 'Bob'], 'Score': [85, 90]})

```

## Créations multiples : CSV et Excel

Pandas propose des fonctions dédiées pour charger directement des données externes stockées dans des fichiers. Ces outils automatisent la conversion des types et la reconnaissance des en-têtes.

```python
# Importation de fichiers externes
# df_csv = pd.read_csv('donnees.csv')
# df_excel = pd.read_excel('donnees.xlsx')

```

## Travailler sur les colonnes

Manipuler les colonnes permet d'ajouter de nouvelles variables, de supprimer celles inutiles ou d'effectuer des calculs par colonne. Les opérations sur les colonnes sont généralement vectorisées, assurant une haute performance.

```python
# Ajout d'une colonne calculée
df['Nouveau_Score'] = df['Score'] + 5

# Suppression d'une colonne
df = df.drop(columns=['Score'])

```

## Travailler sur les lignes

La manipulation des lignes consiste à filtrer, ajouter ou supprimer des enregistrements spécifiques. Cela est souvent effectué via des conditions booléennes qui sélectionnent les lignes répondant à des critères précis.

```python
# Filtrage des lignes où Score > 80
selection = df[df['Score'] > 80]

```

## Travailler sur les labels et index

Modifier l'index ou manipuler les étiquettes permet d'organiser les données selon un identifiant métier plutôt qu'une position. Cela améliore la précision lors des recherches et des jointures.

```python
# Définir une colonne comme index
df = df.set_index('Nom')

```

## Extraction de données

L'extraction utilise `.loc` pour les labels et `.iloc` pour les positions entières. Ces méthodes permettent d'isoler des valeurs, des tranches de données ou des sous-ensembles complexes en une seule commande.

```python
# Extraction par étiquette (.loc) et position (.iloc)
valeur = df.loc['Alice', 'Score']
premiere_ligne = df.iloc[0]

```

### Synthèse

La puissance du DataFrame réside dans sa structure hybride alliant indexation flexible et méthodes d'importation robustes. La maîtrise des accès par labels ou positions est indispensable pour toute analyse de données.

### Exercices

1. Importez un petit fichier CSV de votre choix et affichez les noms des colonnes à l'aide de l'attribut approprié.
2. Créez un DataFrame à partir d'un dictionnaire, puis utilisez `.iloc` pour extraire uniquement la première colonne de toutes les lignes.