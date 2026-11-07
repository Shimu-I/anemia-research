# Personal Notes — Understanding the AnaeCare BPANet Adaptation

These notes are for explaining the experiment to a teammate or supervisor in simple language.

## What is the model trying to do?

The model uses three visual sources from the same patient:
- YEMAS: fingertip video frames
- PALMAS: palm video frames
- UNAS: fingernail image

It also uses Age and Sex as metadata. The model has two kinds of outputs:
1. A project-specific class prediction: Normal, Leve, or Moderada.
2. Hemoglobin estimation tasks: a continuous Hb value and a 48-bin Hb category, inspired by BPANet.

This is intended for research into non-invasive pre-screening, not medical diagnosis.

## How the components work

- **ResNet-50:** extracts visual features from each frame/image. ImageNet-pretrained weights provide an initial visual representation.
- **CSA (Channel-Spatial Attention):** helps the network emphasize useful feature channels and spatial regions. In this notebook it is placed after ResNet Stage 1 and before Stage 2.
- **Bi-LSTM for videos:** processes the sequence of frame features for YEMAS and PALMAS to form a video-level representation.
- **Metadata projection:** maps Age and Sex into the same feature dimension as visual tokens.
- **Modality tags:** learnable tags indicate whether a token came from YEMAS, PALMAS, or UNAS.
- **Fusion attention:** refines information across modality tokens before combining them.
- **Hb regression head:** predicts a continuous hemoglobin value. Its error is measured with MAE in g/dL.
- **48-bin Hb head:** assigns Hb to one of 48 intervals over the configured range; this is part of the BPANet-inspired dual objective.
- **Three-class head:** predicts the AnaeCare target labels Normal, Leve, and Moderada, with class-weighted cross-entropy.

## How the training was evaluated

The notebook reports 806 usable paired patients:
- Training: 560
- Validation: 123
- Test: 123

The model trained for 10 epochs. Validation macro-F1 was used to choose the best checkpoint. The held-out test set was then evaluated once using that selected checkpoint.

## Results to remember

- Accuracy: **69.11%**
- Macro precision: **40.23%**
- Macro recall: **43.30%**
- Macro F1: **41.63%**
- Macro ROC-AUC (one-vs-rest): **76.21%**
- Hb MAE: **1.1565 g/dL**

### What do these mean?

- **Accuracy** is the proportion of all test cases predicted correctly.
- **Precision** asks: when the model predicts a class, how often is that prediction correct?
- **Recall/sensitivity** asks: among real examples of a class, how many did the model find?
- **F1-score** balances precision and recall.
- **Macro metrics** average class-level results equally, so a small class matters as much as the Normal class.
- **ROC-AUC** evaluates ranking of scores across decision thresholds; it does not mean the final class predictions are good.
- **Hb MAE** is the average absolute difference between predicted and recorded Hb values.

## The most important limitation

The test set had 87 Normal, 29 Leve, and only 7 Moderada patients. The confusion matrix shows:
- Normal correctly predicted: 71/87
- Leve correctly predicted: 14/29
- Moderada correctly predicted: 0/7

The model did not predict any case as Moderada. Therefore, do not present 69.11% accuracy alone as evidence that the model performs well. Macro-F1 is 0.4163, and the Moderada class is a clear weakness.

## How to describe the methodology accurately

Use wording such as:

> “We adapted selected BPANet-inspired components to AnaeCare's paired multimodal dataset, including a ResNet-50 visual backbone, channel-spatial attention, age/sex metadata, modality tags, attention-based fusion, and Hb regression plus 48-bin Hb classification objectives. We added a three-class classification head to address AnaeCare's Normal/Leve/Moderada labels and used Bi-LSTM pooling for the video modalities.”

Avoid saying the original BPANet was reproduced exactly. The task, fusion setup, temporal processing, and added three-class head differ from the original paper's setup.

## If asked what should improve next

1. Investigate why Moderada is never predicted.
2. Report class-wise metrics and the confusion matrix alongside accuracy.
3. Compare with the previous best AnaeCare model on the same held-out split.
4. Check whether class weighting, sampling, or calibration improves minority-class recall without worsening other classes.
5. Keep patient-level splitting to avoid the same patient appearing across train/validation/test.
6. Continue to frame the system as a research pre-screening tool, not a diagnosis.

## Files and paths

The Kaggle run saved outputs under `/kaggle/working/AnaeCare_BPAnet_results` and created `/kaggle/working/AnaeCare_BPANet_results.zip`. The notebook output confirms those paths, but this chat only has the notebook file—not the original Kaggle ZIP or binary checkpoint files. The results package prepared here reconstructs metrics/plots from the notebook's saved outputs; it does not include the trained `.pt` checkpoint.
