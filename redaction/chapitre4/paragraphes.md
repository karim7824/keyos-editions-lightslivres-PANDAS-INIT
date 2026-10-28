# Chapitre 4 : Filtrage et Mappage dans les DataFrames Pandas

En programmation Python classique, on utilise souvent les fonctions intégrées **`filter()`** et **`map()`** sur des listes ou des itérables. Sous Pandas, ces concepts existent mais ont été entièrement repensés pour exploiter la vectorisation et offrir des performances de haut niveau sur des millions de lignes.

| Opération | Fonction / Outil Pandas | Cas d'usage principal |
| --- | --- | --- |
| **Filtrer des lignes** | `df[condition]` ou `df.query()` | Extraire un sous-ensemble de données selon des critères chiffrés ou textuels. |
| **Filtrer des colonnes** | `df.filter(like=...)` | Sélectionner rapidement des colonnes par motif ou nom. |
| **Mapper des valeurs** | `serie.map(dictionnaire)` | Remplacer des codes par des libellés (tables de correspondance). |
| **Appliquer une fonction** | `serie.apply()` / `df.map()` | Exécuter une logique de calcul personnalisée sur les données. |

---

## Le Filtrage de données : Bien plus qu'un simple `filter()`

En Pandas, il existe deux manières distinctes de "filtrer" : le **filtrage conditionnel de lignes** (le plus fréquent) et le **filtrage de colonnes/index par nom**.

## Le filtrage conditionnel de lignes (Indexation booléenne)

Pour extraire des lignes selon un critère métier (ex: *tous les produits dont le prix dépasse 50€*), on n'utilise pas la fonction `filter()`, mais l'**indexation booléenne**, qui crée un masque de `True`/`False`.

```python
import pandas as pd

# Données d'exemple (notre catalogue)
df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0],
    'Stock': [10, 40, 5, 0]
})

# Filtrer les produits dont le prix est supérieur à 50
produits_chers = df[df['Prix_HT'] > 50]
print("--- Produits chers (> 50) ---")
print(produits_chers)

# Combiner plusieurs conditions avec les opérateurs logiques (& pour ET, | ou OU)
produits_disponibles_chers = df[(df['Prix_HT'] > 50) & (df['Stock'] > 0)]

```

*Alternative élégante :* La méthode **`.query()`** permet d'écrire les conditions sous forme de chaîne de caractères textuelle, très lisible pour des requêtes complexes :

```python
resultat = df.query("Prix_HT > 50 and Stock > 0")

```

## La méthode `.filter()` : Sélectionner des colonnes ou des index par nom

À l'inverse, la méthode `.filter()` de Pandas sert à **filtrer les étiquettes** (les noms des colonnes ou de l'index), et non les valeurs des lignes.

```python
# Ne conserver que les colonnes dont le nom contient "Prix"
df_filtre_cols = df.filter(like='Prix')

# Sélectionner des colonnes spécifiques par leur nom exact
df_selection = df.filter(items=['Produit', 'Stock'])

```

---

## Le Mappage (`.map()`) : Transformer et traduire des valeurs

La méthode **`.map()`** s'applique à une **Série** (une seule colonne). Elle est idéale pour remplacer des valeurs, traduire des codes numériques en libellés textuels, ou appliquer un dictionnaire de correspondance.

## Traduire des codes catégories

Imaginons une colonne contenant des identifiants numériques de catégories que l'on souhaite transformer en texte lisible :

```python
import pandas as pd

df_ventes = pd.DataFrame({
    'ID_Produit': [1, 2, 3],
    'Code_Cat': [10, 20, 10]
})

# Dictionnaire de correspondance (Mapping)
correspondance_cat = {
    10: 'Accessoire Informatique',
    20: 'Matériel Audio'
}

# Appliquer le map via un dictionnaire
df_ventes['Nom_Cat'] = df_ventes['Code_Cat'].map(correspondance_cat)

print(df_ventes)

```

> **Attention aux valeurs absentes :** Si `.map()` rencontre une valeur qui n'existe pas dans le dictionnaire de correspondance, elle la remplace par `NaN` (valeur manquante). Pour éviter cela, on peut passer une fonction ou utiliser `.replace()`.

---

## Aller plus loin : `.apply()` et `.applymap()` (ou `.map()` global)

Quand le mapping par dictionnaire ne suffit plus et qu'on souhaite appliquer une fonction personnalisée (ou une fonction lambda) sur tout un DataFrame ou une colonne :

* **Sur une Série (`colonne.apply()`)** : Applique une fonction ligne par ligne sur une colonne.
* **Sur tout le DataFrame (`df.map()**` - anciennement `applymap` dans les anciennes versions de Pandas) : Applique une fonction cellule par cellule sur l'ensemble de la table.

```python
# Appliquer une fonction lambda pour appliquer une remise de 10% sur tous les prix
df['Prix_Remise'] = df['Prix_HT'].apply(lambda x: x * 0.9)

print(df[['Produit', 'Prix_HT', 'Prix_Remise']])

```

---
