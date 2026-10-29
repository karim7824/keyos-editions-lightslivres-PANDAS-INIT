# Chapitre 9 : Gestion des dates et séries temporelles

La gestion des séries temporelles est un pilier fondamental en analyse de données. Pour manipuler des historiques de ventes, des logs ou des relevés, la fonction clé de Pandas est **`pd.date_range()`**. Elle permet de générer des séquences de dates régulières en spécifiant un point de départ, une fin (ou un nombre de périodes), et une fréquence (`freq`).

Dans ce chapitre :

* Les 4 modèles fondamentaux du temps dans Pandas
* Manipulation de plages temporelles avec `period_range`
* Combiner `pd.date_range()` avec un dictionnaire de données
* Modèles de temps
* Manipulation de plages temporelles
* Extraire le mois, le jour et l'heure
* Exemple de synthèse
* Exercices de fin de chapitre

---
## Les 4 modèles fondamentaux du temps dans Pandas

### `Timestamp` : Un instant précis (Le point dans le temps)

Un `Timestamp` représente une date et une heure exactes (l'équivalent optimisé du `datetime` de Python). C'est l'élément de base qui compose les index temporels (`DatetimeIndex`).

```python
import pandas as pd

# Créer un instant précis
instant = pd.Timestamp('2026-10-28 14:30:00')

print(instant)
print(type(instant))
# Extraction d'attributs clés
print(f"Année : {instant.year}, Mois : {instant.month}, Jour de la semaine : {instant.day_name()}")

```

---

### `Datetime` (et `to_datetime`) : La conversion et l'indexation

En pratique, les dates proviennent souvent de fichiers CSV sous forme de texte (chaînes de caractères). La fonction `pd.to_datetime()` permet de les convertir proprement en objets temporels exploitables.

```python
# Série de dates brutes (texte)
dates_texte = ['2026-10-01', '2026-10-02', '2026-10-03']

# Conversion en objets Datetime de Pandas
dates_dt = pd.to_datetime(dates_texte)

print(dates_dt)
# Résultat : DatetimeIndex(['2026-10-01', '2026-10-02', '2026-10-03'], dtype='datetime64[ns]', freq=None)

```

---

### `Timedelta` : Une durée ou un écart temporel

Un `Timedelta` représente une durée (par exemple : 5 jours, 3 heures, 45 secondes). Il permet d'effectuer des opérations arithmétiques directement sur les dates (ajouter ou soustraire du temps).

```python
# Définir une date de référence
date_achat = pd.Timestamp('2026-10-01')

# Créer un décalage (Timedelta) de 10 jours et 4 heures
delai = pd.Timedelta(days=10, hours=4)

# Calculer la date de livraison estimée
date_livraison = date_achat + delai

print(f"Date d'achat : {date_achat}")
print(f"Date de livraison estimée : {date_livraison}")

```

---

### `Period` : Une période ou un intervalle temporel fixe

Contrairement au `Timestamp` qui cible une *milliseconde* précise, le `Period` représente une **durée calendaire entière** (un mois complet, un trimestre, une année). C'est particulièrement utile pour la comptabilité et l'agrégation macro-économique.

```python
# Définir une période correspondant au mois d'octobre 2026
periode_mois = pd.Period('2026-10', freq='M')

print(periode_mois)
print(f"Début de la période : {periode_mois.start_time}")
print(f"Fin de la période : {periode_mois.end_time}")

# Avancer d'une période
mois_suivant = periode_mois + 1
print(f"Mois suivant : {mois_suivant}")

```

**En résumé pour ton lecteur :**

* **`Timestamp`** $\rightarrow$ Une date/heure isolée (ex: *le 28/10/2026 à 14h30*).
* **`Datetime`** $\rightarrow$ Le type de données global pour manipuler ces instants.
* **`Timedelta`** $\rightarrow$ Un intervalle de temps mathématique (ex: *+ 7 jours*).
* **`Period`** $\rightarrow$ Une tranche calendaire fixe (ex: *tout le mois d'octobre 2026*).
Créer des intervalles de dates par **jour** et par **mois**, accompagnés d'exemples pratiques.

---

## Manipulation de plages temporelles avec `period_range`

### intervalle de jours
Générer un intervalle de jours (`freq='D'`). Pour créer une séquence quotidienne (par exemple, pour simuler un historique de ventes sur une semaine) :

```python
import pandas as pd

# Créer une série de 7 jours consécutifs à partir d'aujourd'hui
jours = pd.date_range(start='2026-10-01', periods=7, freq='D')

print(jours)
# Résultat : DatetimeIndex(['2026-10-01', '2026-10-02', ..., '2026-10-07'], dtype='datetime64[ns]', freq='D')

```

---
### intervalle de mois

Générer un intervalle de mois (`freq='MS'` ou `'M'`). Pour planifier des bilans ou analyser des données mensuelles, on utilise la fréquence mensuelle.

* `'MS'` (Month Start) place la date au **premier jour** du mois.
* `'M'` place la date au **dernier jour** du mois.

```python
# Créer les dates de début de mois pour l'année 2026 (12 mois)
mois_debut = pd.date_range(start='2026-01-01', periods=12, freq='MS')

print(mois_debut)
# Résultat : DatetimeIndex(['2026-01-01', '2026-02-01', '2026-03-01', ..., '2026-12-01'], freq='MS')

```

---

## Combiner `pd.date_range()` avec un dictionnaire de données

Créer un DataFrame temporel pour nos produits. On peut combiner `pd.date_range()` avec un dictionnaire de données pour structurer un DataFrame prêt à l'analyse temporelle (par exemple, suivre les ventes quotidiennes d'un Clavier) :

```python
import numpy as np

# Générer un index de 5 jours consécutifs
dates = pd.date_range(start='2026-10-01', periods=5, freq='D')

# Associer ces dates à un DataFrame de ventes
df_ventes_journalieres = pd.DataFrame({
    'Date': dates,
    'Produit': ['Clavier', 'Clavier', 'Clavier', 'Clavier', 'Clavier'],
    'Quantite_Vendue': [3, 5, 2, 8, 4]
})

# Transformer la colonne 'Date' en index temporel (très utile pour Pandas)
df_ventes_journalieres = df_ventes_journalieres.set_index('Date')

print(df_ventes_journalieres)

```

---

## Modèles de temps

Pandas offre une gestion précise du temps grâce à quatre types d'objets principaux : `Timestamp` pour un instant précis, `Datetime` pour les types de données temporelles, `Timedelta` pour les durées, et `Period` pour représenter un intervalle de temps défini. Ces outils permettent de manipuler des calendriers complexes avec une grande rigueur syntaxique.

```python
import pandas as pd

# Création d'un instant précis (Timestamp)
instant = pd.Timestamp('2026-07-06 18:00:00')

# Calcul d'une durée (Timedelta)
duree = pd.Timedelta(days=5)
nouveau_temps = instant + duree

```
---

## Manipulation de plages temporelles

La fonction `period_range` permet de générer des séquences régulières de périodes temporelles sur un intervalle donné. Cette fonctionnalité est particulièrement utile pour créer des index basés sur des fréquences spécifiques, comme des mois ou des trimestres.

```python
# Génération d'une plage de périodes mensuelles
plage = pd.period_range(start='2026-01', periods=3, freq='M')

# Utilisation dans une Série
s = pd.Series([10, 20, 30], index=plage)

```
---
## Extraire le mois, le jour et l'heure

Pour extraire des composantes spécifiques (année, mois, jour, heure, minute) à partir d'une colonne de type date (`datetime`) dans Pandas, on utilise l'accesseur **`.dt`**.

Les principaux attributs de l'accesseur `.dt` :

| Attribut | Ce qu'il extrait | Exemple de résultat |
| --- | --- | --- |
| `.dt.year` | L'année | `2026` |
| `.dt.month` | Le numéro du mois (1-12) | `10` |
| `.dt.day` | Le numéro du jour du mois (1-31) | `28` |
| `.dt.hour` | L'heure (0-23) | `14` |
| `.dt.minute` | La minute (0-59) | `30` |
| `.dt.day_name()` | Le nom complet du jour (ex: *Monday*) | `'Wednesday'` |
| `.dt.month_name()` | Le nom complet du mois (ex: *October*) | `'October'` |

```python
import pandas as pd

# Création d'un DataFrame avec des dates et heures précises
df_logs = pd.DataFrame({
    'Timestamp_Brut': ['2026-10-28 08:30:00', '2026-10-29 14:15:45', '2026-10-30 19:00:30'],
    'Valeur': [100, 250, 180]
})

# Conversion indispensable en objet datetime
df_logs['Date_Heure'] = pd.to_datetime(df_logs['Timestamp_Brut'])

# Extraction du mois, du jour et de l'heure via l'accesseur .dt
df_logs['Mois'] = df_logs['Date_Heure'].dt.month
df_logs['Jour'] = df_logs['Date_Heure'].dt.day
df_logs['Heure'] = df_logs['Date_Heure'].dt.hour
df_logs['Jour_Semaine'] = df_logs['Date_Heure'].dt.day_name() # Nom du jour en anglais par défaut

print(df_logs[['Date_Heure', 'Mois', 'Jour', 'Heure', 'Jour_Semaine']])

```
---

## Exemple de synthèse

Rapport de synthèse des ventes par heure de la journée.Imaginons un journal de transactions e-commerce avec des horodatages précis. Nous voulons savoir à quelles heures la structure réalise le plus de chiffre d'affaires.

```python
import pandas as pd

# Données de ventes brutes avec horodatage
df_transactions = pd.DataFrame({
    'Horodatage': [
        '2026-10-01 09:15:00', '2026-10-01 10:30:00', '2026-10-01 14:10:00',
        '2026-10-02 09:45:00', '2026-10-02 14:20:00', '2026-10-02 18:05:00',
        '2026-10-03 10:00:00', '2026-10-03 14:50:00', '2026-10-03 18:30:00'
    ],
    'Montant': [50.0, 25.0, 120.0, 50.0, 80.0, 250.0, 25.0, 75.0, 100.0]
})

# Conversion indispensable en objet datetime
df_transactions['Horodatage'] = pd.to_datetime(df_transactions['Horodatage'])

# Extraction des composantes temporelles via l'accesseur .dt
df_transactions['Heure'] = df_transactions['Horodatage'].dt.hour
df_transactions['Jour'] = df_transactions['Horodatage'].dt.day_name()

# Synthèse : Grouper par heure pour analyser les créneaux les plus rentables
synthese_horaire = df_transactions.groupby('Heure').agg(
    Nombre_Commandes=('Montant', 'count'),  # Volume de transactions
    Chiffre_Affaires=('Montant', 'sum'),    # CA total cumulé
    Panier_Moyen=('Montant', 'mean')        # Montant moyen par commande
).reset_index()

print(synthese_horaire)

```

**La chaîne logique :** On convertit d'abord la colonne brute en `datetime`, on extrait les informations fines (`.dt.hour`, `.dt.day_name()`), puis on structure l'analyse macroscopique avec `.groupby().agg()`.
**La puissance du multi-indicateur :** En une seule passe, `.agg()` permet d'isoler à la fois le volume, la somme et la moyenne par tranche horaire, fournissant un tableau de bord directement exploitable par les équipes métiers.

---
## Exercices de fin de chapitre 

**Exercice 1 :** Créez un `Timestamp` correspondant à la date d'aujourd'hui et ajoutez-lui un `Timedelta` de 48 heures.

**Exercice 2:** Utilisez `period_range` pour générer une liste de 6 périodes trimestrielles commençant au premier trimestre 2026.
