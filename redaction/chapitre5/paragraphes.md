# Chapitre 5 : Manipuler un DataFrame

Dans ce chapitre :

* Travailler sur les colonnes : sélection, calcul et nettoyage
* Travailler sur les lignes : filtrage, ajout et suppression
* Travailler sur les labels et index
* Extraction de données : `.loc` vs `.iloc`
* Exemple de synthèse
* Exercices de fin de chapitre


## Travailler sur les colonnes : sélection, calcul et nettoyage

Manipuler les colonnes est l'une des tâches les plus fréquentes en science des données. Cela permet d'enrichir un jeu de données avec de nouvelles variables, de nettoyer les structures en éliminant le superflu ou de clarifier les étiquettes.

Grâce à la **vectorisation** de Pandas
, ces opérations s'exécutent de manière extrêmement rapide sur de grands volumes de données, évitant ainsi l'utilisation de boucles `for` lentes et inefficaces.

### Ajout d'une colonne calculée (vectorisation)

Vous pouvez créer de nouvelles variables en appliquant des opérations mathématiques directes entre des colonnes existantes.

```python
import pandas as pd
import numpy as np

# Création d'un DataFrame de base
df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2]
})

# Calcul vectorisé du prix TTC (TVA à 20%) et du total par ligne
df['Prix_TTC'] = df['Prix_HT'] * 1.20
df['Total'] = df['Prix_TTC'] * df['Quantite']

print(df)

```

Ajoute une colonne arbitraire avec des valeurs invalides `np.nan`
```python
df["X"] = np.nan

```

Le paramètre `axis` est aussi obligatoire lorsqu'on supprime des éléments avec `drop()` :

```python
# Supprimer une colonne (on se déplace horizontalement sur les colonnes)
df = df.drop(columns='Physique') # Équivalent à df.drop('Physique', axis=1)

# Supprimer une ligne par son index (on se déplace verticalement sur les lignes)
df = df.drop(index=0)            # Équivalent à df.drop(0, axis=0)

```

### Renommer des colonnes (`.rename`)

Pour rendre un jeu de données plus explicite ou harmoniser des noms de colonnes, la méthode `.rename()` est idéale.

```python
# Renommage ciblé via un dictionnaire de correspondance
df_renomme = df.rename(columns={
    'Prix_HT': 'Prix_Hors_Taxe',
    'Quantite': 'Qte_Stock'
})

print(df_renomme.columns)

```

### Supprimer des colonnes (`drop`)

Lorsqu'une variable n'est plus utile ou contient trop de valeurs manquantes, vous pouvez la supprimer du `DataFrame`.

```python
# Suppression d'une ou plusieurs colonnes avec la méthode drop
# axis=1 ou columns=... indique qu'il s'agit d'une colonne
df_reduit = df.drop(columns=['Total'])

print(df_reduit.columns)

```

### Sélectionner et réordonner un sous-ensemble de colonnes

Il est souvent nécessaire de restreindre le `DataFrame` à un sous-ensemble de variables précises et de modifier leur ordre d'affichage.

```python
# Passer une liste de noms de colonnes permet de filtrer et réordonner simultanément
df_ordonne = df[['Produit', 'Quantite', 'Prix_HT', 'Prix_TTC', 'Total']]

print(df_ordonne.head())

```

## Travailler sur les lignes : filtrage, ajout et suppression

La manipulation des lignes permet de sélectionner, d'ajouter ou de supprimer des enregistrements spécifiques au sein d'un `DataFrame`.

Le filtrage repose généralement sur des **conditions booléennes** (ou masques booléen) : Pandas évalue chaque ligne selon un critère pour ne conserver que celles qui remplissent la condition (`True`). Cette approche évite également les boucles itératives inefficaces grâce à la vectorisation.

### Filtrage simple avec une condition booléenne

Vous pouvez isoler un sous-ensemble de lignes en testant la valeur d'une colonne par rapport à un seuil ou une chaîne de caractères.

```python
import pandas as pd

# Création d'un DataFrame de référence
df = pd.DataFrame({
    'Nom': ['Alice', 'Bob', 'Charlie', 'David'],
    'Score': [75, 90, 82, 60],
    'Ville': ['Paris', 'Lyon', 'Marseille', 'Paris']
})

# Filtrage des lignes où le score est strictement supérieur à 80
selection = df[df['Score'] > 80]
print(selection)

```

### Filtrage avancé avec conditions multiples

Il est possible de combiner plusieurs critères en utilisant les opérateurs logiques adaptés : `&` (ET), `|` (OU) et `~` (NON). *Attention : chaque condition doit impérativement être entourée de parenthèses.*

```python
# Sélection des étudiants de Paris dont le score est supérieur à 70
filtre_multiple = df[(df['Score'] > 70) & (df['Ville'] == 'Paris')]
print(filtre_multiple)

```

### Ajouter de nouvelles lignes (`.loc`)

Vous pouvez insérer de nouveaux enregistrements à la volée en ciblant un index inexistant via la propriété `.loc`.

```python
# Ajout d'une ligne à un index spécifique (ex: index 4)
df.loc[4] = ['Eve', 95, 'Bordeaux']

print(df)

```

### Supprimer des lignes (`drop`)

Si certains enregistrements ne sont plus nécessaires ou s'avèrent erronés, vous pouvez les supprimer en ciblant leurs index.

```python
# Suppression de lignes par leurs indices (ex: supprimer les lignes d'index 0 et 3)
df_nettoye = df.drop(index=[0, 3])

print(df_nettoye)

```

## Travailler sur les labels et index

Contrairement aux tableaux NumPy, un `DataFrame` de Pandas ne se limite pas à des positions numériques entières : il s'appuie sur un **index** qui joue le rôle de clé d'identification unique (analogue à une clé primaire en SQL).

Manipuler les étiquettes et l'index permet d'organiser vos données selon un **identifiant métier** plutôt qu'une simple position (ex: nom d'un produit, identifiant client, date). Cela offre plusieurs avantages majeurs :

* **Alignement automatique** des données lors des calculs ou des fusions (jointures).
* **Recherche et accès sémantique** ultra-rapides et précis.
* **Flexibilité** pour restructurer la table à la volée.


### Définir une colonne existante comme index (`set_index`)

Vous pouvez transformer n'importe quelle colonne de votre `DataFrame` en index principal pour identifier vos lignes de manière signifiante.

```python
import pandas as pd

# DataFrame initial avec un index numérique par défaut (0, 1, 2)
df = pd.DataFrame({
    'Code_Emp': ['EMP01', 'EMP02', 'EMP03'],
    'Nom': ['Alice', 'Bob', 'Charlie'],
    'Departement': ['RH', 'IT', 'Finance']
})

# Définition de la colonne 'Code_Emp' comme nouvel index
df_indexe = df.set_index('Code_Emp')
print(df_indexe)

```

### Réinitialiser l'index (`reset_index`)

Si vous avez besoin de replacer l'index actuel dans une colonne standard du tableau (par exemple pour l'exporter vers un fichier CSV classique), la méthode `.reset_index()` est indispensable.

```python
# Retour en arrière : l'index redevient une colonne normale et un index numérique par défaut est recréé
df_reinitialise = df_indexe.reset_index()
print(df_reinitialise)

```

### Renommer des étiquettes d'index (`rename`)

Tout comme pour les colonnes, vous pouvez modifier dynamiquement certaines étiquettes d'index si elles comportent des erreurs ou nécessitent une mise à jour.

```python
# Renommage d'une étiquette d'index spécifique
df_renomme = df_indexe.rename(index={'EMP01': 'EMP_ADMIN_01'})
print(df_renomme)

```

### Sélectionner des données par étiquette avec `.loc`

Une fois l'index configuré, l'attribut `.loc` vous permet d'interroger directement les lignes en utilisant leurs étiquettes textuelles (ce qui est beaucoup plus robuste qu'un numéro de ligne qui pourrait changer).

```python
# Recherche d'une ligne spécifique grâce à son étiquette d'index métier
employe_specifique = df_indexe.loc['EMP02']
print(employe_specifique)

```

## Extraction de données : `.loc` vs `.iloc`

L'extraction de sous-ensembles est une opération quotidienne en science des données. Pandas propose deux attributs principaux pour cibler précisément les cellules, lignes ou colonnes d'un `DataFrame` :

* **`.loc` (Location)** : Indexation basée sur les **étiquettes** (les noms des lignes et des colonnes). *Note importante : les bornes de découpage (slicing) sont inclusives.*
* **`.iloc` (Integer Location)** : Indexation basée strictement sur les **positions numériques entières** (de 0 à n-1), indépendamment des étiquettes. *Note importante : les bornes de découpage excluent la fin, comme les listes Python classiques.*

Ces outils permettent d'isoler des valeurs uniques, des tranches (slices) ou des combinaisons complexes de lignes et de colonnes en une seule commande.

---

### Extraction par étiquette avec `.loc`

Si votre `DataFrame` possède des étiquettes claires (par exemple un nom ou un identifiant), `.loc` permet d'accéder directement à une ligne, une colonne ou une cellule précise.

```python
import pandas as pd

# Création d'un DataFrame avec un index textuel
df = pd.DataFrame({
    'Score': [85, 90, 78],
    'Ville': ['Paris', 'Lyon', 'Marseille']
}, index=['Alice', 'Bob', 'Charlie'])

# Extraction d'une cellule spécifique (ligne 'Alice', colonne 'Score')
valeur = df.loc['Alice', 'Score']
print("Score d'Alice :", valeur)

# Extraction d'une ligne entière par son étiquette
ligne_bob = df.loc['Bob']
print("\nToutes les infos de Bob :\n", ligne_bob)

```

### Extraction par position avec `.iloc`

Lorsque vous ne connaissez pas les étiquettes ou que vous souhaitez automatiser un parcours par position numérique (ex: récupérer la première ligne), `.iloc` est l'outil idéal.

```python
# Extraction de la première ligne (index positionnel 0)
premiere_ligne = df.iloc[0]
print("Première ligne (position 0) :\n", premiere_ligne)

# Extraction d'une valeur précise : 2ème ligne (index 1), 1ère colonne (index 0) -> 'Lyon'
valeur_pos = df.iloc[1, 0]
print("\nValeur à la position [1, 0] :", valeur_pos)

```

### Découpage bidimensionnel (Slicing) de tranches

Les deux méthodes permettent de sélectionner simultanément des plages de lignes et de colonnes en utilisant le format `[lignes, colonnes]`.

```python
# Avec .loc : les bornes textuelles sont INCLUSIVES ('Alice' jusqu'à 'Bob' inclus)
tranche_loc = df.loc['Alice':'Bob', 'Score':'Ville']
print("Slicing avec .loc (inclusif) :\n", tranche_loc)

# Avec .iloc : les bornes numériques excluent la fin (de la ligne 0 à 1 exclue, soit seulement la ligne 0)
tranche_iloc = df.iloc[0:2, 0:1]
print("\nSlicing avec .iloc (exclusif en fin) :\n", tranche_iloc)

```

### Extraction conditionnelle combinée avec `.loc`

Il est très fréquent de combiner un filtre booléen sur les lignes avec `.loc` pour modifier ou extraire uniquement certaines colonnes de manière ciblée.

```python
# Sélectionner la colonne 'Ville' uniquement pour les étudiants dont le Score > 80
villes_bons_scores = df.loc[df['Score'] > 80, 'Ville']
print("Villes des étudiants avec un score > 80 :\n", villes_bons_scores)

```

## Exemple de synthèse

Gestion et analyse d'un catalogue de produits. Ce script de synthèse met en œuvre un flux complet de manipulation de données avec Pandas, combinant toutes les techniques vues précédemment.

```python
import pandas as pd

# CRÉATION DU DATAFRAME (Depuis un dictionnaire de listes)
data = {
    'Reference': ['PROD_01', 'PROD_02', 'PROD_03', 'PROD_04'],
    'Nom': ['Clavier', 'Souris', 'Écran', 'Casque'],
    'Prix_HT': [45.0, 25.0, 199.99, 80.0],
    'Stock': [120, 85, 14, 0],
    'Categorie': ['Accessoire', 'Accessoire', 'Moniteur', 'Audio']
}
df = pd.DataFrame(data)
print("--- DataFrame Initial ---")
print(df)

# EXPLORATION DES PROPRIÉTÉS STRUCTURALES
print("\n--- Propriétés clés ---")
print("Dimensions (.shape) :", df.shape)
print("Colonnes (.columns) :", df.columns.tolist())

# MANIPULATION DES COLONNES (Vectorisation)
# Ajout d'une colonne calculée pour le prix TTC (20% de TVA) et la valeur totale du stock
df['Prix_TTC'] = df['Prix_HT'] * 1.20
df['Valeur_Stock'] = df['Prix_TTC'] * df['Stock']

# GESTION DE L'INDEX
# Utilisation de la colonne 'Reference' comme index métier unique
df = df.set_index('Reference')
print("\n--- DataFrame avec Index Métier ---")
print(df.head())

# FILTRAGE DES LIGNES (Conditions multiples)
# Sélection des produits en stock (Stock > 0) de la catégorie 'Accessoire'
produits_dispos = df[(df['Stock'] > 0) & (df['Categorie'] == 'Accessoire')]
print("\n--- Accessoires disponibles en stock ---")
print(produits_dispos[['Nom', 'Prix_TTC', 'Stock']])

# EXTRACTION DE DONNÉES CIBLÉES (.loc et .iloc)
# Extraction par étiquette textuelle (.loc)
prix_ecran = df.loc['PROD_03', 'Prix_TTC']
print(f"\nPrix TTC de l'Écran via .loc : {prix_ecran} €")

# Extraction par position numérique (.iloc) pour la première ligne
premier_produit = df.iloc[0]
print("\nPremier produit (position 0) :\n", premier_produit[['Nom', 'Prix_TTC']])

```

* **Création & Structure** : Privilégiez les dictionnaires pour la lisibilité et contrôlez toujours vos dimensions avec `.shape`.
* **Performance** : Effectuez vos calculs sur les colonnes de manière globale (vectorisation) plutôt qu'avec des boucles.
* **Sémantique** : Utilisez `.set_index()` pour remplacer les numéros de lignes par des identifiants métiers significatifs.
* **Requêtes** : Utilisez `.loc` lorsque vous connaissez les étiquettes et `.iloc` lorsque vous travaillez sur des positions numériques.

## Exercices de fin de chapitre

**Exercice 1 :** Importez un petit fichier CSV de votre choix et affichez les noms des colonnes à l'aide de l'attribut approprié.

**Exercice 2 :** Créez un DataFrame à partir d'un dictionnaire, puis utilisez `.iloc` pour extraire uniquement la première colonne de toutes les lignes.
