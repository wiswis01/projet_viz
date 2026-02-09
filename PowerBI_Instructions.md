# Instructions Power BI - Dashboard Amazon India

## Fichiers a importer

| Fichier | Description | Lignes |
|---|---|---|
| `amazon_clean.csv` | Table principale des produits | 1 351 |
| `amazon_reviews.csv` | Reviews eclatees avec sentiment | 20 034 |
| `amazon_wordcloud.csv` | Frequence des mots par categorie | 35 174 |

---

## Etape 1 : Import et Power Query

### 1.1 Importer les 3 fichiers CSV
- Accueil > Obtenir des donnees > Texte/CSV
- Importer les 3 fichiers un par un
- Pour chacun, cliquer **Transformer les donnees** (pas Charger directement)

### 1.2 Typage dans Power Query

**Table `amazon_clean` :**

| Colonne | Type a appliquer |
|---|---|
| `product_id` | Texte |
| `product_name` | Texte |
| `discounted_price` | Nombre decimal |
| `actual_price` | Nombre decimal |
| `discount_percentage` | Nombre entier |
| `rating` | Nombre decimal |
| `rating_count` | Nombre entier |
| `main_category` | Texte |
| `sub_category` | Texte |
| `product_type` | Texte |
| `savings` | Nombre decimal |
| `price_range` | Texte |
| `rating_category` | Texte |
| `review_count` | Nombre entier |
| `sentiment_mean` | Nombre decimal |
| `pct_positif` | Nombre decimal |
| `pct_negatif` | Nombre decimal |
| `sentiment_label` | Texte |

**Table `amazon_reviews` :**

| Colonne | Type |
|---|---|
| `product_id` | Texte |
| `main_category` | Texte |
| `review_title` | Texte |
| `review_content` | Texte |
| `sentiment_score` | Nombre decimal |
| `sentiment_label` | Texte |

**Table `amazon_wordcloud` :**

| Colonne | Type |
|---|---|
| `word` | Texte |
| `main_category` | Texte |
| `frequency` | Nombre entier |

Cliquer **Fermer et appliquer** apres le typage.

### 1.3 Relations entre tables

Aller dans **Modele** (icone sur la gauche) :
- Creer une relation : `amazon_clean[product_id]` → `amazon_reviews[product_id]`
  - Cardinalite : **1 a plusieurs (1:N)**
  - Direction du filtre croise : **Unique**
- La table `amazon_wordcloud` est independante (pas de relation directe, filtrage via slicer `main_category`)

---

## Etape 2 : Mesures DAX

Aller dans **Modelisation > Nouvelle mesure** et creer chaque mesure :

```
Avg Rating = AVERAGE('amazon_clean'[rating])
```

```
Total Products = COUNTROWS('amazon_clean')
```

```
Avg Discount = AVERAGE('amazon_clean'[discount_percentage])
```

```
Avg Savings = AVERAGE('amazon_clean'[savings])
```

```
Positive Review % =
    DIVIDE(
        COUNTROWS(FILTER('amazon_reviews', 'amazon_reviews'[sentiment_label] = "Positif")),
        COUNTROWS('amazon_reviews')
    ) * 100
```

```
Total Reviews = COUNTROWS('amazon_reviews')
```

---

## Etape 3 : Les 7 Visualisations

### Layout recommande

```
+----------------------------------------------+
|  [KPI Cards x4]        [Jauge Rating]        |
+----------------------------------------------+
|  [1. Treemap]        |  [2. Scatter Plot]     |
|  Categories          |  Prix vs Rating        |
+----------------------+------------------------+
|  [3. Barres empilees]|  [4. Combo Chart]      |
|  Discount/Categorie  |  Distribution prix     |
+----------------------+------------------------+
|  [5. Matrice Heatmap]                         |
|  Rating par categorie x remise                |
+----------------------------------------------+
|  [6. Donut Sentiment]|  [7. Word Cloud]       |
+----------------------+------------------------+
```

---

### En-tete : KPI Cards + Jauge

**4 Cartes KPI :**
1. Inserer > Visuel > **Carte**
   - Champ : `Total Products` → Titre : "Total Produits"
2. Carte → Champ : `Avg Rating` → Titre : "Rating Moyen"
3. Carte → Champ : `Avg Discount` → Titre : "Remise Moyenne (%)"
4. Carte → Champ : `Positive Review %` → Titre : "% Avis Positifs"

**Jauge :**
- Inserer > Visuel > **Jauge**
- Valeur : `Avg Rating`
- Valeur cible : 4.5 (saisir manuellement dans le volet Format)
- Min : 0, Max : 5
- Titre : "Rating Moyen vs Objectif 4.5"

---

### Graphique 1 — Treemap : Repartition des produits par categorie

1. Inserer > Visuel > **Treemap**
2. Configuration :
   - **Categorie** : `main_category`, puis ajouter `sub_category` en dessous
   - **Valeurs** : `product_id` (Count)
3. Format :
   - Titre : "Repartition des produits par categorie"
   - Etiquettes de donnees : Activer

---

### Graphique 2 — Scatter Plot : Prix vs Rating

1. Inserer > Visuel > **Nuage de points (Scatter Chart)**
2. Configuration :
   - **Axe X** : `discounted_price` (Ne pas resumer / ou Moyenne)
   - **Axe Y** : `rating` (Ne pas resumer / ou Moyenne)
   - **Taille** : `rating_count` (Somme)
   - **Legende** : `main_category`
   - **Details** : `product_name` (optionnel, pour les infobulles)
3. Format :
   - Titre : "Prix vs Rating par categorie"
   - Activer les infobulles

---

### Graphique 3 — Barres empilees : Discount moyen par categorie et rating

1. Inserer > Visuel > **Graphique a barres empilees**
2. Configuration :
   - **Axe Y** : `main_category`
   - **Axe X** : `discount_percentage` (Moyenne)
   - **Legende** : `rating_category`
3. Format :
   - Titre : "Remise moyenne par categorie et niveau de rating"
   - Trier par Axe X decroissant

---

### Graphique 4 — Combo Chart : Distribution des prix avec rating moyen

1. Inserer > Visuel > **Graphique en colonnes et courbes**
2. Configuration :
   - **Axe X** : `price_range`
   - **Valeurs des colonnes** : `product_id` (Count)
   - **Valeurs de la ligne** : `rating` (Moyenne)
3. Format :
   - Titre : "Distribution des prix et rating moyen"
   - Axe Y gauche : "Nombre de produits"
   - Axe Y droit : "Rating moyen"
   - Ordre personnalise du `price_range` : Budget < Mid-range < Premium < Luxe
     (Pour ordonner : creer une colonne de tri `price_range_order` si necessaire,
      ou trier dans Power Query avec une colonne index)

**Astuce tri `price_range` :** Dans Power Query, ajouter une colonne conditionnelle :
- Si `price_range` = "Budget <500" alors 1
- Si `price_range` = "Mid-range 500-2000" alors 2
- Si `price_range` = "Premium 2000-10000" alors 3
- Si `price_range` = "Luxe >10000" alors 4
- Puis dans le modele : cliquer sur `price_range` > Trier par colonne > `price_range_order`

---

### Graphique 5 — Matrice Heatmap : Rating par categorie x tranche de remise

1. D'abord, creer une colonne calculee DAX pour les tranches de remise :
```
Discount Tranche =
    SWITCH(
        TRUE(),
        'amazon_clean'[discount_percentage] < 25, "0-25%",
        'amazon_clean'[discount_percentage] < 50, "25-50%",
        'amazon_clean'[discount_percentage] < 75, "50-75%",
        "75-100%"
    )
```

2. Inserer > Visuel > **Matrice**
3. Configuration :
   - **Lignes** : `main_category`
   - **Colonnes** : `Discount Tranche`
   - **Valeurs** : `rating` (Moyenne)
4. Mise en forme conditionnelle :
   - Clic droit sur une cellule de valeur > Mise en forme conditionnelle > Couleur d'arriere-plan
   - Style : Gradient
   - Couleur minimale : Rouge (rating faible)
   - Couleur maximale : Vert (rating eleve)
5. Format :
   - Titre : "Rating moyen par categorie et tranche de remise"

---

### Graphique 6 — Donut Chart : Repartition du sentiment des avis

1. Inserer > Visuel > **Graphique en anneau (Donut)**
2. Configuration (utiliser la table `amazon_reviews`) :
   - **Legende** : `sentiment_label`
   - **Valeurs** : `sentiment_label` (Count)
3. Format :
   - Titre : "Repartition du sentiment des avis"
   - Etiquettes : Afficher categorie + pourcentage
   - Couleurs : Vert (Positif), Gris (Neutre), Rouge (Negatif)
4. Drill-down :
   - Ajouter un **slicer** sur `main_category` a cote du donut
   - Cela permettra de filtrer le sentiment par categorie

---

### Graphique 7 — Word Cloud : Mots les plus frequents

**Prerequis :** Telecharger le visuel Word Cloud depuis AppSource
- Inserer > Visuel > "..." > Obtenir d'autres visuels
- Rechercher "Word Cloud"
- Installer le visuel officiel Microsoft

1. Inserer le visuel **Word Cloud**
2. Configuration (utiliser la table `amazon_wordcloud`) :
   - **Categorie** : `word`
   - **Valeurs** : `frequency` (Somme)
3. Format :
   - Titre : "Mots les plus frequents dans les avis"
   - Nombre de mots max : 100-150
   - Rotation : Desactiver (pour lisibilite)
4. Ajouter un **slicer** sur `main_category` (de la table `amazon_wordcloud`)
   - Cela filtre le nuage de mots par categorie

---

## Etape 4 : Filtres et interactions

### Slicers recommandes
- **Slicer `main_category`** : en haut du dashboard, filtre toutes les visualisations
- **Slicer `price_range`** : optionnel, pour filtrer par gamme de prix

### Interactions entre visuels
- Par defaut, cliquer sur un element dans un visuel filtre les autres
- Pour ajuster : Format > Modifier les interactions
  - Le Treemap devrait filtrer le Scatter Plot et les barres empilees
  - Le Donut devrait etre filtre par le slicer categorie

---

## Etape 5 : Mise en forme finale

- **Titre du dashboard** : Zone de texte en haut → "Amazon India - Analyse Produits & Avis Clients"
- **Theme** : Affichage > Themes > Choisir un theme sombre ou professionnel
- **Police** : Segoe UI, taille coherente pour les titres
- **Arriere-plan** : Couleur unie claire (#F5F5F5) ou image
- **Bordures** : Activer les bordures arrondies sur chaque visuel

---

## Resume des fichiers du projet

```
/Users/mac/Desktop/projet_viz/
├── amazon.csv              # Donnees brutes (ne pas modifier)
├── clean_amazon.py         # Script de nettoyage Python
├── amazon_clean.csv        # Table principale nettoyee (1 351 lignes)
├── amazon_reviews.csv      # Reviews avec sentiment (20 034 lignes)
├── amazon_wordcloud.csv    # Frequences mots (35 174 lignes)
└── PowerBI_Instructions.md # Ce fichier
```
