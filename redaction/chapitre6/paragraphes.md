# Chapitre 6 : Opérations sur les données

Apprendre à manipuler et transformer vos données est indispensable pour extraire des informations pertinentes. Ce chapitre vous permet de maîtriser le tri, l'itération et les fonctions avancées de transformation pour nettoyer et enrichir vos jeux de données.

Dans ce chapitre :

* Opération de Tri
* Itération sur les lignes et modification
* Utilisation des fonctions `apply`
* Utilisation des `pipe`
* Recherches et critères de filtrage
* Exemple de synthèse
* Exercices de fin de chapitre

## Opération de Tri

Le tri permet de réorganiser vos données selon les valeurs d'une ou plusieurs colonnes pour mieux visualiser les extrêmes ou les tendances. Vous pouvez trier par ordre croissant ou décroissant très simplement.

Tri simple par ordre décroissant (Trouver les meilleures ventes).Pour identifier immédiatement les produits qui génèrent le plus de chiffre d'affaires, on trie la colonne `Total` du plus grand au plus petit (`ascending=False`).

```python

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2],
    'Prix_TTC': [60.0, 30.0, 96.0],
    'Total': [180.0, 300.0, 192.0]
})


# Trier par chiffre d'affaires décroissant
df_tri_desc = df.sort_values(by='Total', ascending=False)
print(df_tri_desc)

```

Tri simple par ordre croissant (Trouver les prix les plus bas).Pour classer les articles du moins cher au plus cher, on utilise le tri croissant par défaut (`ascending=True`).

```python
# Trier par prix unitaire HT croissant
df_tri_asc = df.sort_values(by='Prix_HT', ascending=True)
print(df_tri_asc)

```

Tri multicritères (Hiérarchiser selon plusieurs colonnes). Il est possible de trier selon plusieurs critères en passant une liste à `by` et à `ascending`. Par exemple, trier d'abord par `Quantite` décroissante, puis par `Prix_HT` croissant en cas d'égalité des quantités.

```python
# Tri sur deux colonnes avec des directions différentes
df_multi_tri = df.sort_values(
    by=['Quantite', 'Prix_HT'], 
    ascending=[False, True]
)
print(df_multi_tri)

```

## Itération sur les lignes et modification

Bien que l'itération manuelle soit moins performante que les méthodes vectorisées, elle permet de modifier des lignes complexes individuellement. On utilise généralement `iterrows()` pour parcourir les lignes une par une.

```python
# Itération sur les lignes pour modifier une valeur at pour la celle
for index, row in df.iterrows():
    df.at[index, 'Score'] += 1

```

Pour la lecture ou l'écriture d'une cellule isolée, df.at contourne la surcharge de sélection de Pandas et offre de meilleures performances en termes de temps d'exécution (très utile dans des boucles ou des scripts d'optimisation).

## Utilisation des fonctions `apply`

La méthode `apply` permet d'appliquer une fonction personnalisée à chaque ligne ou chaque colonne d'un DataFrame. C'est un outil très puissant pour réaliser des transformations complexes non supportées nativement par Pandas.

Appliquer une transformation sur une colonne (Série). Pour transformer ou formater les valeurs d'une colonne spécifique (par exemple, mettre tous les noms de produits en majuscules) :

```python
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2],
    'Prix_TTC': [60.0, 30.0, 96.0],
    'Total': [180.0, 300.0, 192.0]
})

# Appliquer une fonction lambda pour mettre en majuscules la colonne 'Produit'
df['Produit_Maj'] = df['Produit'].apply(lambda x: x.upper())

print(df[['Produit', 'Produit_Maj']])

```

---

Appliquer une fonction ligne par ligne (`axis=1`). Lorsque l'on a besoin de croiser plusieurs colonnes d'une **même ligne**, on utilise `axis=1`. Par exemple, pour générer une phrase de synthèse descriptive pour chaque produit :

```python
# Combiner les valeurs de plusieurs colonnes par ligne
df['Resume'] = df.apply(
    lambda row: f"Le {row['Produit']} (qté: {row['Quantite']}) génère un total de {row['Total']}€", 
    axis=1
)

print(df['Resume'])

```

## Utilisation des `pipe`

La méthode `pipe` permet d'enchaîner plusieurs fonctions de transformation de manière lisible et structurée. Cette approche facilite la maintenance du code en isolant chaque étape de traitement dans des fonctions séparées.

Lorsque les transformations se succèdent, `pipe` permet de lier des fonctions personnalisées de manière séquentielle et lisible :

```python
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2],
    'Total': [180.0, 300.0, 192.0]
})

# Première fonction : appliquer une remise promotionnelle
def appliquer_remise(data, taux):
    data['Prix_Promo'] = data['Prix_HT'] * (1 - taux)
    return data

# Seconde fonction : filtrer les produits au-delà d'un seuil de chiffre d'affaires
def filtrer_par_seuil(data, seuil):
    return data[data['Total'] >= seuil].copy()

# Enchaînement propre et lisible avec pipe
df_traite = (
    df.pipe(appliquer_remise, taux=0.10)
      .pipe(filtrer_par_seuil, seuil=190.0)
)

print(df_traite[['Produit', 'Prix_HT', 'Prix_Promo', 'Total']])

```

La méthode .copy() permet de créer une copie indépendante d'un DataFrame (ou d'une Série) en mémoire.

## Recherches et critères de filtrage

Le filtrage consiste à extraire uniquement les lignes qui répondent à une ou plusieurs conditions logiques précises. Vous pouvez combiner plusieurs critères avec des opérateurs booléens pour un ciblage très granulaire.


### Exemple 1 : 

Isoler un catalogue avant une modification de prix et créer une grille tarifaire promotionnelle à partir du catalogue principal sans modifier les prix de base d'origine :

```python
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2],
    'Total': [180.0, 300.0, 192.0]
})

# Création d'une copie indépendante pour les soldes
df_soldes = df.copy()

# Modification sur la copie uniquement (baisse de 10% sur le Prix_HT)
df_soldes['Prix_HT'] = df_soldes['Prix_HT'] * 0.9

print("--- Original inchangé ---")
print(df[['Produit', 'Prix_HT']])

print("\n--- Copie modifiée (Soldes) ---")
print(df_soldes[['Produit', 'Prix_HT']])

```

---

Filtrer et modifier un sous-ensemble (Éviter le `SettingWithCopyWarning`). Lorsque tu isoles des lignes spécifiques (par exemple, les produits dont la quantité vendue est supérieure à 2), l'utilisation conjointe du filtrage et de `.copy()` garantit que tu peux ajouter une nouvelle colonne sans déclencher d'avertissement ni corrompre la mémoire :

```python
# Extraction sécurisée des produits à forte rotation
df_populaires = df[df['Quantite'] > 2].copy()

# Ajout d'une colonne de statut sans risque
df_populaires['Alerte_Stock'] = 'Réassort prioritaire'

print(df_populaires)

```

---

Conserver un historique de référence dans un pipeline de transformation. Dans un enchaînement de traitements, il est fréquent de dupliquer un DataFrame à une étape clé pour pouvoir comparer l'état initial et l'état final :

```python
# Sauvegarde de l'état initial (Baseline)
df_reference = df.copy()

# Traitement sur le DataFrame principal
df['Chiffre_Affaires_Estime'] = df['Prix_HT'] * df['Quantite']

# Comparaison directe entre la référence et le résultat traité
comparaison = pd.DataFrame({
    'Produit': df['Produit'],
    'Total_Ref': df_reference['Total'],
    'Total_Calcule': df['Chiffre_Affaires_Estime']
})

print(comparaison)

```
## Exemple de synthèse

En analyse de données, la synthèse consiste à agréger les indicateurs clés (totaux, moyennes, extrêmes) pour obtenir une vue macroscopique en une seule instruction.

Générer un rapport de synthèse global

```python
import pandas as pd

df = pd.DataFrame({
    'Produit': ['Clavier', 'Souris', 'Casque'],
    'Prix_HT': [50.0, 25.0, 80.0],
    'Quantite': [3, 10, 2],
    'Total': [180.0, 300.0, 192.0]
})

# Utilisation de la méthode .agg() pour combiner plusieurs fonctions d'agrégation
synthese_chiffres = df.agg({
    'Prix_HT': ['mean', 'min', 'max'],
    'Quantite': 'sum',
    'Total': 'sum'
})

print("--- Indicateurs agrégés ---")
print(synthese_chiffres)

# Extraction d'un résumé textuel des performances commerciales
chiffre_affaires_global = df['Total'].sum()
quantite_totale = df['Quantite'].sum()
produit_phare = df.loc[df['Total'].idxmax(), 'Produit']

print("\n--- SYNTHÈSE COMMERCIALE ---")
print(f"Chiffre d'affaires total : {chiffre_affaires_global} €")
print(f"Volume total d'articles vendus : {quantite_totale} unités")
print(f"Produit le plus performant (en CA) : {produit_phare}")

```

* **`.agg()`** permet d'appliquer des fonctions statistiques différentes par colonne en une seule passe (par exemple, calculer la moyenne et les extrêmes sur les prix, mais la somme sur les quantités et les totaux).
* **`idxmax()`** permet de récupérer l'index de la ligne ayant la valeur maximale dans une colonne donnée, ce qui est idéal pour identifier dynamiquement le meilleur produit sans avoir à trier tout le DataFrame.

## Exercices de fin de chapitre


** Exercice 1** : Diagnostic et Audit d'un Dataset de Ventes

Tu récupères un fichier CSV contenant des données de ventes brutes d'une entreprise e-commerce (`ventes_techno.csv`). Avant toute analyse marketing, tu dois mener un audit rigoureux de la structure et de la santé des données.

```python
import pandas as pd
import io

csv_data = """id_vente,produit,categorie,prix_unitaire,quantite,date_vente
101,Clavier RGB,Accessoire,45.5,4,2026-05-10
102,Écran 27,Moniteur,299.0,2,2026-05-11
103,Souris Sans Fil,Accessoire,25.0,NaN,2026-05-11
104,PC Portable Pro,Ordinateur,1200.0,1,2026-05-12
105,Casque Audio,Accessoire,60.0,3,2026-05-13
"""
df_ventes = pd.read_csv(io.StringIO(csv_data))

```

* Importation et typage : Charge ce jeu de données en veillant à convertir la colonne `date_vente` directement en objets temporels (`datetime`) et à spécifier les colonnes catégorielles (`produit`, `categorie`).
* Inspection structurelle : Affiche les dimensions du DataFrame, la liste des axes (`.axes` ou `.columns`/`.index`), ainsi que le diagnostic complet de l'empreinte mémoire avec `info(memory_usage='deep')`.
* Audit des anomalies : Identifie le nombre de valeurs manquantes par colonne.
* Statistiques descriptives : Affiche un résumé statistique global (incluant les variables textuelles/catégorielles) pour évaluer les fourchettes de prix et les volumes.

---

**Exercice 2 :** Pipeline de Transformation et Synthèse Commerciale

À partir du catalogue de produits consolidé, tu dois appliquer une série de transformations sécurisées, calculer des métriques personnalisées ligne par ligne, et extraire un rapport de synthèse décisionnel.


```python
import pandas as pd

df_catalogue = pd.DataFrame({
    'Reference': ['SKU-01', 'SKU-02', 'SKU-03', 'SKU-04'],
    'Produit': ['Clavier', 'Souris', 'Casque', 'Ecran'],
    'Prix_HT': [50.0, 25.0, 80.0, 300.0],
    'Stock': [15, 42, 8, 5]
})

```

* Isolation sécurisée (`.copy()`) : Crée une copie indépendante nommée `df_promo` pour préparer une opération promotionnelle sans risquer de corrompre le catalogue d'origine.
* Transformation par Pipeline (`.pipe()`) : Écris une fonction `appliquer_taxe(df, tva)` qui ajoute une colonne `Prix_TTC` (calculée avec une TVA de 20%), puis enchaîne-la via `.pipe()`.
* Traitement ligne par ligne (`.apply(axis=1)`) : À l'aide de `.apply()` et `axis=1`, génère une colonne `Valeur_Stock_HT` représentant la valorisation financière totale du stock pour chaque produit (`Prix_HT * Stock`).
* Synthèse globale (`.agg()`) : Utilise `.agg()` pour obtenir simultanément le prix moyen, minimum et maximum des produits HT, ainsi que la somme totale de la valeur du stock de l'entreprise.
