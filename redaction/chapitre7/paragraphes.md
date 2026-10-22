# Chapitre 7 : Jointures et fusion de données

La fusion de données est une opération fondamentale pour consolider des informations provenant de sources multiples au sein d'un même projet. Ce chapitre détaille comment combiner vos structures de données pour obtenir une vision unifiée et complète.

Dans ce chapitre :

* Concaténation de structures de données (`concat`)
* Jointures complexes (`merge`, `join`)

## Concaténation de structures

La fonction `concat` permet d'assembler des DataFrames ou des Séries les uns aux autres, soit verticalement (en ajoutant des lignes), soit horizontalement (en ajoutant des colonnes). C'est l'outil idéal pour regrouper des jeux de données de structure identique.

```python
import pandas as pd

# Concaténation verticale de deux DataFrames
df1 = pd.DataFrame({'A': [1, 2]})
df2 = pd.DataFrame({'A': [3, 4]})
resultat = pd.concat([df1, df2])

```

## Jointures complexes

Les méthodes `merge` et `join` permettent d'effectuer des jointures de type SQL basées sur des colonnes communes ou des index. Ces fonctions sont indispensables pour enrichir vos données en croisant des informations provenant de tables différentes.

```python
# Fusion de deux DataFrames sur une colonne commune
df_a = pd.DataFrame({'id': [1, 2], 'val': ['x', 'y']})
df_b = pd.DataFrame({'id': [1, 2], 'prix': [10, 20]})

# Utilisation de merge pour croiser les données
fusion = pd.merge(df_a, df_b, on='id')

```

### Synthèse

La fusion de données via `concat` pour l'empilement et `merge` pour les relations complexes permet de structurer efficacement vos bases d'informations.

### Exercices

1. Créez deux petits DataFrames avec une colonne identique et utilisez `merge` pour les fusionner.
2. Utilisez `pd.concat` pour ajouter une ligne supplémentaire à un DataFrame existant en créant un second DataFrame contenant uniquement cette ligne.