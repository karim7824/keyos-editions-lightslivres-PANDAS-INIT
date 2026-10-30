# Chapitre 17 : Cheatsheet NumPy / Pandas / Matplotlib

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

```

---

## 1. NumPy : le socle

| Besoin | Code |
| --- | --- |
| Créer un tableau | `np.array([1, 2, 3], dtype=np.float64)` |
| Forme / type | `a.shape` · `a.dtype` · `a.ndim` |
| Séquence | `np.arange(0, 10, 2)` · `np.linspace(0, 1, 50)` |
| Slicing 1D | `a[0:3]` · `a[::2]` (fin exclue) |
| Slicing 2D | `m[2, 0]` · `m[:, 1]` · `m[1, :]` |
| Vectorisation | `a * 1.2` (aucune boucle `for`) |
| Broadcasting | `m + v[:, np.newaxis]` |
| Réduction | `np.sum(m, axis=0)` · `np.mean(m, axis=1)` |
| NaN | `np.nan` · `np.isnan(a)` · `a[~np.isnan(a)]` |
| Sans NaN | `np.nansum(a)` · `np.nanmean(a)` · `np.nan_to_num(a, nan=0)` |

> **`axis=0`** ↓ descend les lignes (résultat **par colonne**)  
> **`axis=1`** → traverse les colonnes (résultat **par ligne**)  
> ⚠️ `nan` est contagieux : `np.sum([1, np.nan]) → nan`

---

## 2. Series

Une `Series` = **valeurs** (ndarray) + **index** (labels).

```python
pd.Series([10, 20], index=['a', 'b'])      # liste + index
pd.Series({'FR': 'Paris', 'IT': 'Rome'})   # dict -> clés = index
pd.Series(25, index=['L', 'M', 'Me'])      # scalaire diffusé
```

| Inspection | Rôle |
| --- | --- |
| `.values` · `.index` | données brutes · labels |
| `.shape` · `.size` · `.dtype` | `(n,)` · nb d'éléments · type |
| `.head(n)` · `.tail(n)` | n premières / dernières valeurs |
| `.describe()` | résumé statistique |

### Sélection

| Méthode | Base | Fin du slice |
| --- | --- | --- |
| `s.loc['a':'c']` | **labels** | **incluse** |
| `s.iloc[0:3]` | **positions** | exclue |
| `s[s > 1]` | masque booléen | — |
| `s.loc[['c', 'a']]` | liste de labels (ordre libre) | — |

### Arithmétique et alignement

```python
s1 + s2                      # aligné sur les LABELS, NaN si absent
s1.add(s2, fill_value=0)     # remplace l'absent par 0 avant calcul
# méthodes : .add .sub .mul .div .pow
```

### Index

```python
s.index = ['A', 'B', 'C']              # remplacement global (même taille)
s.rename({'B': 'Delta'})               # renommage ciblé
s.reindex(['A', 'C', 'Z'], fill_value=0)  # réaligne, crée les manquants
```

### Transformer

```python
s * 9/5 + 32                                  # vectorisé (le plus rapide)
s.apply(lambda x: 'Chaud' if x > 22 else 'Frais')  # logique perso
s.map({'H': 'Hivernal', 'P': 'Tempéré'})       # dict ; absent -> NaN
s.astype('int64')  # 'float64' | str | bool | 'category'
```

---

## 3. DataFrame : création et structure

```python
pd.DataFrame({'Nom': ['A', 'B'], 'Age': [25, 30]})          # dict de listes
pd.DataFrame([('Paris', 2.1), ('Lyon', 0.5)], columns=['Ville', 'Pop'])  # tuples
pd.DataFrame([{'ID': 1, 'N': 'A'}, {'ID': 2}])             # liste de dicts (NaN auto)
pd.DataFrame({'X': ser1, 'Y': ser2})                        # Series alignées
```

| Attribut | Rôle |
| --- | --- |
| `.shape` · `.size` · `.ndim` | `(lignes, colonnes)` · nb cellules · 2 |
| `.index` · `.columns` · `.axes` | labels lignes · colonnes · les deux |
| `.dtypes` | type par colonne |
| `.values` / `.to_numpy()` | retour en ndarray |
| `.info(memory_usage='deep')` | types, non-null, mémoire |
| `.describe(include='all')` | stats (num + texte) |
| `.head()` · `.tail()` | aperçu |

> Une colonne `df['col']` est une **Series**. Axes : `axis=0` lignes, `axis=1` colonnes.

### Import

```python
pd.read_csv('f.csv', sep=';', encoding='utf-8', index_col='ID',
            na_values=['NA', '?'], parse_dates=['date'])
pd.read_excel('f.xlsx', sheet_name='Q1', skiprows=2, usecols='A:D')
```

---

## 4. Colonnes

```python
df['TTC'] = df['HT'] * 1.2             # nouvelle colonne calculée
df['X'] = np.nan                       # colonne vide
df[['Produit', 'TTC']]                 # sélection + ordre
df.rename(columns={'HT': 'Prix_HT'})   # renommer
df.drop(columns=['Total'])             # supprimer (= axis=1)
df.filter(like='Prix')                 # colonnes dont le NOM contient 'Prix'
df.filter(items=['A', 'B'])            # colonnes par nom exact
```

## 5. Lignes et filtrage

```python
df[df['Score'] > 80]                              # masque booléen
df[(df['Score'] > 70) & (df['Ville'] == 'Paris')]  # ET  (parenthèses obligatoires)
df[(df['A'] > 1) | (df['B'] < 5)]                  # OU
df[~(df['A'] > 1)]                                 # NON
df[df['Ville'].isin(['Paris', 'Lyon'])]            # appartenance
df.query("Prix > 50 and Stock > 0")                # syntaxe texte
df.loc[4] = ['Eve', 95, 'Bordeaux']                # ajouter une ligne
df.drop(index=[0, 3])                              # supprimer des lignes
```

## 6. Index, `.loc` et `.iloc`

```python
df.set_index('Code')       # colonne -> index
df.reset_index()           # index -> colonne
df.rename(index={'E1': 'X1'})
```

| Besoin | `.loc` (labels, fin incluse) | `.iloc` (positions, fin exclue) |
| --- | --- | --- |
| Une cellule | `df.loc['Alice', 'Score']` | `df.iloc[1, 0]` |
| Une ligne | `df.loc['Bob']` | `df.iloc[0]` |
| Tranche | `df.loc['A':'B', 'Score':'Ville']` | `df.iloc[0:2, 0:1]` |
| Filtre + colonne | `df.loc[df['Score'] > 80, 'Ville']` | — |

---

## 7. Tri, itération, apply, pipe

```python
df.sort_values('Total', ascending=False)
df.sort_values(['Qte', 'Prix'], ascending=[False, True])   # multi-critères
df.sort_index()

df.at[idx, 'Score'] += 1               # accès cellule rapide (dans une boucle)
for i, row in df.iterrows(): ...       # lent : préférer la vectorisation

df['X'].apply(lambda x: x.upper())                       # par valeur
df.apply(lambda r: f"{r['A']}-{r['B']}", axis=1)         # par ligne
df.pipe(f1, taux=0.1).pipe(f2, seuil=190)                # chaîner des fonctions

df2 = df.copy()                        # copie indépendante
df[df['Q'] > 2].copy()                 # évite le SettingWithCopyWarning
```

| Outil | S'applique à | Usage |
| --- | --- | --- |
| `.map(dict)` | Series | codes → libellés (absent = NaN) |
| `.apply(f)` | Series / DataFrame | fonction perso (`axis=1` = par ligne) |
| `.pipe(f)` | DataFrame | pipeline lisible |
| `df.map(f)` | DataFrame | cellule par cellule (ex-`applymap`) |

---

## 8. Nettoyage

### Valeurs manquantes

```python
df.isna().sum()                    # nb de NaN par colonne
df.notna()                         # masque inverse
df.dropna()                        # supprime les lignes avec NaN
df.fillna({'Prix': df['Prix'].mean(), 'Stock': 0})
s.interpolate()                    # estimation linéaire (séries temporelles)
s.mask(s <= 0, np.nan)             # remplace selon condition
```

### Doublons et aberrants

```python
df.duplicated()                                  # masque des doublons
df.drop_duplicates(subset=['ID'], keep='first')  # keep: 'first' | 'last'
df[(df['Prix'] > 0) & (df['Prix'] < 5000)]       # seuil métier
s.quantile(0.05) ; s.quantile(0.95)              # bornes par quantiles
```

### Texte avec `.str`

```python
df['E'].str.strip().str.lower()
df['V'].str.strip().str.capitalize()    # .upper() .title() .contains() .split()
df['M'].str.replace('€', '', regex=False).str.strip().astype(int)
```

### Type `category`

```python
df['Cat'] = df['Cat'].astype('category')   # très gros gain mémoire si répétitif
```

---

## 9. Dates et temps

| Objet | Rôle | Exemple |
| --- | --- | --- |
| `Timestamp` | un instant | `pd.Timestamp('2026-10-28 14:30')` |
| `Timedelta` | une durée | `pd.Timedelta(days=10, hours=4)` |
| `Period` | une période calendaire | `pd.Period('2026-10', freq='M')` |
| `DatetimeIndex` | index de dates | `pd.date_range(...)` |

```python
pd.to_datetime(df['d'])                                  # texte -> datetime
pd.date_range('2026-10-01', periods=7, freq='D')         # jours
pd.date_range('2026-01-01', periods=12, freq='MS')       # 1er du mois
pd.period_range('2026-01', periods=6, freq='Q')          # trimestres
s.loc['2026-10-02':'2026-10-04']                         # slicing par date
s.loc['2026-10']                                         # tout un mois
```

| Accesseur `.dt` | Donne |
| --- | --- |
| `.dt.year` `.dt.month` `.dt.day` | année, mois, jour |
| `.dt.hour` `.dt.minute` | heure, minute |
| `.dt.day_name()` `.dt.month_name()` | noms en toutes lettres |

> Sur un `DatetimeIndex` : `df.index.month`, `df.index.day_name()` (sans `.dt`).  
> Fréquences : `D` jour · `MS` début de mois · `M` fin de mois (`ME` en pandas ≥ 2.2) · `Q` trimestre · `Y` année.

---

## 10. Statistiques et `groupby`

```python
df['x'].mean()  # .sum() .min() .max() .std() .median() .count() .idxmax()
df.loc[df['Total'].idxmax(), 'Produit']     # ligne du maximum

df.agg('sum')                                # une fonction
df.agg(['mean', 'std'])                      # plusieurs
df.agg({'Prix': ['mean', 'max'], 'Qte': 'sum'})   # par colonne
```

### Split → Apply → Combine

```python
df.groupby('Cat')['Total'].sum()

df.groupby('Cat').agg(                       # agrégation nommée
    Prix_Moyen=('Prix_HT', 'mean'),
    Volume=('Quantite', 'sum'),
    CA=('Total', 'sum'),
).reset_index()                              # index -> colonne
```

---

## 11. Fusion de données

| Outil | Lie par | Cas d'usage |
| --- | --- | --- |
| `pd.concat([a, b], ignore_index=True)` | position (axis=0 lignes) | empiler des fichiers mensuels |
| `pd.concat([a, b], axis=1)` | index commun | juxtaposer des colonnes |
| `pd.merge(a, b, on='ID', how='left')` | **colonnes** clés | relier ventes ↔ catalogue |
| `a.join(b)` | **index** | tables déjà indexées pareil |

| `how=` | Garde |
| --- | --- |
| `inner` (défaut) | clés présentes des deux côtés |
| `left` / `right` | toutes les lignes de gauche / droite |
| `outer` | toutes les lignes des deux tables |

Clés de noms différents : `pd.merge(a, b, left_on='x', right_on='y')`

---

## 12. Structures multidimensionnelles

```python
pd.MultiIndex.from_tuples([('Paris', 'Q1'), ('Paris', 'Q2')], names=['Ville', 'Trim'])
pd.MultiIndex.from_product([villes, mois], names=['Ville', 'Mois'])

df.loc['Paris']                       # un niveau
df.loc[('Paris', 'Q1'), 'CA']         # tuple pour plusieurs niveaux
df.loc[pd.IndexSlice[:, 'Q1'], :]     # tranche complexe
df.sort_index()                       # tri requis avant slicing
```

| Fonction | Rôle |
| --- | --- |
| `df.pivot(index, columns, values)` | pivote **sans agrégation** (doublons = erreur) |
| `df.pivot_table(index, columns, values, aggfunc='sum', fill_value=0)` | pivote **avec agrégation** |
| `pd.crosstab(df['A'], df['B'])` | table de fréquences |
| `df.stack()` | colonnes → lignes |
| `df.unstack()` | lignes → colonnes |
| `df.melt(id_vars=['X'], var_name='V', value_name='Val')` | large → long |

---

## 13. Export / persistance

| Format | Lecture | Écriture |
| --- | --- | --- |
| CSV | `pd.read_csv()` | `df.to_csv('f.csv', index=False, sep=';', encoding='utf-8-sig')` |
| JSON | `pd.read_json(orient='records')` | `df.to_json('f.json', orient='records', force_ascii=False, indent=4)` |
| XML | `pd.read_xml(xpath='.//item')` | `df.to_xml('f.xml', root_name='R', row_name='L', index=False)` |
| Excel | `pd.read_excel()` | `df.to_excel('f.xlsx', index=False)` |
| Pickle | `pd.read_pickle()` | `df.to_pickle('f.pkl')` (types conservés, propre à Python) |

> ✅ `index=False` sauf index porteur de sens · `utf-8-sig` pour Excel FR

---

## 14. Visualisation

### Pandas `df.plot()` (prototypage rapide)

| `kind=` | Usage |
| --- | --- |
| `line` | tendance, séries temporelles |
| `bar` / `barh` | comparer des catégories |
| `hist` | distribution d'une variable |
| `scatter` (`x=`, `y=`) | corrélation entre 2 variables |
| `pie` | parts (avec modération) |

```python
df.groupby('Cat')['CA'].sum().plot(kind='bar', color='skyblue', rot=0, title='CA')
df.plot(kind='line', subplots=True, layout=(2, 2), figsize=(10, 6), sharex=True)
```

Options : `figsize=(l, h)` · `color` · `colormap` · `linewidth` · `marker` · `rot` · `title` · `stacked=True`

### Matplotlib (contrôle total)

```python
fig, ax = plt.subplots(figsize=(8, 4))              # Figure + Axes
ax.plot(x, y, label='2026', color='teal', linestyle='--', marker='o', linewidth=2)
ax.set(title='Titre', xlabel='X', ylabel='Y')
ax.legend(loc='upper right'); ax.grid(True, alpha=0.3)

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(10, 4))   # grille 1×2
fig, axes = plt.subplots(2, 2)                          # axes[0, 1]...
plt.tight_layout()
plt.savefig('fig.png', dpi=300, bbox_inches='tight')    # PNG / .svg / .pdf
plt.show(); plt.close()
plt.style.use('ggplot')
```

### Seaborn (statistiques)

```python
import seaborn as sns
sns.set_theme(style='whitegrid')
sns.boxplot(data=df, x='Cat', y='Prix', hue='Canal', palette='Set2')
```

| Outil | Force |
| --- | --- |
| `df.plot()` | rapide, intégré |
| Matplotlib | contrôle chirurgical |
| Seaborn | stats + esthétique, `hue` |

---

## 15. Pipeline type d'analyse

```text
Charger  →  Inspecter  →  Nettoyer  →  Transformer  →  Agréger  →  Visualiser
read_*      shape/info     dropna       colonnes       groupby     plot / savefig
            describe       drop_dup     map/apply      pivot_table
                           .str/astype  merge/concat
```

```python
df = pd.read_excel('f.xlsx').drop_duplicates().copy()
df['Pays'] = df['Pays'].str.strip().str.title().astype('category')
df['Date'] = pd.to_datetime(df['Date'])
top = df.groupby('Pays').agg(Total=('Medaille', 'count')).nlargest(5, 'Total')
top.plot(kind='bar'); plt.savefig('top5.png', dpi=300, bbox_inches='tight')
```

---

## ⚠️ Pièges à retenir

| Piège | Réflexe |
| --- | --- |
| `.loc[a:b]` inclut `b`, `.iloc[a:b]` l'exclut | choisir l'outil selon labels / positions |
| `&` `|` `~` sans parenthèses | `(cond1) & (cond2)` |
| `SettingWithCopyWarning` | `.copy()` après un filtre |
| `NaN` contagieux (NumPy) | `nansum`, `nanmean` · Pandas ignore NaN par défaut |
| `int64` ne supporte pas `NaN` | `.astype('float64')` d'abord |
| `.map(dict)` renvoie NaN si clé inconnue | prévoir un `fillna` ou `.replace()` |
| `concat` garde les anciens index | `ignore_index=True` |
| Boucles `iterrows` lentes | vectoriser, sinon `.apply` |
| Accents illisibles sous Excel | `encoding='utf-8-sig'`, `sep=';'` |
| `df.sort_values` / `drop` / `fillna` ne modifient pas sur place | réassigner : `df = df.drop(...)` |
