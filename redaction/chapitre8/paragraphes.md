# Chapitre 8 : Persistance et formats de fichiers

La sauvegarde et le chargement de vos données sont cruciaux pour assurer la pérennité de votre travail. Ce chapitre explore les méthodes pour importer et exporter vos DataFrames dans différents formats, garantissant ainsi leur réutilisation future.

Dans ce chapitre :

* Lecture et écriture de fichiers CSV
* Persistance des données via le format Pickle

## Lecture et écriture de fichiers CSV

Le format CSV (Comma Separated Values) est le standard pour l'échange de données tabulaires. Pandas facilite cette étape avec des fonctions optimisées pour lire un fichier texte structuré ou pour sauvegarder votre DataFrame actuel sous forme de fichier partageable.

```python
import pandas as pd

# Sauvegarde d'un DataFrame en fichier CSV
df.to_csv('mon_fichier.csv', index=False)

# Chargement d'un fichier CSV dans un DataFrame
df_charge = pd.read_csv('mon_fichier.csv')

```

## Persistance des données via le format Pickle

Le format Pickle permet de sérialiser des objets Python, ce qui signifie que vous pouvez sauvegarder un DataFrame exactement tel qu'il est, en conservant les types de données complexes et la structure des index. Cette méthode est idéale pour stocker des données temporaires entre deux sessions de travail sans perte d'information.

```python
# Sauvegarde au format binaire Pickle
df.to_pickle('donnees.pkl')

# Chargement du fichier Pickle
df_recharge = pd.read_pickle('donnees.pkl')

```

### Synthèse

La persistance des données repose sur le CSV pour l'interopérabilité et le format Pickle pour la préservation intégrale des structures complexes lors de vos sessions de travail.

### Exercices

1. Créez un DataFrame, sauvegardez-le en format CSV, puis rechargez-le dans une nouvelle variable pour vérifier l'intégrité des données.
2. Expérimentez la sauvegarde d'un DataFrame contenant des types de données variés (nombres, textes, dates) au format Pickle et confirmez que les types sont conservés après le rechargement.

