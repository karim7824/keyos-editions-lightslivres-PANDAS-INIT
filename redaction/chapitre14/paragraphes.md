# Chapitre 14: Rappel Pyplot – Les fondations graphiques de Python

Bien que Pandas propose une surcouche intuitive avec `df.plot()` pour prototyper rapidement, l'écosystème graphique de Python repose en profondeur sur **Matplotlib** et son module d'interface haut niveau : **Pyplot** (`plt`). Maîtriser Pyplot est indispensable pour reprendre le contrôle total de la mise en page, structurer des figures complexes et produire des graphiques de qualité professionnelle.

Dans ce chapitre :

* Architecture de Matplotlib : `Figure` et `Axes`
* Tracer un graphique simple : `plt.plot()`
* Titres, étiquettes, légendes et styles
* Plusieurs graphiques dans une figure : `plt.subplots()`
* Enregistrer une figure
* Exemple de synthèse
* Exercices de fin de chapitre

## Architecture de Matplotlib : `Figure` et `Axes`

Pour construire un graphique avec Matplotlib, il faut comprendre sa hiérarchie visuelle, reposant sur deux objets fondamentaux :

* **La `Figure`** : C'est la fenêtre globale ou la "feuille de dessin" qui contient l'ensemble des éléments graphiques (titres globaux, légende générale, sous-graphiques).
* **Les `Axes`** : Ce sont les zones de traçage proprement dites (le système d'axes $X$ et $Y$ où les courbes et barres sont dessinées). Une `Figure` peut contenir un ou plusieurs `Axes`.

Création explicite d'une Figure et d'un unique objet Axes

```python
import matplotlib.pyplot as plt

# Création explicite de la figure (fig) et de la zone de tracé (ax)
fig, ax = plt.subplots(figsize=(6, 3))

# Tracé d'une ligne simple sur l'objet ax
ax.plot([1, 2, 3, 4], [10, 20, 25, 30])

plt.show()

```

Manipulation des propriétés de l'objet Axes

L'objet `ax` centralise toutes les commandes de modification des axes, ce qui rend le code plus modulaire et lisible que l'utilisation de fonctions globales.

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots()

# Utilisation de la méthode .set() pour configurer l'axe en une seule ligne
ax.set(
    title="Évolution des performances",
    xlabel="Temps (en heures)",
    ylabel="Score technique"
)

ax.plot([0, 1, 2], [5, 12, 18])
plt.show()

```

## Tracer un graphique simple : `plt.plot()`

La fonction **`plt.plot()`** est la fonction universelle de Matplotlib pour tracer des graphiques linéaires ou des nuages de points reliés par des lignes. Elle accepte en arguments des listes ou des tableaux NumPy pour les coordonnées $X$ et $Y$.

Tracé linéaire basique à partir de listes Python

```python
import matplotlib.pyplot as plt

# Coordonnées des abscisses (X) et des ordonnées (Y)
jours = ['Lun', 'Mar', 'Mer', 'Jeu', 'Ven']
visiteurs = [120, 250, 180, 300, 420]

# Tracé du graphique linéaire
plt.plot(jours, visiteurs)
plt.show()

```

Personnalisation directe des traits et des marqueurs dans `plt.plot()`

Il est possible de modifier l'apparence des courbes directement via les arguments de la fonction `plt.plot()` (couleurs, épaisseur, style de ligne et symboles de sommets).

```python
import numpy as np

# Génération d'une onde sinusoïdale avec NumPy
x = np.linspace(0, 10, 50)
y = np.sin(x)

# Tracé avec une ligne en tirets ('.--'), de couleur verte, avec des marqueurs ronds ('o')
plt.plot(x, y, color='forestgreen', linestyle='--', linewidth=2, marker='o', markersize=4)
plt.show()

```

## Titres, étiquettes, légendes et styles

Un graphique professionnel doit être immédiatement compréhensible hors de son contexte. Cela nécessite l'ajout rigoureux de titres, d'étiquettes sur les axes (`xlabel`, `ylabel`), de légendes pour identifier les courbes, et éventuellement d'une grille de lecture.

Ajout de titres, libellés d'axes et de légendes

```python
import matplotlib.pyplot as plt

mois = ['Jan', 'Fév', 'Mar', 'Avr']
ca_2025 = [100, 120, 115, 130]
ca_2026 = [110, 135, 140, 155]

# Traçage des deux courbes avec des étiquettes (label)
plt.plot(mois, ca_2025, label='Année 2025', color='gray', linestyle=':')
plt.plot(mois, ca_2026, label='Année 2026', color='teal', linewidth=2)

# Ajout des éléments textuels et de la légende
plt.title("Comparatif annuel du Chiffre d'Affaires")
plt.xlabel("Mois de l'année")
plt.ylabel("Montant (k€)")
plt.legend()       # Affiche la légende en se basant sur les arguments 'label'
plt.grid(True, alpha=0.3) # Ajout d'une grille légère

plt.show()

```

Application d'un style global prédéfini

Matplotlib propose des thèmes visuels intégrés pour modifier instantanément l'esthétique de vos figures sans tout reconfigurer à la main.

```python
import matplotlib.pyplot as plt

# Activer un style visuel moderne (ex: 'seaborn-v0_8-whitegrid' ou 'ggplot')
plt.style.use('ggplot')

x = [1, 2, 3, 4]
y = [10, 25, 15, 30]

plt.plot(x, y, marker='s')
plt.title("Graphique avec le style 'ggplot'")
plt.xlabel("Index")
plt.ylabel("Valeur")
plt.show()

```

## Plusieurs graphiques dans une figure : `plt.subplots()`

Lorsque l'on souhaite comparer plusieurs métriques côte à côte sans surcharger un unique repère, on utilise un système de grille de sous-graphiques (*subplots*) via `plt.subplots(nrows, ncols)`.

Grille de 1 ligne et 2 colonnes

```python
import matplotlib.pyplot as plt

# Création d'une figure contenant 2 sous-graphiques côte à côte (1 ligne, 2 colonnes)
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(10, 4))

# Premier graphique (à gauche) : Les ventes
ax1.plot([1, 2, 3], [10, 20, 15], color='blue', marker='o')
ax1.set_title("Volume des Ventes")
ax1.set_xlabel("Semaine")

# Second graphique (à droite) : Les profits
ax2.plot([1, 2, 3], [2, 8, 5], color='orange', marker='s')
ax2.set_title("Bénéfices nets")
ax2.set_xlabel("Semaine")

# Ajustement automatique de l'espacement entre les graphiques
plt.tight_layout()
plt.show()

```

Grille matricielle de 2x2 sous-graphiques

```python
import matplotlib.pyplot as plt
import numpy as np

# Création d'une grille 2x2 (retourne une matrice d'axes 2x2)
fig, axes = plt.subplots(2, 2, figsize=(8, 6))

# Accès aux sous-graphiques par leurs indices de ligne et colonne
axes[0, 0].plot([1, 2], [3, 4], color='red')
axes[0, 0].set_title("Graphique A")

axes[0, 1].plot([1, 2], [4, 1], color='green')
axes[0, 1].set_title("Graphique B")

axes[1, 0].plot([1, 2], [2, 5], color='purple')
axes[1, 0].set_title("Graphique C")

axes[1, 1].plot([1, 2], [1, 1], color='brown')
axes[1, 1].set_title("Graphique D")

plt.tight_layout()
plt.show()

```

## Enregistrer une figure

Une fois le graphique mis en forme, il est souvent nécessaire de l'exporter sous forme de fichier image (PNG, JPG) ou de format vectoriel haute qualité (PDF, SVG) pour l'insérer dans un rapport ou un e-book. On utilise pour cela la méthode **`plt.savefig()`**.

Exportation standard au format PNG haute résolution

```python
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]
y = [10, 20, 25, 30]

plt.plot(x, y, marker='o')
plt.title("Rapport de performance")
plt.xlabel("Mois")
plt.ylabel("Score")

# Sauvegarde de la figure avec une résolution élevée (dpi=300) et un fond propre
plt.savefig("rapport_performance.png", dpi=300, bbox_inches='tight')
plt.close() # Ferme la figure pour libérer la mémoire

```

Exportation au format vectoriel SVG pour l'édition web ou print

```python
import matplotlib.pyplot as plt

# Traçage d'une figure simple
plt.figure(figsize=(5, 3))
plt.plot([1, 2, 3], [5, 10, 15], color='crimson')
plt.title("Graphique vectoriel")

# Export en SVG (les lignes restent nettes peu importe le zoom)
plt.savefig("graphique_vectoriel.svg", format='svg')
plt.close()

```

## Exemple de synthèse

Ce script complet rassemble l'ensemble des notions de Pyplot abordées dans ce chapitre : création d'une figure structurée, génération d'une grille multi-axes (`plt.subplots`), personnalisation des styles, ajout de légendes et exportation finale sur disque.

```python
import matplotlib.pyplot as plt
import numpy as np

# Préparation des données de test
semaines = ['S1', 'S2', 'S3', 'S4', 'S5']
trafic_web = [1200, 1500, 1350, 1800, 2100]
taux_conversion = [2.1, 2.4, 2.3, 2.8, 3.2]

# Création d'une figure avec 2 sous-graphiques verticaux (2 lignes, 1 colonne)
fig, (ax_trafic, ax_conv) = plt.subplots(2, 1, figsize=(8, 6), sharex=True)

# Premier graphique : Évolution du trafic web (Ligne bleue avec marqueurs)
ax_trafic.plot(semaines, trafic_web, color='royalblue', marker='o', linewidth=2, label='Visiteurs uniques')
ax_trafic.set_title("Tableau de bord marketing - Suivi mensuel", fontsize=14, fontweight='bold')
ax_trafic.set_ylabel("Nombre de visiteurs")
ax_trafic.legend(loc='upper left')
ax_trafic.grid(True, linestyle='--', alpha=0.5)

# Second graphique : Évolution du taux de conversion (Ligne orange avec carrés)
ax_conv.plot(semaines, taux_conversion, color='darkorange', marker='s', linewidth=2, label='Taux de conversion (%)')
ax_conv.set_xlabel("Semaines de l'année")
ax_conv.set_ylabel("Pourcentage (%)")
ax_conv.legend(loc='upper left')
ax_conv.grid(True, linestyle='--', alpha=0.5)

# Mise en page globale et exportation propre
plt.tight_layout()
plt.savefig("tableau_de_bord_marketing.png", dpi=300, bbox_inches='tight')

# Affichage à l'écran
plt.show()

```

## Exercices de fin de chapitre

**Exercice 1 :** Création d'une figure multi-axes personnalisée

Créez un script Python qui génère une figure de taille `(8, 4)` contenant deux graphiques côte à côte (1 ligne, 2 colonnes) à l'aide de `plt.subplots()`.
Sur le premier sous-graphique, tracez une fonction carrée ($y = x^2$ pour $x$ allant de 0 à 5) avec une ligne rouge en pointillés.
Sur le second sous-graphique, tracez une fonction racine carrée ($y = \sqrt{x}$) avec une ligne bleue continue, puis enregistrez l'ensemble sous le nom `exercices_maths.png` à 300 DPI.

**Exercice 2 :** Habillage et superposition de courbes

À l'aide de `plt.plot()`, tracez sur un même repère deux courbes représentant l'évolution hypothétique des ventes de deux produits ("Produit A" et "Produit B") sur 4 trimestres.
Personnalisez le graphique en ajoutant un titre explicite, des libellés sur les axes $X$ et $Y$, une légende positionnée en haut à droite, ainsi qu'une grille de fond en pointillés.
