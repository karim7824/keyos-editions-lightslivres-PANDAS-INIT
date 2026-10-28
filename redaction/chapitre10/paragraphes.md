# Chapitre 9 : Gestion des structures multidimensionnelles

L'analyse de données complexes nécessite souvent de manipuler des dimensions supplémentaires au-delà du simple tableau bidimensionnel. Ce chapitre vous présente les Panel Data et l'utilisation des MultiIndex pour structurer vos informations de manière hiérarchique et précise.

Dans ce chapitre :

* Introduction aux Panel Data
* Utilisation du `MultiIndex`

## Introduction aux Panel Data

Les Panel Data, ou données de panel, permettent de suivre plusieurs entités sur plusieurs périodes de temps. Cette structure enrichit l'analyse en combinant des dimensions temporelles et transversales, offrant une vue dynamique de vos jeux de données.

```python
import pandas as pd

# Les Panel Data organisent les données sur plusieurs dimensions
# Souvent représentées par un DataFrame avec un index hiérarchique
data = {'Valeur': [10, 20, 30, 40]}
index = [('EntrepriseA', 2025), ('EntrepriseA', 2026), 
         ('EntrepriseB', 2025), ('EntrepriseB', 2026)]
df = pd.DataFrame(data, index=pd.MultiIndex.from_tuples(index, names=['Entité', 'Année']))

```

## Utilisation du `MultiIndex`

Le `MultiIndex` est l'outil indispensable pour gérer des dimensions complexes, permettant d'avoir plusieurs niveaux d'indexation sur les lignes ou les colonnes. Il transforme votre DataFrame en une structure multidimensionnelle intuitive, facilitant le filtrage et l'agrégation sur des segments de données spécifiques.

```python
# Accès aux données via le MultiIndex
# Sélection de toutes les données pour l'EntrepriseA
data_a = df.loc['EntrepriseA']

# Sélection d'une valeur précise par entité et année
valeur_specifique = df.loc[('EntrepriseA', 2026)]

```

### Synthèse

La gestion des structures multidimensionnelles repose sur l'utilisation du `MultiIndex`, qui permet de structurer hiérarchiquement vos données pour une analyse fine des Panel Data.

### Exercices

1. Créez un DataFrame avec un `MultiIndex` comportant deux niveaux : "Ville" et "Mois", et remplissez-le avec des données de température fictives.
2. Utilisez `.loc` pour extraire toutes les données relatives à une seule ville de votre DataFrame hiérarchisé.

