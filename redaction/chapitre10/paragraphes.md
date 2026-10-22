# Chapitre 10 : Optimisation et Analyse rapide

L'optimisation et l'analyse exploratoire rapide sont les piliers pour transformer un script lent en une solution professionnelle. Ce chapitre vous montre comment exploiter la puissance de calcul de Pandas et visualiser vos résultats en un clin d'œil.

Dans ce chapitre :

* Importance de la vectorisation
* Analyse exploratoire et visualisation intégrée

## Importance de la vectorisation

La vectorisation consiste à appliquer des opérations sur des colonnes entières de données plutôt que d'utiliser des boucles `for` traditionnelles. Cette approche est infiniment plus rapide car elle délègue le calcul à des fonctions optimisées en langage C.

```python
import pandas as pd
import numpy as np

# Création d'un grand jeu de données
df = pd.DataFrame({'valeurs': np.random.randint(0, 100, 1000000)})

# Évitez les boucles lentes :
# Mauvaise pratique : for i in df['valeurs']: ...

# Utilisez la vectorisation (instantané) :
df['double'] = df['valeurs'] * 2

```

## Analyse exploratoire et visualisation

Pandas intègre des outils d'analyse exploratoire qui permettent de comprendre la distribution de vos données sans configuration complexe. La méthode `.plot()` permet de générer des graphiques directement depuis un DataFrame ou une Série pour une visualisation immédiate.

```python
# Analyse rapide de la distribution d'une série
df['valeurs'].hist(bins=20)

# Visualisation rapide d'une série temporelle ou numérique
df['valeurs'].plot(title="Analyse exploratoire rapide")

```

### Synthèse

Privilégiez toujours la vectorisation pour garantir la performance de votre code et utilisez les méthodes graphiques intégrées pour une exploration visuelle immédiate de vos jeux de données.

### Exercices

1. Comparez le temps d'exécution entre une boucle `for` classique ajoutant 1 à chaque élément d'une liste de 1 million d'entiers et l'opération vectorisée équivalente sur une Series Pandas.
2. À partir d'un DataFrame contenant des données aléatoires, générez un graphique en boîte (boxplot) pour identifier rapidement les valeurs aberrantes.

