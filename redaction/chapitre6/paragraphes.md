# Chapitre 6 : Statistiques et agrégations

L'analyse de données repose sur la capacité à synthétiser de vastes ensembles d'informations en indicateurs clés. Ce chapitre vous présente les outils pour calculer des statistiques descriptives et organiser vos données par catégories afin d'en extraire toute la valeur.

Dans ce chapitre :

* Statistiques descriptives
* Agrégations par catégories avec `groupby`

## Statistiques descriptives

Les statistiques descriptives permettent d'obtenir une vue d'ensemble rapide de la distribution et de la tendance centrale de vos données. Pandas propose des méthodes intégrées pour calculer instantanément des mesures comme la moyenne (`mean`), la somme (`sum`), le minimum (`min`), le maximum (`max`) ou l'écart-type (`std`) sur une série ou un DataFrame.

```python
import pandas as pd

# Création d'un jeu de données de scores
df = pd.DataFrame({'Scores': [10, 20, 30, 40, 50]})

# Calculs statistiques classiques
moyenne = df['Scores'].mean()  # Retourne 30.0
ecart_type = df['Scores'].std() # Mesure la dispersion
maxi = df['Scores'].max()      # Retourne 50

```

## Agrégations par catégories (`groupby`)

Le groupement de données est une étape fondamentale pour analyser des sous-ensembles selon des catégories spécifiques. La méthode `groupby` segmente vos données en groupes, vous permettant d'appliquer des calculs statistiques indépendamment pour chaque catégorie identifiée.

```python
# Données avec une colonne catégorielle
data = {'Equipe': ['A', 'B', 'A', 'B'], 'Scores': [10, 20, 30, 40]}
df = pd.DataFrame(data)

# Calcul de la moyenne par équipe
resultat = df.groupby('Equipe')['Scores'].mean()
# Le résultat montre la moyenne pour A (20) et B (30)

```

### Synthèse

La maîtrise des statistiques descriptives et du `groupby` est essentielle pour passer de la donnée brute à une information intelligible, permettant une analyse fine par segmentation.

### Exercices

1. Calculez la médiane et la somme d'une colonne numérique dans un DataFrame de votre choix.
2. À partir d'un DataFrame contenant des ventes par produit et par vendeur, utilisez `groupby` pour calculer le chiffre d'affaires total par produit.