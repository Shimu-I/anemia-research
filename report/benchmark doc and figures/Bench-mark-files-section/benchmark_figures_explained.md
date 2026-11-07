# Benchmarking Figures: Results and Interpretation

This file explains the four benchmarking figures in `figures/`. All figures compare the **selected (reference) model**, a Bi-LSTM temporal pooling + self-attention fusion model using fingertip video, palm video and nail photograph, with one run from each of 11 published papers (B03–B13), on the same three-class task (Normal, Leve, Moderada).

## How to read all figures

| Marker | Meaning |
|:--|:--|
| Solid bar | B03–B08: five-fold mean ± SD, scored on the **same fixed test set** (123 subjects: 87 Normal, 29 Leve, 7 Moderada) as the reference |
| Dotted bar | B13: five-fold cross-validation over all 823 nail participants (not the same test subjects) |
| Hatched bar | B09–B12: **single runs** on their own splits (no SD, no significance test) |
| Dashed line / band | Reference mean (± SD) |

**Why macro-F1 is the main metric:** always predicting Normal gives accuracy 0.707, so accuracy rewards ignoring anaemia. Macro-F1 weights the three classes equally; macro-AUC is the threshold-free second metric.

---

## Figure 1: Macro-F1 and macro-AUC (`fig1_macro_f1_auc.png`)

![Figure 1](figures/fig1_macro_f1_auc.png)

**Results**

| Run | Macro-F1 | Macro-AUC |
|:--|--:|--:|
| **Reference** | **0.459 ± 0.075** | **0.715 ± 0.033** |
| B06 Late ElasticNet average | 0.419 ± 0.020 | 0.701 ± 0.015 |
| B08 Majority vote, 3 × ResNet-18 | 0.418 ± 0.037 | 0.694 ± 0.015 |
| B11 Nail DenseNet169 † | 0.417 | 0.663 |
| B09 BPANet-adapted † | 0.416 | 0.762 |
| B07 k-NN, colour features | 0.410 ± 0.054 | 0.613 ± 0.022 |
| B05 AFF (ResNet-18) | 0.405 ± 0.035 | 0.702 ± 0.037 |
| B13 EfficientNet-B0 + GeM ‡ | 0.389 ± 0.062 | 0.645 ± 0.066 |
| B03 EfficientNet-B4 + a\* colour | 0.361 ± 0.043 | 0.602 ± 0.052 |
| B04 Intermediate fusion | 0.328 ± 0.022 | 0.591 ± 0.017 |
| B10 MV-HFMD † | 0.267 | 0.607 |
| B12 RCBAM image-only † | 0.266 | 0.577 |

† single run on its own split, ‡ five-fold CV over all nail participants.

**Interpretation**
- The reference has the highest macro-F1 of all 11 runs, but the lead is small.
- **Seven methods lie within one reference SD (0.075)**: B06, B08, B11, B09, B07, B05, B13 (macro-F1 0.389–0.419). The evidence cannot separate them from the reference.
- **Four are clearly lower:** B03, B04 and, by the widest margin, B10 and B12 (≈ 2.6 SD below). B10 and B12 predict Normal for every test subject.
- On AUC only B09 (0.762) is above the reference, but it is one run, which cannot show it ranks subjects better.
- Among the five-fold benchmarks on the fixed test set, **no paired permutation test reached p < 0.0625** (the smallest possible with five folds). No difference is statistically established.

---

## Figure 2: Per-class F1 (`fig2_per_class_f1.png`)

![Figure 2](figures/fig2_per_class_f1.png)

**Results (F1 per class)**

| Run | Normal | Leve | Moderada |
|:--|--:|--:|--:|
| **Reference** | **0.83** | 0.35 | 0.22 |
| B06 Late ElasticNet avg | 0.63 | **0.45** | 0.13 |
| B09 BPANet adapted † | 0.81 | 0.44 | 0.00 |
| B05 AFF | 0.75 | 0.43 | 0.00 |
| B11 Nail DenseNet169 † | 0.62 | 0.41 | 0.22 |
| B13 EffNet-B0 + GeM ‡ | 0.73 | 0.37 | 0.07 |
| B07 k-NN (colour) | 0.81 | 0.34 | 0.20 |
| B08 Majority vote | 0.81 | 0.30 | **0.29** |
| B03 EffNet-B4 + a\* | 0.72 | 0.29 | 0.00 |
| B04 Intermediate fusion | 0.77 | 0.25 | 0.00 |
| B10 MV-HFMD † | 0.80 | 0.00 | 0.00 |
| B12 RCBAM image-only † | 0.80 | 0.00 | 0.00 |

**Interpretation**
- **Normal:** the reference is highest (0.83), but k-NN, BPANet-adapted and majority vote are at 0.81, and even the two all-Normal predictors (B10, B12) reach 0.80. Normal F1 says little about whether anaemia is found, because Normal is the majority class.
- **Leve:** five methods beat the reference (0.35): B06, B09, B05, B11 and B13.
- **Moderada:** six methods score 0.00. Only B08 (0.29) exceeds the reference (0.22); B11 is level with it.
- With 7–11 Moderada subjects in most test sets, **one correct prediction moves recall by 9–14 percentage points**, so no ranking on this class is reliable.

---

## Figure 3: Confusion matrices on the common test set, B03–B08 (`fig3_confusion_common_test_B03-B08.png`)

![Figure 3](figures/fig3_confusion_common_test_B03-B08.png)

Each matrix uses the averaged predictions of the five fold models on the same 123 subjects (rows = true class; large number = count; small number = share of the true class). These runs can be compared directly with each other and with the reference.

**Results (Normal-versus-anaemic screening, Leve + Moderada merged; the models were not trained for this binary task)**

| Run | Sensitivity | Specificity | Moderada found |
|:--|:--|:--|:--|
| Reference | 0.39 (14/36) [0.25, 0.55] | 0.89 (77/87) [0.80, 0.94] | 1/7 |
| B03 EffNet-B4 + a\* | 0.42 (15/36) [0.27, 0.58] | 0.70 (61/87) [0.60, 0.79] | 0/7 |
| B04 Intermediate fusion | 0.33 (12/36) [0.20, 0.50] | 0.79 (69/87) [0.70, 0.86] | 0/7 |
| B05 AFF | 0.61 (22/36) [0.45, 0.75] | 0.69 (60/87) [0.59, 0.78] | 0/7 |
| B06 Late ElasticNet avg | 0.83 (30/36) [0.68, 0.92] | 0.49 (43/87) [0.39, 0.60] | 2/7 |
| B07 k-NN (colour) | 0.31 (11/36) [0.18, 0.47] | 0.89 (77/87) [0.80, 0.94] | 1/7 |
| B08 Majority vote | 0.47 (17/36) [0.32, 0.63] | 0.83 (72/87) [0.73, 0.89] | 3/7 |

(95 % Wilson intervals in brackets.)

**Interpretation**
- The reference is **conservative**: it misses 22 of 36 anaemic subjects but wrongly flags only 10 of 87 Normal subjects.
- **B07 (k-NN) has almost the same profile** as the reference.
- **B06 is the extreme case:** it finds 30 of 36 anaemic subjects but only 43 of 87 Normal subjects are correct, and 15 Normal subjects are called Moderada. Both its intervals are clearly separated from the reference.
- B03 and B05 flag more anaemic subjects at the cost of specificity; their specificity intervals lie just below the reference interval.
- **Main message:** macro-F1 values in Figure 1 are close because the methods trade sensitivity against specificity at different points, not because they make the same errors.

---

## Figure 4: Confusion matrices on their own splits, B09–B13 (`fig4_confusion_own_splits_B09-B13.png`)

![Figure 4](figures/fig4_confusion_own_splits_B09-B13.png)

These runs use different test sets (B09 87/29/7, B10 82/30/11, B11 83/30/11, B12 165/61/22, B13 pooled over folds 550/200/73), so the matrices **describe each run and do not rank it** against the reference.

| Run | Normal recall | Leve recall | Moderada recall | Sensitivity | Specificity |
|:--|--:|--:|--:|:--|:--|
| B09 BPANet adapted † | 0.816 (71/87) | 0.483 (14/29) | 0.000 (0/7) | 0.53 | 0.82 |
| B10 MV-HFMD † | 1.000 (82/82) | 0.000 (0/30) | 0.000 (0/11) | 0.00 | 1.00 |
| B11 Nail DenseNet169 † | 0.518 (43/83) | 0.500 (15/30) | 0.364 (4/11) | 0.71 | 0.52 |
| B12 RCBAM image-only † | 1.000 (165/165) | 0.000 (0/61) | 0.000 (0/22) | 0.00 | 1.00 |
| B13 EffNet-B0 + GeM ‡ | 0.702 (386/550) | 0.420 (84/200) | 0.068 (5/73) | 0.54 | 0.70 |

**Interpretation**
- **B10 and B12 collapse to the majority class.** B10's checkpoint was chosen by validation accuracy and fell on epoch 2, where its validation macro-F1 (0.267) equals an all-Normal predictor; B12's image branch had no class weighting or augmentation.
- **B11 is the only single run that finds Moderada at a useful rate (4 of 11)**, but only 43 of 83 Normal subjects are correct and 16 are called Moderada.
- B09 finds 14 of 29 Leve subjects but none of the seven Moderada subjects.
- B13 is more moderate, with only 5 of 73 Moderada found.
- Single runs are noisy: B09 stayed at the all-Normal level for five of its first six epochs (peak validation macro-F1 0.328) yet scored 0.416 on test.

---

## Overall conclusions

1. **The reference model keeps its place, but its lead is small.** No run exceeds it on macro-F1, but seven are within one SD and no test reached significance. Read it as a statistical tie with a small edge, not a clear win.
2. **The methods differ mainly in operating point**, from conservative (reference, B07) to aggressive (B06, B11), not in overall accuracy of ranking.
3. **Nail-only models work only when training is sound.** Pretrained, augmented, class-weighted nail models reach 0.417 (B11) and 0.389 (B13), about 91 % and 85 % of the reference; the unweighted B12 predicts Normal for everyone. This suggests the nail photograph carries a usable part of the signal and that the training recipe matters more than architecture, but splits and budgets differ, so the claim is weak.
4. **Poor results reflect the adaptation, not necessarily the paper's method.** B10 fails through its checkpoint rule on an imbalanced validation set, B12 through missing class weighting, and B03/B04 through three-class re-implementation of originally binary papers. Published accuracies (0.65–0.99, other data) cannot be ranked against these values.

## Limitations to state alongside the figures
- Three protocols were used (fixed test set, all-participant CV, own-split single runs).
- Single runs have no run-to-run variation, and B13's folds include the reference test subjects.
- The reference was selected among several configurations on the same test set, which may favour it.
- Other runs exist that are not shown: for example naive Bayes in B07 (macro-F1 0.480 ± 0.017, p = 0.6875) scored above the reference mean, so the reference is not the highest of all variants evaluated.
- Moderada has only 7–73 test subjects; clinical suitability and computational cost were not measured.
- B03–B08 confusion counts were read from the notebooks' saved images; the reference's full matrix is not included (only its diagonal is available).

## Source data
`figures/benchmark_comparison_data.csv` holds the per-variant metrics behind Figures 1–2.
