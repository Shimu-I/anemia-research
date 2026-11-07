# Methodology Notes: 3-Class Anemia Severity Classification

Source: `methodology-1.ipynb` (Kaggle run, 2× Tesla T4). All numbers below come from that notebook's printed outputs and embedded figures.

---

## What is the model trying to do?

The notebook predicts **anemia severity** from a single photo or video frame of one body site. It is a **3-class** problem, not anemic vs. non-anemic:

| Label ID | Class | Meaning |
|---|---|---|
| 0 | Normal | No anemia |
| 1 | Leve | Mild anemia |
| 2 | Moderada | Moderate anemia |

Severe anemia is not a class. The notebook states that severe cases are excluded unless a clinically approved policy is defined.

Three body sites are modelled **separately** (one set of models per site, no fusion across sites):

- **palmas**: palm (video frames)
- **unas**: fingernail (one still image per participant)
- **yemas**: fingertip pad (video frames)

Data used: **40,970 labelled images from 843 participants** (27,367 Normal, 9,946 Leve, 3,657 Moderada images; 563 / 204 / 76 participants). Labels were read from the **folder names** (100% coverage). The notebook also configures hemoglobin cutoffs (Normal ≥ 11.0, Leve 10.0 to <11.0, Moderada 7.0 to <10.0 g/dL), but that fallback never ran because folder labels already covered everything.

---

## How the components work

1. **Indexing.** Every image is found on disk. The participant ID (`ID###`) is parsed from the folder or file name, and the severity label from the path text (normal / leve / moderada).
2. **Colour-feature extraction (for the classic models).** Each image is resized to 128×128 and converted to CIE L\*a\*b\*. A pixel mask keeps pixels with 15 < L < 95 and max channel < 250 and min channel > 5 (drops too-dark and saturated pixels; if fewer than 50 pixels survive, all pixels are used). Seven features are averaged over the mask: **a\*, b\*, G** (the three the notebook attributes to the source paper) plus **L, R, B** and the **standard deviation of a\*** (extras).
3. **Classic classifiers** on those 7 features:

| Model | Setup |
|---|---|
| Naive Bayes | StandardScaler + GaussianNB |
| Decision Tree | min_samples_leaf=10, min_samples_split=5 |
| kNN | StandardScaler + k=5, Euclidean, uniform weights |
| SVM | StandardScaler + sigmoid kernel, C=100, probability=True |

4. **CNN** on the raw 128×128 RGB image (no handcrafted features, no explicit ROI segmentation):
   - Random translation (±5%), then four Conv(3×3)+BatchNorm blocks with 32, 64, 128, 256 filters (max-pooling after the first three), global average pooling, Dense(128), Dropout(0.4), 3-way softmax. L2 regularisation (1e-4) on conv and dense weights.
   - Training: SGD, learning rate 0.01, momentum 0.9, batch 32, up to 30 epochs, **balanced class weights** (e.g. yemas: 0.50 / 1.38 / 3.67 for Normal / Leve / Moderada), random horizontal flip on training images.
   - **Early stopping** on validation loss (patience 8, best weights restored).
5. **Outputs** per site and model: metrics JSON, saved test-set probabilities, fitted model, and summary plots.

---

## How the training was evaluated

- **Participant-level split**, stratified by class: 70% train / 15% validation / 15% test (589 / 127 / 127 participants). The same participant split is applied to every site, and the code asserts that no participant appears in more than one split.
- **Training** uses the train split only. The validation split is used for CNN early stopping. The **test split is held out** and scored once.
- **Metrics:** accuracy, macro precision, macro recall, macro F1, macro one-vs-rest ROC-AUC, and confusion matrices. Per-class precision, recall and F1 come from `classification_report`.
- **Grouped cross-validation (classic models only):** 5-fold `StratifiedGroupKFold` on train+validation, reporting accuracy and macro-F1. CV was **not** run for the CNN.
- **Scoring unit is the image/frame**, not the participant. Frames from the same participant are not aggregated into one prediction.
- Test-set sizes: palmas 2,000 images (125 participants), unas 123 images (123 participants), yemas 4,064 images (127 participants).

---

## Results to remember

Majority-class baseline (always predict Normal): about **0.67 accuracy** on every site (palmas 0.672, unas 0.667, yemas 0.669). Macro-F1 for that strategy is about **0.27**.

| Site | Best model by macro-F1 | Accuracy | Macro-F1 | Macro AUC |
|---|---|---|---|---|
| palmas | CNN | 0.5580 | 0.3802 | 0.6572 |
| unas | kNN | 0.6585 | 0.3541 | 0.5322 |
| yemas | **CNN** | **0.6777** | **0.4874** | **0.7299** |

- **Best overall: yemas CNN**, with Leve recall 0.22 and Moderada recall 0.41 (Normal recall 0.88).
- **SVM collapsed** on every site: it predicts essentially only Normal, so its accuracy equals the baseline while its macro-F1 is about 0.27.
- **unas** results are very small-sample (123 test images; only 11 Moderada) and several AUCs fall below 0.5.
- Classic-model grouped-CV accuracy is only **0.24 to 0.32**, far below their test accuracy (see limitation below).

### What do these mean?

- Accuracy is misleading here because about two-thirds of the data is Normal. A model that never detects anemia still scores about 0.67. Macro-F1 and per-class recall are the fairer measures.
- Only the yemas CNN clearly separates classes better than the baseline on macro-F1 and AUC, and even then its accuracy (0.678) is only about 1 point above the all-Normal baseline (0.669).
- A macro AUC of about 0.73 means moderate ranking ability, not a usable screening tool. Palmas (0.66) is weak, and unas (0.53) is essentially chance.
- Moderada is the hardest class everywhere and Leve is the second hardest. The models mostly learn "Normal vs. everything else", poorly.

---

## The most important limitation

**The signal is weak and the evidence is thin.** Apart from the yemas CNN, none of the models meaningfully beat an always-Normal predictor, and even the yemas CNN misses roughly three out of four Leve cases. At the same time, the test set contains only about **11 Moderada participants** per site (76 Moderada participants overall, 15% in test). Thousands of frames from the same few people are counted as independent test samples, so the real statistical sample size is the number of participants, not the number of images. A single split with a single seed and no confidence intervals cannot support claims of reliable severity grading.

A related warning sign: grouped CV accuracy for the classic models (0.24 to 0.32) is **below the 0.67 majority rate and far below their held-out test accuracy**. The notebook does not explain this gap. It needs to be investigated before any classic-model test number is trusted.

---

## How to describe the methodology accurately

**Safe wording:**

> "We built a 3-class anemia severity classifier (Normal / Mild / Moderate) from palm, fingernail and fingertip-pad images, comparing four classic classifiers on CIE L\*a\*b\*/RGB colour features against a small CNN. Evaluation used a stratified, participant-level 70/15/15 split so no participant appeared in more than one split. Results are reported on the held-out test split per image."

**Do say:**
- It is an **adaptation** of a published pipeline (the notebook says so), not a replication.
- Labels come from dataset folder names; Hb thresholds were configured but not used.
- Performance is modest; the best result is the yemas CNN (macro-F1 0.487, macro AUC 0.730).
- Splits are participant-level, so frames from one person never straddle train and test.

**Don't say:**
- That it "detects" or "diagnoses" anemia, or that it is clinically validated.
- That it estimates hemoglobin. **No hemoglobin regression is performed in this notebook.**
- That accuracy shows it works; the 0.67 baseline makes accuracy uninformative.
- That results were aggregated per participant, or that CV supports the test numbers.
- That it matches the source paper's setup (split ratio, kNN setting and feature handling differ; see the results interpretation file).

---

## If asked what should improve next

1. **Report participant-level results** (aggregate frame predictions per participant) and add confidence intervals, for example by bootstrapping participants.
2. **Use repeated or nested participant-grouped CV** for all models, including the CNN, instead of one split. Investigate why the classic-model CV scores sit below chance.
3. **Address Moderada scarcity:** more Moderada participants, or merge/relax classes, since about 11 test participants cannot support per-class claims.
4. **Stabilise CNN training:** lower learning rate or Adam, learning-rate schedule, more patience; the validation curve is highly erratic and the chosen weights come from epochs 2 to 4.
5. **Add proper ROI segmentation** (nail bed, palm crease, fingertip pad) and colour/illumination normalisation, since camera and lighting differences across sessions can swamp small colour shifts.
6. **Try pretrained backbones** and fuse the three sites per participant.
7. **Add threshold tuning and calibration** (for example class-specific thresholds or probability calibration), and fix the SVM, whose sigmoid kernel collapses to all-Normal.
8. **If Hb is available, add a hemoglobin regression head** and evaluate with MAE / RMSE / R², rather than relying only on folder labels.

---

## Files and paths

**Input data (Kaggle):**
- `/kaggle/input/datasets/shimu080/dataset2/Dataset2.csv`
- `/kaggle/input/datasets/shimu080/metadata/metadata.csv`
- `/kaggle/input/datasets/shimu080/unas-palmas-yemas/yemas_frames/yemas_frames`
- `/kaggle/input/datasets/shimu080/unas-palmas-yemas/palmas_frames/palmas_frames`
- `/kaggle/input/datasets/shimu080/unas-palmas-yemas/unas_images/unas_images`

**Working directory:** `/kaggle/working/anemia_ckpt_3class/`

| Path | Contents |
|---|---|
| `features/{site}.csv` | Extracted colour features per image |
| `features/{site}_partial.pkl` | Feature-extraction checkpoints |
| `splits/patient_split.csv` | Participant → train / val / test assignment |
| `results/{site}__{model}.json` | Validation, test and CV metrics |
| `results/{site}__{model}_probs.npz` | Test labels and predicted probabilities |
| `models/{site}__{model}.joblib` | Fitted classic models |
| `cnn/{site}_final.keras`, `cnn/{site}_best.keras` | CNN weights (final and best-validation checkpoint) |
| `summary_test_metrics.csv` | Test comparison table (15 rows) |
| `roc_curves_3class.png` | One-vs-rest ROC curves |
| `confusion_matrices_3class.png` | 3×5 grid of confusion matrices |

**Archive:** `/kaggle/working/anemia_ckpt_3class.zip` (about 25.7 MB).

**Config note:** `FORCE_RERUN=True`, so existing checkpoints were ignored in this run.
