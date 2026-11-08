# Projet : prédiction de troubles psychiatriques à partir de données génétiques et cliniques

## Objectif

Ce projet cherche à prédire le **trouble psychiatrique** d'une personne à partir de ses données génétiques et cliniques. La variable cible comporte **4 classes** :

| Code | Classe |
|---|---|
| 0 | Aucun trouble |
| 1 | Dépression |
| 2 | Trouble bipolaire |
| 3 | Schizophrénie |

Il s'agit d'un problème de **classification multiclasse** en grande dimension : beaucoup plus de variables que d'observations.

## Données

- **205 observations** et **1 041 variables**
- Variables numériques (données génétiques) et catégorielles (`gender`, `tissue`, `race`, `disease state`)
- Échantillons provenant de trois régions cérébrales : hippocampe, cortex préfrontal et striatum associatif
- Aucune valeur manquante, mais présence de **valeurs infinies** (`Inf`)
- Présence de variables administratives sans intérêt pour la prédiction

> Source des données : https://www.nature.com/articles/s41398-019-0492-8

## Prétraitement

Les étapes de nettoyage sont appliquées dans cet ordre :

1. **Encodage des variables catégorielles** : `gender`, `tissue`, `race`, `disease state`.
2. **Suppression des variables administratives**.
3. **Valeurs infinies** :
   - si au moins 30 % des valeurs d'une variable valent `Inf`, la variable est supprimée ;
   - sinon, les `Inf` sont remplacés par la médiane de la variable.
4. **Transformation logarithmique** : appliquée aux variables dont le rapport entre le quantile à 99 % et la médiane dépasse 10⁶, afin de réduire l'amplitude des valeurs.
5. **Valeurs aberrantes**, détectées avec l'écart interquartile (IQR) :
   - si au moins 40 % des valeurs d'une variable sont jugées aberrantes, la variable est supprimée ;
   - sinon, les valeurs aberrantes sont remplacées par la médiane.
6. **Suppression des variables à faible variance**.
7. **Suppression des variables trop corrélées** (seuil de corrélation : 0,7).

## Méthodologie

- Séparation du jeu de données : **80 % entraînement / 20 % test**
- Entraînement des modèles sur le jeu d'entraînement
- Pour chaque observation, le modèle calcule la probabilité d'appartenir à chacune des 4 classes ; la **classe prédite** est celle dont la probabilité est la plus élevée
- Évaluation par l'**accuracy** (proportion de cas bien prédits) sur le jeu de test

## Modèles testés

### 1. Régression logistique multinomiale sans pénalisation

Modèle de référence, choisi parce que la variable cible est catégorielle.

- Accuracy : **0,366**
- Accuracy en supprimant les variables corrélées à plus de 0,9 : **0,268**

Ces performances ne sont pas satisfaisantes. Le modèle suivant conserve donc les variables corrélées et ajoute une pénalisation.

### 2. Elastic Net

- Le paramètre α varie de 0 (Ridge) à 1 (Lasso) par pas de 0,01.
- Pour chaque α, on retient le λ qui minimise l'erreur moyenne de prédiction en validation croisée.
- On choisit l'α qui donne la meilleure accuracy.

Résultat : meilleure accuracy de **0,634** pour **α = 0,87**, avec une précision de **0,637**.

### 3. ACP + régression logistique sans pénalisation

- Réduction de dimension avec le PCA
- Conservation des **50 premières composantes principales**
- Accuracy : **0,463**

### 4. Division du jeu de données par tissu + Elastic Net

Le jeu de données est divisé en trois sous-ensembles selon la région cérébrale :

1. hippocampe ;
2. cortex préfrontal ;
3. striatum associatif.

Un modèle Elastic Net est entraîné sur chaque sous-ensemble. La performance globale est la moyenne des accuracies, pondérée par la taille des sous-ensembles :

$$\overline{Accuracy} = \sum_{k=1}^{K} w_k \cdot Accuracy_k$$

- Accuracy moyenne pondérée : 0.64

### 5. Division du jeu de données par tissu + Elastic Net + SVM

Même découpage que la méthode 4. Les variables sélectionnées par chaque Elastic Net servent ensuite à entraîner un **SVM** sur chaque sous-ensemble.

- Accuracy moyenne pondérée : 0.30

## Résultats

| Méthode | Accuracy |
|---|---|
| Régression logistique sans pénalisation | 0,366 |
| Elastic Net (α = 0,87) | 0,634 |
| PCA (50 composantes) + régression logistique | 0,463 |
| **Division par tissu + Elastic Net** | 0.64 |
| Division par tissu + Elastic Net + SVM | 0.30 |

**Meilleur modèle : division du jeu de données par tissu + Elastic Net.**

Entraîner un modèle par région cérébrale permet de tenir compte des différences d'expression génétique entre tissus, et la pénalisation Elastic Net limite le surapprentissage dans ce contexte de grande dimension.

## Structure du dépôt

```
├── GSE53987_first1000genes.csv/    # dataset
├── projet_code_final/              # prétraitement et modèles
└── README.md
```

