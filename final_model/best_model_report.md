# Best Comparable Model: Image-Only Bi-LSTM + Self-Attention Fusion (Ex-1, 5-fold)

**Notebook:** `ex-1-with-bi-lstm-without-demographic-5fold.ipynb` (included in the separate zip)

## 1. Which model is "best"?

Two answers are needed, because the raw numbers and the trustworthy numbers point to different models.

- **Highest raw scores:** the Ex-3 Bi-LSTM + demographics model (5-fold macro-F1 0.668 concat / 0.687 self-attention). These results are **not reliable**: that notebook is the only one that read its images from different input folders, it classifies all 29 Leve test subjects correctly (no other run exceeds F1 0.471 on that class; demographic-only logistic regression gets 0.377), and its single-split run on the common folders gave only 0.433 / 0.512. It was excluded from the selection until it is re-run on the common image folders.
- **Best among fairly comparable runs (recommended):** **image-only model, Bi-LSTM temporal pooling, self-attention fusion, 3-class, 5-fold CV (Ex-1).** It has the highest mean 5-fold macro-F1 (0.459 ± 0.075) and the highest fold-averaged macro-F1 (0.468) and accuracy (0.707) of the comparable configurations.

The evidence is **inconclusive**: the lead over the other comparable configurations is within one fold standard deviation, and no difference was statistically significant. This is a provisional candidate, not a proven winner.

## 2. Configuration

| Item | Value |
|---|---|
| Task | 3-class: Normal / Leve (mild) / Moderada (moderate) |
| Inputs | Fingertip video (32 frames), palm video (16 frames), nail photo (1 image); 224×224, ImageNet normalisation, no augmentation |
| Encoders | One ImageNet-pretrained ResNet-18 per modality, 256-d embeddings |
| Temporal pooling | Bi-LSTM (128 hidden units per direction, projected to 256) |
| Fusion | Self-attention: 3 modality tokens + learned modality embeddings → 1 Transformer encoder layer (4 heads) → mean → 64-unit layer → 3 logits |
| Demographic features | None |
| Loss / optimiser | Class-weighted cross-entropy (inverse frequency per fold); Adam, lr 1e-4; batch 16; dropout 0.3 |
| Protocol | 5-fold patient-level stratified CV on 685 train+validation subjects (548 train / 137 val per fold), 10 epochs per fold, seed 42 |
| Checkpoint rule | Highest validation macro-F1 per fold |
| Test set | Fixed, 123 subjects (87 Normal, 29 Leve, 7 Moderada), never used for training or checkpoint selection |
| Hardware | 2 × Tesla T4 (Kaggle) |

## 3. Test results

### 3.1 Per-fold test results (self-attention)

| Fold | Accuracy | Macro-precision | Macro-recall | Macro-F1 | Macro-AUC (OvR) |
|---|---|---|---|---|---|
| 1 | 0.683 | 0.356 | 0.352 | 0.337 | 0.683 |
| 2 | 0.683 | 0.541 | 0.517 | 0.525 | 0.764 |
| 3 | 0.707 | 0.592 | 0.507 | 0.513 | 0.720 |
| 4 | 0.675 | 0.540 | 0.431 | 0.451 | 0.687 |
| 5 | 0.602 | 0.471 | 0.494 | 0.470 | 0.720 |

### 3.2 Mean ± SD over the five fold models

| Model | Accuracy | Macro-precision | Macro-recall | Macro-F1 | Macro-AUC (OvR) |
|---|---|---|---|---|---|
| **Self-attention (this model)** | 0.670 ± 0.040 | 0.500 ± 0.092 | 0.460 ± 0.069 | 0.459 ± 0.075 | 0.715 ± 0.033 |
| Concatenation (same inputs, for reference) | 0.626 ± 0.063 | 0.384 ± 0.043 | 0.385 ± 0.042 | 0.380 ± 0.041 | 0.675 ± 0.043 |

### 3.3 Fold-averaged (ensemble) predictions on the test set

Probabilities of the five fold models were averaged, then scored once.

| Model | Accuracy | Macro-precision | Macro-recall | Macro-F1 | Macro-AUC (OvR) |
|---|---|---|---|---|---|
| **Self-attention** | 0.707 | 0.562 | 0.446 | 0.468 | 0.756 |
| Concatenation | 0.634 | 0.353 | 0.360 | 0.356 | 0.701 |

### 3.4 Per-class results (self-attention, fold-averaged)

| Class | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Normal | 0.778 | 0.885 | 0.828 | 87 |
| Leve | 0.409 | 0.310 | 0.353 | 29 |
| Moderada | 0.500 | 0.143 | 0.222 | 7 |

Confusion matrix (rows = true class, columns = predicted; read from the saved figure):

| True / Predicted | Normal | Leve | Moderada |
|---|---|---|---|
| Normal | 77 | 10 | 0 |
| Leve | 19 | 9 | 1 |
| Moderada | 3 | 3 | 1 |

![Fold-averaged confusion matrices (left: concatenation, right: self-attention)](cm.png)

### 3.5 Statistical comparison with concatenation

Paired sign-flip permutation test on per-fold macro-F1 (2000 iterations): mean difference (self-attention − concatenation) = +0.0798, two-sided p = 0.1784, **not significant**. With only five folds the smallest attainable two-sided p is 2/32 = 0.0625, so this test cannot reach 0.05 regardless of effect size.

## 4. How it compares with the other comparable 5-fold configurations

| Configuration | Fusion | Macro-F1 (mean ± SD) | Macro-AUC (mean ± SD) |
|---|---|---|---|
| **Image-only, Bi-LSTM** | **Self-attention** | **0.459 ± 0.075** | **0.715 ± 0.033** |
| Image-only, mean pooling | Concatenation | 0.440 ± 0.089 | 0.711 ± 0.056 |
| Image-only, mean pooling | Self-attention | 0.433 ± 0.040 | 0.708 ± 0.027 |
| + Demographics, mean pooling | Self-attention | 0.410 ± 0.065 | 0.692 ± 0.047 |
| + Demographics, mean pooling | Concatenation | 0.404 ± 0.053 | 0.721 ± 0.036 |
| Image-only, Bi-LSTM | Concatenation | 0.380 ± 0.041 | 0.675 ± 0.043 |

For reference, a tabular logistic regression on the demographic features alone reaches macro-F1 0.407 and macro-AUC 0.717 on the same test set.

## 5. Limitations

- **Accuracy is not evidence here.** Predicting Normal for everyone gives 87/123 = 0.707, which equals this model's fold-averaged accuracy.
- **Minority classes are weak.** Leve F1 is only 0.353 and Moderada is found in just 1 of 7 cases (F1 0.222, based on one correct subject).
- **Not stable across protocols.** The single-split run of the same configuration scored macro-F1 0.406, macro-AUC 0.697 and accuracy 0.569, lower than its concatenation counterpart (0.439).
- **Small test set.** 123 subjects with 7 Moderada; the ± values reflect training/validation variation, not test-set sampling uncertainty.
- **Fold spread is large.** Per-fold macro-F1 ranges from 0.337 to 0.525.
- **Early overfitting.** In the training histories, mean validation loss is lowest after epoch 1–2 and rises afterwards while training loss keeps falling.
- No claims are made about computational efficiency, robustness or clinical suitability; none was measured.

## 6. Suggested next steps (proposed, not performed)

1. Re-run the Ex-3 Bi-LSTM + demographics experiment on the common image folders and audit the stored images class by class.
2. Re-evaluate this model and the simpler mean-pooling + concatenation control with several seeds and repeated cross-validation.
3. Add sensitivity-oriented measures and class-specific thresholds once the intended use is defined.
4. Benchmark this model against other model frameworks under the same cohort, labels, split and metrics.
