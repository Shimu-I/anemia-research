# Results Interpretation: 3-Class Anemia Severity Classification (EfficientNet-B0 + Participant-Level 5-Fold CV)

Source: `methodology-3.ipynb`. Every figure below is taken from the notebook's printed tables, training logs, and embedded plots. Numbers marked "derived" were computed from those printed values (for example macro precision as the mean of the printed per-class precisions). Class order is always **normal, leve (mild), moderada (moderate)**. All headline metrics are **participant-level**, pooled over the five held-out CV folds.

---

## 1. Run summary

- **Task:** 3-class severity classification (normal / leve / moderada), run separately on three body sites: unas (fingernail), yemas (fingertip pad), palmas (palm).
- **Environment:** Kaggle notebook, CUDA GPU, torch 2.11.0+cu128, timm 1.0.29, albumentations 2.0.8, seed 42, images resized to 224×224.
- **Data:** 40,970 images indexed (labels from folder names, none missing). Frames per participant were reduced to 8 for yemas and palmas.

| Site | Images indexed | Images used | Participants | Normal | Leve | Moderada |
|---|---|---|---|---|---|---|
| unas | 826 | 826 | 823 | 550 | 200 | 73 |
| yemas | 26,816 | 6,672 | 834 | 558 | 201 | 75 |
| palmas | 13,328 | 6,632 | 829 | 554 | 201 | 74 |

- **Split:** one shared, stratified, participant-level 5-fold split over 839 participants (167 to 168 per fold; per fold 112 to 113 Normal, 40 to 41 Leve, 15 to 16 Moderada). Inside each fold the training participants are split 85/15 into train and validation.

| Site | Train images / fold | Val images / fold | Test images / fold |
|---|---|---|---|
| unas | 559 to 564 | 99 to 101 | 162 to 167 |
| yemas | 4,528 to 4,544 | 800 to 808 | 1,320 to 1,344 |
| palmas | 4,496 to 4,520 | 800 | 1,312 to 1,336 |

- **Models per site (2):** a non-CNN baseline (mean RGB → StandardScaler → balanced multinomial logistic regression) and an ImageNet-pretrained EfficientNet-B0 with GeM pooling and a linear head (AdamW, lr 1e-3, cosine schedule, balanced class weights, early stopping on validation macro-F1).
- **Headline result:** the **yemas CNN** is the best model by macro-F1 (0.447) and macro AUC (0.689). Differences between the CNN and the baseline are small, and every model's accuracy is below the always-Normal rate.

---

## 2. Held-out test metrics

Participant-level, pooled over the 5 test folds. Accuracy, macro-F1 and macro AUC are printed in the notebook; macro precision and macro recall are **derived** as the mean of the printed per-class values.

| Site | Model | Accuracy | Macro P | Macro R | Macro F1 | Macro AUC |
|---|---|---|---|---|---|---|
| unas | CNN | 0.5772 | 0.3906 | 0.3968 | **0.3916** | **0.6254** |
| unas | Non-CNN | 0.3670 | 0.3718 | 0.3771 | 0.3186 | 0.5839 |
| yemas | CNN | 0.5228 | 0.4562 | 0.4952 | **0.4470** | **0.6891** |
| yemas | Non-CNN | 0.4976 | 0.4335 | 0.4797 | 0.4012 | 0.6746 |
| palmas | CNN | 0.5356 | 0.4080 | 0.4224 | **0.4024** | 0.6482 |
| palmas | Non-CNN | 0.4765 | 0.4015 | 0.4289 | 0.3758 | **0.6505** |

CNN across folds, mean ± std over the 5 folds (participant-level within each fold):

| Site | Accuracy | Macro-F1 | Macro AUC |
|---|---|---|---|
| unas | 0.578 ± 0.075 | 0.389 ± 0.062 | 0.645 ± 0.066 |
| yemas | 0.523 ± 0.082 | 0.442 ± 0.062 | 0.696 ± 0.035 |
| palmas | 0.535 ± 0.071 | 0.392 ± 0.042 | 0.661 ± 0.032 |

How to read this:

- **Baseline:** always predicting Normal gives about 0.668 accuracy (unas 0.668, yemas 0.669, palmas 0.668; derived from class counts) and macro-F1 about 0.267. **All six models score below 0.668 accuracy**, because balanced class weights deliberately give up Normal recall. Their macro-F1 values (0.32 to 0.45) are above the 0.267 baseline.
- **CNN vs baseline:** the CNN has higher macro-F1 on all three sites (+0.073 unas, +0.046 yemas, +0.027 palmas), but each gap is smaller than the CNN's own fold-to-fold standard deviation (0.04 to 0.06). The AUC gap is +0.04 on unas, +0.015 on yemas and none on palmas (0.648 vs 0.651). Treat the CNN advantage as suggestive, not demonstrated.
- **Best site:** yemas has the highest macro-F1 and AUC. unas has the highest CNN accuracy (0.577) but a lower macro-F1, since it barely detects Moderada.
- **Fold variability:** accuracy standard deviation is 0.07 to 0.08. Per-fold image-level test accuracy of the CNN (printed during training) ranges 0.49 to 0.65 for unas, 0.42 to 0.59 for yemas and 0.45 to 0.59 for palmas.
- **Pooled vs mean of folds:** pooled macro-F1 (0.447 yemas) and the fold mean (0.442) are close, so pooling does not hide a fold-level problem.

---

## 3. Class-wise results

Participant-level precision / recall / F1 / one-vs-rest AUC per class (test support in the headers). All values are from the notebook's per-class table.

### unas (normal 550 · leve 200 · moderada 73)

| Model | Normal P / R / F1 / AUC | Leve P / R / F1 / AUC | Moderada P / R / F1 / AUC |
|---|---|---|---|
| Non-CNN | 0.711 / 0.386 / 0.500 / 0.604 | 0.304 / 0.280 / 0.292 / 0.596 | 0.100 / 0.466 / 0.164 / 0.551 |
| CNN | 0.754 / 0.702 / 0.727 / 0.664 | 0.336 / 0.420 / 0.373 / 0.628 | 0.082 / 0.069 / 0.075 / 0.585 |

### yemas (normal 558 · leve 201 · moderada 75)

| Model | Normal P / R / F1 / AUC | Leve P / R / F1 / AUC | Moderada P / R / F1 / AUC |
|---|---|---|---|
| Non-CNN | 0.816 / 0.574 / 0.674 / 0.698 | 0.324 / 0.239 / 0.275 / 0.607 | 0.160 / 0.627 / 0.255 / 0.720 |
| CNN | 0.831 / 0.545 / 0.658 / 0.704 | 0.303 / 0.488 / 0.374 / 0.632 | 0.235 / 0.453 / 0.309 / 0.731 |

### palmas (normal 554 · leve 201 · moderada 74)

| Model | Normal P / R / F1 / AUC | Leve P / R / F1 / AUC | Moderada P / R / F1 / AUC |
|---|---|---|---|
| Non-CNN | 0.808 / 0.561 / 0.662 / 0.692 | 0.257 / 0.239 / 0.247 / 0.590 | 0.140 / 0.487 / 0.218 / 0.670 |
| CNN | 0.798 / 0.648 / 0.715 / 0.707 | 0.281 / 0.309 / 0.294 / 0.595 | 0.146 / 0.311 / 0.198 / 0.643 |

Take-aways:

- **Normal** is the best-learned class (F1 0.50 to 0.73) and the only one with precision above 0.7, but its recall is only 0.39 to 0.70 because class weights push predictions toward the other classes.
- **Leve** is poorly recognised: best F1 is 0.374 (unas CNN and yemas CNN) and precision never exceeds 0.34, so most predicted-Leve participants are not Leve.
- **Moderada** is weakest in precision everywhere (0.08 to 0.24). Recall is high only where the model over-predicts the class (yemas baseline 0.627 recall at 0.160 precision, i.e. most flagged participants are not Moderada).
- **unas CNN** essentially fails on Moderada (recall 0.069, F1 0.075).
- **yemas CNN** is the most balanced model: F1 0.66 / 0.37 / 0.31.
- **palmas CNN** ranks Normal best (F1 0.715) but is at or below the baseline for Moderada (F1 0.198 vs 0.218).

---

## 4. Confusion matrix interpretation

Rows are actual classes, columns are predicted classes, both ordered normal, leve, moderada. Counts are participants (read from the embedded confusion-matrix figure; each row sums to the class support).

| Site | Model | Normal row | Leve row | Moderada row |
|---|---|---|---|---|
| unas | Non-CNN | 212 / 110 / 228 | 65 / 56 / 79 | 21 / 18 / 34 |
| unas | CNN | 386 / 132 / 32 | 92 / 84 / 24 | 34 / 34 / 5 |
| yemas | Non-CNN | 320 / 86 / 152 | 58 / 48 / 95 | 14 / 14 / 47 |
| yemas | CNN | 304 / 192 / 62 | 54 / 98 / 49 | 8 / 33 / 34 |
| palmas | Non-CNN | 311 / 116 / 127 | 59 / 48 / 94 | 15 / 23 / 36 |
| palmas | CNN | 359 / 125 / 70 | 74 / 62 / 65 | 17 / 34 / 23 |

What the matrices show:

- **Errors are spread across all classes**, not mostly adjacent ones. In yemas CNN, 49 of 201 Leve participants are called Moderada and 54 are called Normal; 8 of 75 Moderada participants are called Normal.
- **unas CNN** puts almost everything into Normal or Leve: only 5 of 73 Moderada are right and 34 are called Normal. Only 61 of 823 participants are predicted Moderada in total, so the class is almost never predicted.
- **unas baseline** is close to random: Normal recall 0.39, with 228 of 550 Normal participants called Moderada.
- **yemas CNN** correctly classifies 304 / 98 / 34 participants (436 of 834, accuracy 0.523). The yemas baseline gets Moderada recall higher (47 of 75) by calling 152 Normal participants Moderada.
- **palmas CNN** gets 359 / 62 / 23 correct (444 of 829). Most Moderada participants are called Leve (34) or Normal (17), and 65 Leve participants are called Moderada.
- **Overall pattern:** no model separates the three severity levels reliably. The CNN mainly reallocates errors compared with the baseline, and mostly improves Leve recall (yemas 0.488 vs 0.239, unas 0.420 vs 0.280).

---

## 5. ROC-AUC interpretation

Macro one-vs-rest AUC is in section 2; per-class AUC is in section 3. The combined CNN ROC plot shows the same numbers (rounded):

| Site | Normal | Leve | Moderada |
|---|---|---|---|
| unas CNN | 0.66 | 0.63 | 0.58 |
| yemas CNN | 0.70 | 0.63 | 0.73 |
| palmas CNN | 0.71 | 0.59 | 0.64 |

Reading the curves:

- **0.5 is chance.** Every AUC is between 0.55 and 0.73, so the models rank participants only weakly. No class reaches 0.75.
- **yemas is the only site with a clearly informative Moderada ranking** (0.73 CNN, 0.72 baseline). The same number for palmas is 0.64 to 0.67 and for unas 0.55 to 0.59.
- **Leve is the weakest class by AUC on every site** (0.59 to 0.63 for the CNN), which matches its low F1.
- **The baseline matches the CNN on ranking.** Yemas macro AUC is 0.675 vs 0.689, palmas is 0.651 vs 0.648. The CNN's advantage appears mainly at the argmax decision, not in the probabilities themselves.
- **AUC and accuracy disagree:** unas CNN has the highest accuracy (0.577) but a lower AUC (0.625) than yemas (0.689, accuracy 0.523).
- **Uncertainty:** with only 73 to 75 Moderada participants, the AUC curves are step-like and noisy, and the fold standard deviations (0.03 to 0.07 on macro AUC) show how much a single split can move them.

---

## 6. Hemoglobin regression interpretation

**Not applicable: this notebook does not perform hemoglobin regression.**

- There is no regression model, no Hb target, and no regression metric (MAE, RMSE, R²) anywhere in the notebook or its outputs.
- `metadata.csv` (909 rows, 22 columns) contains a `Hemoglobina` column and `Dataset2.csv` (999 rows) contains `Class`, `Anemia Predict` and `Split` columns. Both files are only **loaded and previewed** in Cell 3. They are not used for labels, splits, training or evaluation. Labels come from the image folder names.
- No claim about predicting hemoglobin levels, or about agreement between predicted classes and measured Hb, can be made from these results.

If a regression result is needed, it requires adding a continuous Hb target, a regression head or model, and evaluating with MAE / RMSE / R² (plus agreement with the class labels) on the same participant-level folds.

---

## 7. Training trend

Only the CNN has epoch-level logs. Early stopping (patience 10 on validation macro-F1, best weights restored) ended 13 of 15 runs before the 40-epoch limit. The unas fold 1 run reached epoch 40 (early stop triggered at the last epoch) and unas fold 2 ran the full 40 epochs.

| Site | Epochs run per fold (folds 0 to 4) | Best-F1 epoch per fold | Best val macro-F1 per fold | Train loss, first → last epoch (range over folds) |
|---|---|---|---|---|
| unas | 16, 40, 40, 20, 20 | 6, 30, 34, 10, 10 | 0.438, 0.508, 0.483, 0.383, 0.569 | 1.4 to 1.7 → 0.24 to 0.97 |
| yemas | 36, 21, 30, 22, 12 | 26, 11, 20, 12, 2 | 0.444, 0.435, 0.484, 0.432, 0.403 | 1.1 to 1.2 → 0.59 to 0.98 |
| palmas | 13, 17, 16, 14, 19 | 3, 7, 6, 4, 9 | 0.394, 0.397, 0.512, 0.510, 0.485 | 1.1 to 1.2 → 0.06 to 0.19 |

What the trend suggests:

- **palmas overfits quickly.** Training loss falls to 0.06 to 0.19 while validation loss climbs from about 1.0 to 2.6 to 3.9. The best epochs are 3 to 9 and validation macro-F1 never improves afterwards. This is the clearest case of memorising participants.
- **yemas learns more slowly and less severely.** Training loss ends at 0.59 to 0.98 and validation loss stays near 0.9 to 1.5, with one spike (6.67 in fold 0, epoch 5). The chosen epoch varies a lot (2 to 26), so the selected checkpoint depends on noise.
- **unas is extremely unstable.** Validation accuracy swings from 0.13 to 0.67 between epochs and validation loss from 1.03 to 2.17. In folds 1 and 2, validation loss rises steeply after about epoch 20 while validation macro-F1 keeps creeping up (to about 0.5), so loss and F1 give opposite signals. The best epochs range from 6 to 34 across folds.
- **Validation sets are small** (about 100 participants for unas, 800 frames of about 100 participants for yemas and palmas), so validation macro-F1, used for model selection, is itself noisy. This probably explains some of the fold-to-fold spread in the test results.
- **Class weights shape the curves.** Loss and accuracy can move in opposite directions because the weighted loss rewards minority-class recall.
- The notebook suggests lowering the learning rate to 3e-4 "if unstable"; this was not tried.

---

## 8. Grad-CAM, methodology summary and fidelity caveat

**Grad-CAM (qualitative).** One random test image per class was explained for each site, using the **fold-0** CNN and the **predicted** class. Predictions on those 9 images: unas normal→normal, leve→normal, moderada→normal; yemas normal→normal, leve→normal, moderada→moderada; palmas normal→leve, leve→leve, moderada→moderada.

- **unas:** in the normal and moderada examples the strongest activation is on the image edge or background, not the nail. Only in the leve example does the heat sit on the nail.
- **yemas:** heat maps are coarse and diffuse, often covering large regions or the image border rather than the fingertip pad.
- **palmas:** activation sits mostly on fingers and knuckles, and partly on the palm creases; the normal example is a closed fist.
- Nine images from one fold are an illustration, not evidence. Even so, the unas and yemas maps do not show clear focus on the intended skin region, which fits the weak results and the possible role of background or lighting.

**Summary.** Each of three body sites is modelled independently. A baseline logistic regression on mean RGB is compared with an ImageNet-pretrained EfficientNet-B0 (GeM pooling) trained on 224×224 RGB images with class-weighted loss, augmentation and early stopping. Evaluation uses a stratified participant-level 5-fold CV shared across sites, with frame probabilities averaged per participant, and metrics pooled over the five test folds.

**Fidelity caveat.** This notebook is an *adapted* pipeline, not a faithful replication of Kato et al., *Br J Haematol* 2024. Based on the notebook's own comments:

- The task is changed to **3-class severity** with a multinomial logistic regression baseline.
- The notebook uses **40 epochs** where the paper used 60. Batch size 32 and lr 1e-3 follow the paper.
- The paper averaged left and right eye predictions per participant; here the **average is over frames** (8 per participant for yemas and palmas, 1 to 2 for unas).
- Frames are subsampled to 8 per participant to save time.
- Early stopping on validation macro-F1 (patience 10) and balanced class weights are choices of this notebook.
- Hb thresholds and hemoglobin values were **not used** (labels come from folder names).
- Results should therefore **not be compared numerically with the paper's reported numbers**. They also should not be compared directly with the earlier single-split, per-image notebook (e.g. its yemas CNN scored macro-F1 0.487 on one 70/15/15 split): the evaluation design is different, and this notebook's participant-level CV is the more conservative of the two.

---

## 9. Limitations and recommended next steps

**Limitations**

1. **Weak signal.** Macro AUC is 0.58 to 0.69 and macro-F1 0.32 to 0.45. Every model's accuracy is below the always-Normal rate (about 0.668).
2. **Small Moderada sample.** Only 73 to 75 Moderada participants per site (about 15 per fold), so Moderada metrics move a lot between folds.
3. **No confidence intervals, single seed.** Only the fold standard deviation is reported. The CNN-vs-baseline gaps (0.03 to 0.07 macro-F1) are smaller than or comparable to the fold standard deviation (0.04 to 0.08).
4. **Mean-RGB baseline is almost as good as the CNN.** Colour alone carries most of the usable signal, and colour can also reflect lighting, camera or session differences that are not controlled.
5. **Unstable training and noisy model selection.** Validation sets are small, validation accuracy and loss swing widely, and the chosen epoch varies from 2 to 34 across folds.
6. **palmas overfitting** (training loss 0.06 to 0.19 vs validation loss up to 3.9), and a validation-loss spike of 6.67 in yemas fold 0.
7. **Frame subsampling.** Only 8 frames per participant (of up to 64 and 32 available) are used for yemas and palmas, and the notebook assumes they are near-identical without testing it.
8. **unas has one image per participant** (at most 2) and the smallest validation sets, so its results are the least reliable.
9. **Labels from folder names only.** No Hb check or regression, and the provided CSVs are unused.
10. **Class weights lower accuracy** below the majority baseline; no threshold tuning or calibration was done.
11. **Grad-CAM** shows 9 images from one fold only, and attention on borders or background cannot be excluded.
12. **No external validation.**

**Recommended next steps**

1. Add bootstrap confidence intervals over participants and repeat the CV with several seeds before ranking models.
2. Gather more Moderada participants, or merge classes, or use an ordinal target.
3. Stabilise training: lr 3e-4, stronger regularisation for palmas, a larger validation set, selection on validation loss or AUC.
4. Use more frames per participant and add ROI segmentation with colour and illumination normalisation.
5. Fuse the three sites per participant using the shared folds.
6. Tune thresholds and calibrate probabilities, since the models rank (AUC 0.6 to 0.7) better than their argmax decisions suggest.
7. If hemoglobin values are available (`metadata.csv`), add an Hb regression head and evaluate with MAE / RMSE / R².
8. Validate on an external dataset before any claim of screening usefulness.

---

## 10. Included files

All under `/kaggle/working/metrics_all/ALL_modalities/` (plus per-site folders; the whole `metrics_all` folder is zipped as `/kaggle/working/ALL_RESULTS.zip`):

| File | Description |
|---|---|
| `final_comparison_all_modalities.csv` | The 6-row participant-level comparison table (section 2) |
| `per_class_all_modalities.csv` | Per-class precision / recall / F1 / AUC / support (section 3) |
| `comparison_bar_chart.png` | Accuracy, macro-F1 and macro AUC by site and model |
| `comparison_confusion_matrices.png` | 2×3 grid of row-normalised confusion matrices with counts (section 4) |
| `comparison_roc_cnn.png` | CNN one-vs-rest ROC curves, all sites (section 5) |
| `gradcam_all_modalities.png` | Grad-CAM for 3 images per site (section 8) |
| `{site}/cnn_per_fold_metrics.csv`, `cnn_fold_mean_std.csv`, `cnn_image_level_metrics.csv` | Fold-level and image-level CNN metrics |
| `{site}/cnn_learning_curves.png` | Validation loss and macro-F1 per fold (section 7) |
| `{site}/{baseline_meanRGB_LogReg,cnn}_report.png`, `..._participant_predictions.csv` | Per-model confusion matrices, ROC and predictions |
