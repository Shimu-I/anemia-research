# My Methodology Notes — MV-HFMD Adaptation for AnaeCare

## Why I ran this experiment

I adapted the MV-HFMD (Multi-View Classification Using Hybrid Fusion and Mutual Distillation) approach for an AnaeCare three-class classification experiment. I wanted to test whether information from three visual views—fingertip, palm and nail images—could be combined to distinguish Normal, Leve and Moderada.

This is an exploratory methodology adaptation. It is not a completed or clinically validated anemia-detection system, and the results should be interpreted honestly.

## How the inputs are prepared

The notebook matches patient records across YEMAS, PALMAS and UNAS using patient IDs. One representative image/frame is selected from each modality for each patient. The three images are resized/cropped to 224 × 224 and normalized using ImageNet statistics.

The patient-level split recorded in the notebook contains 572 training examples, 123 validation examples and 123 test examples. The test set contains 82 Normal, 30 Leve and 11 Moderada examples.

Because the model receives one representative image from each video modality, it does not learn temporal changes across video frames. Age, sex and other metadata are also not used in this run.

## Main model components

- **Hybrid visual backbone:** The notebook uses the pretrained `vit_small_r26_s32_224` model. It combines CNN-style and Transformer-style processing to represent local visual patterns and broader relationships.
- **Shared encoder:** The same backbone processes each of the three views.
- **View embeddings and token-level fusion:** Learnable view embeddings help distinguish the source modality. The model combines the class token with patch tokens from the three views.
- **Individual-view predictions:** The model also produces a prediction from each view separately.
- **Fused prediction:** A combined representation produces the final three-class output.
- **Mutual distillation:** The loss encourages agreement between the fused output and the average single-view output, while retaining supervised classification losses.

## Training configuration recorded in the notebook

The run uses SGD with learning rate 0.01, momentum 0.9, weight decay 0.0005, One-Cycle learning-rate scheduling, 50 epochs, gradient clipping at 80, distillation temperature 4 and distillation weight 0.1. The checkpoint is selected using validation accuracy, without early stopping. The notebook uses a logical training batch size of 64 and smaller GPU micro-batches to manage memory.

These are the settings in this AnaeCare notebook; they should not be presented as a bit-for-bit reproduction of the paper's original dataset experiment.

## What the test result tells me

The saved test report gives 66.67% accuracy, macro precision 0.22, macro recall 0.33, macro F1 0.27 and macro one-vs-rest ROC-AUC 0.6069. The confusion matrix shows that all 123 test examples were predicted as Normal.

That means the model correctly classified the Normal examples but failed to identify any Leve or Moderada examples. Accuracy alone therefore gives a misleadingly reassuring impression. Macro F1 and class-wise recall are much more informative for this imbalanced three-class task.

## What I would do next

1. Check the saved predictions and confusion matrix before interpreting any new run.
2. Debug why the model predicts only Normal; inspect class counts, label mapping, logits and per-class probabilities.
3. Keep the patient-level split fixed when comparing experiments.
4. Try controlled changes one at a time and report per-class precision/recall/F1, macro F1, macro ROC-AUC and the confusion matrix.
5. Treat any improvement as experimental evidence only, not clinical validation.

## Important distinction

These notes describe my MV-HFMD adaptation for AnaeCare. They do not claim to reproduce the original paper's dataset, results or exact experimental setup.
