# Chapitre 3 : Création et manipulation de DataFrames

Le DataFrame est la structure de données pivot de Pandas, permettant de manipuler des tableaux bidimensionnels de manière flexible et efficace. Ce chapitre détaille comment structurer, créer et extraire des données au sein de cet environnement tabulaire.

Dans ce chapitre :

* Series vs DataFrame (et comparaison avec NumPy)
* Organisation interne et propriétés d'un DataFrame
* Gestion des données invalides (`NaN`)
* Créations multiples : listes, tuples et dictionnaires
* Chargement de fichiers externes : CSV et Excel
* Travailler sur les colonnes : sélection, calcul et nettoyage
* Travailler sur les lignes : filtrage, ajout et suppression
* Travailler sur les labels et index
* Extraction de données : `.loc` vs `.iloc`
* Exemple de synthèse
* Exercices de fin de chapitre

## Series vs DataFrame (et comparaison avec NumPy)

La différence fondamentale entre les structures de Pandas (`Series` et `DataFrame`) et les tableaux NumPy (`ndarray`) réside dans l'**indexation**. Alors qu'un `ndarray` se base uniquement sur des positions entières implicites, les objets Pandas intègrent des étiquettes explicites :

* **`Series`** : Tableau unidimensionnel étiqueté pouvant contenir n'importe quel type de données.
* **`DataFrame`** : Structure tabulaire bidimensionnelle composée de lignes et de colonnes étiquetées (comparable à une table SQL).

Cette structure permet un **accès sémantique** aux données, facilitant grandement leur interprétation et leur manipulation.

---

### Création d'une `Series` avec un index personnalisé

Contrairement à un tableau simple, une `Series` associe chaque valeur à une étiquette explicite.

```python
import pandas as pd

# Création d'une Series représentant des notes par matière
notes = pd.Series([15, 18, 12], index=['Maths', 'Physique', 'Informatique'])
print(notes)

```

### Création d'un `DataFrame` (structure 2D)

Le `DataFrame` étend ce concept à deux dimensions en combinant un index de lignes et des étiquettes de colonnes.

```python
import pandas as pd

# Un DataFrame associe des lignes étiquetées à des colonnes nommées
data = {
    'Nom': ['Alice', 'Bob', 'Charlie'],
    'Âge': [25, 30, 35],
    'Ville': ['Paris', 'Lyon', 'Marseille']
}
df = pd.DataFrame(data)
print(df)

```

### Accès sémantique aux données (`.loc`)

Grâce aux étiquettes, il est possible d'interroger les données par leur nom plutôt que par leur position numérique, ce qui rend le code plus lisible.

```python
# Accès direct à une valeur dans la Series via son étiquette textuelle
print(notes['Physique'])  # Affiche : 18

# Accès à une colonne entière du DataFrame par son nom
print(df['Ville'])

```

### Comparaison directe avec un `ndarray` NumPy

Cette comparaison illustre la limite d'un tableau NumPy brut face à un `DataFrame` lorsque les données nécessitent un contexte sémantique.

```python
import numpy as np
import pandas as pd

# NumPy : indexation uniquement par position entière (pas de nom de colonne)
arr = np.array([[10, 20], [30, 40]])
print("NumPy (sans contexte) :")
print(arr)

# DataFrame Pandas : ajout d'un index et de noms de colonnes pour l'interprétation
df_comparaison = pd.DataFrame(arr, index=['Ligne_1', 'Ligne_2'], columns=['Col_A', 'Col_B'])
print("\nPandas DataFrame (avec étiquettes sémantiques) :")
print(df_comparaison)

```

---

## Organisation interne et propriétés d'un DataFrame

Un `DataFrame` possède une architecture interne rigoureuse : il peut être visualisé comme un dictionnaire ordonné ou un conteneur de **`Series`** partageant un **index commun**. Techniquement, chaque colonne d'un `DataFrame` est donc une `Series` indépendante.

Ses propriétés principales permettent d'inspecter rapidement sa structure, ses métadonnées et ses types de données sans avoir à parcourir tout le tableau :

* **`.shape`** : Renvoie un tuple indiquant le nombre de lignes et de colonnes `(lignes, colonnes)`.
* **`.index`** : Retourne les étiquettes des lignes.
* **`.columns`** : Retourne les noms des colonnes.
* **`.dtypes`** : Indique le type de données (entier, texte, flottant, etc.) pour chaque colonne.
* **`.values`** (ou `.to_numpy()`) : Permet d'extraire les données sous forme de tableau NumPy brut.

---

### Vérifier l'organisation interne (une colonne = une Series)

L'extraction d'une colonne d'un `DataFrame` renvoie bien un objet de type `Series`.

```python
import pandas as pd

data = {
    'Produit': ['Clavier', 'Souris', 'Écran'],
    'Prix': [45.0, 25.0, 199.99],
    'Stock': [120, 85, 14]
}
df = pd.DataFrame(data, index=['P1', 'P2', 'P3'])

# Vérification du type d'une colonne individuelle
colonne_prix = df['Prix']
print(type(colonne_prix))  # Affiche : <class 'pandas.core.series.Series'>

```

### Explorer les propriétés structurelles (`.shape`, `.index`, `.columns`, `.dtypes`)

Ces attributs sont indispensables lors de l'exploration initiale d'un jeu de données pour en comprendre la taille et la nature.

```python
# Dimensions du DataFrame (3 lignes, 3 colonnes)
print("Dimensions (.shape) :", df.shape)

# Identifiants des lignes
print("Index (.index) :", df.index)

# Noms des colonnes
print("Colonnes (.columns) :", df.columns)

# Types de données par colonne
print("\nTypes des données (.dtypes) :\n", df.dtypes)

```

### Les axes d'un DataFrame

L'argument `axis` indique la direction d'une opération : `axis=0` (ou `'index'`) agit verticalement le long des lignes, tandis que `axis=1` (ou `'columns'`) agit horizontalement à travers les colonnes.

L'axe 0 (axis=0 ou 'index') représente les lignes (vertical). Les opérations descendent le long des lignes.
L'axe 1 (axis=1 ou 'columns') représente les colonnes (horizontal). Les opérations se déplacent de gauche à droite à travers les colonnes.

`df.mean(axis=0)` : Calcule la moyenne de **chaque colonne** (en descendant les lignes).
`df.sum(axis=1)` : Calcule la somme des valeurs de **chaque ligne** (de gauche à droite).
`df.drop('ma_colonne', axis=1)` : Supprime **une colonne** entière du DataFrame.
`df.dropna(axis=0)` : Supprime les **lignes** qui contiennent au moins une valeur manquante.

Utilisation de `.axes`
```python
import pandas as pd

df = pd.DataFrame({
    'Maths': [10, 15, 20],
    'Physique': [12, 14, 16]
}, index=['Élève_A', 'Élève_B', 'Élève_C'])

# Afficher les deux axes du DataFrame
print(df.axes)

```

df.axes[0] correspond à l'axe vertical (les lignes).
df.axes[1] correspond à l'axe horizontal (les colonnes).

```python
[Index(['Élève_A', 'Élève_B', 'Élève_C'], dtype='object'), 
 Index(['Maths', 'Physique'], dtype='object')]

```

Calculer la moyenne par colonne (`axis=0`).C'est le comportement par défaut de Pandas. On calcule la moyenne de chaque matière pour l'ensemble des élèves en descendant les lignes.

```python
print(df.mean(axis=0))
# Résultat :
# Maths      15.0
# Physique   14.0
# dtype: float64

```

Calculer la moyenne par ligne (`axis=1`). Si l'on souhaite calculer la moyenne générale de chaque élève (en additionnant ses notes de gauche à droite) :

```python
print(df.mean(axis=1))
# Résultat :
# 0    11.0
# 1    14.5
# 2    18.0
# dtype: float64

```

```python
# Supprimer une colonne (on se déplace horizontalement sur les colonnes)
df = df.drop(columns='Physique') # Équivalent à df.drop('Physique', axis=1)

# Supprimer une ligne par son index (on se déplace verticalement sur les lignes)
df = df.drop(index=0)            # Équivalent à df.drop(0, axis=0)

```

### Accéder aux valeurs brutes (`.values`)

Si vous avez besoin de convertir ou de manipuler les données pures sans les étiquettes Pandas (par exemple pour les passer à un modèle de Machine Learning avec Scikit-Learn), la propriété `.values` extrait un tableau NumPy (`ndarray`).

```python
# Extraction des données sous forme de tableau NumPy bidimensionnel
matrice_brute = df.values
print(matrice_brute)
print(type(matrice_brute))  # Affiche : <class 'numpy.ndarray'>

```
---
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
---

## Créations multiples : listes, tuples et dictionnaires

La création d'un `DataFrame` est extrêmement flexible et s'adapte à la structure initiale de vos données. Selon que vous partez de tableaux bruts, de flux JSON (API) ou de variables isolées, plusieurs approches s'offrent à vous :

* **Liste de listes ou de tuples** : Idéal pour transformer une matrice numérique ou un ensemble de lignes brutes en tableau étiqueté.
* **Dictionnaire de listes (ou de tableaux)** : La méthode la plus intuitive et courante, où chaque clé du dictionnaire devient automatiquement le nom d'une colonne.
* **Liste de dictionnaires** : Parfait pour traiter des données orientées lignes (comme des fichiers JSON ou des réponses d'API), où chaque dictionnaire représente une ligne.

---

### Création à partir d'une liste de listes ou de tuples

Cette approche nécessite de spécifier explicitement les noms des colonnes (`columns`) et éventuellement l'index des lignes (`index`).

```python
import pandas as pd

# Données sous forme de liste de tuples (ou listes imbriquées)
donnees_tuples = [
    ('Paris', 2140000),
    ('Lyon', 513275),
    ('Marseille', 861635)
]

# Création avec définition explicite des colonnes et de l'index
df_tuples = pd.DataFrame(donnees_tuples, columns=['Ville', 'Population'], index=['Île-de-France', 'Auvergne-Rhône-Alpes', 'PACA'])
print(df_tuples)

```

### Création à partir d'un dictionnaire de listes (orientation colonnes)

C'est la méthode standard la plus lisible. Chaque clé associe un nom de colonne à une séquence de valeurs de même longueur.

```python
import pandas as pd

# Chaque clé du dictionnaire correspond à une colonne du DataFrame
data_dict = {
    'Produit': ['Ordinateur', 'Tablette', 'Smartphone'],
    'Prix': [1200, 450, 800],
    'En_Stock': [True, False, True]
}

df_dict = pd.DataFrame(data_dict)
print(df_dict)

```

### Création à partir d'une liste de dictionnaires (orientation lignes)

Si vos données proviennent d'une source au format JSON (comme une API web), chaque élément de la liste représente une ligne complète. Les clés manquantes sont automatiquement remplies par des valeurs manquantes (`NaN`).

```python
import pandas as pd

# Liste de dictionnaires (chaque dictionnaire = une ligne)
utilisateurs = [
    {'ID': 1, 'Nom': 'Alice', 'Role': 'Admin'},
    {'ID': 2, 'Nom': 'Bob', 'Role': 'Utilisateur'},
    {'ID': 3, 'Nom': 'Charlie'}  # La clé 'Role' est absente ici
]

df_lignes = pd.DataFrame(utilisateurs)
print(df_lignes)

```

### Combinaison de `Series` indépendantes

Vous pouvez également assembler des objets `Series` préexistants dans un dictionnaire pour former un `DataFrame` cohérent. Pandas alignera automatiquement les données selon leurs index.

```python
import pandas as pd

# Création de deux Series distinctes
titres = pd.Series(['Livre A', 'Livre B', 'Livre C'])
ventes = pd.Series([150, 320, 95])

# Assemblage dans un DataFrame via un dictionnaire
df_series = pd.DataFrame({
    'Ouvrage': titres,
    'Exemplaires_Vendus': ventes
})

print(df_series)

```
---

## Chargement de fichiers externes : CSV et Excel

Pandas propose des fonctions hautement performantes pour importer directement des données tabulaires stockées dans des fichiers (`pd.read_csv` et `pd.read_excel`). Ces fonctions automatisent de nombreuses tâches complexes, telles que :

* La reconnaissance automatique ou manuelle des en-têtes de colonnes.
* La détection des types de données (entiers, flottants, chaînes de caractères, dates).
* La gestion des valeurs manquantes (`NaN`).

Pour être pleinement efficace, ces fonctions acceptent de nombreux paramètres de configuration (comme le séparateur de champs, l'encodage des caractères ou le choix des feuilles de calcul).

---

### Importation de base d'un fichier CSV

C'est le cas le plus simple. Par défaut, `pd.read_csv` considère que les colonnes sont séparées par une virgule (`,`) et que la première ligne contient les en-têtes.

```python
import pandas as pd

# Chargement d'un fichier CSV standard
# df_csv = pd.read_csv('clients.csv')

# Simulation avec un chemin fictif ou URL directe
url = "https://raw.githubusercontent.com/datasciencedojo/datasets/master/titanic.csv"
df = pd.read_csv(url)

# Dimensions globales
print(f"Dimensions (lignes, colonnes) : {df.shape}")
print(f"Nombre total de cellules : {df.size}")
print(f"Nombre de dimensions : {df.ndim}") # Renvoie 2 pour un DataFrame

# Structure des index et des colonnes
print(f"Type d'index : {type(df.index)}")
print(f"Plage des index : {df.index.min()} à {df.index.max()}")
print(f"Liste des colonnes : {df.columns.tolist()}")

# Dictionnaire des types de données (très utile pour du code programmatique)
print(df.dtypes.to_dict())

# Vérifier la structure et les types réels
df.info(memory_usage='deep')

# Obtenir un premier résumé statistique global
print(df.describe(include='all'))

# Aperçu des 5 premières lignes
print(dfc.head())

```

### CSV avec paramètres avancés (séparateur, encodage, index)

En Europe, les fichiers CSV utilisent souvent le point-virgule (`;`) comme séparateur et des accents nécessitant un encodage spécifique (`utf-8` ou `latin-1`). Vous pouvez aussi définir directement une colonne comme index.

```python
import pandas as pd

# Importation d'un CSV personnalisé :
# - sep=';' : délimiteur de colonnes point-virgule
# - encoding='utf-8' : gestion correcte des caractères accentués
# - index_col='ID' : utilise la colonne 'ID' comme index du DataFrame
# df_avance = pd.read_csv('ventes_france.csv', sep=';', encoding='utf-8', index_col='ID')

```

### Importation basique d'un fichier Excel

La fonction `pd.read_excel` permet de charger des classeurs Excel (`.xlsx` ou `.xls`). Par défaut, elle importe la **première feuille** du fichier.

```python
import pandas as pd

# Chargement de la première feuille d'un fichier Excel
# df = pd.read_excel('budget_annuel.xlsx')

```

### Excel avancé (choix de la feuille et des lignes à ignorer)

Les fichiers Excel contiennent souvent des en-têtes publicitaires, des titres ou plusieurs onglets. Vous pouvez cibler précisément la zone utile grâce à `sheet_name` et `skiprows`.

```python
import pandas as pd

# Importation ciblée d'un fichier Excel :
# - sheet_name='2026_Q1' : sélectionne une feuille spécifique par son nom
# - skiprows=2 : ignore les 2 premières lignes (ex: titres du tableau)
# - usecols="A:D" : limite l'importation aux colonnes de A à D
# df_Q1 = pd.read_excel('entreprise.xlsx', sheet_name='2026_Q1', skiprows=2, usecols="A:D")

```
---

## Travailler sur les colonnes : sélection, calcul et nettoyage

Manipuler les colonnes est l'une des tâches les plus fréquentes en science des données. Cela permet d'enrichir un jeu de données avec de nouvelles variables, de nettoyer les structures en éliminant le superflu ou de clarifier les étiquettes.

Grâce à la **vectorisation** de Pandas
, ces opérations s'exécutent de manière extrêmement rapide sur de grands volumes de données, évitant ainsi l'utilisation de boucles `for` lentes et inefficaces.

---

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
---

## Travailler sur les lignes : filtrage, ajout et suppression

La manipulation des lignes permet de sélectionner, d'ajouter ou de supprimer des enregistrements spécifiques au sein d'un `DataFrame`.

Le filtrage repose généralement sur des **conditions booléennes** (ou masques booléen) : Pandas évalue chaque ligne selon un critère pour ne conserver que celles qui remplissent la condition (`True`). Cette approche évite également les boucles itératives inefficaces grâce à la vectorisation.

---

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
---

## Travailler sur les labels et index

Contrairement aux tableaux NumPy, un `DataFrame` de Pandas ne se limite pas à des positions numériques entières : il s'appuie sur un **index** qui joue le rôle de clé d'identification unique (analogue à une clé primaire en SQL).

Manipuler les étiquettes et l'index permet d'organiser vos données selon un **identifiant métier** plutôt qu'une simple position (ex: nom d'un produit, identifiant client, date). Cela offre plusieurs avantages majeurs :

* **Alignement automatique** des données lors des calculs ou des fusions (jointures).
* **Recherche et accès sémantique** ultra-rapides et précis.
* **Flexibilité** pour restructurer la table à la volée.

---

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
---

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

---

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

--- 

## Exercices de fin de chapitre

**Exercice 1** Importez un petit fichier CSV de votre choix et affichez les noms des colonnes à l'aide de l'attribut approprié.

**Exercice 2** Créez un DataFrame à partir d'un dictionnaire, puis utilisez `.iloc` pour extraire uniquement la première colonne de toutes les lignes.
