# Dataset Preprocessing Report — PALMAS, YEMAS, UNAS

This document describes, step by step, how each of the three modalities in the dataset was preprocessed. It is based on three Google Colab notebooks:

| Modality | Source notebook | Raw input | Processed output |
|---|---|---|---|
| **PALMAS** (palm videos) | `palmas_frame_extraction_ALL_DONE_.ipynb` | `.mp4/.mov/.avi` video per subject | 16 JPEG frames per subject, 224×224 |
| **YEMAS** (fingertip videos) | `yemas_frame_extraction_ALL_DONE_.ipynb` | `.mp4/.mov/.avi` video per subject | 32 JPEG frames per subject, 224×224 |
| **UNAS** (fingernail photos) | `UNAS_Preprocessing_ALL.ipynb` | One photo (`.jpg/.jpeg/.png`) per subject | 1 JPEG image per subject, 224×224, nail-centered |

All three modalities are split into the same three severity classes: **Normal**, **Leve** (mild) and **Moderada** (moderate). Each file is named by subject ID (e.g. `ID009`), and the ID is preserved through preprocessing.

---

## 1. Overview of the pipeline

```
Raw data (Google Drive)                      Processed data (Google Drive)
──────────────────────                      ─────────────────────────────
Dataset/palmas/{Class}/IDxxx.mp4   ──►  dataset_palmas_frames/palmas_frames/{Class}/IDxxx/frame_000..015.jpg
Dataset/yemas/{Class}/IDxxx.mp4    ──►  dataset_yemas_frames/yemas_frames/{Class}/IDxxx/frame_000..031.jpg
dataset/Dataset/unas/{Class}/IDxxx.jpg ──► Dataset_processed/unas_processed/{Class}/IDxxx.jpg
```

Common design principles across all three notebooks:

1. **Same code path for every class.** Normal, Leve and Moderada are processed by identical code, with no per-class special cases. This was introduced after an earlier inconsistency, where one class ended up at raw capture resolution instead of the target size.
2. **Everything ends at 224×224 pixels**, the standard input size for common CNN/ViT image backbones.
3. **Safe, local-first Google Drive I/O** (see Section 5). Each file is copied to local Colab disk, processed there, copied back, and verified.
4. **Logging and self-checks.** Corrupted files, copy mismatches and resolution mismatches are written to log files rather than found by hand later.

Software used: Python, OpenCV (`cv2`), NumPy, tqdm, Pillow (for verification), and for UNAS also pandas and matplotlib. Everything ran on Google Colab with Google Drive mounted.

> **What is *not* done:** none of the notebooks apply pixel normalization (mean/std scaling), data augmentation, color-space conversion for the saved images, denoising, or train/validation/test splitting. The output is plain resized RGB/BGR JPEG images. Any normalization and augmentation must be applied later, in the training pipeline.

---

## 2. PALMAS (palm videos)

### 2.1 Input
- Folder structure: `Dataset/palmas/{Leve, Moderada, Normal}/*.mp4` (the code also accepts `.mov` and `.avi`, in upper or lower case).
- One video per subject. The subject ID is the file name without its extension.
- Source counts (from the progress bars): **Leve 203, Moderada 74, Normal 556** videos, 833 in total.

### 2.2 Configuration

| Parameter | Value | Meaning |
|---|---|---|
| `PALMAS_DISCARD_START_PCT` | **0.25** | Drop the first 25% of frames of each video |
| `PALMAS_DISCARD_END_PCT` | **0.10** | Drop the last 10% of frames of each video |
| `PALMAS_NUM_FRAMES` | **16** | Number of frames sampled per video |
| `IMG_SIZE` | **(224, 224)** | Final size of every saved frame |

### 2.3 Step-by-step processing of each video

1. **Open the video** with `cv2.VideoCapture`. If it cannot be opened, the video is logged as `could_not_open_video` and skipped.
2. **Read the total frame count** (`CAP_PROP_FRAME_COUNT`). If it is zero or unreadable, the video is logged as `zero_or_unreadable_frame_count` and skipped.
3. **Compute the usable window** (`get_frame_indices`):
   - `start = int(total_frames × 0.25)`
   - `end = int(total_frames × (1 − 0.10))`
   - The discards are **percentages of that video's own length**, not fixed frame counts, so videos of different lengths are trimmed in proportion.
   - The 25% at the start removes the setup phase (hand being positioned, camera stabilizing). The 10% at the end removes the hand being withdrawn.
   - **Degenerate case:** if `end <= start` (a very short clip), the window falls back to the whole video (`0 … total_frames`).
4. **Sample 16 evenly spaced frames** with `np.linspace(start, end − 1, 16)`, rounded to integers and clipped to the valid range `[0, total_frames − 1]`. This gives uniform temporal coverage of the retained segment.
   - If a video has fewer than 16 frames in total, all of its frames are returned instead. This is the only case where a subject could have fewer than 16 frames.
5. **Seek and read each selected frame** with `cap.set(CAP_PROP_POS_FRAMES, idx)` followed by `cap.read()`. Frames that fail to read are skipped.
6. **Resize to 224×224** with `cv2.resize(..., interpolation=cv2.INTER_AREA)`.
   - `INTER_AREA` is the recommended interpolation for downscaling because it limits aliasing.
   - The resize is **unconditional**: it is applied to every frame of every class. The aspect ratio is **not** preserved (no padding or cropping), so the full frame is squeezed into a square.
7. **Save as JPEG** with `cv2.imwrite` as `frame_000.jpg … frame_015.jpg` in `palmas_frames/{Class}/{subject_id}/`. OpenCV's default JPEG quality (95) is used.
8. If no frame at all could be read, the video is logged as `no_frames_could_be_read`.

### 2.4 Run history and resume logic

- The first full run processed Leve (203/203, ~32 min) and Moderada (74/74, ~12 min). Normal was interrupted at 105/556 (~19%) by a Colab or Drive problem.
- A **resume cell** then re-ran **Normal only**. Before processing, `is_already_done()` checks whether the subject's output folder already has exactly 16 `frame_*.jpg` files and whether the first frame is exactly 224×224. If so, the subject is skipped. This makes re-runs idempotent and also re-processes any subject that was previously written at the wrong size.
- The resume cell also wraps each subject in `try / except OSError / finally`. A transient Drive error (e.g. `Errno 107 Transport endpoint is not connected`) is logged as `io_error:…` and the run continues. The `finally` block always deletes the temporary local files. Any subject that failed can be picked up by re-running the cell.
- The Normal re-run finished 556/556 in about 1 h 24 min.

### 2.5 Results and validation

- **Corrupted/mismatched log:** none (`No corrupted or mismatched videos.`).
- **Resolution self-check:** `[PASS] Every subject in ['Leve', 'Moderada', 'Normal'] has exactly 16 frames at 224x224.`
- **Expected output size:** 833 subjects × 16 frames = 13,328 JPEG frames.
- **Note on the summary CSV:** `palmas_frame_summary.csv` has 554 rows. The `summary_rows` list was reset to empty before the Normal re-run, so the CSV only describes subjects handled in that last run. It is not a complete manifest of all 833 subjects. The authoritative record of completeness is the folder contents plus the self-check above.

---

## 3. YEMAS (fingertip videos)

### 3.1 Input
- Folder structure: `Dataset/yemas/{Leve, Moderada, Normal}/*.mp4` (same accepted extensions as PALMAS).
- One video per subject. Per the notebook, the protocol has the **finger covering the whole camera lens**, so the frame content is the fingertip filling the view and no spatial cropping or localization is needed.
- Source counts: **Leve 203, Moderada 75, Normal 560** videos, 838 in total.

### 3.2 Configuration

| Parameter | Value | Meaning |
|---|---|---|
| `YEMAS_DISCARD_START_PCT` | **0.10** | Drop the first 10% of frames |
| `YEMAS_DISCARD_END_PCT` | **0.10** | Drop the last 10% of frames |
| `YEMAS_NUM_FRAMES` | **32** | Number of frames sampled per video |
| `IMG_SIZE` | **(224, 224)** | Final size of every saved frame |

### 3.3 Step-by-step processing of each video

The code is the same as for PALMAS (`get_frame_indices` and `process_video`); only the parameters differ.

1. Open the video (`could_not_open_video` if it fails).
2. Read the total frame count (`zero_or_unreadable_frame_count` if ≤ 0).
3. Compute the window: `start = int(total × 0.10)`, `end = int(total × 0.90)`. Fall back to the whole clip if `end <= start`.
4. Sample **32** evenly spaced indices with `np.linspace(start, end − 1, 32)`, rounded and clipped. If the video has fewer than 32 frames, all frames are used.
5. Seek to each index, read the frame, skip it on failure.
6. Resize to **224×224** with `INTER_AREA`. This is unconditional and the aspect ratio is not preserved.
7. Save as `frame_000.jpg … frame_031.jpg` in `yemas_frames/{Class}/{subject_id}/`.
8. If nothing could be read, log `no_frames_could_be_read`.

### 3.4 Why PALMAS and YEMAS differ

| | PALMAS | YEMAS |
|---|---|---|
| Frames discarded at start | 25% | 10% |
| Frames discarded at end | 10% | 10% |
| Frames kept per video | 16 | 32 |

The notebooks state these as modality-specific settings. YEMAS uses the milder 10%/10% trim (the same method already used for its Normal and Moderada classes) and twice as many frames. They do not record the reason for the PALMAS-specific 25% start trim.

### 3.5 Run history and resume logic

- First run: Leve 203/203 (~26 min), Moderada 75/75 (~10 min). Normal crashed after 5/560 videos with `OSError: [Errno 107] Transport endpoint is not connected` while copying `ID009.mp4` from Drive, meaning Drive's FUSE mount had disconnected.
- A **resume cell** adds `is_already_done()` (32 frames and a 224×224 first frame) and the `try / except OSError / finally` wrapper. Re-running Leve and Moderada took only seconds, because all of their subjects were already complete and skipped. Normal then completed 560/560 in ~1 h 18 min.

### 3.6 Results and validation

- **Corrupted/mismatched log:** none.
- **Summary CSV:** `yemas_frame_summary.csv` with 838 rows, one per subject, which is complete here.
- **Resolution self-check:** `[PASS] Every subject in ['Leve', 'Moderada', 'Normal'] has exactly 32 frames at 224x224.`
- **Expected output size:** 838 subjects × 32 frames = 26,816 JPEG frames.

---

## 4. UNAS (fingernail photos)

UNAS is a still-image modality, so there is no frame extraction. Preprocessing is a **nail-centered crop** followed by a resize.

### 4.1 Input
- Folder structure: `dataset/Dataset/unas/{Normal, Leve, Moderada}/*.jpg|jpeg|png`.
- Source counts: **Normal 551, Leve 202, Moderada 73** images, 826 in total.
- Capture style: fingertips pressed against a surface over a **dark, fairly uniform background**, with **no palm in frame**. The target finger is extended furthest "up" in the frame.

### 4.2 Why a nail-centered crop was needed

Two problems were found while reviewing the earlier UNAS output:

1. **The old crop was centered on the frame, not on the nail.** A fixed-fraction center crop of the whole photo produced crops that were mostly background, with the nail in a corner (confirmed on `ID104`). Unlike YEMAS, where the finger fills the lens by protocol, the hand is not guaranteed to be in the middle of a nail photo.
2. **MediaPipe hand landmarks were tried and rejected.** The detector reported "no hand detected" on every sample, even those with a clearly visible hand. This was a real model mismatch: it expects a full hand with the palm visible, and these photos only show fingertips.

The adopted solution is **skin-color segmentation**, which needs no external model and suits this capture style.

### 4.3 Configuration

| Parameter | Value | Meaning |
|---|---|---|
| `CROP_SIZE` | **900 px** | Side of the square crop window around the nail |
| `OUTPUT_SIZE` | **224 px** | Final square size |
| `VERTICAL_OFFSET_FRACTION` | **0.12** | Shifts the crop window *up* by 0.12 × 900 = **108 px** |
| `FALLBACK_ROI_FRACTION` | **0.75** | Center-crop fraction used only if no skin is found |
| `min_area_fraction` | **0.01** | Smallest skin blob accepted (1% of image area) |
| YCrCb lower bound | `[0, 133, 77]` | Skin-color range, lower |
| YCrCb upper bound | `[255, 173, 127]` | Skin-color range, upper |

### 4.4 Step-by-step processing of each image

**Step 1: Read the image** with `cv2.imread`. If it returns `None`, the image is logged as `cannot_open_image`.

**Step 2: Locate the nail** (`find_nail_center_skin`):
1. Convert BGR → **YCrCb**. This color space separates luminance (Y) from chrominance (Cr, Cb), so skin tones fall in a compact range that is fairly robust to brightness.
2. Apply `cv2.inRange` with the bounds above to get a binary skin mask.
3. Clean the mask with morphology:
   - **Opening** with a 5×5 kernel removes small speckle.
   - **Closing** with a 15×15 kernel fills small holes inside the finger.
4. Find external contours (`cv2.findContours`, `RETR_EXTERNAL`) and keep the **largest** one.
5. If the largest contour's area is below **1% of the image area**, treat it as "no hand found".
6. Take the **topmost point** (smallest y) of that contour as the nail location. This matches the dataset's composition, where the target finger extends highest in the frame. It was checked on a real sample (`ID245`), where it landed almost exactly on the fingertip.

**Step 3: Crop around the nail** (`crop_centered_on_nail`):
1. Take a 900×900 window centered horizontally on the detected x.
2. **Shift the window up by 108 px.** The detected point is where visible *skin* starts, but the pale or white nail tip is often outside the YCrCb skin range and sits above it. Without the shift the tip was clipped by the top edge of the crop (confirmed on `ID005`, `ID010`, `ID014` and `ID044` in the Moderada class). The shift gives it headroom and places the nail lower in the frame.
3. Clamp the window to the image boundaries with `max(0, …)` and `min(w/h, …)`. Near an image edge the crop can therefore be smaller than 900×900, and possibly non-square, before the final resize.
4. If the crop comes out empty, fall back to the center crop.

**Step 4: Fallback (rare, flagged).** If no sufficiently large skin region exists, the code uses the *old* center crop (the central 75% of the frame). The image is still saved, but it is **logged to `unas_fallback_log.txt`** for manual review.

**Step 5: Resize** the crop to **224×224** with `cv2.resize(..., INTER_AREA)`.

**Step 6: Save** as `unas_processed/{Class}/{subject_id}.jpg` with `cv2.imwrite`. A failed write is logged as `failed_to_write_output`.

### 4.5 Visual spot-check before the full run

`QUICK_TEST_MODE = True` runs 5 images per class entirely in memory and plots **original (top) vs. nail-centered crop (bottom)**. Each panel is titled "detected" (green) or "FALLBACK (no hand found)" (red). The intended use is to confirm by eye that the nail lands in the middle, and to tune `CROP_SIZE`, the YCrCb bounds or the vertical offset if not, before committing to the full run. The skin-color thresholds are sensitive to lighting and skin tone, so they are the first thing to tune if detection is off for a subset of images.

### 4.6 Results and validation

- **Error log** (`error_log.txt`): 0 entries, "No corrupted UNAS files detected."
- **Fallback log** (`unas_fallback_log.txt`): **0 entries**. A skin region was detected in every image, so no image used the old center crop.
- **Verification counts:**

| Class | Source | Processed | Missing |
|---|---|---|---|
| Normal | 551 | 551 | 0 |
| Leve | 202 | 202 | 0 |
| Moderada | 73 | 73 | 0 |

- A `unas_summary_rows.csv` manifest (modality, class, subject_id, 1 image) was also saved.

---

## 5. Safe Drive I/O pattern (all three modalities)

Writing many small files straight to a Drive-mounted folder can fail silently or lag behind what Python reports. During the earlier YEMAS/PALMAS runs, the script reported 203/203 successes while Drive's UI showed only 175. The fix is the same in all three notebooks, applied **one subject at a time**:

1. **Copy** one source file from Drive to local Colab disk (`/content/palmas_tmp`, `/content/yemas_tmp`, `/content/unas_tmp`).
2. **Process** it locally and write the output to a local folder, never straight to Drive.
3. **Copy** the finished output back to Drive (`shutil.copytree` for frame folders, `shutil.copy2` for a single image).
4. **Verify** the copy:
   - PALMAS/YEMAS: the number of files in the local output folder must equal the number on Drive (`drive_copy_mismatch_{local}_vs_{drive}` otherwise).
   - UNAS: the byte size of the Drive file must equal the local file's size.
5. **Clean up** the local video/image and its output, so Colab's disk never fills up over hundreds of files.

This means there is never a large batch of unverified writes waiting in Drive's sync queue.

---

## 6. Quality checks

| Check | PALMAS | YEMAS | UNAS |
|---|---|---|---|
| Unreadable / corrupted input logged | ✅ `palmas_corrupted_log.csv` | ✅ `yemas_corrupted_log.csv` | ✅ `error_log.txt` |
| Drive copy verified | ✅ file count | ✅ file count | ✅ byte size |
| Resume support for interrupted runs | ✅ `is_already_done` | ✅ `is_already_done` | ❌ not needed (single run) |
| Post-run self-check (count + size) | ✅ PASS (16 @ 224×224) | ✅ PASS (32 @ 224×224) | counts per class: all matched |
| Manual-review list | n/a | n/a | ✅ `unas_fallback_log.txt` (empty) |
| Visual spot-check | n/a | n/a | ✅ original vs. crop plots |

The PALMAS/YEMAS self-check (Section 7 of each notebook) walks every subject folder. It confirms exactly the expected number of `frame_*.jpg` files and that the **first** frame is 224×224. It checks only the first frame per subject, which is representative because all frames of a subject come from the same `process_video()` call. Any failure is written to `*_resolution_mismatch_log.csv`.

---

## 7. Final output summary

| Modality | Subjects | Per-subject output | Total files | Size |
|---|---|---|---|---|
| PALMAS | 833 (203 Leve / 74 Moderada / 556 Normal) | 16 frames | 13,328 | 224×224 |
| YEMAS | 838 (203 Leve / 75 Moderada / 560 Normal) | 32 frames | 26,816 | 224×224 |
| UNAS | 826 (202 Leve / 73 Moderada / 551 Normal) | 1 image | 826 | 224×224 |

Total counts are computed from the per-class counts shown in the notebook outputs.

### Points to keep in mind when using the data
- **Class imbalance:** every modality is dominated by Normal (roughly 65–67% of subjects), with Moderada the smallest class (about 9%). Use class weights, resampling or stratified evaluation.
- **Split by subject:** frames from the same subject are highly correlated, so train/validation/test splits must be made at the **subject (ID) level**, never at the frame level, to avoid leakage.
- **Aspect ratio:** PALMAS/YEMAS frames and UNAS crops are resized to a square without padding, so non-square sources are geometrically stretched.
- **Different subject counts per modality:** the three modalities have slightly different subject counts (833 / 838 / 826), and the notebooks do not show a cross-modality ID-matching step. Check ID alignment before any multimodal fusion.
- **Normalization and augmentation** are not part of this preprocessing and must be handled in the model pipeline.
