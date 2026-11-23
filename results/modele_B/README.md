# Modèle B – YOLOv8n (augmentations réduites)

Modèle comparé au modèle A. Paramètres : yolov8n.pt, imgsz 512, 15 époques, seed 42, **scale 0.2, translate 0.05** (le modèle A utilise les valeurs par défaut : scale 0.5, translate 0.1).
Version d'Ultralytics : 8.4.215. Durée de l'entraînement : environ 0.058 h (3 à 4 minutes, GPU T4).

## Résultats sur la validation (225 images)

Valeurs de la validation finale faite à la fin de l'entraînement, avec le meilleur modèle (best.pt).

| Métrique | Valeur |
|---|---|
| Précision | 0.927 |
| Rappel | 0.874 |
| mAP50 | 0.940 |
| mAP50-95 | 0.771 |

## Résultats par classe

| Classe | Précision | Rappel | mAP50 | mAP50-95 |
|---|---|---|---|---|
| buffalo | 0.909 | 0.876 | 0.952 | 0.800 |
| elephant | 0.907 | 0.857 | 0.917 | 0.754 |
| rhino | 0.949 | 0.869 | 0.944 | 0.792 |
| zebra | 0.944 | 0.895 | 0.948 | 0.738 |

## Analyse des erreurs (validation, seuil de confiance 0.25)

- Animaux manqués : 28
- Mauvaise classe : 10
- Boîtes en trop (fausses détections ou doublons) : 39
- Images avec au moins une erreur : 52 sur 225

Ces nombres dépendent du seuil de confiance 0.25, choisi pour l'analyse visuelle.

## Contenu du dossier

- results.png et results.csv : courbes et valeurs par époque
- args.yaml : paramètres complets de l'entraînement
- confusion_matrix.png et confusion_matrix_normalized.png : matrices de confusion
- BoxPR_curve.png et BoxF1_curve.png : courbes précision-rappel et F1
- val_batch0_labels.jpg et val_batch0_pred.jpg : exemples de validation
- erreurs_B.png : exemples d'erreurs (vert = vérité, rouge = prédiction)
- metriques_par_classe_val.csv : métriques par classe

## Poids

Les fichiers .pt ne sont pas versionnés sur GitHub. Les poids sont sur Google Drive : Projet_CNN/poids/modele_B_best.pt
