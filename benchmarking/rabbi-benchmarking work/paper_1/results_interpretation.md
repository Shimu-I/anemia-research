# Results Interpretation: 3-Class Anemia Severity Classification

Source: `methodology-1.ipynb`. Every figure below is taken from the notebook's printed tables, classification reports, training logs, and the embedded ROC and confusion-matrix plots. Class order is always **Normal, Leve (mild), Moderada (moderate)**.

---

## 1. Run summary

- **Task:** 3-class severity classification (Normal / Leve / Moderada), run separately on three body sites: palmas (palm), unas (fingernail), yemas (fingertip pad).
- **Environment:** Kaggle notebook, 2× Tesla T4 GPUs, seed 42, images resized to 128×128.
- **Data:** 40,970 labelled images from 843 participants, labels taken from folder names (100% coverage).

| Site | Images | Normal | Leve | Moderada |
|---|---|---|---|---|
| palmas | 13,328 | 8,896 | 3,248 | 1,184 |
| unas | 826 | 551 | 202 | 73 |
| yemas | 26,816 | 17,920 | 6,496 | 2,400 |

- **Participants:** 563 Normal, 204 Leve, 76 Moderada (843 total).
- **Split:** participant-level, stratified, 70 / 15 / 15 → 589 train, 127 validation, 127 test participants. No participant appears in two splits.

| Site | Train images | Val images | Test images |
|---|---|---|---|
| palmas | 9,328 | 2,000 | 2,000 |
| unas | 579 | 124 | 123 |
| yemas | 18,688 | 4,064 | 4,064 |

- **Models per site (5):** Naive Bayes, Decision Tree, kNN (k=5), SVM (sigmoid, C=100) on 7 colour features, and a 4-block CNN on raw RGB.
- **Headline result:** the **yemas CNN** is the best model (accuracy 0.6777, macro-F1 0.4874, macro AUC 0.7299). Every other model is weak, and several are no better than predicting "Normal" for everything.

---

## 2. Held-out test metrics

Macro-averaged over the three classes. `CV acc` is mean 5-fold participant-grouped CV accuracy on train+validation (classic models only; not run for the CNN).

| Site | Model | Accuracy | Macro P | Macro R | Macro F1 | Macro AUC | CV acc |
|---|---|---|---|---|---|---|---|
| palmas | CNN | 0.5580 | 0.3867 | 0.3990 | **0.3802** | **0.6572** | n/a |
| palmas | kNN | 0.5625 | 0.3581 | 0.3580 | 0.3578 | 0.5423 | 0.2857 |
| palmas | NaiveBayes | 0.5695 | 0.3765 | 0.3589 | 0.3546 | 0.6106 | 0.2911 |
| palmas | DecisionTree | 0.5720 | 0.3565 | 0.3512 | 0.3508 | 0.5439 | 0.2844 |
| palmas | SVM | 0.6720 | 0.2240 | 0.3333 | 0.2679 | 0.4960 | 0.3088 |
| unas | kNN | 0.6585 | 0.3743 | 0.3715 | **0.3541** | 0.5322 | 0.3220 |
| unas | DecisionTree | 0.6260 | 0.3421 | 0.3623 | 0.3465 | 0.5206 | 0.3003 |
| unas | NaiveBayes | 0.5041 | 0.3372 | 0.3789 | 0.3356 | 0.5851 | 0.2562 |
| unas | CNN | 0.4146 | 0.3022 | 0.3341 | 0.2832 | 0.5265 | n/a |
| unas | SVM | 0.6667 | 0.2222 | 0.3333 | 0.2667 | 0.4496 | 0.3199 |
| yemas | CNN | 0.6777 | 0.5272 | 0.5038 | **0.4874** | **0.7299** | n/a |
| yemas | kNN | 0.6270 | 0.4644 | 0.4190 | 0.4292 | 0.6286 | 0.3184 |
| yemas | DecisionTree | 0.5960 | 0.4080 | 0.3970 | 0.4005 | 0.6117 | 0.3177 |
| yemas | NaiveBayes | 0.4417 | 0.4226 | 0.4738 | 0.3753 | 0.6945 | 0.2436 |
| yemas | SVM | 0.6602 | 0.2221 | 0.3288 | 0.2651 | 0.6844 | 0.3216 |

How to read this:

- **Baseline:** always predicting Normal gives about 0.67 accuracy (palmas 0.672, unas 0.667, yemas 0.669) and macro-F1 near 0.27. The palmas and unas SVMs sit exactly on that baseline; their high accuracy is not skill.
- **Accuracy and macro-F1 disagree.** For example, palmas SVM has the highest accuracy (0.672) and the lowest macro-F1 (0.268). Rank models by macro-F1 and per-class recall, not accuracy.
- **Only the yemas CNN** is meaningfully above baseline on macro-F1 (0.487 vs about 0.27) and macro AUC (0.73). Its accuracy gain over the all-Normal baseline is under 1 percentage point.
- **CV vs. test discrepancy:** classic-model grouped-CV accuracy is only 0.24 to 0.32, which is below the 0.67 majority rate and much lower than their test accuracy (0.44 to 0.68). The notebook does not explain this. Treat the classic-model test numbers cautiously until it is investigated; it may reflect participant- or session-specific effects that do not generalise across people.

---

## 3. Class-wise results

Precision / recall / F1 per class on the test set (test support in the table headers).

### palmas (Normal 1,344 · Leve 480 · Moderada 176)

| Model | Normal P / R / F1 | Leve P / R / F1 | Moderada P / R / F1 |
|---|---|---|---|
| NaiveBayes | 0.714 / 0.770 / 0.741 | 0.336 / 0.165 / 0.221 | 0.079 / 0.142 / 0.102 |
| DecisionTree | 0.683 / 0.763 / 0.721 | 0.307 / 0.223 / 0.258 | 0.080 / 0.068 / 0.074 |
| kNN | 0.708 / 0.731 / 0.719 | 0.281 / 0.269 / 0.275 | 0.086 / 0.074 / 0.079 |
| SVM | 0.672 / 1.000 / 0.804 | 0 / 0 / 0 | 0 / 0 / 0 |
| CNN | 0.809 / 0.635 / 0.711 | 0.323 / 0.540 / 0.404 | 0.028 / 0.023 / 0.025 |

### unas (Normal 82 · Leve 30 · Moderada 11)

| Model | Normal P / R / F1 | Leve P / R / F1 | Moderada P / R / F1 |
|---|---|---|---|
| NaiveBayes | 0.721 / 0.537 / 0.615 | 0.290 / 0.600 / 0.391 | 0 / 0 / 0 |
| DecisionTree | 0.693 / 0.854 / 0.765 | 0.333 / 0.233 / 0.275 | 0 / 0 / 0 |
| kNN | 0.694 / 0.915 / 0.790 | 0.429 / 0.200 / 0.273 | 0 / 0 / 0 |
| SVM | 0.667 / 1.000 / 0.800 | 0 / 0 / 0 | 0 / 0 / 0 |
| CNN | 0.660 / 0.402 / 0.500 | 0.247 / 0.600 / 0.350 | 0 / 0 / 0 |

### yemas (Normal 2,720 · Leve 992 · Moderada 352)

| Model | Normal P / R / F1 | Leve P / R / F1 | Moderada P / R / F1 |
|---|---|---|---|
| NaiveBayes | 0.893 / 0.482 / 0.626 | 0.197 / 0.241 / 0.217 | 0.178 / 0.699 / 0.283 |
| DecisionTree | 0.716 / 0.770 / 0.742 | 0.316 / 0.282 / 0.298 | 0.192 / 0.139 / 0.161 |
| kNN | 0.721 / 0.806 / 0.761 | 0.354 / 0.310 / 0.330 | 0.319 / 0.142 / 0.197 |
| SVM | 0.666 / 0.986 / 0.795 | 0 / 0 / 0 | 0 / 0 / 0 |
| CNN | 0.763 / 0.878 / 0.816 | 0.545 / 0.225 / 0.318 | 0.273 / 0.409 / 0.328 |

Take-aways:

- **Normal** is learned reasonably well by almost everything (F1 about 0.7 to 0.8), largely because it is the majority class.
- **Leve** is poorly recognised: the best Leve F1 is 0.404 (palmas CNN), with yemas CNN recall only 0.225.
- **Moderada** is the weakest class. On palmas every model has Moderada F1 ≤ 0.10, and on unas every model scores 0 (only 11 test images).
- **yemas CNN** is the only model with non-trivial F1 on all three classes (0.82 / 0.32 / 0.33).
- **yemas Naive Bayes** trades precision for recall on Moderada (recall 0.70, precision 0.18), producing many false Moderada alarms.

---

## 4. Confusion matrix interpretation

Rows are actual classes, columns are predicted classes, both ordered Normal, Leve, Moderada. Each cell below lists the three rows in order.

| Site | Model | Normal row | Leve row | Moderada row |
|---|---|---|---|---|
| palmas | NaiveBayes | 1035 / 116 / 193 | 303 / 79 / 98 | 111 / 40 / 25 |
| palmas | DecisionTree | 1025 / 219 / 100 | 335 / 107 / 38 | 141 / 23 / 12 |
| palmas | kNN | 983 / 268 / 93 | 305 / 129 / 46 | 101 / 62 / 13 |
| palmas | SVM | 1344 / 0 / 0 | 480 / 0 / 0 | 176 / 0 / 0 |
| palmas | CNN | 853 / 420 / 71 | 152 / 259 / 69 | 49 / 123 / 4 |
| unas | NaiveBayes | 44 / 38 / 0 | 12 / 18 / 0 | 5 / 6 / 0 |
| unas | DecisionTree | 70 / 11 / 1 | 23 / 7 / 0 | 8 / 3 / 0 |
| unas | kNN | 75 / 6 / 1 | 24 / 6 / 0 | 9 / 2 / 0 |
| unas | SVM | 82 / 0 / 0 | 30 / 0 / 0 | 11 / 0 / 0 |
| unas | CNN | 33 / 49 / 0 | 12 / 18 / 0 | 5 / 6 / 0 |
| yemas | NaiveBayes | 1310 / 872 / 538 | 152 / 239 / 601 | 5 / 101 / 246 |
| yemas | DecisionTree | 2093 / 497 / 130 | 636 / 280 / 76 | 193 / 110 / 49 |
| yemas | kNN | 2191 / 483 / 46 | 624 / 307 / 61 | 225 / 77 / 50 |
| yemas | SVM | 2683 / 37 / 0 | 992 / 0 / 0 | 352 / 0 / 0 |
| yemas | CNN | 2387 / 167 / 166 | 552 / 223 / 217 | 189 / 19 / 144 |

What the matrices show:

- **SVM collapse:** on every site the SVM predicts Normal for essentially everything (palmas and unas: 100%; yemas: all but 37 images). Its Leve and Moderada columns are empty. This is a modelling failure (sigmoid kernel, C=100), not a data finding.
- **yemas CNN:** 2,387 of 2,720 Normal correct; Leve is mostly mislabelled as Normal (552) or Moderada (217), with only 223 of 992 correct; Moderada gets 144 of 352 correct, with 189 called Normal. Severity errors are spread, not neatly adjacent.
- **palmas CNN:** shifts predictions toward Leve (420 Normal images called Leve) and almost never gets Moderada right (4 of 176; 123 called Leve). It buys Leve recall at the cost of Normal recall.
- **yemas Naive Bayes:** heavily over-predicts Moderada (1,385 predictions, only 246 correct), and 872 Normal images are called Leve.
- **unas:** no model ever correctly predicts a Moderada image. With only 11 Moderada and 30 Leve test images, individual cells move by many percentage points with one participant.
- **Overall pattern:** errors run mostly toward the majority class (Normal) or one class over. The matrices show no model that reliably separates the three severity levels.

---

## 5. ROC-AUC interpretation

Macro one-vs-rest AUC is in section 2. Per-class one-vs-rest AUC read from the ROC plot legends:

| Site | Model | Normal | Leve | Moderada |
|---|---|---|---|---|
| palmas | NaiveBayes | 0.61 | 0.60 | 0.62 |
| palmas | DecisionTree | 0.55 | 0.54 | 0.54 |
| palmas | kNN | 0.57 | 0.52 | 0.53 |
| palmas | SVM | 0.48 | 0.52 | 0.49 |
| palmas | CNN | 0.68 | 0.66 | 0.64 |
| unas | NaiveBayes | 0.57 | 0.57 | 0.61 |
| unas | DecisionTree | 0.59 | 0.57 | 0.40 |
| unas | kNN | 0.58 | 0.56 | 0.46 |
| unas | SVM | 0.45 | 0.46 | 0.43 |
| unas | CNN | 0.53 | 0.52 | 0.52 |
| yemas | NaiveBayes | 0.78 | 0.54 | 0.76 |
| yemas | DecisionTree | 0.64 | 0.59 | 0.60 |
| yemas | kNN | 0.66 | 0.61 | 0.62 |
| yemas | SVM | 0.71 | 0.69 | 0.65 |
| yemas | CNN | 0.79 | 0.64 | 0.76 |

Reading the curves:

- **0.5 is chance.** Most palmas and unas values sit between 0.45 and 0.65, which is little better than random ranking. Several unas and SVM values are below 0.5.
- **yemas is the only site with clear signal**, and the CNN is the best (Normal 0.79, Moderada 0.76, Leve 0.64). Leve is consistently the weakest class in AUC as well as F1 for both Naive Bayes (0.54) and the CNN (0.64).
- **AUC and accuracy can disagree.** The yemas SVM has a macro AUC of 0.68 yet predicts almost only Normal. Its probabilities carry some ranking information, but the argmax decision rule throws it away. Thresholding or calibration could recover some of it.
- **Naive Bayes AUC is decent on yemas (0.69 macro) but its accuracy is the lowest (0.44)**, because its probabilities are poorly calibrated toward the majority class.
- **unas AUCs are unreliable:** with 11 Moderada and 30 Leve test images, the curves are step-like and noisy, and values below 0.5 (for example Decision Tree Moderada 0.40) are as likely noise as a real inverted relationship.
- AUC is computed per image, so frames from the same participant make the curves look smoother and more certain than the 127 test participants justify.

---

## 6. Hemoglobin regression interpretation

**Not applicable: this notebook does not perform hemoglobin regression.**

- There is no regression model, no Hb target, and no regression metric (MAE, RMSE, R²) anywhere in the notebook or its outputs.
- Hb thresholds exist only in the configuration (Normal ≥ 11.0, Leve 10.0 to <11.0, Moderada 7.0 to <10.0 g/dL) and are meant for an optional label-mapping fallback. That path **never executed**, because 100% of images already had a class label from their folder names.
- No claim about predicting hemoglobin levels, or about agreement between predicted classes and measured Hb, can be made from these results.

If a regression result is needed, it requires adding a continuous Hb target, a regression head or model, and evaluating with MAE / RMSE / R² (plus agreement with the class labels) on the same participant-level test split.

---

## 7. Training trend

Only the CNN has epoch-level logs. Early stopping (patience 8 on validation loss, best weights restored) ended all three runs well short of the 30-epoch cap, so **the reported test results come from very early epochs**.

| Site | Epochs run | Best epoch (val loss) | Best val loss | Val acc at best epoch | Train acc, first → last epoch | Val acc range across epochs |
|---|---|---|---|---|---|---|
| palmas | 11 | 3 | 0.8987 | 0.6065 | 0.459 → 0.548 | 0.238 to 0.687 |
| unas | 12 | 4 | 1.0108 | 0.4677 | 0.354 → 0.383 | 0.081 to 0.661 |
| yemas | 10 | 2 | 0.9164 | 0.6198 | 0.454 → 0.487 | 0.282 to 0.620 |

What the trend suggests:

- **Training is slow and low.** Training accuracy rises only a few points (palmas the most, 0.46 to 0.55; yemas almost flat at about 0.47 to 0.49). This is closer to under-fitting than classic over-fitting. Note that training accuracy is measured with augmentation and dropout active, so it understates fit.
- **Validation is extremely unstable.** Validation accuracy swings by 40 to 50 points between consecutive epochs (for example palmas 0.664 → 0.238 → 0.664 across epochs 1 to 7; unas dips as low as 0.08). This is far more than ordinary noise and points to training instability (SGD at learning rate 0.01 with momentum, BatchNorm statistics) and/or a validation set that differs in distribution from training participants. The notebook does not isolate the cause.
- **Early stopping selected epochs 2 to 4.** The final CNNs are therefore barely trained, chosen because validation loss never improved after those epochs. Longer or steadier training might help, but this was not tested.
- **Class weights shape the curves.** Balanced class weights (up to about 3.7× for Moderada) make loss and accuracy move in different directions, which partly explains why validation accuracy and loss do not track together.
- **unas is the least stable**, with only 579 training images and one image per participant.

---

## 8. Methodology summary and fidelity caveat

**Summary.** Each of three body sites is modelled independently. Classic models (Naive Bayes, Decision Tree, kNN, sigmoid-kernel SVM) use 7 colour features computed from a masked 128×128 image (CIE a\*, b\*, plus G, L, R, B and a\* standard deviation). The CNN uses the raw 128×128 RGB image with class-weighted loss and early stopping. Evaluation uses a stratified participant-level 70/15/15 split shared across sites, and metrics are computed per image on the untouched test split.

**Fidelity caveat.** This notebook is an *adapted* pipeline, not a faithful replication of the paper it draws on. Based on the notebook's own comments:

- The task is changed to **3-class severity**, whereas the source describes a different (anemic vs. non-anemic) setup.
- The code's split is 70/15/15, while a comment notes the paper used **70/10/20**.
- The notebook says the paper's kNN description is internally inconsistent (100 neighbours vs. k=2), so **k=5** was chosen.
- Features are the paper's **a\*, b\*, G plus extras** (L, R, B, a\* std), and the "ROI" step is a pixel brightness mask on the whole resized image rather than a dedicated ROI segmentation.
- Hb thresholds are configurable, flagged "verify against study criteria", and were **not used** (labels came from folder names).
- Results should therefore **not be compared numerically with the paper's reported numbers**.

---

## 9. Limitations and recommended next steps

**Limitations**

1. **Weak signal.** Only the yemas CNN clearly beats an always-Normal baseline, and even it recognises about 22% of Leve cases.
2. **Tiny effective sample for Moderada.** About 11 Moderada test participants per site; thousands of correlated frames count as independent test samples, so confidence is overstated.
3. **Per-image scoring.** No participant-level aggregation of frame predictions.
4. **Single split, single seed, no confidence intervals.** Differences of a few points between models are not trustworthy.
5. **Unexplained CV vs. test gap** for the classic models (CV accuracy 0.24 to 0.32 vs. test 0.44 to 0.68).
6. **SVM collapse** on every site, making its test numbers meaningless as a comparison.
7. **CNN instability and very early stopping** (epochs 2 to 4), so the CNN is barely trained.
8. **No Hb-based validation or regression**, and labels are taken at face value from folder names.
9. **Class imbalance** (about 67% Normal) makes accuracy misleading.
10. **Possible acquisition confounds** (lighting, camera, session) are not controlled, and no external dataset is used.
11. Validation-set metrics were computed and saved but not shown in the notebook output, so the test results cannot be compared against validation performance.

**Recommended next steps**

1. Aggregate predictions **per participant**, and report participant-level metrics with bootstrap confidence intervals.
2. Run **repeated participant-grouped cross-validation** for every model, including the CNN, and investigate the below-chance classic-model CV scores.
3. Gather more Moderada participants, or reconsider the class definition (for example merge or use ordinal modelling).
4. Stabilise CNN training (lower learning rate or Adam, scheduler, longer patience) and try **pretrained backbones** with colour/illumination normalisation.
5. Add real **ROI segmentation**, and consider fusing the three sites per participant.
6. Tune decision thresholds and calibrate probabilities (the SVM and Naive Bayes outputs carry more ranking information than their argmax predictions show).
7. If Hb measurements exist, add a **hemoglobin regression** target and evaluate with MAE / RMSE / R².
8. Validate externally before any claim of screening usefulness.

---

## 10. Included files

All under `/kaggle/working/anemia_ckpt_3class/` (also zipped as `/kaggle/working/anemia_ckpt_3class.zip`, about 25.7 MB):

| File | Description |
|---|---|
| `summary_test_metrics.csv` | The 15-row test comparison table (section 2) |
| `roc_curves_3class.png` | One-vs-rest ROC curves, one panel per site (section 5) |
| `confusion_matrices_3class.png` | 3×5 grid of test confusion matrices (section 4) |
| `splits/patient_split.csv` | Participant → train / val / test assignment |
| `features/{site}.csv`, `features/{site}_partial.pkl` | Extracted colour features and checkpoints |
| `results/{site}__{model}.json` | Per-model validation, test and CV metrics |
| `results/{site}__{model}_probs.npz` | Test labels and predicted probabilities |
| `models/{site}__{model}.joblib` | Fitted classic models |
| `cnn/{site}_final.keras`, `cnn/{site}_best.keras` | CNN weights per site |
