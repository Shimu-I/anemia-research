# Results Interpretation: Three-Class Anemia Severity Replication

Companion to `methodology_notes.md`. All figures come from the executed notebook `methodology-2.ipynb` (outputs and embedded plots). Where a number was not printed by the notebook and had to be derived, this is stated.

**Read this first:** the high scores in this run (97-99.9%) are not trustworthy performance estimates. The EHR features include three columns that are the label itself, the train/test split puts frames of the same person on both sides, and the hyperparameter search silently failed. Sections 2 and 9 explain the consequences. The only results free from the label leak are the image-only CNN results.

---

## 1. Run summary

| Item | Value |
|---|---|
| Date / hardware | 2026-10-28, Kaggle, 2 x Tesla T4, TensorFlow 2.20.0 |
| Task | 3-class severity: Normal (0), Leve (1), Moderada (2) |
| Image sources | palmas (13,328 frames), unas (826 photos), yemas (26,816 frames); each analysed separately |
| Total labelled images | 40,970 (Normal 27,367 / Leve 9,946 / Moderada 3,657) |
| Labels from | folder names (100% coverage); hemoglobin thresholds were not needed |
| Split | stratified random 70/30 by image per source, 10% of train used as CNN validation; seed 42 |
| Test images | palmas 3,999; unas 248; yemas 8,045 |
| Majority-class baseline | 66.7% (palmas), 66.5% (unas), 66.8% (yemas) |
| CNN training | MobileNetV2 backbone, Adam 1e-4, batch 32, max 20 epochs, early stopping (patience 5) |
| Experiments | A: EHR only; B: image only; C: CNN embedding + EHR; D: Grad-CAM RGB + EHR |
| Repeats | single run, no confidence intervals |

---

## 2. Held-out test metrics

Values are percentages: accuracy / macro ROC-AUC (one-vs-rest) / macro F1.

**B. Image only (the cleanest evidence of what the images contain)**

| Source | Model | Acc | AUC | Macro P | Macro R | Macro F1 | MCC |
|---|---|---|---|---|---|---|---|
| palmas | MobileNetV2 | 72.69 | 89.75 | 65.12 | 74.70 | 67.61 | 54.07 |
| palmas | CBAM | **87.37** | **95.81** | 86.09 | 78.74 | **81.90** | 73.15 |
| palmas | RCBAM | 44.26 | 66.76 | 36.12 | 39.62 | 30.84 | 15.49 |
| unas | MobileNetV2 | 66.94 | 56.14 | 39.07 | 34.22 | 28.78 | 8.67 |
| unas | CBAM | 66.53 | 52.60 | 22.18 | 33.33 | 26.63 | 0.00 |
| unas | RCBAM | 66.53 | 57.71 | 22.18 | 33.33 | 26.63 | 0.00 |
| yemas | MobileNetV2 | 65.21 | 63.25 | 28.99 | 36.54 | 31.92 | 9.31 |
| yemas | CBAM | 66.58 | 61.55 | 36.88 | 34.89 | 31.01 | 8.25 |
| yemas | RCBAM | 63.59 | 71.16 | 45.13 | 43.69 | 43.98 | 20.02 |

**A. EHR only (invalid: label leakage)**

| Source | LR | DT | RF | SVM | NB | KNN |
|---|---|---|---|---|---|---|
| palmas | 98.85 / 99.58 / 98.54 | 75.39 / 72.49 / 61.05 | 98.85 / 99.19 / 98.54 | 98.80 / 99.12 / 98.42 | 98.85 / 99.12 / 98.54 | 99.80 / 99.91 / 99.81 |
| unas | 97.98 / 98.32 / 97.43 | 75.00 / 72.00 / 60.62 | 97.98 / 98.93 / 97.43 | 97.98 / 97.13 / 97.43 | 97.98 / 98.32 / 97.43 | 97.18 / 97.17 / 96.69 |
| yemas | 98.73 / 99.58 / 98.41 | 75.54 / 72.52 / 61.06 | 98.73 / 98.81 / 98.41 | 98.69 / 98.93 / 98.32 | 98.73 / 98.99 / 98.41 | 99.76 / 99.89 / 99.78 |

**C. Fusion: CNN embedding + EHR, SelectKBest(50) + logistic regression (invalid: same leak)**

| Source | MobileNetV2 | CBAM | RCBAM |
|---|---|---|---|
| palmas | 99.10 / 99.82 / 99.04 | 99.42 / 99.94 / 99.34 | 98.85 / 99.66 / 98.73 |
| unas | 97.98 / 96.65 / 97.43 | 97.98 / 97.72 / 97.43 | 97.98 / 98.36 / 97.43 |
| yemas | 98.74 / 99.48 / 98.44 | 98.73 / 99.48 / 98.41 | 98.69 / 99.27 / 98.34 |

**D. Grad-CAM RGB + EHR (invalid: same leak, and effectively a copy of A)**

| Source | Variant | KNN | LR | RF | SVM | DT |
|---|---|---|---|---|---|---|
| palmas | tuned | 99.87 / 99.91 / 99.88 | 98.85 / 99.62 / 98.54 | 98.85 / 98.88 / 98.54 | 98.85 / 98.84 / 98.55 | 75.39 / 72.49 / 61.05 |
| palmas | paper params | 99.37 / 99.97 / 99.38 | same | same | same | same |
| unas | tuned | 96.77 / 97.16 / 96.33 | 97.98 / 98.44 / 97.43 | 97.98 / 98.19 / 97.43 | 97.98 / 97.56 / 97.43 | 75.00 / 72.00 / 60.62 |
| unas | paper params | 97.98 / 97.26 / 97.43 | same | same | same | same |
| yemas | tuned | 99.85 / 99.96 / 99.86 | 98.73 / 99.60 / 98.41 | 98.73 / 98.67 / 98.41 | 98.69 / 98.85 / 98.32 | 75.54 / 72.52 / 61.06 |
| yemas | paper params | 99.81 / 99.98 / 99.83 | same | same | same | same |

**How to read this.**

- Rows A, C and D are all in a narrow 97-99.9% band regardless of image model. Fusion barely beats EHR-only (palmas CBAM fusion 99.42% vs EHR-only LR 98.85%), and D is numerically identical to A for LR, RF, SVM and DT. The image contributes almost nothing; the EHR leak does the work.
- "Tuned" and "paper parameters" are identical for LR, RF, SVM and DT because the grid search failed (every score was NaN because `f1` is binary-only), so the first grid entry was used. It differs only for KNN (k=3 vs k=19). The ~75% DT figure is a depth-1 stump.
- The decision-tree results are the lowest EHR-model results, not because trees are poor here, but because the stump has a single split.
- Only B tells you something about the images. Even there the evaluation is optimistic, because frames from one person appear in both train and test.

---

## 3. Class-wise results

The notebook stores per-class precision, recall and F1 in each result JSON but does not print them. The values below are **derived by me from the 3 x 3 confusion matrices** of the Random Forest (`HAND_TUNED`) shown in the embedded plots. They describe the leaked D-experiment, so they show how easy the leaked task is, not how well images identify severity. RF in `HAND_PAPER` has identical headline metrics.

| Source | Class | Support | Precision | Recall | F1 |
|---|---|---|---|---|---|
| palmas | Normal | 2,669 | 98.63 | 100.00 | 99.31 |
| palmas | Leve | 975 | 99.05 | 96.21 | 97.61 |
| palmas | Moderada | 355 | 100.00 | 97.46 | 98.72 |
| unas | Normal | 165 | 97.63 | 100.00 | 98.80 |
| unas | Leve | 61 | 98.28 | 93.44 | 95.80 |
| unas | Moderada | 22 | 100.00 | 95.45 | 97.67 |
| yemas | Normal | 5,376 | 98.48 | 100.00 | 99.23 |
| yemas | Leve | 1,949 | 98.99 | 95.74 | 97.34 |
| yemas | Moderada | 720 | 100.00 | 97.36 | 98.66 |

Pattern: Normal always has 100% recall; the minority classes lose a few percent of recall, and the errors are always between neighbouring classes (see section 4).

For the image-only CNNs no per-class numbers were printed. What the macro metrics imply:

- **unas CBAM and RCBAM:** macro recall 33.33% and MCC 0.00 mean only one class is ever predicted (Normal), so Leve and Moderada recall is 0%.
- **palmas CBAM:** macro recall 78.7% against macro precision 86.1% suggests the minority classes are under-detected, but the exact split cannot be confirmed without the stored JSON.

---

## 4. Confusion matrix interpretation

Random Forest, `HAND_TUNED` (rows = true class, columns = predicted):

| palmas | Normal | Leve | Moderada |
|---|---|---|---|
| **Normal** | 2,669 | 0 | 0 |
| **Leve** | 37 | 938 | 0 |
| **Moderada** | 0 | 9 | 346 |

| unas | Normal | Leve | Moderada |
|---|---|---|---|
| **Normal** | 165 | 0 | 0 |
| **Leve** | 4 | 57 | 0 |
| **Moderada** | 0 | 1 | 21 |

| yemas | Normal | Leve | Moderada |
|---|---|---|---|
| **Normal** | 5,376 | 0 | 0 |
| **Leve** | 83 | 1,866 | 0 |
| **Moderada** | 0 | 19 | 701 |

- No Normal image is ever called anemic, and no Moderada is ever called Normal (and vice versa): every error is a one-step error.
- All mistakes run in one direction, towards the less severe class: Leve is called Normal (37 / 4 / 83) and Moderada is called Leve (9 / 1 / 19). If these were genuine, that would mean mild cases are the clinically risky miss. Here it is more likely an artifact of the leaked features and SMOTE rather than a real property of the data.
- A perfectly clean Normal column plus 100% precision for Moderada is unusually tidy for a three-class medical problem, which supports the leakage reading.
- The image-only CNN confusion matrices were not plotted, so none are available for B. The `unas` CNN matrices can be inferred: all 248 predictions go to Normal.

---

## 5. ROC-AUC interpretation

The AUC reported everywhere is the **macro one-vs-rest AUC**: each class is scored against the other two and the three values are averaged. Per-class AUCs were not printed.

- **EHR / fusion / handcrafted (A, C, D):** 96.6-99.98%. DT is the exception at 72-72.5%, because a depth-1 tree outputs only a few distinct probabilities and the ROC curve collapses to straight segments (visible as the straight DT and RF-Moderada lines in the plots). These AUCs reflect the leak and should not be quoted.
- **Image-only CNNs (B):**
  - palmas CBAM 95.8% and MobileNetV2 89.8% show real ranking ability.
  - palmas RCBAM 66.8% is weak because early stopping restored epoch-1 weights (section 7).
  - unas 52.6-57.7% is essentially chance (50%).
  - yemas 61.6-71.2% is poor but above chance; RCBAM is best here (71.2%).
- **AUC vs accuracy:** a model can have AUC well above 50% while its accuracy equals the majority baseline (yemas, unas), since AUC measures ranking and accuracy measures the argmax. Do not read yemas 71% AUC as a usable classifier.
- **Plots:** the one-vs-rest ROC curves for the handcrafted models hug the top-left corner for every model except DT. The precision-recall curves tell the same story. The calibration panel is noisy and jagged because several classifiers produce few distinct probabilities and some bins have few samples; it should not be over-interpreted.
- With only 22 Moderada images in the unas test set, any AUC involving that class is unstable.

---

## 6. Hemoglobin regression interpretation

**Not performed.** The notebook has no regression model, no MAE/RMSE/R², and no continuous hemoglobin target. This is a classification-only run.

What the notebook does with hemoglobin:

- The metadata file has a `Hemoglobina` column, but the keyword filter (`hb`, `hemo`) removes it from the EHR features on purpose, to avoid leaking the label.
- The Hb thresholds (`HB_NORMAL_THRESHOLD = 11.0`; below `HB_MODERADA_THRESHOLD = 8.0` is Moderada; in between is Leve) exist only as a fallback for deriving labels when folder names are missing. They were **not triggered** because folder names labelled 100% of images. The code comment itself says these thresholds should be confirmed against the clinical protocol.

If a hemoglobin regression section is needed for a report, it has to be built and run separately; nothing in the current outputs can be interpreted for it.

---

## 7. Training trend

Read from the epoch logs printed in Cell 7 (training accuracy is the last printed or best-epoch value; validation loss is what early stopping watches).

| Run | Epochs run | Best val-loss epoch (weights kept) | Train acc at best | Val acc at best | Val acc range over run |
|---|---|---|---|---|---|
| palmas MobileNetV2 | 9 | 4 | 98.1% | 71.4% | 28.1-78.1% |
| palmas CBAM | 13 | 8 | 98.8% | 86.5% | 33.2-86.8% |
| palmas RCBAM | 6 | **1** | 70.1% | 42.7% | 23.8-69.0% |
| unas MobileNetV2 | 6 | **1** | 61.5% | 67.2% | 67.2-69.0% |
| unas CBAM | 6 | **1** | 64.8% | 67.2% | 67.2% flat |
| unas RCBAM | 6 | **1** | 60.6% | 67.2% | 65.5-67.2% |
| yemas MobileNetV2 | 6 | **1** | 66.0% | 65.1% | 12.7-65.1% |
| yemas CBAM | 6 | **1** | 66.9% | 67.2% | 49.4-67.2% |
| yemas RCBAM | 8 | 3 | 75.0% | 62.8% | 52.3-68.1% |

What this shows:

1. **Training accuracy climbs to 94-99% in every run, but validation loss is lowest at epoch 1-3 in 7 of 9 runs.** Early stopping then restored near-untrained weights. That is why palmas RCBAM scores 44% on test (it is the epoch-1 model) and why unas/yemas CNNs sit at the majority baseline.
2. **Validation accuracy is extremely erratic** (e.g. palmas MobileNetV2 jumps between 28% and 78%; yemas MobileNetV2 drops to 12.7% at epoch 2). Because validation is a random 10% of the same pool as training, an ordinary overfitting gap would be smaller and smoother. The instability points to optimisation problems, with the likeliest candidates being BatchNorm behaviour in a fully unfrozen MobileNetV2, and the lr/batch settings. This cause is a hypothesis; the notebook does not test it.
3. **unas validation accuracy is flat at 67.24%** (39/58 = the validation majority share) while training accuracy rises to 94-98%: the model memorises the 520 training images but learns nothing transferable.
4. **palmas CBAM is the only run that trained usefully** (validation accuracy up to 86.8%, test 87.4%), and it ran longest (13 epochs, best at 8).
5. **Same early-stopping point, same validation loss in two runs:** unas CBAM and yemas CBAM both report best val-loss 0.831; this is as printed and likely coincidence, but worth a glance at the CSV logs.
6. Classical models have no training curve; only their final test metrics exist.

---

## 8. Methodology summary and fidelity caveat

**Summary.** Images are cropped to a region of interest (entropy + triangle threshold), classified with MobileNetV2 / CBAM / reverse-CBAM, explained with Grad-CAM, and the highlighted-region R/G/B percentages and CNN embeddings are fused with EHR features and fed to six classical classifiers with SMOTE and (intended) grid search. Metrics are accuracy, macro precision/recall/F1, macro one-vs-rest AUC and MCC on a random 30% image-level test set.

**Fidelity caveat.** This is an adaptation, not a faithful replication of the paper:

- The paper's task is binary (anemic vs non-anemic) on conjunctiva images; this run is three-class on palm, nail and fingertip images, mostly video frames.
- The paper's 710-image dataset (214 test images, per the notebook's own comment) is not comparable to 40,970 frames from about 826 people, where many frames are near-duplicates.
- Several settings the paper did not state were chosen here: epochs, learning rate, dropout, the Grad-CAM threshold, SMOTE neighbours and the RCBAM details. The RCBAM block is the notebook author's reading of the paper's description.
- The grid search did not operate as intended, so "GridSearchCV tuned" results are not tuned.
- The EHR set differs from the paper's clinical features and contains label columns.

So paper numbers and these numbers should not be compared side by side, and the paper's conclusion that RCBAM is best was **not reproduced** (it is the worst on palmas and not clearly better elsewhere).

---

## 9. Limitations and recommended next steps

**Limitations (most to least serious)**

1. Target leakage: `Normal`, `Leve`, `Moderado` columns are in the selected EHR features. Invalidates experiments A, C and D.
2. Frame-level random split: frames and EHR vectors of the same person are in train and test. Inflates every result, including image-only ones.
3. Grid search scoring (`f1`, binary) failed for three classes, so no tuning occurred.
4. CNNs are effectively undertrained: early stopping restored epoch 1-3 weights in 7 of 9 runs; unas and yemas are at the majority baseline.
5. Grad-CAM RGB features have almost no class signal (class means differ by under 1 point; yemas is about 99.7% red) and come from the weakest model (RCBAM).
6. Patient IDs are blank for frame folders, so EHR statistics came almost entirely from the 826 unas rows, and a patient-wise split cannot work as coded.
7. Small test sets for the minority class (22 Moderada in unas); single seed; no confidence intervals; no external validation.
8. Hb-derived labelling thresholds are unconfirmed (unused in this run).
9. Not clinically validated; not suitable for any diagnostic use.

**Recommended next steps**

1. Drop the leaked columns and re-run all four experiments.
2. Parse person IDs from `.../ID004/frame_000.jpg`, set `SPLIT_BY_PATIENT = True`, and keep each person entirely in one split.
3. Set `GS_SCORING = "f1_macro"` and verify that the selected parameters are not simply the first grid entries.
4. Fix CNN training (staged unfreezing, BatchNorm inference mode, class weights, augmentation, lower LR) and check that validation accuracy tracks training accuracy before trusting any CNN number.
5. Report per-class recall (especially Leve and Moderada), repeated seeds or cross-validation, and confidence intervals.
6. Replace or drop the Grad-CAM colour features unless they show class signal on clean data.
7. Only then decide whether the fusion with EHR adds anything beyond the symptoms alone.

---

## 10. Included files

Produced by the notebook under `/kaggle/working/anemia_ramzan_3class_ckpt/` (not supplied to me; the numbers above come from the notebook's printed output and embedded plots):

| Path | Content |
|---|---|
| `summary_all_metrics.csv` | all experiments x models x sources, with confusion matrices as JSON strings |
| `results/<site>__<exp>__<model>.json` | per-run metrics, per-class precision/recall/F1, confusion matrix, best parameters |
| `results/<site>__<exp>__<model>_probs.npz` | test labels and class probabilities |
| `plots/<site>__HAND_TUNED.png`, `<site>__HAND_PAPER.png` | one-vs-rest ROC, precision-recall, calibration and RF 3 x 3 confusion matrix |
| `plots/<site>__feature_importance.png` | Random Forest feature importance (HAND_TUNED) |
| `cnn/<site>__<variant>_log.csv` | per-epoch training logs (use these to verify section 7) |
| `cnn/<site>__<variant>.weights.h5` | trained CNN weights |
| `features/index.csv`, `features/split.csv` | image index with labels and split assignment |
| `features/<site>_gradcam_rgb.csv` | Grad-CAM R/G/B percentage features |
| `models/*.joblib` | fitted classical pipelines |

Companion document: `methodology_notes.md`.
