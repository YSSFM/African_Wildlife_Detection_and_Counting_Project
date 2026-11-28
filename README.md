# Détection et comptage de la faune africaine

Projet CNN – filière IASciD – S5, année universitaire 2026-2027.
Équipe : Moussa, Salma, Fatima, Rachida.

Système de **détection et de comptage** de quatre espèces (buffle, éléphant, rhinocéros, zèbre) avec **YOLOv8n**, à partir du dataset *African Wildlife* (Ultralytics). Deux politiques d'augmentation géométrique sont comparées, avec une attention aux petits objets (animaux éloignés).

> **Question scientifique :** une meilleure mAP garantit-elle un meilleur comptage ? → **Non** (voir [Conclusion](#conclusion)).

## Structure du projet

```
Projet/
├── notebooks/
│   ├── African_Wildlife_Dataset_Exploration.ipynb   # Étape 1 : exploration du dataset
│   ├── entrainement_A.ipynb                         # Étape 2 : modèle A (augmentations par défaut)
│   ├── entrainement_B.ipynb                         # Étape 3 : modèle B (scale=0.2, translate=0.05)
│   └── evaluation.ipynb                             # Étapes 4 à 6 : seuils, petits/grands objets, comptage
├── src/                                             # scripts Python réutilisables
├── datasets/                                        # African Wildlife (images + labels YOLO), non versionné
├── runs/                                            # sorties d'entraînement Ultralytics (poids, courbes), non versionné
├── results/
│   ├── annotations_all.csv                          # annotations + classe petit/grand
│   ├── size_rule.json                               # règle petit/grand (seuil 0.227)
│   ├── 12_exemples_annotes.png                      # exemples annotés
│   └── resultats_evaluation/                        # tableaux CSV, figures, seuils figés
├── config.yaml                                      # configuration du projet
├── .gitignore
└── README.md
```

---

## Contenu du dépôt

| Fichier | Rôle |
|---|---|
| `01_African_Wildlife_Dataset_Exploration.ipynb` | Étape 1 : exploration, statistiques, règle petit/grand |
| `entrainement_A.ipynb` | Étape 2 : modèle A (augmentations par défaut) |
| `entrainement_B.ipynb` | Étape 3 : modèle B (`scale=0.2`, `translate=0.05`) |
| `evaluation.ipynb` | Étapes 4 à 6 : seuils, petits/grands objets, comptage, scènes difficiles |
| `modele_A_best.pt`, `modele_B_best.pt` | Poids entraînés (`best.pt`) |
| `results/` | Tableaux CSV, figures et seuils figés générés par l'évaluation |

## Données

African Wildlife : 1 504 images, 4 classes, format YOLO. **Partitions officielles conservées** : train 1 052 / val 225 / test 227 (379 animaux en validation, 375 en test). Aucune anomalie dans les annotations.

## Protocole expérimental

| Paramètre | Modèle A | Modèle B |
|---|---|---|
| Poids initiaux | `yolov8n.pt` | `yolov8n.pt` |
| Résolution / époques / seed | 512 px / 15 / 42 | 512 px / 15 / 42 |
| `scale` | 0.5 (défaut) | **0.2** |
| `translate` | 0.1 (défaut) | **0.05** |

Seuls `scale` et `translate` diffèrent.

---

## Évaluation (`evaluation.ipynb`)

### Méthode

1. **Seuils (validation uniquement).** 2 confiances (0.25, 0.50) × 2 IoU de NMS (0.45, 0.70), pour chaque modèle. Règle de choix : précision ≥ 0.90, puis meilleur rappel, puis meilleur mAP50-95. Seuils figés dans `frozen_thresholds.json` **avant** le test.
2. **Petits / grands objets.** Petit = aire normalisée de la boîte < médiane de la validation (≈ 0.2271, soit environ 244 × 244 px à 512 px). Appariement prédiction ↔ annotation **un-à-un, même classe, IoU ≥ 0.5** (par confiance décroissante). Rappel calculé par taille.
3. **Test.** Évaluation avec les seuils figés, plus la mAP « standard » Ultralytics (`conf=0.001`).
4. **Comptage.** Nombre prédit par classe = nombre de boîtes gardées. Erreur = prédit − réel ; **MAE** globale (image × classe), par classe, et par image. Huit scènes difficiles choisies **sur les annotations du test** (pas sur les prédictions).

### Seuils figés

| Modèle | Confiance | IoU NMS |
|---|---|---|
| A | 0.25 | 0.70 |
| B | 0.25 | 0.70 |

Pour B, la règle de rappel donnait IoU 0.45 ; **0.70 a été retenu** (meilleure mAP50 et mAP50-95), choix fixé avant le test.

### Résultats – validation (4 configurations)

| Modèle | Conf | IoU | mAP50 | mAP50-95 | Préc. | Rappel | MAE comptage |
|---|---|---|---|---|---|---|---|
| A | 0.25 | 0.45 | 0.8869 | 0.7366 | 0.9336 | 0.8901 | 0.0700 |
| A | 0.25 | 0.70 | 0.8924 | 0.7444 | 0.9204 | 0.8931 | 0.0789 |
| A | 0.50 | 0.45 | 0.8613 | 0.7219 | 0.9363 | 0.8742 | 0.0767 |
| A | 0.50 | 0.70 | 0.8628 | 0.7261 | 0.9236 | 0.8764 | 0.0789 |
| B | 0.25 | 0.45 | 0.9033 | 0.7386 | 0.9350 | 0.8802 | 0.0733 |
| B | 0.25 | 0.70 | 0.9106 | 0.7444 | 0.9303 | 0.8698 | 0.0878 |
| B | 0.50 | 0.45 | 0.8511 | 0.7113 | 0.9495 | 0.8597 | 0.0711 |
| B | 0.50 | 0.70 | 0.8579 | 0.7145 | 0.9359 | 0.8663 | 0.0700 |

### Résultats – rappel par taille (validation, IoU ≥ 0.5)

| Modèle | Petits (189) | Grands (190) | Très petits < Q25 (95) |
|---|---|---|---|
| A | 0.868 | **0.937** | 0.790 |
| B | **0.921** | 0.932 | **0.853** |

B est nettement meilleur sur les petits objets ; les grands sont équivalents.

### Résultats – test (seuils figés)

| Modèle | mAP50 | mAP50-95 | mAP50 (std) | mAP50-95 (std) | Préc. | Rappel |
|---|---|---|---|---|---|---|
| A | **0.935** | **0.798** | **0.967** | **0.820** | **0.957** | **0.909** |
| B | 0.910 | 0.769 | 0.954 | 0.799 | 0.947 | 0.860 |

### Résultats – comptage

| Modèle | MAE test (image × classe) | Erreur par image (test) | Biais (prédit − réel) | MAE 8 scènes |
|---|---|---|---|---|
| A | **0.074** | **0.260** | +0.18 | **0.208** |
| B | 0.087 | 0.278 | +0.14 | 0.292 |

MAE de comptage par classe (test) : buffle A 0.022 / B 0.066 ; éléphant 0.075 / 0.101 ; rhinocéros 0.026 / 0.053 ; **zèbre 0.172 / 0.128**.

Figures et scènes : voir `results/scenes/` et le notebook.

---

## Conclusion

- **A est globalement meilleur que B sur le test** (mAP, précision, rappel) et compte un peu mieux (écart d'environ 0.02 animal par image).
- **B est meilleur sur les petits objets** en validation (+5 pts de rappel), sans gain visible au global sur le test.
- **Une meilleure mAP ne garantit pas un meilleur comptage :**
  - sur le zèbre, A a la meilleure mAP50 et le meilleur rappel mais compte moins bien (précision plus basse, boîtes en double) ;
  - pour B, la configuration de seuils à meilleure mAP est celle dont la MAE de comptage est la pire ;
  - les deux modèles sur-comptent en moyenne (biais positif) malgré un rappel < 1.
- **Cas difficiles :** les groupes denses sont **sous-comptés** (animaux cachés), et les animaux superposés **sur-comptés** (boîtes imbriquées avec un NMS à 0.70).

## Limites

- Un seul entraînement et une seule graine par modèle : les écarts de 1 à 2 pts ne sont pas fiables.
- ~375 animaux par partition ; seulement 66 buffles dans le test.
- Le test ne contient **aucune image multi-espèces** ; les 8 scènes sont complétées autrement.
- Petit / grand est relatif (peu d'objets vraiment minuscules) et lié à la classe (zèbre plus souvent « petit »).
- Précision/rappel Ultralytics : pris au seuil de confiance qui maximise le F1, pas exactement à `conf=0.25`.
- Rappel par taille analysé uniquement sur la validation.

## Reproduire

Ouvrir les notebooks dans Google Colab (GPU T4), dans l'ordre : exploration → `entrainement_A` → `entrainement_B` → `evaluation`. Déposer `modele_A_best.pt` et `modele_B_best.pt` dans `/content/` (ou à la racine). Graine : **42**. Dépendances : `ultralytics`, `numpy`, `pandas`, `matplotlib`, `Pillow`.
