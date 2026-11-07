# AnaeCare BPANet-Adapted Model — Results and Interpretation

## 1. Run summary

This report is based on the saved outputs in the uploaded Kaggle notebook `anaecare-bpanet-methodology-adapted-corrected-v2.ipynb`. The notebook output shows 806 usable paired patients after modality-file validation, split into 560 training, 123 validation, and 123 held-out test patients. Training ran for 10 epochs. The best checkpoint was selected using validation macro-F1; the saved test evaluation then loaded that best checkpoint.

The notebook reports two Tesla T4 GPUs available, although the code's device output says `cuda`; this report does not assume multi-GPU training was used.

## 2. Held-out test metrics

| Metric | Result |
|---|---:|
| Test loss | 1.95735 |
| Accuracy | 0.691057 (69.11%) |
| Macro precision | 0.402273 |
| Macro recall | 0.432950 |
| Macro F1-score | 0.416310 |
| Macro ROC-AUC (one-vs-rest) | 0.762055 |
| Hemoglobin regression MAE | 1.156462 g/dL |

The CSV `test_metrics.csv` contains the exact values printed in the notebook's saved test output.

## 3. Class-wise results

| Class | Precision | Recall | F1-score | Test support |
|---|---:|---:|---:|---:|
| Normal | 0.81 | 0.82 | 0.81 | 87 |
| Leve | 0.40 | 0.48 | 0.44 | 29 |
| Moderada | 0.00 | 0.00 | 0.00 | 7 |
| Macro average | 0.40 | 0.43 | 0.42 | 123 |
| Weighted average | 0.66 | 0.69 | 0.68 | 123 |

The notebook's classification report is rounded to two decimal places. The `per_class_report.csv` preserves that displayed precision.

## 4. Confusion matrix interpretation

Rows are actual classes and columns are predicted classes:

| Actual \ Predicted | Normal | Leve | Moderada |
|---|---:|---:|---:|
| Normal | 71 | 16 | 0 |
| Leve | 15 | 14 | 0 |
| Moderada | 2 | 5 | 0 |

The model correctly predicts 71 of 87 Normal cases and 14 of 29 Leve cases. It does not predict any test patient as Moderada, so Moderada recall and F1 are both zero. This is a major limitation. The overall accuracy of 69.11% should therefore not be interpreted alone as strong performance.

## 5. ROC-AUC interpretation

The reported macro one-vs-rest ROC-AUC is 0.7621. AUC reflects how well the predicted scores rank class membership across thresholds, while the current confusion matrix reflects predictions at the selected argmax decision rule. Thus, a moderate macro AUC can coexist with weak macro F1 and failure to predict the Moderada class. See `test_roc.png` for the saved ROC plot from the notebook.

## 6. Hemoglobin regression interpretation

The notebook reports an Hb regression mean absolute error of 1.1565 g/dL on the held-out test set. This is an average absolute deviation between predicted and recorded hemoglobin values. It is not a diagnosis result, and clinical suitability cannot be claimed from this experiment alone.

## 7. Training trend

Training loss fell from 2.9396 at epoch 1 to 1.7157 at epoch 10. Validation macro-F1 was highest at epoch 4 (0.3283 among the printed epochs), after which it fluctuated between 0.2686 and 0.3237. This indicates that lower training loss did not consistently translate into better validation classification performance. The best checkpoint was selected by validation macro-F1, not by the final epoch.

The `training_history_from_notebook_output.csv` records the epoch values printed in the notebook. It is reconstructed from the saved cell output; it is not a substitute for the original full `training_history.csv` that may exist in the Kaggle working directory.

## 8. Methodology summary and fidelity caveat

The notebook implements an AnaeCare adaptation inspired by BPANet:
- ImageNet-pretrained ResNet-50 visual backbone.
- Channel-spatial attention after ResNet Stage 1 and before Stage 2.
- Age and sex metadata projection.
- Learnable modality/body-part tags for YEMAS, PALMAS, and UNAS.
- External-attention-style fusion across three modality tokens.
- Hb regression and 48-bin Hb classification dual objectives.
- Additional weighted 3-class AnaeCare head for Normal, Leve, and Moderada.
- Bi-LSTM pooling for YEMAS and PALMAS frame sequences.
- Morphological opening preprocessing as implemented in the notebook.

This is **not an exact reproduction of the original BPANet authors' code**. The original paper's task and data setup differ; this notebook adapts its described components to the project's paired multimodal videos/images and adds a project-specific three-class objective. The notebook's method should be described as “BPANet-inspired/adapted,” not “the original BPANet reproduced exactly.”

## 9. Limitations and recommended next steps

1. The test set contains only 7 Moderada cases, making that class's estimate uncertain.
2. The model predicts no Moderada cases in this run; class imbalance, split composition, loss weighting, thresholds, or model calibration need investigation.
3. Macro-F1 (0.4163) is modest despite 69.11% accuracy, so accuracy alone is misleading.
4. Compare against simple baselines and the project's previous best model using the same patient-level test split before claiming improvement.
5. Inspect labels, split integrity, class weights, and prediction distributions before further tuning.
6. Because this is pre-screening research, do not describe the model as a clinical diagnostic system.

## 10. Included files

- `test_metrics.csv`
- `test_confusion_matrix.csv`
- `per_class_report.csv`
- `training_history_from_notebook_output.csv`
- `test_confusion_matrix.png`
- `test_roc.png`
