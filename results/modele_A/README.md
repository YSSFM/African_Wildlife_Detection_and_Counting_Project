# Modèle A – YOLOv8n (augmentations par défaut)

Modèle de référence du projet. Paramètres : yolov8n.pt, imgsz 512, 15 époques, seed 42, scale 0.5, translate 0.1 (valeurs par défaut).
Version d'Ultralytics : 8.4.216

## Résultats sur la validation (225 images)

Valeurs de la validation finale faite à la fin de l'entraînement, avec le meilleur modèle (best.pt).

| Métrique | Valeur |
|---|---|
| Précision | 0.920 |
| Rappel | 0.894 |
| mAP50 | 0.933 |
| mAP50-95 | 0.772 |

## Résultats par classe

| Classe | Précision | Rappel | mAP50 | mAP50-95 |
|---|---|---|---|---|
| buffalo | 0.955 | 0.854 | 0.937 | 0.782 |
| elephant | 0.859 | 0.890 | 0.916 | 0.733 |
| rhino | 0.940 | 0.927 | 0.939 | 0.814 |
| zebra | 0.926 | 0.904 | 0.940 | 0.758 |

## Contenu du dossier

- results.png et results.csv : courbes et valeurs par époque
- args.yaml : paramètres complets de l'entraînement
- confusion_matrix.png et confusion_matrix_normalized.png : matrices de confusion
- BoxPR_curve.png et BoxF1_curve.png : courbes précision-rappel et F1
- val_batch0_labels.jpg et val_batch0_pred.jpg : exemples de validation
- metriques_par_classe_val.csv : métriques par classe

## Poids

Les fichiers .pt ne sont pas versionnés sur GitHub. Les poids sont sur Google Drive : Projet_CNN/modele_A/weights/best.pt
