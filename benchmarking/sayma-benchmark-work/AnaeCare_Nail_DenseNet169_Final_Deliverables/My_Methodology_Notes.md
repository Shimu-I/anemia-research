# My Methodology Notes — AnaeCare Nail-Based Anemia Classification

## Why I ran this experiment

For this AnaeCare experiment, I used DenseNet169 to explore whether nail photographs could help distinguish three categories: Normal, Leve and Moderada. I focused on the UNAS nail-image modality only, so this experiment does not combine nail images with palm or fingertip videos, and it does not use patient metadata as model input.

This is an exploratory research prototype for non-invasive pre-screening. It is not a clinically validated diagnostic system.

## Dataset and labels

The notebook indexes the nail images and maps the original categories as follows:

| Class | Label |
|---|---:|
| Normal | 0 |
| Leve | 1 |
| Moderada | 2 |

The notebook found 826 images associated with 823 unique patient IDs. It then created a patient-level, stratified 70/15/15 train/validation/test split. The saved split counts are:

| Split | Normal | Leve | Moderada | Total patients |
|---|---:|---:|---:|---:|
| Train | 385 | 140 | 51 | 576 |
| Validation | 82 | 30 | 11 | 123 |
| Test | 83 | 30 | 11 | 124 |

The split manifest is important because it records which images and patient IDs belong to each split. Keeping each patient in only one split helps reduce the risk of patient-level leakage.

## Image preparation

Images are resized and center-cropped to 224 × 224 pixels, then normalized with ImageNet mean and standard deviation. Random augmentation is applied to training images only. These are practical preprocessing choices in this adaptation; they should not be described as an exact reproduction of the reference paper's segmentation procedure.

## Model and training

I used ImageNet-pretrained DenseNet169 as the visual backbone and replaced its final classification layer with an output layer for the three AnaeCare classes. DenseNet's dense connections allow later layers to reuse features from earlier layers.

The notebook uses a class-weighted cross-entropy loss. The weights are calculated from the training-image counts, rather than from the validation or test set. In this run, the weights for Normal, Leve and Moderada were approximately 0.500, 1.364 and 3.771 respectively.

The optimizer is Adam with a learning rate of 0.0001 and weight decay of 0.0001. The notebook is configured for up to 20 epochs and uses early stopping after five consecutive epochs without an improvement in validation loss. The best checkpoint is selected by validation loss, not by test performance. Training stopped after epoch 15, and the selected checkpoint came from epoch 10.

## Evaluation approach

The notebook reports three-class accuracy, balanced accuracy, macro precision, macro recall, macro F1, macro one-vs-rest ROC-AUC and a 3 × 3 confusion matrix. It also implements patient-level probability averaging when multiple images belong to one patient.

In this particular test split, the number of test images and test patients is the same (124), so the reported image-level and patient-level metrics are numerically identical. I keep the two evaluation perspectives conceptually separate because they would differ if patients contributed multiple test images.

## What the results show

The saved patient-level test results are:

- Accuracy: 0.5000
- Balanced accuracy: 0.4606
- Macro precision: 0.4282
- Macro recall: 0.4606
- Macro F1-score: 0.4168
- Macro one-vs-rest ROC-AUC: 0.6629

The model does predict examples from all three classes, but performance is uneven. Normal recall is about 0.52, Leve recall is 0.50, and Moderada recall is about 0.36. Only 4 of the 11 Moderada test cases were correctly classified. This is not enough to claim reliable anemia classification.

## What I would do next

1. Inspect errors and class-wise probabilities, especially for Moderada.
2. Keep the patient-level split fixed when comparing model changes.
3. Compare controlled alternatives one at a time, including class weighting and augmentation settings.
4. Report macro F1, balanced accuracy, class-wise recall and the confusion matrix alongside accuracy.
5. Preserve the checkpoint, split manifest, training history and test predictions so the results can be reproduced.

## Scope and limitations

This is a paper-informed DenseNet169 adaptation for UNAS nail images, not an exact reproduction of the reference paper's dataset or protocol. The reported metrics come from this notebook's saved execution outputs. They are research results only and do not establish clinical validity or replace a hemoglobin test.
