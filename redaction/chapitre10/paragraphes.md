# Chapitre 10 : Statistiques et agrégations

L'analyse de données repose sur la capacité à synthétiser de vastes ensembles d'informations en indicateurs clés. Ce chapitre vous présente les outils pour calculer des statistiques descriptives et organiser vos données par catégories afin d'en extraire toute la valeur.

Dans ce chapitre :

* Statistiques descriptives
* Agrégations par catégories (`groupby`)
* Agrégation avec plusieurs fonctions de réduction (`.agg`)
* `.agg` vs `.groupby`
* Exemple de synthèse
* Exercices de fin de chapitre

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

La méthode **`groupby()`** est l'équivalent sous Pandas du fameux `GROUP BY en SQL`. Elle repose sur le concept puissant du **Split-Apply-Combine** (Diviser - Appliquer - Combiner) : elle segmente le DataFrame en sous-groupes selon une ou plusieurs catégories, applique une fonction d'agrégation sur chacun d'eux, puis rassemble le tout dans un nouveau tableau synthétique.

```python
# Données avec une colonne catégorielle
data = {'Equipe': ['A', 'B', 'A', 'B'], 'Scores': [10, 20, 30, 40]}
df = pd.DataFrame(data)

# Calcul de la moyenne par équipe
resultat = df.groupby('Equipe')['Scores'].mean()
# Le résultat montre la moyenne pour A (20) et B (30)

```

```python
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam', 'Tapis'],
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio', 'Accessoire'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0, 15.0],
    'Quantite': [3, 10, 2, 5, 20],
    'Total': [150.0, 250.0, 160.0, 350.0, 300.0]
})

```

Calculer une métrique simple par catégorie. Pour obtenir rapidement le chiffre d'affaires total généré par chaque famille de produits :

```python
# Grouper par catégorie et sommer la colonne 'Total'
ca_par_categorie = df.groupby('Categorie')['Total'].sum()

print(ca_par_categorie)

```

* **Ce qui se passe :** Pandas isole les lignes par `Categorie` ('Accessoire' d'un côté, 'Audio' de l'autre), additionne les valeurs de la colonne `Total` pour chaque groupe, et renvoie une Série indexée par les catégories.

Appliquer plusieurs fonctions d'agrégation (`.agg()`). `.agg()` s'associe parfaitement à `groupby` pour croiser plusieurs indicateurs par groupe :

```python
# Appliquer des fonctions différentes selon les colonnes pour chaque catégorie
synthese_categorie = df.groupby('Categorie').agg({
    'Prix_HT': 'mean',        # Prix moyen par catégorie
    'Quantite': 'sum',        # Volume total d'articles vendus
    'Total': ['sum', 'max']   # CA total et CA maximal atteint
})

print(synthese_categorie)

```

* **Ce qui se passe :** Pandas génère un tableau croisé dynamique à colonnes multiples (MultiIndex) permettant d'auditer finement chaque rayon.

L'agrégation nommée pour des rapports propres. Pour éviter d'obtenir des noms de colonnes complexes ou imbriqués lors d'agrégations multiples, la syntaxe moderne de **l'agrégation nommée** permet de renommer directement les colonnes à la volée :

```python
# Création d'un rapport synthétique avec des noms de colonnes explicites
rapport_propre = df.groupby('Categorie').agg(
    Prix_Moyen=('Prix_HT', 'mean'),
    Volume_Ventes=('Quantite', 'sum'),
    Chiffre_Affaires=('Total', 'sum')
).reset_index()

print(rapport_propre)

```

* **Ce qui se passe :** L'utilisation de `.reset_index()` transforme l'index catégoriel en colonne classique, ce qui rend le DataFrame immédiatement exploitable pour un export ou la génération de graphiques.

## Agrégation avec plusieurs fonctions de réduction (`.agg`) 

La méthode **`.agg()`** (abréviation d'*aggregate*, ou son alias `.aggregate()`) est l'outil de synthèse par excellence sous Pandas. Elle permet d'appliquer une ou plusieurs fonctions de réduction (moyenne, somme, minimum, maximum, etc.) sur un DataFrame ou une Série, de manière globale ou personnalisée par colonne.

Appliquer une fonction unique à toutes les colonnes numériques

```python
# Calcule la somme de toutes les colonnes numériques
print(df.agg('sum'))

```

Appliquer plusieurs fonctions à la fois. En passant une liste de fonctions, Pandas retourne un tableau récapitulatif structuré :

```python
# Calcule à la fois la moyenne et l'écart-type sur les colonnes numériques
print(df.agg(['mean', 'std']))

```

Appliquer des fonctions différentes selon les colonnes (Le plus puissant). En passant un **dictionnaire**, tu peux spécifier exactement quel indicateur calculer pour chaque colonne, ce qui est particulièrement utile pour un rapport de synthèse :

```python
synthese = df.agg({
    'Prix_HT': ['mean', 'min', 'max'], # Moyenne et extrêmes pour les prix
    'Quantite': 'sum',               # Volume total vendu
    'Total': ['sum', 'mean']         # CA total et panier moyen global
})

print(synthese)

```

Contrairement aux méthodes directes comme `df['Prix_HT'].mean()` qui s'exécutent colonne par colonne séparément, `.agg()` centralise la logique de calcul et permet d'obtenir une **matrice de synthèse multi-métriques** en une seule ligne de code propre et lisible.

## `.agg` vs `.groupby`

C'est la syntaxe standard en analyse de données pour obtenir des rapports structurés et multi-indicateurs par catégorie :
* groupby() : Le découpeur (Structure)
* agg() (ou aggregate) : Le calculateur (Synthèse)

```python 
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam'],
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0],
    'Quantite': [3, 10, 2, 5]
})

# GROUPBY crée les groupes ('Accessoire', 'Audio')
# AGG calcule les métriques spécifiques pour chaque groupe
rapport = df.groupby('Categorie').agg(
    Prix_Moyen=('Prix_HT', 'mean'),
    Volume_Total=('Quantite', 'sum')
)

print(rapport)
```

## Exemple de synthèse

Rapport de synthèse commercial par catégorie
Reprenons notre catalogue enrichi avec une colonne 'Categorie'

```python
import pandas as pd

# Données de départ
df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam', 'Tapis'],
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio', 'Accessoire'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0, 15.0],
    'Quantite': [3, 10, 2, 5, 20],
    'Total': [150.0, 250.0, 160.0, 350.0, 300.0]
})

# Combinaison de groupby et agg (avec l'agrégation nommée)
synthese_par_categorie = df.groupby('Categorie').agg(
    Nombre_References=('Produit', 'count'),    # Nombre de produits distincts dans la catégorie
    Prix_Moyen_HT=('Prix_HT', 'mean'),          # Prix moyen unitaire
    Volume_Ventes=('Quantite', 'sum'),          # Nombre total d'articles vendus
    Chiffre_Affaires_Total=('Total', 'sum')     # CA total généré
).reset_index()

print(synthese_par_categorie)
```

## Exercices de fin de chapitre

**Exercice 1 :** Rapport de synthèse décisionnel par catégorie (`groupby` et `agg`)

En tant qu'analyste de données, la direction te demande de produire un tableau de bord synthétique pour analyser les performances des ventes par grande catégorie de produits.

```python
import pandas as pd

df_catalogue = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam', 'Tapis', 'Écran'],
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio', 'Accessoire', 'Moniteur'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0, 15.0, 250.0],
    'Quantite_Vendue': [12, 45, 8, 15, 30, 5],
    'Chiffre_Affaires': [600.0, 1125.0, 640.0, 1050.0, 450.0, 1250.0]
})

```

À l'aide d'un enchaînement `groupby()` et `agg()`, construis un DataFrame de synthèse qui regroupe les données par `Categorie` et calcule pour chacune d'elles :

* Le nombre total de références distinctes (`count` sur la colonne `Produit`).
* Le prix unitaire moyen HT (`mean` sur la colonne `Prix_HT`), renommé explicitement `Prix_Moyen`.
* Le volume total d'articles vendus (`sum` sur la colonne `Quantite_Vendue`), renommé `Volume_Total`.
* Le chiffre d'affaires cumulé (`sum` sur la colonne `Chiffre_Affaires`), renommé `CA_Total`.
* Transforme l'index de groupe en colonne classique pour obtenir un tableau propre et plat.

**Exercice 2 :** Pipeline de transformation sécurisée et enrichissement (`copy`, `pipe`, `apply`)

Préparer une simulation commerciale : appliquer une politique de remise promotionnelle ciblée et générer des libellés textuels automatisés pour chaque ligne, sans altérer le jeu de données initial.

```python
import pandas as pd

df_stocks = pd.DataFrame({
    'Reference': ['REF-01', 'REF-02', 'REF-03'],
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_Actuel': [50.0, 25.0, 80.0],
    'Stock_Restant': [10, 25, 4]
})

```

* Isolation sécurisée :** Créer une copie indépendante du DataFrame nommée `df_simulation` pour travailler sans risque de modification de l'original.
* Pipeline de remise :** Écrire une fonction personnalisée `appliquer_promo(df, taux)` qui calcule un nouveau prix `Prix_Promo` en appliquant le taux de réduction fourni (par exemple 15%), puis intègre cette fonction dans un appel `.pipe()`.
* Traitement ligne par ligne :** À l'aide de `.apply()` avec `axis=1`, ajoute une colonne `Alerte_Reappro` qui retourne la chaîne de caractères `"Prioritaire"` si le stock est strictement inférieur à 5, et `"Normal"` dans le cas contraire.
