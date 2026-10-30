# Chapitre 15 : Visualisation de données avec Pandas (`df.plot()`)

Ce chapitre vous guide à travers les techniques de visualisation de données en Python, combinant la simplicité d'utilisation de la méthode df.plot() de Pandas avec la puissance graphique de Matplotlib et de Seaborn.

Dans ce chapitre :

* Visualisation avec Matplotlib, Pyplot et Seaborn
* Le catalogue des représentations graphiques
* Habillage, personnalisation et multi-graphiques
* Exemple de synthèse
* Exercices de fin de chapitre

---

## Visualisation avec Matplotlib, Pyplot et Seaborn

Bien que la méthode `df.plot()` suffise amplement pour l'exploration rapide et la génération de graphiques simples, l'écosystème Python de la visualisation repose sur trois piliers complémentaires. Pour concevoir des rapports de production avancés ou des analyses statistiques poussées, il est indispensable de savoir sortir de Pandas pour exploiter directement **Matplotlib**, son module **Pyplot**, et la bibliothèque statistique **Seaborn**.

### La hiérarchie des outils de visualisation

Pour comprendre comment ces outils s'articulent, il faut voir Pandas comme une surcouche de simplification :

**Pandas (`df.plot()`)** : Intégré nativement, idéal pour prototyper et visualiser directement depuis un DataFrame. Il délague le travail d'affichage à Matplotlib en arrière-plan.
**Matplotlib / Pyplot (`plt`)** : Le moteur graphique de référence en Python. Il offre un contrôle absolu sur chaque pixel, chaque axe, chaque annotation et la structure globale de la figure.
**Seaborn (`sns`)** : Une bibliothèque construite *au-dessus* de Matplotlib, spécialisée dans la **visualisation statistique**. Elle gère nativement les regroupements complexes, les distributions et propose des designs modernes par défaut.

### Matplotlib et Pyplot (`plt`) : Le contrôle total sur la figure

Quand les options de Pandas montrent leurs limites (par exemple pour superposer des graphiques de types différents, ajouter des annotations textuelles complexes ou gérer des figures multi-axes sophistiquées), on fait appel directement à `matplotlib.pyplot`.

```python
import pandas as pd
import matplotlib.pyplot as plt

# Données d'exemple
Mois = ['Jan', 'Fév', 'Mar', 'Avr', 'Mai']
CA_2025 = [10000, 12000, 11000, 13500, 15000]
CA_2026 = [12500, 14000, 13800, 16000, 18500]

# Création explicite d'une figure et d'une zone de dessin (axes)
fig, ax = plt.subplots(figsize=(8, 4))

# Tracé de deux courbes superposées avec personnalisation fine
ax.plot(Mois, CA_2025, label='2025', color='gray', linestyle='--', marker='o')
ax.plot(Mois, CA_2026, label='2026', color='teal', linewidth=2.5, marker='s')

# Habillage professionnel via l'objet ax
ax.set(title="Comparatif du Chiffre d'Affaires N-1 vs N",
       xlabel="Mois",
       ylabel="Euros (€)")
ax.legend()
ax.grid(True, alpha=0.3)

plt.show()

```

### Seaborn (`sns`) : L'expert en statistiques et esthétique

Seaborn brille particulièrement lorsqu'il s'agit d'analyser des distributions statistiques ou de croiser plusieurs variables catégorielles sans avoir à manipuler laborieusement les index de Pandas. Il intègre nativement des thèmes visuels élégants et gère les regroupements complexes en un seul argument (`hue`, `col`, etc.).

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Application d'un thème Seaborn moderne
sns.set_theme(style="whitegrid")

# Données de test (répartition des prix par catégorie)
df_produits = pd.DataFrame({
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio', 'Moniteur', 'Moniteur', 'Accessoire', 'Audio'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0, 250.0, 300.0, 40.0, 90.0],
    'Canal': ['Web', 'Boutique', 'Web', 'Boutique', 'Web', 'Web', 'Boutique', 'Boutique']
})

# Génération d'un diagramme en boîte (boxplot) avec Seaborn
# Il gère automatiquement la répartition statistique (médiane, quartiles, valeurs aberrantes)
plt.figure(figsize=(8, 4))
sns.boxplot(
    data=df_produits, 
    x='Categorie', 
    y='Prix_HT', 
    hue='Canal',          # Regroupement secondaire instantané
    palette='Set2'
)

plt.title("Distribution des prix par catégorie et canal de vente")
plt.show()

```

### Synthèse : Quel outil choisir pour votre projet ?

| Outil | Niveau d'abstraction | Force principale | Cas d'usage idéal |
| --- | --- | --- | --- |
| **Pandas (`df.plot()`)** | Élevé (Intégré) | Rapidité d'exécution | Exploration de données rapide, graphiques simples intégrés à un script de nettoyage. |
| **Matplotlib (`plt`)** | Bas / Intermédiaire | Contrôle chirurgical | Rapports de production sur-mesure, annotations complexes, multi-graphiques avancés. |
| **Seaborn (`sns`)** | Intermédiaire (Statistique) | Esthétique moderne et inférence | Analyse exploratoire statistique, distributions, corrélations et graphiques multi-variables. |

---

## Le catalogue des représentations graphiques 

* **Le graphique linéaire (`kind='line'`)** : Suivi des tendances et des séries temporelles (ex: évolution du chiffre d'affaires au fil des mois).
* **Le diagramme en barres (`kind='bar'` / `barh`)** : Comparaison de catégories (ex: volume des ventes par catégorie de produits).
* **L'histogramme (`kind='hist'`)** : Analyse de la distribution statistique d'une variable numérique (ex: répartition des prix unitaires).
* **Le nuage de points (`kind='scatter'`)** : Croisement de deux variables pour identifier des corrélations (ex: Prix vs Quantité vendue).
* **Le diagramme circulaire (`kind='pie'`)** : Représentation des parts de marché (avec modération).

Le DataFrame de référence pour la visualisation

```python
import pandas as pd
import matplotlib.pyplot as plt

# Création du DataFrame de test
df_visualisation = pd.DataFrame({
    'Date': pd.date_range(start='2026-10-01', periods=10, freq='D'),
    'Produit': ['Clavier', 'Souris', 'Casque', 'Webcam', 'Tapis', 'Écran', 'Clavier', 'Souris', 'Casque', 'Webcam'],
    'Categorie': ['Accessoire', 'Accessoire', 'Audio', 'Audio', 'Accessoire', 'Moniteur', 'Accessoire', 'Accessoire', 'Audio', 'Audio'],
    'Prix_HT': [50.0, 25.0, 80.0, 70.0, 15.0, 250.0, 50.0, 25.0, 80.0, 70.0],
    'Quantite': [5, 12, 3, 8, 15, 2, 7, 10, 4, 6]
})

# Calcul d'une colonne dérivée pour l'analyse
df_visualisation['Chiffre_Affaires'] = df_visualisation['Prix_HT'] * df_visualisation['Quantite']

print(df_visualisation)

```

---

### Diagramme en barres (`kind='bar'`) : Comparer par catégorie

Idéal pour visualiser le chiffre d'affaires cumulé par grande famille de produits en couplant `groupby` et `plot()` :

```python
# Agréger le CA par catégorie et générer un graphique en barres
df_visualisation.groupby('Categorie')['Chiffre_Affaires'].sum().plot(
    kind='bar',
    color='skyblue',
    title="Chiffre d'Affaires par Catégorie",
    rot=0 # Garde les libellés horizontaux
)
plt.ylabel('Euros (€)')
plt.show()

```

### Graphique linéaire (`kind='line'`) : Suivre une série temporelle

Parfait pour observer l'évolution des ventes au fil des jours :

```python
# Indexer par date pour un rendu temporel propre
df_temporel = df_visualisation.set_index('Date')

# Tracer l'évolution de la quantité vendue par jour
df_temporel['Quantite'].plot(
    kind='line',
    marker='o',
    color='green',
    title="Évolution journalière des volumes vendus",
    figsize=(10, 4)
)
plt.ylabel('Quantité')
plt.show()

```

### Nuage de points (`kind='scatter'`) : Analyser une corrélation

Utile pour étudier la relation entre le prix unitaire et les quantités achetées :

```python
# Croiser le prix et la quantité
df_visualisation.plot(
    kind='scatter',
    x='Prix_HT',
    y='Quantite',
    color='purple',
    title="Relation Prix / Quantité vendue"
)
plt.show()

```

---

## Habillage, personnalisation et multi-graphiques

Un graphique brut généré par défaut est utile pour l'exploration rapide, mais un rapport professionnel exige un habillage rigoureux : des axes lisibles, des titres explicites, des couleurs adaptées et une mise en page soignée. Puisque Pandas s'appuie directement sur **Matplotlib** sous le capot, il est possible de personnaliser finement chaque rendu.

### Personnalisation des axes et des titres

Il est indispensable de nommer clairement les axes et d'ajouter un titre pour que le graphique soit compréhensible hors de son contexte. Ces paramètres peuvent être passés directement dans la méthode `.plot()` ou complétés via les fonctions de `matplotlib.pyplot`.

```python
import pandas as pd
import matplotlib.pyplot as plt

# Données d'exemple (ventes par jour)
df_ventes = pd.DataFrame({
    'Jour': ['Lun', 'Mar', 'Mer', 'Jeu', 'Ven'],
    'CA': [1200, 1500, 950, 1800, 2200]
}).set_index('Jour')

# Génération du graphique avec habillage intégré
df_ventes.plot(
    kind='bar',
    title="Chiffre d'affaires journalier",  # Titre principal
    rot=0                                  # Garde les libellés de l'axe X horizontaux
)

# Personnalisation explicite des axes et de la légende
plt.xlabel("Jours de la semaine")
plt.ylabel("Montant en Euros (€)")
plt.legend(["CA Global"])                  # Affichage et nom de la légende
plt.show()

```

### Mise en forme avancée : Dimensions, couleurs et styles

Pour adapter la lisibilité d'un graphique (notamment pour l'intégrer dans un support de cours ou un rapport PDF), on joue sur trois leviers majeurs :

* **`figsize=(largeur, hauteur)`** : Ajuste les proportions physiques de la figure en pouces.
* **`colormap`** : Applique une palette de couleurs cohérente (ex: `'viridis'`, `'plasma'`, `'tab10'`).
* **`linewidth` / `linestyle**` : Modifie l'épaisseur et le style des lignes pour les courbes.

```python
# Traçage d'une courbe stylisée
df_ventes.plot(
    kind='line',
    figsize=(9, 4),           # Figure large et moins haute
    color='teal',             # Couleur principale
    linewidth=3,              # Ligne épaisse
    marker='o',               # Ajout de points sur les sommets
    markersize=8,
    title="Suivi de la performance hebdomadaire"
)

plt.grid(True, linestyle='--', alpha=0.6) # Ajout d'une grille en pointillés légers
plt.ylabel("Euros (€)")
plt.show()

```

### Multi-subplots : Organiser plusieurs graphiques sur une grille

Lorsque l'on souhaite analyser plusieurs métriques en parallèle sans saturer un seul graphique, l'argument **`subplots=True`** ordonne à Pandas de créer un sous-graphique indépendant pour chaque colonne numérique. L'argument **`layout`** permet d'organiser ces graphiques sous forme de grille (lignes, colonnes).

```python
# DataFrame avec plusieurs métriques numériques
df_multimetriques = pd.DataFrame({
    'Date': pd.date_range(start='2026-10-01', periods=4, freq='D'),
    'Visiteurs': [150, 200, 180, 250],
    'Ventes': [12, 18, 15, 22],
    'Panier_Moyen': [50.0, 55.0, 48.0, 52.0]
}).set_index('Date')

# Création d'une grille de graphiques (2 lignes, 2 colonnes)
df_multimetriques.plot(
    kind='line',
    subplots=True,            # Active les graphiques multiples
    layout=(2, 2),            # Grille de 2x2
    figsize=(10, 6),          # Dimensions globales de la figure
    sharex=True,              # Partage l'axe des abscisses (dates)
    marker='.'
)

plt.suptitle("Tableau de bord multi-métriques", fontsize=14, fontweight='bold')
plt.tight_layout()            # Évite le chevauchement des sous-graphiques
plt.show()

```

* **Les libellés (`xlabel`, `ylabel`, `title`)** garantissent l'autonomie sémantique du graphique.
* **`figsize` et `colormap**` assurent l'harmonie visuelle et professionnelle de vos rapports.
* **`subplots=True` couplé à `layout**` transforme un DataFrame multi-colonnes en un tableau de bord analytique structuré en une seule ligne de code.

---

## Exemple de synthèse

Utilisation combinée de la visualisation sous Pandas, Matplotlib et Seaborn.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# Application d'un thème Seaborn moderne pour les graphiques statistiques
sns.set_theme(style="whitegrid")

# Préparation d'un DataFrame de synthèse (ventes e-commerce)
np.random.seed(42)
dates = pd.date_range(start='2026-10-01', periods=30, freq='D')
df_synthese = pd.DataFrame({
    'Date': dates,
    'Canal': np.choice(['Web', 'Mobile', 'Boutique'], size=30),
    'Panier_Moyen': np.random.normal(65.0, 15.0, size=30).round(2),
    'Volume_Ventes': np.random.randint(10, 50, size=30)
})

# Calcul d'une colonne dérivée
df_synthese['CA_Journalier'] = df_synthese['Panier_Moyen'] * df_synthese['Volume_Ventes']

print("--- 1. Aperçu des données de synthèse ---")
print(df_synthese.head())

# Utilisation de Pandas (df.plot) pour un aperçu rapide du CA journalier
plt.figure(figsize=(10, 4))
df_synthese.set_index('Date')['CA_Journalier'].plot(
    kind='line', 
    color='teal', 
    linewidth=2, 
    marker='o',
    title="Évolution globale du Chiffre d'Affaires journalier"
)
plt.ylabel("Euros (€)")
plt.show()

# Utilisation de Matplotlib (plt.subplots) pour une figure multi-axes avancée
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4))

# Graphique 1 : Nuage de points (Prix vs Volume)
ax1.scatter(df_synthese['Panier_Moyen'], df_synthese['Volume_Ventes'], color='purple', alpha=0.7)
ax1.set_title("Relation Panier Moyen / Volume")
ax1.set_xlabel("Panier Moyen (€)")
ax1.set_ylabel("Volume de ventes")

# Graphique 2 : Histogramme de distribution du panier moyen
ax2.hist(df_synthese['Panier_Moyen'], bins=6, color='skyblue', edgecolor='black')
ax2.set_title("Distribution des paniers moyens")
ax2.set_xlabel("Montant (€)")
ax2.set_ylabel("Fréquence")

plt.tight_layout()
plt.show()

# Utilisation de Seaborn (sns) pour analyser la répartition par canal de vente
plt.figure(figsize=(8, 4))
sns.boxplot(
    data=df_synthese, 
    x='Canal', 
    y='CA_Journalier', 
    palette='Set2'
)
plt.title("Répartition du Chiffre d'Affaires selon le canal de vente")
plt.xlabel("Canal")
plt.ylabel("CA Journalier (€)")
plt.show()

```
## Exercices de fin de chapitre

**Exercice 1 :** Création d'un graphique linéaire multi-courbes et habillage personnalisé

Vous suivez l'évolution mensuelle des ventes de deux produits phares d'une entreprise au cours du premier semestre 2026. Vous devez représenter ces tendances sur un même graphique en veillant à soigner l'habillage visuel.

À l'aide de Pandas, créez un `DataFrame` nommé `df_suivi` contenant les données suivantes, avec les mois comme index :
* **`Mois`** : `['Janvier', 'Février', 'Mars', 'Avril', 'Mai', 'Juin']`
* **`Produit_A`** : `[1200, 1350, 1500, 1420, 1600, 1750]`
* **`Produit_B`** : `[900, 1100, 1050, 1300, 1450, 1600]`
Définissez la colonne `'Mois'` comme index du DataFrame.
Utilisez la méthode `.plot()` pour tracer ces deux séries sous forme de graphique linéaire (`kind='line'`).
Personnalisez le graphique en ajoutant :
* Une dimension physique de `(8, 4)` via `figsize`.
* Un titre explicite (ex: *"Comparatif des ventes - 1er semestre 2026"*).
* Des libellés clairs sur les axes (`xlabel` et `ylabel`).
* Une grille en pointillés (`plt.grid()`).


---

**Exercice 2 :** Génération d'une grille de graphiques multi-subplots

La direction souhaite analyser simultanément plusieurs indicateurs de performance d'un site web (visites, conversions et taux de rebond) sur une semaine. Plutôt que de multiplier les figures séparées, vous devez concevoir un tableau de bord condensé.

Créez un `DataFrame` nommé `df_dashboard` à l'aide de la structure suivante :
```python
import pandas as pd

df_dashboard = pd.DataFrame({
    'Date': pd.date_range(start='2026-10-01', periods=5, freq='D'),
    'Visiteurs': [1200, 1450, 1300, 1600, 1850],
    'Conversions': [45, 60, 52, 70, 85],
    'Taux_Rebond': [0.45, 0.42, 0.48, 0.39, 0.35]
}).set_index('Date')

```

Utilisez la méthode `.plot()` en combinant les arguments **`subplots=True`** et **`layout=(2, 2)`** pour répartir les métriques dans une grille de sous-graphiques.
Ajustez les dimensions globales à `(10, 6)`, activez le partage de l'axe horizontal (`share=True`), et ajoutez des marqueurs sur les points (`marker='o'`).
Ajoutez un titre global à la figure (`plt.suptitle(...)`) et utilisez `plt.tight_layout()` pour éviter tout chevauchement avant d'afficher le résultat.


