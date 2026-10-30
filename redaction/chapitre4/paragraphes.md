# Chapitre 4 : Présentation un DataFrame

Le DataFrame est la structure de données pivot de Pandas, permettant de manipuler des tableaux bidimensionnels de manière flexible et efficace. Ce chapitre détaille comment structurer, créer et extraire des données au sein de cet environnement tabulaire.

Dans ce chapitre :

* Series vs DataFrame (et comparaison avec NumPy)
* Organisation interne et propriétés d'un DataFrame
* Gestion des données invalides (`NaN`)
* Créations multiples : listes, tuples et dictionnaires
* Chargement de fichiers externes : CSV et Excel
* Exemple de synthèse
* Exercices de fin de chapitre

## Series vs DataFrame (et comparaison avec NumPy)

La différence fondamentale entre les structures de Pandas (`Series` et `DataFrame`) et les tableaux NumPy (`ndarray`) réside dans l'**indexation**. Alors qu'un `ndarray` se base uniquement sur des positions entières implicites, les objets Pandas intègrent des étiquettes explicites :

* **`Series`** : Tableau unidimensionnel étiqueté pouvant contenir n'importe quel type de données.
* **`DataFrame`** : Structure tabulaire bidimensionnelle composée de lignes et de colonnes étiquetées (comparable à une table SQL).

Cette structure permet un **accès sémantique** aux données, facilitant grandement leur interprétation et leur manipulation.

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

## Organisation interne et propriétés d'un DataFrame

Un `DataFrame` possède une architecture interne rigoureuse : il peut être visualisé comme un dictionnaire ordonné ou un conteneur de **`Series`** partageant un **index commun**. Techniquement, chaque colonne d'un `DataFrame` est donc une `Series` indépendante.

Ses propriétés principales permettent d'inspecter rapidement sa structure, ses métadonnées et ses types de données sans avoir à parcourir tout le tableau :

* **`.shape`** : Renvoie un tuple indiquant le nombre de lignes et de colonnes `(lignes, colonnes)`.
* **`.index`** : Retourne les étiquettes des lignes.
* **`.columns`** : Retourne les noms des colonnes.
* **`.dtypes`** : Indique le type de données (entier, texte, flottant, etc.) pour chaque colonne.
* **`.values`** (ou `.to_numpy()`) : Permet d'extraire les données sous forme de tableau NumPy brut.

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
# df.axes[0] correspond à l'axe vertical (les lignes).
# df.axes[1] correspond à l'axe horizontal (les colonnes).
print(df.axes)

[Index(['Élève_A', 'Élève_B', 'Élève_C'], dtype='object'), Index(['Maths', 'Physique'], dtype='object')]

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
# Élève_A    11.0
# Élève_B    14.5
# Élève_C    18.0
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

## Créations multiples : listes, tuples et dictionnaires

La création d'un `DataFrame` est extrêmement flexible et s'adapte à la structure initiale de vos données. Selon que vous partez de tableaux bruts, de flux JSON (API) ou de variables isolées, plusieurs approches s'offrent à vous :

* **Liste de listes ou de tuples** : Idéal pour transformer une matrice numérique ou un ensemble de lignes brutes en tableau étiqueté.
* **Dictionnaire de listes (ou de tableaux)** : La méthode la plus intuitive et courante, où chaque clé du dictionnaire devient automatiquement le nom d'une colonne.
* **Liste de dictionnaires** : Parfait pour traiter des données orientées lignes (comme des fichiers JSON ou des réponses d'API), où chaque dictionnaire représente une ligne.


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

## Chargement de fichiers externes : CSV et Excel

Pandas propose des fonctions hautement performantes pour importer directement des données tabulaires stockées dans des fichiers (`pd.read_csv` et `pd.read_excel`). Ces fonctions automatisent de nombreuses tâches complexes, telles que :

* La reconnaissance automatique ou manuelle des en-têtes de colonnes.
* La détection des types de données (entiers, flottants, chaînes de caractères, dates).
* La gestion des valeurs manquantes (`NaN`).

Pour être pleinement efficace, ces fonctions acceptent de nombreux paramètres de configuration (comme le séparateur de champs, l'encodage des caractères ou le choix des feuilles de calcul).

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
print(df.head())

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
## Exemple de synthèse

```python
import pandas as pd
import numpy as np

# Création d'un DataFrame e-commerce à partir d'un dictionnaire
data = {
    'Produit': ['Clavier', 'Souris', 'Casque', 'Écran', 'Webcam'],
    'Prix_HT': [50.0, 25.0, 80.0, 200.0, np.nan], # Contient une valeur manquante (NaN)
    'Stock': [120, 85, 45, 14, 60]
}
df_catalogue = pd.DataFrame(data, index=['P1', 'P2', 'P3', 'P4', 'P5'])

print("--- 1. DataFrame d'origine ---")
print(df_catalogue)

# Inspection structurelle
print("\n--- 2. Propriétés structurelles ---")
print("Dimensions (shape) :", df_catalogue.shape)
print("Colonnes disponibles :", df_catalogue.columns.tolist())

# Gestion des données invalides (Remplacement du NaN par la moyenne des prix)
prix_moyen = df_catalogue['Prix_HT'].mean()
df_catalogue['Prix_HT'] = df_catalogue['Prix_HT'].fillna(prix_moyen)

print("\n--- 3. Après nettoyage des valeurs manquantes ---")
print(df_catalogue)

# Opérations vectorisées et calculs par axe (axis)
# Calcul de la valeur totale du stock par produit (Prix_HT * Stock)
df_catalogue['Valeur_Stock'] = df_catalogue['Prix_HT'] * df_catalogue['Stock']

# Calcul de la somme totale de la valeur du stock (en descendant les lignes, axis=0 par défaut)
valeur_totale_parc = df_catalogue['Valeur_Stock'].sum(axis=0)

print(f"\n--- 4. Valeur totale globale du parc en stock : {valeur_totale_parc} € ---")

# Extraction finale sous forme de tableau NumPy brut (ex: pour export ou modèle)
matrice_brute = df_catalogue[['Prix_HT', 'Stock']].values
print("\n--- 5. Extraction brute sous forme de ndarray NumPy ---")
print(matrice_brute)

```

## Exercices de fin de chapitre


**Exercice 1 :** Création d'un DataFrame à partir d'un dictionnaire et manipulation des propriétés structurelles

Vous gérez le parc informatique d'une PME. Vous disposez des informations relatives à trois ordinateurs portables en stock, que vous devez structurer sous la forme d'un `DataFrame` Pandas pour en analyser les caractéristiques.

À l'aide d'un dictionnaire de listes, créez un `DataFrame` nommé `df_parc` contenant les données suivantes :
* **`Modele`** : `['Latitude 5420', 'ThinkPad T14', 'MacBook Pro']`
* **`Marque`** : `['Dell', 'Lenovo', 'Apple']`
* **`Prix_HT`** : `[950.0, 1100.0, 1850.0]`
* **`En_Stock`** : `[True, False, True]`

Attribuez un index explicite personnalisé à votre DataFrame en utilisant la liste d'identifiants : `['PC_01', 'PC_02', 'PC_03']`.
Affichez les dimensions du DataFrame via l'attribut `.shape` ainsi que les types de données de chaque colonne via `.dtypes`.
Extrayez la colonne `Prix_HT` dans une variable, puis vérifiez et affichez son type exact à l'aide de la fonction `type()` pour confirmer qu'il s'agit bien d'une `Series`.

---

**Exercice 2 :** Utilisation des axes (`axis`) et calculs statistiques orientés

Un centre de formation souhaite analyser les notes semestrielles obtenues par ses étudiants dans différentes matières scientifiques. Vous devez charger ces données et exploiter le paramètre `axis` pour automatiser les calculs.

Créez un `DataFrame` nommé `df_notes` à partir du dictionnaire suivant, en définissant les noms des étudiants comme index (`index=['Élève_A', 'Élève_B', 'Élève_C']`) :
```python
data_notes = {
    'Algorithmique': [14, 16, 10],
    'Base_de_Donnees': [12, 18, 15],
    'Reseaux': [16, 14, 13]
}

```

Calculez et affichez la **moyenne par matière** en utilisant explicitement le paramètre adapté (`axis=0`).
Calculez et affichez la **moyenne générale de chaque élève** en modifiant la direction du calcul avec l'axe horizontal (`axis=1`).
Supprimez la colonne `'Reseaux'` du DataFrame en utilisant la méthode `.drop()` avec l'argument `axis=1` (ou `columns=...`), puis affichez le DataFrame résultant.

