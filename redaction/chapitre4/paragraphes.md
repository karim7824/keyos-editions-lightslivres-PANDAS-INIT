# Chapitre 4 : Nettoyage et préparation (Data Wrangling)

La préparation des données, souvent appelée "Data Wrangling", est une étape cruciale pour garantir la fiabilité de vos analyses. Ce chapitre aborde les techniques essentielles pour traiter les valeurs manquantes et transformer les données textuelles au sein de vos DataFrames.

Dans ce chapitre :

* Gestion des données invalides (`NaN`)
* Manipulation de texte avec l'objet `.str`

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

## Manipulation de texte avec l'objet `.str`

L'accesseur `.str` permet d'appliquer des méthodes de traitement de chaîne de caractères sur une colonne entière de type texte. Cette API dédiée simplifie considérablement le nettoyage et la normalisation des données textuelles comme le passage en majuscules ou le remplacement de motifs.

```python
# Création d'une série de noms
noms = pd.Series(['alice', 'bob', 'charlie'])

# Conversion en majuscules et remplacement d'une lettre
noms_maj = noms.str.upper()
noms_remplace = noms.str.replace('a', 'o')

```

### Synthèse

Le nettoyage des données repose sur la gestion rigoureuse des valeurs manquantes via `dropna` ou `fillna` , ainsi que sur l'utilisation de l'API `.str` pour harmoniser et transformer efficacement les colonnes textuelles.

### Exercices

1. Créez un DataFrame contenant des valeurs `None` et utilisez `fillna` pour les remplacer par la moyenne de la colonne.
2. À partir d'une colonne contenant des prénoms en minuscules, utilisez l'accesseur `.str` pour les mettre en majuscules et extraire uniquement les trois premières lettres de chaque prénom.