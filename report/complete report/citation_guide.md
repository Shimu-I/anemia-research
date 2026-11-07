# Citation guide: where each `\cite{p-N}` is used and the BibTeX entry to add

Every citation in the report is written as `\cite{p-N}`, where `p-N` is the serial number in `thesis_cited_papers.xlsx`. For each paper, add one entry to `fydp.bib` whose **key is exactly `p-N`**. Fill in the empty fields from the paper itself.

## Steps

1. Open `fydp.bib`. Delete the 18 unrelated sample entries (for example `kleinberg2006algorithm`) and rename the existing dataset entry `Valles-Coral_Injante_...` to **`p-17`**. It is already filled in.
2. Add the 51 other entries from the list below, with the key `p-N` as given.
3. In `fydp.tex`, delete the temporary `\chapter*{References} ... [Reference list: ...]` block, then remove the `%` in front of `\bibliographystyle{unsrt}` and `\bibliography{fydp}`.
4. Compile with LaTeX, then BibTeX, then LaTeX twice (Overleaf does this automatically). The `[?]` marks become numbers.
5. Keep the `p-N` keys exactly as written. A key that is missing from `fydp.bib` shows as `[?]` again.

The template for every entry is:

```bibtex
@article{p-N,
  title={...},
  author={...},
  journal={...},
  year={...},
  volume={...},
  number={...},
  pages={...},
  doi={...}
}
```
Use `@inproceedings` with `booktitle` for conference papers, and `@misc` or `@article` with `journal={arXiv preprint}` for preprints.

## Entries

### p-1

- **Paper:** Deep Learning-Based Model for Non-invasive Hemoglobin Estimation via Body Parts Images: A Retrospective Analysis and a Prospective Emergency Department Study
- **Benchmark run:** B09
- **Cited in:**
  - Ch. 1 Introduction
  - 1.1 (sub) Background and Context
  - 1.1 (sub) Problem Statement
  - 1.2 (sub) What the Literature Leaves Open
  - 1.4 Methodology
  - 2.2.3 Multi-Site and Multi-Source Anemia Detection
  - 2.2.6 Attention-Based Fusion and Temporal Aggregation
  - 2.2.9 Methods Selected for Benchmarking
  - 2.3 Gap Analysis
  - 3.2.8.1 Benchmarked Methods
  - 4.3.6 Discussion

```bibtex
@article{p-1,
  title={Deep Learning-Based Model for Non-invasive Hemoglobin Estimation via Body Parts Images: A Retrospective Analysis and a Prospective Emergency Department Study},
  author={},
  journal={},
  year={}
}
```

### p-2

- **Paper:** Iron deficiency anemia detection using machine learning models: A comparative study of fingernails, palm and conjunctiva of the eye images
- **Benchmark run:** B07
- **Cited in:**
  - Ch. 1 Introduction
  - 1.1 (sub) Background and Context
  - 1.1 (sub) Problem Statement
  - 1.2 (sub) What the Literature Leaves Open
  - 1.4 Methodology
  - 2.2.2 Image-Based Detection from a Single Body Site
  - 2.2.9 Methods Selected for Benchmarking
  - 2.3 Gap Analysis
  - 3.2.8.1 Benchmarked Methods
  - 4.3.6 Discussion

```bibtex
@article{p-2,
  title={Iron deficiency anemia detection using machine learning models: A comparative study of fingernails, palm and conjunctiva of the eye images},
  author={},
  journal={},
  year={}
}
```

### p-3

- **Paper:** Anaemia Detection using Deep Learning by processing Conjunctiva, Fingernails, Palm and Tongue Images
- **Benchmark run:** -
- **Cited in:**
  - Ch. 1 Introduction
  - 1.1 (sub) Problem Statement
  - 2.2.3 Multi-Site and Multi-Source Anemia Detection

```bibtex
@article{p-3,
  title={Anaemia Detection using Deep Learning by processing Conjunctiva, Fingernails, Palm and Tongue Images},
  author={},
  journal={},
  year={}
}
```

### p-4

- **Paper:** Non-Invasive Hemoglobin prediction: A machine learning approach
- **Benchmark run:** -
- **Cited in:**
  - 1.1 (sub) Background and Context
  - 2.2.1 Signal-Based and Smartphone-Based Hemoglobin Estimation

```bibtex
@article{p-4,
  title={Non-Invasive Hemoglobin prediction: A machine learning approach},
  author={},
  journal={},
  year={}
}
```

### p-5

- **Paper:** A Multi-input Deep Neural Network Framework for Non-invasive Detection of Anemia using Finger Nail Images
- **Benchmark run:** B03
- **Cited in:**
  - 1.1 (sub) Background and Context
  - 1.1 (sub) Problem Statement
  - 1.4 Methodology
  - 2.2.2 Image-Based Detection from a Single Body Site
  - 2.2.9 Methods Selected for Benchmarking
  - 2.3 Gap Analysis
  - 3.2.8.1 Benchmarked Methods
  - 4.3.6 Discussion

```bibtex
@article{p-5,
  title={A Multi-input Deep Neural Network Framework for Non-invasive Detection of Anemia using Finger Nail Images},
  author={},
  journal={},
  year={}
}
```

### p-6

- **Paper:** Anemia Detection from Eyes, Palm and Fingernails with Machine Learning Models
- **Benchmark run:** -
- **Cited in:**
  - 1.1 (sub) Background and Context
  - 1.2 (sub) What the Literature Leaves Open
  - 2.2.3 Multi-Site and Multi-Source Anemia Detection

```bibtex
@article{p-6,
  title={Anemia Detection from Eyes, Palm and Fingernails with Machine Learning Models},
  author={},
  journal={},
  year={}
}
```

### p-7

- **Paper:** Fingertip Video Dataset for Non-Invasive Diagnosis of Anemia Using ResNet-18 Classifier
- **Benchmark run:** -
- **Cited in:**
  - 1.1 (sub) Background and Context
  - 1.1 (sub) Problem Statement
  - 1.2 (sub) What the Literature Leaves Open
  - 2.2.1 Signal-Based and Smartphone-Based Hemoglobin Estimation
  - 2.2.4 The Dataset Used in This Research and Related Work
  - 2.3 Gap Analysis

```bibtex
@article{p-7,
  title={Fingertip Video Dataset for Non-Invasive Diagnosis of Anemia Using ResNet-18 Classifier},
  author={},
  journal={},
  year={}
}
```

### p-8

- **Paper:** Image-Based Sensing of Leukonychia for Early Diagnosis of Anemia Using a Smartphone Application
- **Benchmark run:** -
- **Cited in:**
  - 1.1 (sub) Background and Context
  - 2.2.1 Signal-Based and Smartphone-Based Hemoglobin Estimation

```bibtex
@article{p-8,
  title={Image-Based Sensing of Leukonychia for Early Diagnosis of Anemia Using a Smartphone Application},
  author={},
  journal={},
  year={}
}
```

### p-9

- **Paper:** Machine/deep learning-assisted hemoglobin level prediction using palpebral conjunctival images
- **Benchmark run:** B13
- **Cited in:**
  - 1.1 (sub) Background and Context
  - 1.1 (sub) Problem Statement
  - 1.4 Methodology
  - 2.2.1 Signal-Based and Smartphone-Based Hemoglobin Estimation
  - 2.2.7 Explainability in Medical Deep Learning
  - 2.2.9 Methods Selected for Benchmarking
  - 2.3 Gap Analysis
  - 3.2.8.1 Benchmarked Methods
  - 4.3.6 Discussion

```bibtex
@article{p-9,
  title={Machine/deep learning-assisted hemoglobin level prediction using palpebral conjunctival images},
  author={},
  journal={},
  year={}
}
```

### p-10

- **Paper:** Machine vision model using nail images for non-invasive detection of iron deficiency anemia in university students
- **Benchmark run:** B11
- **Cited in:**
  - 1.1 (sub) Background and Context
  - 1.1 (sub) Problem Statement
  - 1.2 (sub) What the Literature Leaves Open
  - 1.2 (sub) Why Fusion, and Why a Benchmark
  - 1.4 Methodology
  - 2.2.2 Image-Based Detection from a Single Body Site
  - 2.2.4 The Dataset Used in This Research and Related Work
  - 2.2.9 Methods Selected for Benchmarking
  - 2.3 Gap Analysis
  - 3.2.8.1 Benchmarked Methods
  - 4.3.6 Discussion

```bibtex
@article{p-10,
  title={Machine vision model using nail images for non-invasive detection of iron deficiency anemia in university students},
  author={},
  journal={},
  year={}
}
```

### p-11

- **Paper:** An Explainable AI and Optimized Multi-Branch Convolutional Neural Network Model for Eye Anemia Diagnosis
- **Benchmark run:** -
- **Cited in:**
  - 1.1 (sub) Problem Statement
  - 2.2.2 Image-Based Detection from a Single Body Site
  - 2.2.7 Explainability in Medical Deep Learning

```bibtex
@article{p-11,
  title={An Explainable AI and Optimized Multi-Branch Convolutional Neural Network Model for Eye Anemia Diagnosis},
  author={},
  journal={},
  year={}
}
```

### p-12

- **Paper:** A Neural Network Optimization Approach for Anemia Prediction Using Image Pixels and Hemoglobin Data
- **Benchmark run:** -
- **Cited in:**
  - 1.1 (sub) Problem Statement
  - 2.2.1 Signal-Based and Smartphone-Based Hemoglobin Estimation

```bibtex
@article{p-12,
  title={A Neural Network Optimization Approach for Anemia Prediction Using Image Pixels and Hemoglobin Data},
  author={},
  journal={},
  year={}
}
```

### p-13

- **Paper:** Iron Deficiency Anemia Detection Using Deep CNN and Ensemble Learning
- **Benchmark run:** B08
- **Cited in:**
  - 1.1 (sub) Problem Statement
  - 1.2 (sub) What the Literature Leaves Open
  - 1.4 Methodology
  - 2.2.3 Multi-Site and Multi-Source Anemia Detection
  - 2.2.9 Methods Selected for Benchmarking
  - 2.3 Gap Analysis
  - 3.2.8.1 Benchmarked Methods

```bibtex
@article{p-13,
  title={Iron Deficiency Anemia Detection Using Deep CNN and Ensemble Learning},
  author={},
  journal={},
  year={}
}
```

### p-14

- **Paper:** Enhancing anemia detection through multimodal data fusion: a non-invasive approach using EHRs and conjunctiva images
- **Benchmark run:** B12
- **Cited in:**
  - 1.2 (sub) What the Literature Leaves Open
  - 1.4 Methodology
  - 2.2.3 Multi-Site and Multi-Source Anemia Detection
  - 2.2.6 Attention-Based Fusion and Temporal Aggregation
  - 2.2.9 Methods Selected for Benchmarking
  - 2.3 Gap Analysis
  - 3.2.8.1 Benchmarked Methods

```bibtex
@article{p-14,
  title={Enhancing anemia detection through multimodal data fusion: a non-invasive approach using EHRs and conjunctiva images},
  author={},
  journal={},
  year={}
}
```

### p-15

- **Paper:** Multimodal Anemia Detection Using Nail and Eye Images with Numerical Parameters
- **Benchmark run:** -
- **Cited in:**
  - 1.2 (sub) What the Literature Leaves Open
  - 2.2.3 Multi-Site and Multi-Source Anemia Detection

```bibtex
@article{p-15,
  title={Multimodal Anemia Detection Using Nail and Eye Images with Numerical Parameters},
  author={},
  journal={},
  year={}
}
```

### p-16

- **Paper:** Revolutionizing anemia detection: integrative machine learning models and advanced attention mechanisms
- **Benchmark run:** -
- **Cited in:**
  - 1.2 (sub) What the Literature Leaves Open
  - 2.2.2 Image-Based Detection from a Single Body Site
  - 2.2.3 Multi-Site and Multi-Source Anemia Detection
  - 2.2.6 Attention-Based Fusion and Temporal Aggregation

```bibtex
@article{p-16,
  title={Revolutionizing anemia detection: integrative machine learning models and advanced attention mechanisms},
  author={},
  journal={},
  year={}
}
```

### p-17

- **Paper:** Non-invasive multimodal dataset for the detection of iron deficiency anemia in young adults: fingertip videos, palm videos, and nail photographs
- **Benchmark run:** Dataset
- **Cited in:**
  - 1.2 (sub) What the Literature Leaves Open
  - 1.4 Methodology
  - 2.1.1 Anemia and Its Clinical Assessment
  - 2.2.4 The Dataset Used in This Research and Related Work
  - 2.3 Gap Analysis
  - 3.2.2.1 Source and Collection Protocol
  - 3.2.2.3 Cohort Composition
  - 3.2.3.6 Preprocessed Inputs

```bibtex
@article{p-17,
  title={Non-invasive multimodal dataset for the detection of iron deficiency anemia in young adults: fingertip videos, palm videos, and nail photographs},
  author={},
  journal={},
  year={}
}
```

### p-18

- **Paper:** Multimodal fusion with deep neural networks for leveraging CT imaging and electronic health record: a case-study in pulmonary embolism detection
- **Benchmark run:** B06
- **Cited in:**
  - 1.2 (sub) What the Literature Leaves Open
  - 1.4 Methodology
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning
  - 2.2.9 Methods Selected for Benchmarking
  - 2.3 Gap Analysis
  - 3.2.8.1 Benchmarked Methods

```bibtex
@article{p-18,
  title={Multimodal fusion with deep neural networks for leveraging CT imaging and electronic health record: a case-study in pulmonary embolism detection},
  author={},
  journal={},
  year={}
}
```

### p-19

- **Paper:** Exploring Fusion Strategies for Multi-Modal Pneumonia Classification with Modality-Specific Explainability
- **Benchmark run:** B04
- **Cited in:**
  - 1.2 (sub) What the Literature Leaves Open
  - 1.4 Methodology
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning
  - 2.2.7 Explainability in Medical Deep Learning
  - 2.2.9 Methods Selected for Benchmarking
  - 2.3 Gap Analysis
  - 3.2.8.1 Benchmarked Methods

```bibtex
@article{p-19,
  title={Exploring Fusion Strategies for Multi-Modal Pneumonia Classification with Modality-Specific Explainability},
  author={},
  journal={},
  year={}
}
```

### p-20

- **Paper:** Attentional Feature Fusion
- **Benchmark run:** B05
- **Cited in:**
  - 1.2 (sub) What the Literature Leaves Open
  - 1.4 Methodology
  - 2.2.6 Attention-Based Fusion and Temporal Aggregation
  - 2.2.9 Methods Selected for Benchmarking
  - 2.3 Gap Analysis
  - 3.2.8.1 Benchmarked Methods

```bibtex
@article{p-20,
  title={Attentional Feature Fusion},
  author={},
  journal={},
  year={}
}
```

### p-21

- **Paper:** Multi-view Classification Using Hybrid Fusion and Mutual Distillation
- **Benchmark run:** B10
- **Cited in:**
  - 1.2 (sub) What the Literature Leaves Open
  - 1.4 Methodology
  - 2.2.6 Attention-Based Fusion and Temporal Aggregation
  - 2.2.9 Methods Selected for Benchmarking
  - 2.3 Gap Analysis
  - 3.2.8.1 Benchmarked Methods

```bibtex
@article{p-21,
  title={Multi-view Classification Using Hybrid Fusion and Mutual Distillation},
  author={},
  journal={},
  year={}
}
```

### p-22

- **Paper:** A review of deep learning-based information fusion techniques for multimodal medical image classification
- **Benchmark run:** -
- **Cited in:**
  - 1.2 (sub) Why Fusion, and Why a Benchmark
  - 2.1.4 Multimodal Fusion
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-22,
  title={A review of deep learning-based information fusion techniques for multimodal medical image classification},
  author={},
  journal={},
  year={}
}
```

### p-23

- **Paper:** A Systematic Review of Intermediate Fusion in Multimodal Deep Learning for Biomedical Applications
- **Benchmark run:** -
- **Cited in:**
  - 1.2 (sub) Why Fusion, and Why a Benchmark
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-23,
  title={A Systematic Review of Intermediate Fusion in Multimodal Deep Learning for Biomedical Applications},
  author={},
  journal={},
  year={}
}
```

### p-24

- **Paper:** Multimodal deep learning for biomedical data fusion: a review
- **Benchmark run:** -
- **Cited in:**
  - 1.2 (sub) Why Fusion, and Why a Benchmark
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-24,
  title={Multimodal deep learning for biomedical data fusion: a review},
  author={},
  journal={},
  year={}
}
```

### p-25

- **Paper:** Non-invasive prediction of hemoglobin levels by principal component and back propagation artificial neural network
- **Benchmark run:** -
- **Cited in:**
  - 2.2.1 Signal-Based and Smartphone-Based Hemoglobin Estimation

```bibtex
@article{p-25,
  title={Non-invasive prediction of hemoglobin levels by principal component and back propagation artificial neural network},
  author={},
  journal={},
  year={}
}
```

### p-26

- **Paper:** Enhancing Anaemia Prediction Through Multi-Modal Data Integration and Generative Learning
- **Benchmark run:** -
- **Cited in:**
  - 2.2.3 Multi-Site and Multi-Source Anemia Detection

```bibtex
@article{p-26,
  title={Enhancing Anaemia Prediction Through Multi-Modal Data Integration and Generative Learning},
  author={},
  journal={},
  year={}
}
```

### p-27

- **Paper:** Fusion of medical imaging and electronic health records using deep learning: a systematic review and implementation guidelines
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-27,
  title={Fusion of medical imaging and electronic health records using deep learning: a systematic review and implementation guidelines},
  author={},
  journal={},
  year={}
}
```

### p-28

- **Paper:** Research and Application of Multimodal Data Fusion in Medical Images
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-28,
  title={Research and Application of Multimodal Data Fusion in Medical Images},
  author={},
  journal={},
  year={}
}
```

### p-29

- **Paper:** Early vs Late Fusion in Multimodal Convolutional Neural Networks
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-29,
  title={Early vs Late Fusion in Multimodal Convolutional Neural Networks},
  author={},
  journal={},
  year={}
}
```

### p-30

- **Paper:** MedFMIT: A Foundation Model-Driven Multimodal Fusion Method for Image-Text Disease Diagnosis
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-30,
  title={MedFMIT: A Foundation Model-Driven Multimodal Fusion Method for Image-Text Disease Diagnosis},
  author={},
  journal={},
  year={}
}
```

### p-31

- **Paper:** HyPCA-Net: Advancing Multimodal Fusion in Medical Image Analysis
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-31,
  title={HyPCA-Net: Advancing Multimodal Fusion in Medical Image Analysis},
  author={},
  journal={},
  year={}
}
```

### p-32

- **Paper:** Explainable Multimodal Deep Learning Framework for Early Disease Diagnosis
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning
  - 2.2.7 Explainability in Medical Deep Learning

```bibtex
@article{p-32,
  title={Explainable Multimodal Deep Learning Framework for Early Disease Diagnosis},
  author={},
  journal={},
  year={}
}
```

### p-33

- **Paper:** A review on multimodal medical image fusion: Compendious analysis of medical modalities, multimodal databases, fusion techniques and quality metrics
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-33,
  title={A review on multimodal medical image fusion: Compendious analysis of medical modalities, multimodal databases, fusion techniques and quality metrics},
  author={},
  journal={},
  year={}
}
```

### p-34

- **Paper:** A Review of Multimodal Medical Image Fusion Techniques
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-34,
  title={A Review of Multimodal Medical Image Fusion Techniques},
  author={},
  journal={},
  year={}
}
```

### p-35

- **Paper:** Generative AI in Multimodal Medical Image Fusion: A Comparative Analysis of GANs, VAEs, and DDPMs
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-35,
  title={Generative AI in Multimodal Medical Image Fusion: A Comparative Analysis of GANs, VAEs, and DDPMs},
  author={},
  journal={},
  year={}
}
```

### p-36

- **Paper:** A Multi-Modal Medical Image Fusion Technique Using Structural Similarity Index Measure and Correlation Coefficient
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-36,
  title={A Multi-Modal Medical Image Fusion Technique Using Structural Similarity Index Measure and Correlation Coefficient},
  author={},
  journal={},
  year={}
}
```

### p-37

- **Paper:** A Recent Advancements in Multi-Modal Medical Image Fusion Techniques using SWT, NSST, NSCT, and CNN
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-37,
  title={A Recent Advancements in Multi-Modal Medical Image Fusion Techniques using SWT, NSST, NSCT, and CNN},
  author={},
  journal={},
  year={}
}
```

### p-38

- **Paper:** An Enhanced Multi-Modal Medical Image Fusion Technique using Stationary Wavelet Transform and Method Noise based Wiener Filter
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-38,
  title={An Enhanced Multi-Modal Medical Image Fusion Technique using Stationary Wavelet Transform and Method Noise based Wiener Filter},
  author={},
  journal={},
  year={}
}
```

### p-39

- **Paper:** Multimodal Medical Image Registration and Fusion for Quality Enhancement
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-39,
  title={Multimodal Medical Image Registration and Fusion for Quality Enhancement},
  author={},
  journal={},
  year={}
}
```

### p-40

- **Paper:** Multimodal Medical Image Fusion using Hybrid Layer Decomposition with CNN-based Feature Mapping and Structural Clustering
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-40,
  title={Multimodal Medical Image Fusion using Hybrid Layer Decomposition with CNN-based Feature Mapping and Structural Clustering},
  author={},
  journal={},
  year={}
}
```

### p-41

- **Paper:** MACTFusion: Lightweight Cross Transformer for Adaptive Multimodal Medical Image Fusion
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-41,
  title={MACTFusion: Lightweight Cross Transformer for Adaptive Multimodal Medical Image Fusion},
  author={},
  journal={},
  year={}
}
```

### p-42

- **Paper:** MATR: Multimodal Medical Image Fusion via Multiscale Adaptive Transformer
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-42,
  title={MATR: Multimodal Medical Image Fusion via Multiscale Adaptive Transformer},
  author={},
  journal={},
  year={}
}
```

### p-43

- **Paper:** A review: Deep learning for medical image segmentation using multi-modality fusion
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-43,
  title={A review: Deep learning for medical image segmentation using multi-modality fusion},
  author={},
  journal={},
  year={}
}
```

### p-44

- **Paper:** NMD-FusionNet: a multimodal fusion-based medical imaging-assisted diagnostic model for liver cancer
- **Benchmark run:** -
- **Cited in:**
  - 2.2.5 Fusion Strategies in Multimodal Deep Learning

```bibtex
@article{p-44,
  title={NMD-FusionNet: a multimodal fusion-based medical imaging-assisted diagnostic model for liver cancer},
  author={},
  journal={},
  year={}
}
```

### p-45

- **Paper:** Hierarchical Attentional Feature Fusion for Surgical Instrument Segmentation
- **Benchmark run:** -
- **Cited in:**
  - 2.2.6 Attention-Based Fusion and Temporal Aggregation

```bibtex
@article{p-45,
  title={Hierarchical Attentional Feature Fusion for Surgical Instrument Segmentation},
  author={},
  journal={},
  year={}
}
```

### p-46

- **Paper:** A Review of Deep Learning-based Multi-modal Medical Image Fusion
- **Benchmark run:** -
- **Cited in:**
  - 2.2.6 Attention-Based Fusion and Temporal Aggregation

```bibtex
@article{p-46,
  title={A Review of Deep Learning-based Multi-modal Medical Image Fusion},
  author={},
  journal={},
  year={}
}
```

### p-47

- **Paper:** Cross-Modal Multistep Fusion Network With Co-Attention for Visual Question Answering
- **Benchmark run:** -
- **Cited in:**
  - 2.2.6 Attention-Based Fusion and Temporal Aggregation

```bibtex
@article{p-47,
  title={Cross-Modal Multistep Fusion Network With Co-Attention for Visual Question Answering},
  author={},
  journal={},
  year={}
}
```

### p-48

- **Paper:** Enhancing lung diseases recognition through CNN-RNN methodologies
- **Benchmark run:** -
- **Cited in:**
  - 2.2.6 Attention-Based Fusion and Temporal Aggregation

```bibtex
@article{p-48,
  title={Enhancing lung diseases recognition through CNN-RNN methodologies},
  author={},
  journal={},
  year={}
}
```

### p-49

- **Paper:** Survey of Explainable AI Techniques in Healthcare
- **Benchmark run:** -
- **Cited in:**
  - 2.2.7 Explainability in Medical Deep Learning

```bibtex
@article{p-49,
  title={Survey of Explainable AI Techniques in Healthcare},
  author={},
  journal={},
  year={}
}
```

### p-50

- **Paper:** Deep Learning Models for Classification of Red Blood Cells in Microscopy Images to Aid in Sickle Cell Anemia Diagnosis
- **Benchmark run:** -
- **Cited in:**
  - 2.2.8 Blood-Smear-Based Anemia Analysis

```bibtex
@article{p-50,
  title={Deep Learning Models for Classification of Red Blood Cells in Microscopy Images to Aid in Sickle Cell Anemia Diagnosis},
  author={},
  journal={},
  year={}
}
```

### p-51

- **Paper:** Detection of the Top Anemic Diseases in Blood Smear Images Using Image Quantization Followed by Ensemble of Classifiers
- **Benchmark run:** -
- **Cited in:**
  - 2.2.8 Blood-Smear-Based Anemia Analysis

```bibtex
@article{p-51,
  title={Detection of the Top Anemic Diseases in Blood Smear Images Using Image Quantization Followed by Ensemble of Classifiers},
  author={},
  journal={},
  year={}
}
```

### p-52

- **Paper:** Detection of some anaemia types in human blood smears using neural networks
- **Benchmark run:** -
- **Cited in:**
  - 2.2.8 Blood-Smear-Based Anemia Analysis

```bibtex
@article{p-52,
  title={Detection of some anaemia types in human blood smears using neural networks},
  author={},
  journal={},
  year={}
}
```
