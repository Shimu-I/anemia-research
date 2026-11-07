# Methodology Notes: 3-Class Anemia Severity Classification (EfficientNet-B0 + Participant-Level 5-Fold CV)

Source: `methodology-3.ipynb` (Kaggle run, CUDA GPU, torch 2.11.0+cu128, timm 1.0.29, albumentations 2.0.8). All numbers below come from that notebook's printed outputs and embedded figures. Where a number is not printed directly, it is derived from printed tables and marked "derived".

---

## What is the model trying to do?

The notebook predicts **anemia severity** from a photo or video frame of one body site, then combines the frames of each participant into **one participant-level prediction**. It is a **3-class** problem, not anemic vs. non-anemic:

| Label ID | Class | Meaning |
|---|---|---|
| 0 | normal | No anemia |
| 1 | leve | Mild anemia |
| 2 | moderada | Moderate anemia |

(`moderado` in a folder name is also mapped to class 2.) Severe anemia is not a class.

Three body sites ("modalities") are modelled **separately** (one set of models per site, no fusion across sites):

- **unas**: fingernail (still images, at most 2 per participant)
- **yemas**: fingertip pad (video frames)
- **palmas**: palm (video frames)

Data indexed: **40,970 images** (unas 826, yemas 26,816, palmas 13,328). Labels were read from the **folder names** (Normal / Leve / Moderada), with 0 images missing an ID or label. A check asserts that no participant has two different labels, within or across modalities.

| Site | Images indexed | Participants | Normal | Leve | Moderada | Images used for training/testing |
|---|---|---|---|---|---|---|
| unas | 826 | 823 | 550 | 200 | 73 | 826 (all) |
| yemas | 26,816 | 834 | 558 | 201 | 75 | 6,672 (8 per participant) |
| palmas | 13,328 | 829 | 554 | 201 | 74 | 6,632 (8 per participant) |

The union of participants across the three sites is **839** (561 Normal, 202 Leve, 76 Moderada; counted from the fold table). The notebook methodology is "adapted from Kato et al., *Br J Haematol* 2024".

The input files `Dataset2.csv` (999 rows) and `metadata.csv` (909 rows; includes a `Hemoglobina` column) are **only opened and displayed** in Cell 3. They are not used for labels, splits or any model.

---

## How the components work

1. **Indexing.** Every `.jpg/.jpeg/.png/.bmp` is found on disk. The participant ID (`ID###`) and the class label are parsed from the folder or file path.
2. **Frame subsampling.** For yemas and palmas only **8 frames per participant** are kept (`MAX_FRAMES`), chosen at evenly spaced positions in the sorted file list. The notebook comment says the frames are near-identical, so more frames would only be slower. unas keeps everything. Observed maximum images per participant before subsampling: unas 2, yemas 64, palmas 32.
3. **One shared participant-level 5-fold split.** `StratifiedKFold(5, shuffle=True, random_state=42)` is run on the union of all participants, stratified by class, and saved to `folds_global.json`. **The same folds are used for all three sites** so the comparison between sites is fair. Folds hold 167 to 168 participants each, with 112 to 113 Normal, 40 to 41 Leve and 15 to 16 Moderada per fold.
4. **Non-CNN baseline** (per site): the mean R, G, B of each image (3 features) → `StandardScaler` → multinomial `LogisticRegression` (`max_iter=2000`, `class_weight="balanced"`). Trained on all participants outside the test fold; no validation set is used.
5. **CNN** (per site and fold), on the full RGB image resized to **224×224**:
   - Backbone: ImageNet-pretrained **EfficientNet-B0** (timm, weights downloaded), then **GeM pooling** (learnable p, initial 3.0), Dropout 0.3, one Linear layer with 3 outputs. No handcrafted features and no ROI segmentation.
   - Augmentation (Albumentations, train only): horizontal and vertical flip (p=0.5 each); affine (scale 0.9 to 1.1, translate ±5%, rotate ±15°, p=0.5); elastic **or** grid distortion (p=0.3); coarse dropout (1 to 6 holes of 8 to 32 px, p=0.3); Gaussian **or** motion blur (p=0.3); ImageNet normalisation.
   - Training: **AdamW**, lr 1e-3, weight decay 1e-2, **cosine annealing** (T_max=40), batch size 32, up to **40 epochs**, mixed precision (AMP). Loss is cross-entropy with **balanced class weights** computed from the training fold.
   - **Early stopping** and model selection on **validation macro-F1**, patience **10** epochs. The best-F1 weights are reloaded for testing.
   - Each fold's checkpoints are saved every epoch, and the cell can be re-run to resume.
6. **Participant-level aggregation.** The softmax probabilities of all frames from one participant are **averaged**, and the argmax of the mean is that participant's prediction. The same is done for the baseline.
7. **Grad-CAM** from the fold-0 CNN, one random test image per class and site.
8. **Outputs** per site: metrics CSVs, confusion matrices, ROC plots, learning curves, per-fold metrics, saved predictions, and a combined comparison across sites.

---

## How the training was evaluated

- **Participant-level 5-fold cross-validation.** Every participant is in the test fold exactly once. Test predictions from the five folds are **pooled** (one prediction per participant per site) and the headline metrics are computed on the pooled set. Pooled test size: **unas 823, yemas 834, palmas 829 participants**.
- **Validation inside each fold.** The four training folds are split again at participant level, stratified: 85% train / 15% validation. Typical sizes (images): unas 559 to 564 / 99 to 101 / 162 to 167; yemas 4,528 to 4,544 / 800 to 808 / 1,320 to 1,344; palmas 4,496 to 4,520 / 800 / 1,312 to 1,336. The validation split drives early stopping only. The test fold is scored once with the selected weights.
- **Seeds:** split seed 42; each fold uses seed 42 + fold index. One run only, no repeated seeds.
- **Metrics:** accuracy, macro and weighted precision / recall / F1, macro one-vs-rest ROC-AUC, per-class precision / recall / F1 / AUC, and raw and row-normalised confusion matrices. The CNN also gets **per-fold** metrics (mean ± std over the 5 folds) and image-level metrics (saved to CSV but not shown in the notebook output).
- **Scoring unit is the participant** (frame probabilities are averaged). Image-level test accuracy per fold is printed during training only.
- **Statistical uncertainty:** only the fold-to-fold standard deviation is reported (no bootstrap confidence intervals, no significance tests).

---

## Results to remember

Majority-class baseline (always predict Normal), derived from the class counts: about **0.668 accuracy** and about **0.267 macro-F1** on every site.

| Site | Model | Accuracy | Macro-F1 | Macro AUC |
|---|---|---|---|---|
| unas | Non-CNN (mean RGB + LogReg) | 0.3670 | 0.3186 | 0.5839 |
| unas | CNN | **0.5772** | **0.3916** | **0.6254** |
| yemas | Non-CNN | 0.4976 | 0.4012 | 0.6746 |
| yemas | CNN | 0.5228 | **0.4470** | **0.6891** |
| palmas | Non-CNN | 0.4765 | 0.3758 | **0.6505** |
| palmas | CNN | **0.5356** | **0.4024** | 0.6482 |

- **Best overall by macro-F1 and AUC: yemas CNN** (accuracy 0.523, macro-F1 0.447, macro AUC 0.689). unas CNN has the highest accuracy (0.577) but a lower macro-F1.
- **CNN mean ± std over the 5 folds:** unas acc 0.578 ± 0.075, F1 0.389 ± 0.062, AUC 0.645 ± 0.066; yemas acc 0.523 ± 0.082, F1 0.442 ± 0.062, AUC 0.696 ± 0.035; palmas acc 0.535 ± 0.071, F1 0.392 ± 0.042, AUC 0.661 ± 0.032.
- The CNN is better than the mean-RGB baseline on macro-F1 on all three sites, but the margins are small (+0.07 unas, +0.05 yemas, +0.03 palmas) compared with the fold standard deviation (0.04 to 0.06).
- **Every model's accuracy is below the 0.668 majority baseline.** This is a side effect of balanced class weights, which trade Normal recall for Leve and Moderada recall. Macro-F1 (all above 0.267) and per-class recall are the useful measures.
- **Moderada is hardest, and Leve is second.** Best Moderada recall comes from the yemas models (CNN 0.453; baseline 0.627, with precision only 0.160). The unas CNN almost never finds Moderada (recall 0.069, 5 of 73).

### What do these mean?

- The signal is weak. Macro AUC of 0.58 to 0.69 means little better than weak ranking ability, and not a usable screening tool.
- yemas is the most informative site (AUC 0.69), then palmas (0.65), then unas (0.63).
- A 3-feature mean-RGB logistic regression gets most of what the CNN gets (palmas AUC is the same, 0.65). This suggests that overall colour carries much of the usable signal here, and that the pretrained CNN adds little on top. It also leaves open that colour differences between sessions, lighting or cameras contribute.

---

## The most important limitation

**The signal is weak, the sample of Moderada participants is small, and the CNN training is unstable.** Only 73 to 75 Moderada participants exist per site, so each fold tests about 15 of them. Fold-to-fold standard deviations of 0.04 to 0.08 are as large as the gaps between the CNN and the baseline, so the ranking of models is not statistically supported. The unas site has only one image per participant, a validation set of about 100 images (about 9 Moderada), and wildly swinging validation scores (validation accuracy from 0.13 to 0.67), so the selected epoch (anywhere from 6 to 34 across folds) is largely noise.

A related point: the palmas CNN clearly overfits. Training loss falls to 0.06 to 0.19 while validation loss rises to 2.6 to 3.9, and the chosen weights come from epochs 3 to 9 (best epochs 3, 7, 6, 4, 9). The yemas CNN also has a spike (validation loss 6.67 in fold 0, epoch 5).

---

## How to describe the methodology accurately

**Safe wording:**

> "We built a 3-class anemia severity classifier (normal / leve / moderada) from fingernail, fingertip-pad and palm images, comparing an ImageNet-pretrained EfficientNet-B0 (GeM pooling) with a logistic regression on mean RGB. Evaluation used a stratified, participant-level 5-fold cross-validation with the same folds for all three sites, and frame-level predictions were averaged per participant. Metrics are pooled over the five held-out folds."

**Do say:**
- It is an **adaptation** of Kato et al. (2024), not a replication (see the fidelity caveat in the results interpretation file).
- Labels come from dataset folder names. Hemoglobin values and the CSV files were not used.
- Performance is modest; the best result is the yemas CNN (macro-F1 0.447, macro AUC 0.689).
- Splits are participant-level, so frames from one person never straddle train, validation and test. Predictions are aggregated per participant.
- Only 8 frames per participant were used for yemas and palmas.

**Don't say:**
- That it "detects" or "diagnoses" anemia, or that it is clinically validated.
- That it estimates hemoglobin. **No hemoglobin regression is performed in this notebook.**
- That the accuracy shows it works: every model is below the always-Normal accuracy.
- That the CNN is clearly better than the baseline; the gap is within fold noise.
- That Grad-CAM shows the model looks at the right region (see the results interpretation file).
- That it matches the source paper's setup (epochs, frame averaging, 3-class task and other details differ).

---

## If asked what should improve next

1. **Add confidence intervals** (bootstrap over participants) and **repeat the CV with several seeds**; only then compare the CNN with the baseline.
2. **Address Moderada scarcity:** more Moderada participants, or merge or relax classes, or model severity as an ordinal target.
3. **Stabilise and regularise the CNN:** lower learning rate (the notebook itself suggests 3e-4), fewer epochs or stronger regularisation for palmas, a larger validation set, early stopping on validation loss or AUC instead of noisy macro-F1.
4. **Use more of the available frames** (currently 8 per participant for yemas and palmas), and add ROI segmentation (nail bed, palm, fingertip pad).
5. **Colour and illumination normalisation**, and a check for camera or session confounds, since mean RGB alone performs almost as well as the CNN.
6. **Fuse the three sites per participant** (the same folds already make this possible) and compare with single-site models.
7. **Threshold tuning and calibration**, since class weights moved accuracy well below the majority baseline.
8. **If Hb is available** (`metadata.csv` has a `Hemoglobina` column), add a hemoglobin regression head and evaluate with MAE / RMSE / R².
9. **External validation** before any claim of screening usefulness.

---

## Files and paths

**Input data (Kaggle):**
- `/kaggle/input/datasets/shimu080/dataset2/Dataset2.csv` (opened only)
- `/kaggle/input/datasets/shimu080/metadata/metadata.csv` (opened only)
- `/kaggle/input/datasets/shimu080/unas-palmas-yemas/yemas_frames/yemas_frames`
- `/kaggle/input/datasets/shimu080/unas-palmas-yemas/palmas_frames/palmas_frames`
- `/kaggle/input/datasets/shimu080/unas-palmas-yemas/unas_images/unas_images`

**Working directories:** `/kaggle/working/ckpt_all/` (checkpoints) and `/kaggle/working/metrics_all/` (metrics and plots), each with one sub-folder per site.

| Path | Contents |
|---|---|
| `ckpt_all/folds_global.json` | Participant → fold (0 to 4), shared by all sites |
| `ckpt_all/{site}/rgb_feats.npy` | Mean RGB features per image |
| `ckpt_all/{site}/baseline_fold{k}_preds.csv` | Baseline test probabilities per fold |
| `ckpt_all/{site}/cnn_fold{k}_preds.csv` | CNN test probabilities per image per fold |
| `ckpt_all/{site}/fold{k}_best.pt`, `fold{k}_last.pt` | Best-validation weights and resume checkpoint |
| `ckpt_all/{site}/fold{k}_history.json` | Per-epoch train loss, val loss, val accuracy, val macro-F1 |
| `metrics_all/{site}/{baseline_meanRGB_LogReg,cnn}_overall_metrics.csv` | Overall participant-level metrics |
| `metrics_all/{site}/{...}_per_class_report.csv` | Per-class precision / recall / F1 / AUC / support |
| `metrics_all/{site}/{...}_confusion_matrix_counts.csv`, `..._normalised.csv` | Confusion matrices |
| `metrics_all/{site}/{...}_report.png` | Confusion matrices and ROC curves |
| `metrics_all/{site}/{...}_participant_predictions.csv` | Per-participant probabilities and correctness |
| `metrics_all/{site}/cnn_per_fold_metrics.csv`, `cnn_fold_mean_std.csv`, `cnn_image_level_metrics.csv` | Fold-level and image-level CNN metrics |
| `metrics_all/{site}/cnn_learning_curves.png`, `results.json` | Validation curves; all metrics and matrices |
| `metrics_all/ALL_modalities/final_comparison_all_modalities.csv` | The 6-row comparison table |
| `metrics_all/ALL_modalities/per_class_all_modalities.csv` | Per-class results, all sites and models |
| `metrics_all/ALL_modalities/comparison_bar_chart.png`, `comparison_confusion_matrices.png`, `comparison_roc_cnn.png`, `gradcam_all_modalities.png` | Summary figures |

**Archive:** `/kaggle/working/ALL_RESULTS.zip` (the whole `metrics_all` folder).

**Config note:** `RESUME_FROM = None` and the log shows "Created and saved split", with no "RESUMED" or "skipping" messages, so this was a fresh run end to end.
