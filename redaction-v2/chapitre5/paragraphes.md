# Chapitre 5 : Filtrage et Mappage dans les DataFrames Pandas

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
## Exemple de synthèse

Le nettoyage et l'enrichissement du catalogue e-commerce. Imaginons que nous récupérons un fichier de données brut comportant des codes de catégories chiffrés et des prix bruts. Notre objectif est de :

**Filtrer** les articles disponibles et dont le prix dépasse un certain seuil.
**Traduire** les codes de catégories en libellés explicites grâce à un dictionnaire de mapping.
**Appliquer** une fonction de calcul personnalisée pour simuler un nouveau tarif promotionnel.

```python
import pandas as pd

# Données brutes issues de l'importation
df_brut = pd.DataFrame({
    'Reference': ['SKU-01', 'SKU-02', 'SKU-03', 'SKU-04', 'SKU-05'],
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam', 'Tapis'],
    'Code_Cat': [10, 20, 30, 20, 10],
    'Prix_Brut': [60.0, 25.0, 120.0, 85.0, 15.0],
    'En_Stock': [True, False, True, True, True]
})

print("--- Données brutes d'origine ---")
print(df_brut)

# ==========================================
# APPLICATION DU FILTRAGE ET DU MAPPING
# ==========================================

# Filtrer uniquement les produits en stock ET dont le prix dépasse 20€
# (Utilisation d'un masque booléen combiné)
df_filtre = df_brut[(df_brut['En_Stock'] == True) & (df_brut['Prix_Brut'] > 20)].copy()

# Mappage des codes de catégories vers des libellés métier explicites
dictionnaire_categories = {
    10: 'Accessoires',
    20: 'Périphériques',
    30: 'Audio & Son'
}
df_filtre['Categorie'] = df_filtre['Code_Cat'].map(dictionnaire_categories)

# Appliquer une fonction personnalisée (.apply) : Remise de 15% sur les prix
df_filtre['Prix_Promo'] = df_filtre['Prix_Brut'].apply(lambda x: round(x * 0.85, 2))

# ==========================================
# SÉLECTION FINALE (Propreté du rapport)
# ==========================================
# On isole uniquement les colonnes pertinentes pour l'analyse
df_synthese = df_filtre[['Reference', 'Produit', 'Categorie', 'Prix_Brut', 'Prix_Promo']]

print("\n--- Tableau de synthèse final (Filtré, Mappé et Calculé) ---")
print(df_synthese)

```

---

### Les clés de la réussite à rappeler à votre lecteur :

**Le filtrage se fait sur les lignes** via des conditions booléennes (`df[...]` ou `.query()`), tandis que `.filter()` est réservé aux étiquettes de colonnes ou d'index.

**Le `.map()` est l'outil roi** pour enrichir une colonne à partir d'un dictionnaire de correspondance, à condition de bien anticiper la gestion des valeurs inconnues (`NaN`).

**Le `.apply()` offre une flexibilité totale** pour injecter du code Python personnalisé (comme des fonctions `lambda`) lorsque les opérations vectorisées de base ne suffisent plus.

## Exercices de fin de chapitre

**Exercice 1:** Analyse temporelle et extraction des pics d'activité
Manipuler les séries temporelles, convertir des chaînes de caractères en objets `datetime`, extraire des composantes temporelles et réaliser une agrégation croisée.

Vous disposez d'un journal de transactions brutes sous forme de texte. La direction souhaite identifier quels jours de la semaine génèrent le plus de chiffre d'affaires.

Créez un DataFrame contenant ces données brutes :
```python
import pandas as pd

df_logs = pd.DataFrame({
    'Horodatage': ['2026-10-01 09:30:00', '2026-10-02 14:15:00', '2026-10-03 18:45:00', 
                   '2026-10-04 10:00:00', '2026-10-05 16:20:00', '2026-10-06 11:10:00'],
    'Montant_EUR': [120.5, 45.0, 300.0, 85.0, 210.0, 95.0]
})

```
Convertissez la colonne `Horodatage` en un type `datetime` exploitable.
Extrayez le **nom du jour de la semaine** (ex: *Monday*, *Tuesday*...) dans une nouvelle colonne nommée `Jour_Semaine`.
Réalisez un regroupement (`groupby`) par `Jour_Semaine` pour calculer la somme totale du chiffre d'affaires (`Montant_EUR`) et le nombre de transactions par jour.


**Exercice 2:**  Enrichissement relationnel, Mapping et Filtrage
Croiser deux tables hétérogènes avec `pd.merge()`, traduire des codes via `.map()` et appliquer un filtre conditionnel multicritère.

Vous gérez deux sources de données distinctes dans votre boutique : un historique de ventes contenant des identifiants produits, et un catalogue général fournissant les détails et les catégories.

Partez des deux DataFrames suivants :
```python
# Table des ventes
df_ventes = pd.DataFrame({
    'ID_Produit': [101, 102, 101, 103, 102],
    'Quantite': [2, 1, 5, 3, 2]
})

# Table du catalogue produits
df_catalogue = pd.DataFrame({
    'ID_Produit': [101, 102, 103, 104],
    'Nom': ['Clavier', 'Souris', 'Casque', 'Webcam'],
    'Code_Cat': [1, 2, 3, 2],
    'Prix_Unitaire': [50.0, 25.0, 80.0, 70.0]
})

```


Réalisez une jointure (`pd.merge()`) de type **`left`** en prenant `df_ventes` comme table principale pour y intégrer les informations du catalogue.
À l'aide d'un dictionnaire de correspondance, créez une colonne `Categorie` en mappant les `Code_Cat` selon la règle suivante :
* `1` : *Bureautique*
* `2` : *Accessoire*
* `3` : *Audio*

Calculez le montant total de chaque ligne (`Quantite * Prix_Unitaire`) dans une nouvelle colonne `CA_Ligne`.
Filtrez enfin le DataFrame final pour n'afficher que les lignes dont le `CA_Ligne` dépasse 100 €.
