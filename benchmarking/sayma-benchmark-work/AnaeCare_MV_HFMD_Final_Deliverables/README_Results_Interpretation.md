# AnaeCare MV-HFMD — Results and Interpretation

## At a glance

This package documents an MV-HFMD paper-method adaptation for AnaeCare using three visual views: YEMAS, PALMAS and UNAS. It is an exploratory experiment, not a clinically validated anemia detector.

## Test-set results

| Metric | Result |
|---|---:|
| Test accuracy | 0.6667 |
| Macro precision | 0.2222 |
| Macro recall | 0.3333 |
| Macro F1-score | 0.2667 |
| Macro one-vs-rest ROC-AUC | 0.6069 |
| Test loss | 1.7615 |

The held-out test set contains 123 examples: 82 Normal, 30 Leve and 11 Moderada.

## Class-wise report

| Class | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| Normal | 0.67 | 1.00 | 0.80 | 82 |
| Leve | 0.00 | 0.00 | 0.00 | 30 |
| Moderada | 0.00 | 0.00 | 0.00 | 11 |

## Confusion matrix

Rows are true labels and columns are predicted labels, in the order Normal, Leve, Moderada.

| Actual \ Predicted | Normal | Leve | Moderada |
|---|---:|---:|---:|
| Normal | 82 | 0 | 0 |
| Leve | 30 | 0 | 0 |
| Moderada | 11 | 0 | 0 |

The model predicted Normal for every test example. Consequently, the 66.67% accuracy mostly reflects the Normal-class share of the test set. It does not demonstrate useful recognition of anemia-related classes.

## Training and checkpoint selection

The notebook reports 50 completed epochs. The checkpoint selected by validation accuracy is from epoch 2, with validation accuracy 0.6667. Later epochs do not improve that selection criterion. This early best checkpoint and the final all-Normal test predictions are important context when interpreting the result.

## Limitations to disclose

- This is an adaptation for AnaeCare, not a bit-for-bit reproduction of the paper's original dataset experiment.
- One representative frame/image is used per modality; video temporal dynamics are not modelled.
- Metadata is not included in this model run.
- The test set is class-imbalanced.
- The model fails to detect Leve and Moderada in the reported test set.
- The results are not clinical validation and should not be described as a successful anemia-detection system.

## Files

- `test_metrics.csv`: summary metrics from the notebook's displayed test output.
- `test_confusion_matrix.csv`: confusion matrix from the notebook's displayed output.
- `test_classification_report.txt`: class-wise classification report.
- `training_history_from_notebook_output.csv`: epoch values parsed from the saved notebook training log. These are reconstructed from notebook output, not the original runtime CSV.
- `plots/`: plots embedded in the notebook's post-training output.

The original notebook includes code to generate `test_predictions.csv`, `test_per_class_auc.csv`, the checkpoints and the complete results ZIP in Kaggle. Those runtime files were not attached separately with the uploaded notebook, so this package does not invent missing per-patient predictions or checkpoint weights.
