# AnaeCare Nail-Based DenseNet169 — Results and Interpretation

## Experiment overview

- Modality: UNAS nail photographs only
- Model: ImageNet-pretrained DenseNet169 with a three-class output layer
- Labels: Normal = 0, Leve = 1, Moderada = 2
- Patient-level split: train 576, validation 123, test 124
- Test class counts: Normal 83, Leve 30, Moderada 11
- Best checkpoint: epoch 10, selected by validation loss
- Training stopped after epoch 15 due to early stopping

## Primary test results: patient level

| Metric | Result |
|---|---:|
| Accuracy | 0.5000 |
| Balanced accuracy | 0.4606 |
| Macro precision | 0.4282 |
| Macro recall | 0.4606 |
| Macro F1-score | 0.4168 |
| Macro ROC-AUC (one-vs-rest) | 0.6629 |
| Number of patients | 124 |

## Class-wise performance

| Class | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| Normal | 0.7818 | 0.5181 | 0.6232 | 83 |
| Leve | 0.3488 | 0.5000 | 0.4110 | 30 |
| Moderada | 0.1538 | 0.3636 | 0.2162 | 11 |

## Patient-level confusion matrix

Rows are actual labels and columns are predictions. Label order is Normal, Leve, Moderada.

| Actual \\ Predicted | Normal | Leve | Moderada |
|---|---:|---:|---:|
| Normal | 43 | 24 | 16 |
| Leve | 9 | 15 | 6 |
| Moderada | 3 | 4 | 4 |

The model identifies some examples in each category, but class-wise performance remains uneven. It correctly classifies 4 of 11 Moderada cases. The overall accuracy of 50% is not sufficient evidence of reliable anemia pre-screening, especially given the small Moderada test group.

## Training details

- DenseNet169 pretrained on ImageNet
- Input size: 224 × 224
- Optimizer: Adam
- Learning rate: 0.0001
- Weight decay: 0.0001
- Loss: class-weighted cross-entropy
- Class weights [Normal, Leve, Moderada]: approximately [0.4996, 1.3641, 3.7712]
- Configured maximum: 20 epochs
- Early-stopping patience: 5 epochs without validation-loss improvement
- Best validation-loss checkpoint: epoch 10
- Completed epochs: 15

## Important interpretation notes

1. The experiment uses UNAS nail images only. It is not a multimodal model.
2. The three original classes are retained; they are not merged into a binary label.
3. Patient-level splitting is used to reduce leakage risk.
4. In this test split, there is one test image per patient, so image-level and patient-level results are numerically identical.
5. The method is a paper-informed adaptation, not an exact reproduction of the reference paper.
6. The results are exploratory and do not constitute clinical validation.

## Package contents

- Original uploaded notebook
- `My_Methodology_Notes.md`
- `README_Results_Interpretation.md`
- `PACKAGE_MANIFEST.json`
- `results/test_metrics.csv`
- `results/image_level_test_metrics.csv`
- `results/patient_level_per_class_report.csv`
- `results/patient_level_test_confusion_matrix.csv`
- `results/training_history_from_notebook_output.csv`
- Three plot images extracted from the notebook's saved output

The uploaded notebook's output reports the original runtime ZIP and checkpoint paths, but the checkpoint and runtime result files were not attached separately. Therefore this package does not fabricate or include missing checkpoint weights or per-patient prediction rows.
