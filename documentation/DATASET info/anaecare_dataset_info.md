# AnaeCare Dataset — Reference Info

Non-invasive multimodal dataset for iron deficiency anemia (IDA) detection,
used in the "AnaeCare: An Explainable Multimodal Deep Learning Framework for
Non-Invasive Anemia Detection" project.

## Source
- Public clinical dataset for non-invasive iron deficiency anemia detection.
- Collected at **Universidad Nacional de San Martín, Peru**.

## Participants
- **909 young adults**, ages 18–25.
- **540 Female | 369 Male**.

## Data Acquisition
- Device: **Samsung Galaxy A73 5G** (camera).
- Hemoglobin reference measurement: **Rad-67 Hb Measurement Device**.
- Standardized collection protocol.

## Modalities (3 total)

| Modality | Spanish folder name | Type | Count | Capture details |
|---|---|---|---|---|
| Fingertip | `yemas` | Video | 838 videos | 30 seconds, 60 FPS |
| Palm | `palmas` | Video | 833 videos | Controlled hand opening motion |
| Nail | `unas` | Photo | 826 images | Light-box image, standardized lighting |

Rationale: fingertip, palm, and nail-bed areas visibly pale when hemoglobin
drops, making them usable as non-invasive visual proxies for anemia.

## Class Distribution (per modality, from metadata)

| Modality | Class | Samples |
|---|---|---|
| palmas | Leve (Mild) | 203 |
| palmas | Moderada (Moderate) | 74 |
| palmas | Normal | 556 |
| unas | Leve (Mild) | 202 |
| unas | Moderada (Moderate) | 73 |
| unas | Normal | 551 |
| yemas | Leve (Mild) | 203 |
| yemas | Moderada (Moderate) | 75 |
| yemas | Normal | 560 |

Classes: **Normal**, **Leve** (Mild), **Moderada** (Moderate) — based on
clinical hemoglobin-based anemia severity.

For binary classification tasks, labels are commonly mapped as:
`Normal → 0`, `Leve + Moderada → 1` (~2.06:1 class imbalance ratio,
i.e. roughly 2 "Normal" for every 1 "anemic" case).

## Dataset Folder Structure

```
dataset/
├── palmas/
│   ├── Leve/
│   │   ├── ID004.mp4
│   │   ├── ID015.mp4
│   │   └── ...
│   ├── Moderada/
│   └── Normal/
├── unas/
│   ├── Leve/
│   │   ├── ID004.jpg
│   │   └── ...
│   ├── Moderada/
│   └── Normal/
├── yemas/
│   ├── Leve/
│   │   ├── ID004.mp4
│   │   └── ...
│   ├── Moderada/
│   └── Normal/
└── metadata.csv
```

- Videos/images are organized by modality → class folder → per-subject file
  (mp4 for palmas/yemas, jpg for unas), named by subject ID (e.g. `ID004`).
- `metadata.csv` holds participant info, clinical labels, and file paths.

## Metadata / CSV Fields

Each participant record in `metadata.csv` contains:

**Demographics & vitals:**
- Anonymous Participant ID (e.g. `ID001`)
- Sex (`M` / `F`)
- Age
- Hemoglobin (Hb) level (g/dL)
- Heart Rate (Frecuencia Cardiaca)
- Oxygen Saturation (SpO₂ / Oxígeno)
- Weight (Peso)
- Height (Talla)

**Clinical symptoms (binary flags, Spanish column names in raw CSV):**
- Fatigue (Fatiga)
- Weakness (Debilidad)
- Irregular heartbeat (Latidos irregulares)
- Shortness of breath (Dificultad para respirar)
- Dizziness (Mareos o aturdimiento)
- Chest pain (Dolor en el pecho)
- Cold hands and feet (Manos y pies fríos)
- Headache (Dolores de cabeza)

**Labels:**
- Clinical Anemia Category: `Normal` / `Leve` (Mild) / `Moderada` (Moderate)
  — also present as one-hot-style columns (`Normal`, `Leve`, `Moderado`)
- `Anemia Predict` — binary anemia label derived from the category

**File paths (in the processed/split CSV, e.g. `Dataset2.csv`):**
- YEMAS Path — e.g. `dataset/yemas/Normal/20230706_211043.mp4`
- PALMAS Path — e.g. `dataset/palmas/Normal/20230706_211218.mp4`
- UNAS Path — e.g. `dataset/unas/Normal/20230706_211246.jpg`
- `Split` column — `train` / `validation` / `test`

Note: some rows may have missing modality files (`N/A`) — not every subject
has all 3 modalities present on disk (see "existence check" step below).

## Train / Val / Test Split

- **Patient-level stratified split**, ratio **70 / 15 / 15**.
- Example counts (one run): **Train: 584, Validation: 126, Test: 111**.
- Stratified by the binary anemia label (`Anemia Predict`) to preserve class
  balance across splits.
- Train/val/test sets are guaranteed **patient-ID disjoint** (no subject
  appears in more than one split).
- If a provided `Split` column exists in the metadata, it's used directly;
  otherwise a fresh stratified split is generated (70% train, then remaining
  30% split 15/15 into val/test).

## Preprocessing Pipeline

1. **Existence check**: verify each subject actually has files for all 3
   modalities on disk (frame folder for yemas/palmas, single image for unas);
   subjects missing any modality are dropped from the usable set.
2. **Frame sampling** (video modalities only, no OpenCV video decoding —
   frames are pre-extracted and just sampled):
   - Yemas (fingertip): commonly **32 frames** sampled evenly per video
     (also tested at 8/16 frames in later experiments).
   - Palmas (palm): commonly **16 frames** sampled evenly per video
     (also tested at 8/32 frames in later experiments).
   - Unas (nail): used directly as a single image, no frame extraction.
3. **Resize**: all frames/images resized to **224×224 px**.
4. **Normalization**: ImageNet mean/std normalization
   (`mean=[0.485, 0.456, 0.406]`, `std=[0.229, 0.224, 0.225]`).

## Baseline Single-Modality Results (ResNet-18, FYDP-1 phase)

Fine-tuned ResNet-18 (pretrained backbone + 1-neuron binary head), BCE loss
with class-imbalance weighting, AdamW optimizer, cosine LR schedule, 10
epochs, evaluated on the 15% test split.

| Modality | Accuracy | AUC-ROC | F1-score | Specificity | Sensitivity |
|---|---|---|---|---|---|
| Unas (nail) | 0.5636 | 0.6203 | 0.6308 | 0.6774 | 0.519 |
| Palmas (palm) | 0.663 | 0.673 | 0.49 | 0.71 | 0.519 |
| Yemas (fingertip) | 0.6637 | 0.6557 | 0.5185 | 0.7154 | 0.5564 |

## Improved Single-Modality Results (FYDP-2 phase — best configs per modality)

| Modality | Config | Accuracy | AUC-ROC | F1-Score | Sensitivity |
|---|---|---|---|---|---|
| Unas | Best (Combined, class-imbalance handling + augmentation) | 0.6727 | 0.7123 | 0.5135 | 0.6129 |
| Palmas | Best (8-frame + imbalance handling) | 0.672 | 0.689 | 0.746 | 0.67 |
| Yemas | Best (32-frame + augmentation) | 0.7387 | 0.7427 | 0.5672 | 0.6129 |

Techniques explored on top of the baseline: class-imbalance handling
(e.g. weighted loss / resampling) and data augmentation, tested across
different frame-sampling counts (8 / 16 / 32 frames).

## Multimodal Fusion (current/planned phase)

Goal: fuse all 3 modalities (+ optionally clinical metadata) rather than
using them independently. Fusion strategies under consideration/testing:

- **Input/Early fusion** — merge raw modalities before the backbone (e.g.
  channel-stacking).
- **Single-level (intermediate) fusion** — extract per-modality features
  first (e.g. via per-modality ResNet-18 encoders), then fuse once before
  classification. Two variants implemented so far:
  - `ConcatFusion` — simple concatenation of the 3 modality embeddings.
  - `SelfAttentionFusion` — the 3 modality embeddings attend to each other
    via a Transformer encoder layer, then mean-pooled.
- **Hierarchical fusion** — fusion at multiple feature depths (low/mid/high).
- **Attention-based fusion** — learned per-modality/feature weighting.
- **Output/Late fusion** — combine independent per-modality model
  predictions (decision-level).

In the modeling pipeline used so far: each of the 2 video modalities
(yemas, palmas) is encoded per-frame by a shared-architecture ResNet-18
`FrameEncoder` → temporal pooling (mean-pooling, or optionally an LSTM) →
a per-modality embedding vector; the nail image is encoded directly by its
own ResNet-18 encoder; the 3 resulting embeddings are then fused
(concat or self-attention) and passed to a small classifier head.

## Explainability

**Grad-CAM** is used post-hoc on trained models to visualize which regions
of the nail and fingertip images most influenced the anemia prediction
(e.g. highlighting nail-bed pallor or perfusion patterns).

## Key Numbers Summary (quick reference)

- Total participants: **909** (540 F / 369 M), ages 18–25
- Fingertip videos: **838**
- Palm videos: **833**
- Nail photos: **826**
- Classes: Normal / Leve (Mild) / Moderada (Moderate)
- Class imbalance (binary): Normal : Anemic ≈ **2.06 : 1**
- Split: **70% train / 15% val / 15% test**, patient-level, stratified
- Frame sampling: yemas up to 32 frames, palmas 8–32 frames (evenly sampled)
- Image size: **224×224**, ImageNet-normalized
