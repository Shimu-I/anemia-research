# Methodology Notes: Three-Class Anemia Severity Replication (Ramzan et al. 2024 pipeline)

Source: `methodology-2.ipynb`, executed on Kaggle (2 x Tesla T4, TensorFlow 2.20.0, Python 3.13) on 2026-10-28. Seed 42 throughout. Every number below is taken from the notebook's printed output or from the figures embedded in it.

---

## What is the model trying to do?

The notebook tries to reproduce the pipeline of Ramzan et al. (2024, *Discover Artificial Intelligence* 4:100), which combines an image model with electronic health record (EHR) features to detect anemia. Two things differ from the paper:

- **Three classes instead of two.** The task is **Normal (0) / Leve (1, mild) / Moderada (2, moderate)**, not Anemic / Non-anemic. Labels come from the folder names of the image files (100% of images were labelled this way).
- **Different image modality.** The paper used conjunctiva images. Here three image sources stand in for that modality, each analysed separately:
  - `palmas` (palms), 13,328 frames, extracted from video
  - `unas` (nails), 826 still photos
  - `yemas` (fingertips), 26,816 frames, extracted from video

In total there are 40,970 labelled images from about 826 people: Normal 27,367, Leve 9,946, Moderada 3,657 (66.8% / 24.3% / 8.9%).

Four experiments are run for each image source:

| Exp. | Name in code | Inputs | Classifier |
|---|---|---|---|
| A | `TEXT` | EHR features only | LR, DT, RF, SVM, NB, KNN |
| B | `IMG` | ROI image only | MobileNetV2, CBAM-MobileNetV2, RCBAM-MobileNetV2 |
| C | `FUSION_SKB` | CNN embedding (1280-d) + EHR, top 50 by SelectKBest | Logistic regression |
| D | `HAND_TUNED` / `HAND_PAPER` | Grad-CAM R/G/B percentages + EHR | SVM, DT, LR, KNN, RF |

---

## How the components work

**1. ROI extraction (Cell 4).** Each image is resized to 224 x 224 and converted to grayscale. A local-entropy map (disk radius 5) is computed, normalised to 0-255 and thresholded with the triangle method. Pixels outside the mask are set to black. If the mask covers under 5% or over 98% of the image, the whole image is kept instead.

**2. EHR preparation (Cell 5).**
- Text columns are factorised to integer codes, columns with 50% or more missing values are dropped, remaining gaps are filled with the median, and constant columns are removed.
- For each column a chi-square test and (for binary columns) an odds ratio are computed against the label. Columns with p < 0.1 are kept as "notable".
- Columns whose name contains `age`, `gender` or `sex` are force-included. Because the columns are Spanish, this matched `Sexo` but not `Edad` (age), which had p = 0.87 and was dropped.
- Hemoglobin (`Hemoglobina`) is deliberately excluded from the features by the keyword filter.

**3. The three CNN variants (Cell 7).** All use an ImageNet-pretrained MobileNetV2 backbone with no layers frozen, followed by an optional attention block, a layer named `feature_map` (the Grad-CAM target, 7 x 7), global average pooling, `Dense(1280, relu)` (named `emb`), dropout 0.3 and a 3-way softmax.
- **MobileNetV2:** no attention block.
- **CBAM:** channel attention then spatial attention (shared MLP on average and max pooling; 7 x 7 convolution for the spatial map).
- **RCBAM ("reverse CBAM"):** spatial attention then channel attention, applied at three scales (1.0, 0.5, 0.25), resized back and fused with learned sigmoid weights. This is the notebook's interpretation of the paper's description.

Training settings: Adam, learning rate 1e-4, batch 32, up to 20 epochs, early stopping on validation loss (patience 5, best weights restored). No data augmentation, no class weighting, no SMOTE for the CNNs.

**4. Grad-CAM to RGB features (Cell 8).** For the trained RCBAM model, the Grad-CAM heatmap of the predicted class is thresholded at 0.5 (threshold chosen here; not given in the paper). Within that highlighted, non-black region the share of red, green and blue intensity is computed and stored as three features per image.

**5. Classical classifiers (Cells 6 and 9).** Each is a pipeline of `StandardScaler`, then SMOTE (5 neighbours, applied only to training data inside the pipeline), then the classifier. `GridSearchCV` (5-fold) is meant to tune it. `HAND_PAPER` instead uses the paper's reported best parameters (SVM C=0.1 linear; DT depth 1; LR C=0.01; KNN k=19; RF depth 1, 50 trees).

**6. Metrics.** Accuracy, macro-averaged precision/recall/F1, macro one-vs-rest ROC-AUC and Matthews correlation coefficient (MCC), plus the 3 x 3 confusion matrix and per-class precision/recall/F1 stored in the JSON results.

---

## How the training was evaluated

- **Split:** a separate stratified random 70 / 30 split of *images* is made for each image source (`SPLIT_BY_PATIENT = False`, as in the paper). 10% of the training part is held out as validation for CNN early stopping only.
- **Classical models** are fit on train + validation and scored on the 30% test set. The CNNs are scored on the same test set after early stopping.
- **Test set sizes:**

| Source | Test images | Normal | Leve | Moderada |
|---|---|---|---|---|
| palmas | 3,999 | 2,669 | 975 | 355 |
| unas | 248 | 165 | 61 | 22 |
| yemas | 8,045 | 5,376 | 1,949 | 720 |

- **One run, one seed.** There are no repeated runs, no confidence intervals and no external dataset.
- **Majority-class baseline** (always predicting Normal) is 66.7% for palmas, 66.5% for unas and 66.8% for yemas. Any image-only accuracy near these numbers means the model has learned almost nothing.

---

## Results to remember

Test-set values, macro-averaged over the three classes (accuracy / macro AUC / macro F1, in %).

**B. Image only (CNN)**

| Source | MobileNetV2 | CBAM-MobileNetV2 | RCBAM-MobileNetV2 |
|---|---|---|---|
| palmas | 72.69 / 89.75 / 67.61 | **87.37 / 95.81 / 81.90** | 44.26 / 66.76 / 30.84 |
| unas | 66.94 / 56.14 / 28.78 | 66.53 / 52.60 / 26.63 | 66.53 / 57.71 / 26.63 |
| yemas | 65.21 / 63.25 / 31.92 | 66.58 / 61.55 / 31.01 | 63.59 / 71.16 / 43.98 |

**A. EHR only** (best and worst model per source)

| Source | Best | Worst |
|---|---|---|
| palmas | KNN 99.80 / 99.91 / 99.81 | DT 75.39 / 72.49 / 61.05 |
| unas | LR, RF, SVM, NB all 97.98 acc, F1 97.43 | DT 75.00 / 72.00 / 60.62 |
| yemas | KNN 99.76 / 99.89 / 99.78 | DT 75.54 / 72.52 / 61.06 |

**C. Fusion (CNN embedding + EHR, logistic regression)**

| Source | MobileNetV2 | CBAM | RCBAM |
|---|---|---|---|
| palmas | 99.10 / 99.82 / 99.04 | 99.42 / 99.94 / 99.34 | 98.85 / 99.66 / 98.73 |
| unas | 97.98 / 96.65 / 97.43 | 97.98 / 97.72 / 97.43 | 97.98 / 98.36 / 97.43 |
| yemas | 98.74 / 99.48 / 98.44 | 98.73 / 99.48 / 98.41 | 98.69 / 99.27 / 98.34 |

**D. Grad-CAM RGB + EHR** (`HAND_TUNED`; `HAND_PAPER` is identical except KNN)

| Source | KNN | LR | RF | SVM | DT |
|---|---|---|---|---|---|
| palmas | 99.87 | 98.85 | 98.85 | 98.85 | 75.39 |
| unas | 96.77 | 97.98 | 97.98 | 97.98 | 75.00 |
| yemas | 99.85 | 98.73 | 98.73 | 98.69 | 75.54 |

(Accuracy %. `HAND_PAPER` KNN: palmas 99.37, unas 97.98, yemas 99.81.)

### What do these mean?

- **Only the image-only rows (B) measure what the image carries.** They are weak: palmas with CBAM reaches 87% accuracy, but unas and yemas sit at the majority-class baseline. For unas, CBAM and RCBAM score exactly 66.53%, which is 165/248, i.e. they predict "Normal" for every image (MCC = 0.00).
- **The 97-99.9% rows (A, C, D) are not evidence of a good image model.** EHR-only models already reach the same level, and adding image features changes almost nothing (see the next section for why).
- **RCBAM is not the best variant here.** It is the worst on palmas and roughly equal elsewhere, so the paper's claimed advantage was not reproduced.
- **Decision-tree results (about 75%) come from a depth-1 stump**, not from a tuned model (see "most important limitation", point 3).

---

## The most important limitation

**The headline numbers (97-99.9%) are inflated and should not be reported as performance.** Three problems combine, in order of severity:

1. **Label leakage through the EHR features.** The metadata file contains three columns named `Normal`, `Leve` and `Moderado`, which are one-hot encodings of the target itself. The leak filter only excluded names containing words like `label`, `class`, `sever`, `anem` and `hb`, so these three slipped through. The selected EHR list printed by the notebook includes all three, and their chi-square values (about 790, p around 1e-170) are far above every genuine symptom. Any model given them can read the answer.
2. **Frame-level split with duplicated EHR rows.** Frames from the same video, and therefore identical EHR vectors, land in both train and test. KNN can simply memorise a person, which is why KNN scores highest (99.8-99.9%) and why a stump-based DT is the only EHR model that does not.
3. **GridSearchCV never actually tuned anything.** `GS_SCORING = "f1"` is binary-only, and with three classes every fold's score failed (the notebook shows the `Target is multiclass but average='binary'` error many times). All scores become NaN, so the first parameter combination in each grid is selected. That first combination equals the paper's parameters for SVM, DT, LR and RF, which is why `HAND_TUNED` and `HAND_PAPER` are identical for those four and differ only for KNN (k=3 vs k=19). The 75% for DT is a depth-1 tree.

Two secondary issues:

- The Grad-CAM RGB features carry almost no class signal. Class means differ by under 1 percentage point (e.g. palmas red 40.0 / 39.7 / 39.6), and yemas is about 99.7-99.8% red for every class. So experiment D is effectively experiment A.
- Patient IDs were not parsed for the frame folders (the patient column is blank for palmas and yemas). The EHR feature statistics were therefore computed essentially on the 826 `unas` rows only, and `SPLIT_BY_PATIENT = True` would not work without fixing `PATIENT_ID_FN`.

---

## How to describe the methodology accurately

Suggested wording:

> We adapted the multimodal anemia pipeline of Ramzan et al. (2024) to a three-class severity task (Normal / Leve / Moderada) using palm, nail and fingertip images in place of conjunctiva images. Images were reduced to a region of interest by entropy-based thresholding and classified with MobileNetV2, CBAM-MobileNetV2 and a multi-scale reverse-CBAM variant. Grad-CAM colour proportions and CNN embeddings were fused with EHR features and passed to classical classifiers. Evaluation used a single random 70/30 split of images per modality with one seed. Image-only models were weak (best: 87.4% accuracy on palm images with CBAM; near the majority-class baseline on nails and fingertips). Fusion and EHR-based results in this run are not valid performance estimates because of target leakage in the EHR features and a frame-level split.

Do not say:

- "The model detects anemia severity with 99% accuracy."
- "RCBAM outperforms CBAM" (it did not here).
- "Hyperparameters were tuned by grid search" (the search failed silently).
- "Faithful replication of the paper" (see the fidelity caveat in `results_interpretation.md`).
- Anything implying clinical use. This is a research notebook on a small, single-site dataset.

---

## If asked what should improve next

In priority order:

1. **Remove the leaked columns** (`Normal`, `Leve`, `Moderado`, and check `Split`) from the EHR candidates, and rerun everything.
2. **Split by person, not by frame.** Parse the person ID from the folder name (e.g. `Leve/ID004/frame_000.jpg` gives `ID004`), set `SPLIT_BY_PATIENT = True`, and make sure EHR statistics are computed per person.
3. **Fix the tuning metric.** Use `GS_SCORING = "f1_macro"` so GridSearchCV works for three classes, and confirm the selected parameters differ from the first grid entry.
4. **Stabilise CNN training.** Validation accuracy swings wildly while training accuracy climbs; try a frozen-then-unfrozen backbone, BatchNorm in inference mode, class weights, augmentation and a lower learning rate. Seven of nine runs restored epoch 1 to 3 weights (see `results_interpretation.md`, section 7), so the CNNs are barely trained.
5. **Report per-class metrics and uncertainty:** per-class recall for Leve and Moderada, repeated seeds or cross-validation, and confidence intervals. The `unas` test set has only 22 Moderada images.
6. **Reconsider the Grad-CAM colour features,** or replace them with learned features; the current ones carry no signal.
7. **Check the Hb thresholds** (`HB_NORMAL_THRESHOLD = 11.0`, `HB_MODERADA_THRESHOLD = 8.0`) against the clinical protocol if hemoglobin is ever used to derive labels. They were not used in this run.
8. **Optionally add a hemoglobin regression head** if the aim is a continuous estimate; none exists yet.

---

## Files and paths

**Notebook:** `methodology-2.ipynb` (uploaded as `/mnt/user-data/uploads/methodology-2.ipynb`).

**Input data (Kaggle):**
- `/kaggle/input/datasets/shimu080/dataset2/Dataset2.csv` (999 rows: ID, Class, Anemia Predict, Split, video/image paths)
- `/kaggle/input/datasets/shimu080/metadata/metadata.csv` (909 rows, 22 columns: symptoms, vitals, hemoglobin, one-hot severity columns)
- `/kaggle/input/datasets/shimu080/unas-palmas-yemas/yemas_frames/yemas_frames`
- `/kaggle/input/datasets/shimu080/unas-palmas-yemas/palmas_frames/palmas_frames`
- `/kaggle/input/datasets/shimu080/unas-palmas-yemas/unas_images/unas_images`

**Outputs:** everything is written under `/kaggle/working/anemia_ramzan_3class_ckpt/` (the introductory markdown cell says `anemia_ramzan_ckpt`, but the code uses the `_3class_` name):

| Folder / file | Content |
|---|---|
| `roi/` | 40,970 ROI PNGs (`<site>_<index>.png`) |
| `features/` | `index.csv`, `split.csv`, `<site>_gradcam_rgb.csv`, CNN embeddings `<site>__<variant>_emb.npz` |
| `cnn/` | `<site>__<variant>.weights.h5`, training logs `<site>__<variant>_log.csv`, backup folders |
| `results/` | `<site>__<exp>__<model>.json` (metrics, confusion matrix, per-class scores) and `_probs.npz` (test probabilities) |
| `models/` | fitted classical pipelines (`.joblib`) |
| `plots/` | `<site>__HAND_TUNED.png`, `<site>__HAND_PAPER.png`, `<site>__feature_importance.png` |
| `summary_all_metrics.csv` | one row per experiment x model x site |

Note: only the notebook was provided, not these output files; the figures above were read from the notebook's embedded outputs.
