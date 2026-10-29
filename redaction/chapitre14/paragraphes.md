# Chapitre 14 : Étude de cas

Analyse complète d'un jeu de données (Jeux Olympiques de Paris 2024) : https://github.com/rythamsaini/Paris-Olympic-Games-2024

Cette étude de cas finale rassemble l'ensemble des compétences et des bonnes pratiques abordées dans ce manuel. De l'importation d'un fichier Excel brut jusqu'à la restitution visuelle des résultats, nous allons mener une analyse complète de données en conditions réelles, en prenant pour support un jeu de données simulant les résultats des **Jeux Olympiques de Paris 2024**.

Dans ce chapitre :

* Présentation du jeu de données et des questions métier
* Chargement et inspection
* Nettoyage et préparation
* Agrégations et croisements
* Visualisation et restitution des résultats
* Exemple de synthèse
* Exercices de fin de chapitre

---

## Présentation du jeu de données et des questions métier

Pour mener une analyse pertinente, il est indispensable de définir clairement le contexte et les objectifs avant d'écrire la moindre ligne de code.

Nous disposons d'un fichier Excel nommé `paris_2024_resultats.xlsx` contenant les performances des athlètes et des délégations. Le classeur est structuré autour de plusieurs informations clés : le nom de l'athlète, son pays d'origine, la discipline sportive, le type de médaille obtenue (Or, Argent, Bronze) et la date de l'épreuve.

La direction d'un institut d'analyse sportive souhaite obtenir des réponses précises aux questions suivantes :

**Quelles sont les nations qui dominent le tableau général des médailles ?**
**Quelle est la répartition des médailles (Or, Argent, Bronze) par grande discipline sportive ?**
**Existe-t-il des temps forts temporels dans l'attribution des médailles au fil des jours de compétition ?**

---

## Chargement et inspection

La première étape consiste à charger le fichier Excel en mémoire à l'aide de `pd.read_excel()` et à auditer sa structure pour identifier d'éventuelles anomalies.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Chargement de la feuille principale du fichier Excel
df_brut = pd.read_excel('paris_2024_resultats.xlsx', sheet_name='Performances')

# Inspection de la structure globale
print("--- Dimensions du DataFrame ---")
print(df_brut.shape)

print("\n--- Aperçu des premières lignes ---")
print(df_brut.head())

print("\n--- Diagnostic technique (.info) ---")
df_brut.info(memory_usage='deep')

```

---

## Nettoyage et préparation

Les données brutes comportent souvent des imperfections : espaces superflus dans les noms de pays, valeurs manquantes ou types de données mal interprétés. Nous allons nettoyer ce jeu de données.

```python
# A. Suppression des doublons stricts éventuels
df_propre = df_brut.drop_duplicates().copy()

# Nettoyage des chaînes de caractères (pays et disciplines)
df_propre['Pays'] = df_propre['Pays'].str.strip().str.title()
df_propre['Discipline'] = df_propre['Discipline'].str.strip()

# Gestion des valeurs manquantes (ex: absence de médaille remplacée par 'Aucune')
df_propre['Type_Medaille'] = df_propre['Type_Medaille'].fillna('Aucune')

# Conversion des colonnes catégorielles pour optimiser la mémoire
df_propre['Pays'] = df_propre['Pays'].astype('category')
df_propre['Discipline'] = df_propre['Discipline'].astype('category')

# Conversion de la date de l'épreuve au format datetime
df_propre['Date_Epreuve'] = pd.to_datetime(df_propre['Date_Epreuve'])

print("--- Données après nettoyage ---")
print(df_propre.head())

```

---

## Agrégations et croisements

Une fois le jeu de données assaini, nous pouvons répondre aux questions métier en mobilisant les outils d'agrégation (`groupby`, `agg`) et de croisement (`pivot_table`).

### Le tableau des médailles par pays

Pour identifier les nations les plus performantes, nous filtrons les médailles attribuées (Or, Argent, Bronze) et regroupons les données par pays.

```python
# Filtrer uniquement les lignes ayant rapporté une médaille
df_medailles = df_propre[df_propre['Type_Medaille'].isin(['Or', 'Argent', 'Bronze'])]

# Création d'un tableau croisé dynamique pour compter les médailles par type et par pays
tableau_medailles = df_medailles.pivot_table(
    index='Pays',
    columns='Type_Medaille',
    values='Athlete',
    aggfunc='count',
    fill_value=0
).reindex(columns=['Or', 'Argent', 'Bronze']) # Réordonner les colonnes logiquement

# Calcul du total des médailles par pays et tri décroissant
tableau_medailles['Total'] = tableau_medailles.sum(axis=1)
tableau_medailles = tableau_medailles.sort_values(by=['Or', 'Argent', 'Bronze', 'Total'], ascending=False)

print("--- Top 5 du Tableau des Médailles ---")
print(tableau_medailles.head(5))

```

---

## Visualisation et restitution des résultats

Pour valoriser les résultats auprès des équipes métiers, nous traduisons ce tableau de synthèse en un graphique visuel clair et soigné à l'aide de l'écosystème Matplotlib/Pandas.

```python
# Sélectionner le Top 5 des pays pour la visualisation
top_5_pays = tableau_medailles.head(5)[['Or', 'Argent', 'Bronze']]

# Génération d'un diagramme en barres empilées
ax = top_5_pays.plot(
    kind='bar',
    stacked=True,
    color=['gold', 'silver', 'peru'],
    figsize=(9, 5)
)

# Habillage professionnel du graphique
plt.title("Top 5 des Nations - Jeux Olympiques de Paris 2024", fontsize=14, fontweight='bold')
plt.xlabel("Pays", fontsize=11)
plt.ylabel("Nombre de Médailles", fontsize=11)
plt.xticks(rotation=0)
plt.legend(title="Type de Médaille")
plt.grid(axis='y', linestyle='--', alpha=0.6)

# Sauvegarde de la figure haute résolution
plt.savefig("top_5_medailles_paris2024.png", dpi=300, bbox_inches='tight')
plt.show()

```

---

## Exemple de synthèse 

Le script ci-dessous résume le pipeline complet de l'analyse, de l'importation à la génération du rapport chiffré.

```python
import pandas as pd
import matplotlib.pyplot as plt

def executer_etude_cas(chemin_fichier):
    # 1. Chargement
    df = pd.read_excel(chemin_fichier, sheet_name='Performances')
    
    # 2. Nettoyage
    df = df.drop_duplicates().copy()
    df['Pays'] = df['Pays'].str.strip().str.title()
    df['Type_Medaille'] = df['Type_Medaille'].fillna('Aucune')
    
    # 3. Traitement analytique (Top 3 pays par nombre total de médailles d'or)
    podiums = df[df['Type_Medaille'] == 'Or']
    compte_or = podiums.groupby('Pays')['Type_Medaille'].count().reset_index(name='Total_Or')
    top_or = compte_or.sort_values(by='Total_Or', ascending=False).head(3)
    
    print("--- Résumé de l'analyse : Top 3 Or ---")
    print(top_or)
    
    return top_or

# Simulation d'appel (nécessite un fichier valide)
# executer_etude_cas('paris_2024_resultats.xlsx')

```

* **Méthodologie rigoureuse** : Toujours suivre les étapes séquentielles (Inspection $\rightarrow$ Nettoyage $\rightarrow$ Transformation $\rightarrow$ Restitution).
* **Sécurité des types** : L'utilisation de `.astype('category')` et du typage date sécurise les calculs futurs et allège l'empreinte mémoire.
* **Clarté de la restitution** : Un graphique propre doté d'un habillage explicite (titres, axes, légende) transforme des chiffres bruts en un outil d'aide à la décision percutant.

---

## Exercices de fin de chapitre

**Exercice 1 :** Analyse par discipline sportive

À l'aide du DataFrame nettoyé `df_propre` :

Réalisez un groupement (`groupby`) sur la colonne `Discipline` pour compter le nombre total d'épreuves disputées.
Identifiez la discipline qui a mobilisé le plus grand nombre d'athlètes uniques.

**Exercice 2 :** Suivi temporel des performances

Convertissez la colonne `Date_Epreuve` en index temporel (`DatetimeIndex`).
Utilisez un regroupement par date (`resample` ou `groupby` sur la date) pour tracer l'évolution quotidienne du nombre total de médailles décernées au cours de la quinzaine olympique.
