# Chapitre 3 : Opérations sur les données

Apprendre à manipuler et transformer vos données est indispensable pour extraire des informations pertinentes. Ce chapitre vous permet de maîtriser le tri, l'itération et les fonctions avancées de transformation pour nettoyer et enrichir vos jeux de données.

Dans ce chapitre :

* Opération de Tri
* Itération sur les lignes et modification
* Utilisation des fonctions `apply` pour les transformations
* Utilisation des `pipe` pour les transformations
* Recherches et critères de filtrage

## Opération de Tri

Le tri permet de réorganiser vos données selon les valeurs d'une ou plusieurs colonnes pour mieux visualiser les extrêmes ou les tendances. Vous pouvez trier par ordre croissant ou décroissant très simplement.

```python
# Tri des données selon une colonne spécifique
df_tri = df.sort_values(by='Score', ascending=False)

```

## Itération sur les lignes et modification

Bien que l'itération manuelle soit moins performante que les méthodes vectorisées, elle permet de modifier des lignes complexes individuellement. On utilise généralement `iterrows()` pour parcourir les lignes une par une.

```python
# Itération sur les lignes pour modifier une valeur
for index, row in df.iterrows():
    df.at[index, 'Score'] += 1

```

## Utilisation des fonctions `apply`

La méthode `apply` permet d'appliquer une fonction personnalisée à chaque ligne ou chaque colonne d'un DataFrame. C'est un outil très puissant pour réaliser des transformations complexes non supportées nativement par Pandas.

```python
# Application d'une fonction pour transformer les scores
df['Score'] = df['Score'].apply(lambda x: x * 1.1)

```

## Utilisation des `pipe`

La méthode `pipe` permet d'enchaîner plusieurs fonctions de transformation de manière lisible et structurée. Cette approche facilite la maintenance du code en isolant chaque étape de traitement dans des fonctions séparées.

```python
# Enchaînement de transformations avec pipe
def augmenter_score(df):
    df['Score'] += 5
    return df

df = df.pipe(augmenter_score)

```

## Recherches et critères de filtrage

Le filtrage consiste à extraire uniquement les lignes qui répondent à une ou plusieurs conditions logiques précises. Vous pouvez combiner plusieurs critères avec des opérateurs booléens pour un ciblage très granulaire.

```python
# Filtrage selon deux critères
resultat = df[(df['Score'] > 80) & (df['Nom'] == 'Alice')]

```

### Synthèse

La manipulation efficace des données repose sur l'utilisation de méthodes vectorisées, le chaînage via `pipe` et le filtrage conditionnel, garantissant un code propre et performant.

### Exercices

1. Triez votre DataFrame par ordre croissant sur une colonne numérique et affichez les trois premières lignes.
2. Créez une fonction qui double la valeur d'une colonne et utilisez `apply` pour l'appliquer à votre DataFrame.
