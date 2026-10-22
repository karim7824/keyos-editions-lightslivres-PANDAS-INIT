# Chapitre 5 : Gestion des dates et séries temporelles

La maîtrise des séries temporelles est essentielle pour toute analyse de données comportant une dimension chronologique. Vous allez apprendre à utiliser les outils spécifiques de Pandas pour structurer, manipuler et générer des séquences basées sur le temps.

Dans ce chapitre :

* Utilisation des modèles de temps (`Timestamp`, `Datetime`, `Timedelta`, `Period`)
* Manipulation de plages temporelles avec `period_range`

## Modèles de temps

Pandas offre une gestion précise du temps grâce à quatre types d'objets principaux : `Timestamp` pour un instant précis, `Datetime` pour les types de données temporelles, `Timedelta` pour les durées, et `Period` pour représenter un intervalle de temps défini. Ces outils permettent de manipuler des calendriers complexes avec une grande rigueur syntaxique.

```python
import pandas as pd

# Création d'un instant précis (Timestamp)
instant = pd.Timestamp('2026-07-06 18:00:00')

# Calcul d'une durée (Timedelta)
duree = pd.Timedelta(days=5)
nouveau_temps = instant + duree

```

## Manipulation de plages temporelles

La fonction `period_range` permet de générer des séquences régulières de périodes temporelles sur un intervalle donné. Cette fonctionnalité est particulièrement utile pour créer des index basés sur des fréquences spécifiques, comme des mois ou des trimestres.

```python
# Génération d'une plage de périodes mensuelles
plage = pd.period_range(start='2026-01', periods=3, freq='M')

# Utilisation dans une Série
s = pd.Series([10, 20, 30], index=plage)

```

### Synthèse

La gestion du temps dans Pandas repose sur des objets spécialisés comme `Timestamp` et `Period`, couplés à des générateurs puissants comme `period_range` pour structurer vos analyses chronologiques.

### Exercices

1. Créez un `Timestamp` correspondant à la date d'aujourd'hui et ajoutez-lui un `Timedelta` de 48 heures.
2. Utilisez `period_range` pour générer une liste de 6 périodes trimestrielles commençant au premier trimestre 2026.