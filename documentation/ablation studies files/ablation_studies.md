# 1 Ablation Studies

The system studied in this thesis estimates anaemia status from three non-invasive inputs: a fingertip video, a palm video and a nail photograph. Each input is encoded by its own ResNet-18, the two video encoders are followed by a temporal pooling step, and the three embeddings are fused before a small classification head. Some experiments add a fourth input, a short vector of demographic and symptom data. This design contains several choices that could change performance. The ablation experiments reported here were run to find out which of these choices matter in practice: how the modality embeddings are fused, how frames are pooled over time, whether demographic data are added, and whether the task is binary or three-class. They also served a second purpose, which is to decide which configuration is worth carrying forward to a wider comparison with other model frameworks.

## 1.1 Ablation Study Design

All experiments share the same backbone and input pipeline. A fingertip video contributes 32 pre-extracted frames, a palm video contributes 16 frames, and the nail photograph is used as one image. Every image is 224×224 pixels with ImageNet normalisation, and no data augmentation is applied. Each modality has its own ImageNet-pretrained ResNet-18 that produces a 256-dimensional embedding. Training uses the Adam optimiser with a learning rate of $10^{-4}$, a batch size of 16 and a dropout rate of 0.3. The three-class models use cross-entropy loss with inverse-frequency class weights computed from the training labels, so that the rare classes are not ignored. In the binary experiment (Ex-0) the loss is plain binary cross-entropy without class weights. In every run the checkpoint with the best validation score is kept, using macro-F1 for the three-class runs and ROC-AUC for the binary runs.

Four components were varied. (i) *Fusion:* either concatenation of the three embeddings followed by a 128-unit layer, or self-attention, in which the embeddings plus learned modality embeddings pass through one Transformer encoder layer with four heads, are averaged, and feed a 64-unit layer. (ii) *Temporal pooling:* either a simple mean over frame embeddings, or a bidirectional LSTM (Bi-LSTM) with 128 hidden units per direction whose final states are projected back to 256 dimensions. (iii) *Demographic data:* either absent, or added as a fourth token produced by a small multilayer perceptron from twelve features (age, sex, heart rate, weight, height and seven symptom flags: fatigue, weakness, irregular heartbeat, shortness of breath, dizziness, chest pain and headache). (iv) *Task:* three severity classes (Normal, Leve for mild anaemia, Moderada for moderate anaemia) or a binary Normal-versus-anaemic task. The three-class experiments cover every combination of fusion, pooling and demographic data, which gives eight configurations. Table 1 lists the experiments.

**Table 1.** Experiments included in the ablation study. Every three-class experiment was run with both fusion methods. Usable subjects are those with all three modalities available on disk. † marks the configuration whose 5-fold result could not be treated as reliable (Section 1.4).

| Experiment | Task | Pooling | Demographics | Protocols run | Usable / test subjects |
|:---|:---|:---|:---|:---|:---|
| Ex-0 | Binary | Mean | No | Single split (20 epochs); 5-fold (10 epochs) | 737 / 110 |
| Ex-1 (mean) | Three-class | Mean | No | Single split; 5-fold | 808 / 123 |
| Ex-1 (Bi-LSTM) | Three-class | Bi-LSTM | No | Single split; 5-fold | 808 / 123 |
| Ex-2 (supplementary) | Three-class | Mean | Yes | 5-fold only; AdamW, weight decay $10^{-4}$ | 808 / 123 |
| Ex-3 (mean) | Three-class | Mean | Yes | Single split; 5-fold | 808 / 123 |
| Ex-3 (Bi-LSTM) | Three-class | Bi-LSTM | Yes | Single split; 5-fold† | 808 / 123 |

Each three-class configuration was evaluated under two protocols. In the single-split protocol, a model was trained for 20 epochs on 562 subjects, its checkpoint was chosen on 123 validation subjects, and it was scored once on 123 test subjects. In the 5-fold protocol, the training and validation subjects were pooled (685 subjects) and divided into five patient-level stratified folds of 548 training and 137 validation subjects. Each fold trained for 10 epochs, and the best checkpoint of each fold was scored on the same fixed test set. The test set is identical in both protocols and contains 87 Normal, 29 Leve and only 7 Moderada subjects. Results of the 5-fold protocol are given as mean ± standard deviation over the five fold models. This spread reflects changes in the training and validation data, but it does not capture the sampling uncertainty of the small test set. Because the two protocols differ in training length and training-set size, they are compared within each protocol and not against each other.

Macro-F1 is used as the main metric. It was also the checkpoint-selection criterion, and it gives the three classes equal weight, which matters because Moderada subjects are about 8% of the data. For class $c$ with $n_c$ training subjects, $N$ training subjects in total and $K=3$ classes, the class weight and the macro-F1 are

$$
w_c = \frac{N}{K\,n_c}, \qquad \text{macro-F1} = \frac{1}{K}\sum_{c=1}^{K} F1_c .
$$

Macro one-vs-rest ROC-AUC and accuracy are reported as supporting metrics. Accuracy needs caution, because predicting Normal for every test subject would already give $87/123 = 0.707$. Two tabular baselines, logistic regression and a random forest trained only on the twelve demographic and symptom features, were scored on the same test set to show what the demographic data can do without images.

## 1.2 Quantitative Comparison

Table 2 gives the 5-fold results, which are the most stable evidence in the archive. Six configurations (the image-only and the mean-pooling demographic configurations, each with both fusion methods) can be compared fairly with one another. Their mean macro-F1 lies between 0.380 and 0.459, and their mean macro-AUC between 0.675 and 0.721. The fold-to-fold standard deviation of macro-F1 is 0.040 to 0.089, so most gaps between these configurations are smaller than the spread within a single configuration. Mean accuracy (0.623 to 0.670) stays below the 0.707 reached by always predicting Normal.

**Table 2.** Three-class results under the 5-fold protocol on the fixed test set (123 subjects). Values are mean ± standard deviation over five fold models. The tabular baselines are single fits scored once on the same test set. † See Section 1.4.

| Configuration | Fusion | Macro-F1 | Macro-AUC | Accuracy |
|:---|:---|:---|:---|:---|
| Image-only, mean pooling (Ex-1) | Concatenation | 0.440 ± 0.089 | 0.711 ± 0.056 | 0.667 ± 0.048 |
|  | Self-attention | 0.433 ± 0.040 | 0.708 ± 0.027 | 0.623 ± 0.058 |
| Image-only, Bi-LSTM pooling (Ex-1) | Concatenation | 0.380 ± 0.041 | 0.675 ± 0.043 | 0.626 ± 0.063 |
|  | Self-attention | 0.459 ± 0.075 | 0.715 ± 0.033 | 0.670 ± 0.040 |
| + Demographics, mean pooling (Ex-3) | Concatenation | 0.404 ± 0.053 | 0.721 ± 0.036 | 0.655 ± 0.054 |
|  | Self-attention | 0.410 ± 0.065 | 0.692 ± 0.047 | 0.634 ± 0.038 |
| + Demographics, Bi-LSTM pooling (Ex-3)† | Concatenation | 0.668 ± 0.032 | 0.891 ± 0.027 | 0.878 ± 0.032 |
|  | Self-attention | 0.687 ± 0.034 | 0.912 ± 0.032 | 0.836 ± 0.064 |
| Demographics, mean pooling (Ex-2, supplementary) | Concatenation | 0.427 ± 0.077 | 0.711 ± 0.047 | 0.646 ± 0.052 |
|  | Self-attention | 0.417 ± 0.057 | 0.718 ± 0.045 | 0.589 ± 0.050 |
| Tabular logistic regression (features only) | — | 0.407 | 0.717 | 0.480 |
| Tabular random forest (features only) | — | 0.344 | 0.699 | 0.699 |

The tabular logistic regression reaches a macro-F1 of 0.407 and a macro-AUC of 0.717 from demographic data alone. No comparable image-based configuration exceeds its macro-F1 by more than 5.3 percentage points. The one configuration that stands far above the rest is the Bi-LSTM model with demographic data, which reaches macro-F1 values of 0.668 and 0.687. This result is treated with caution for the reasons given in Section 1.4.

Table 3 shows the single-split results. They paint a less consistent picture. With image-only inputs and mean pooling, self-attention beats concatenation by 13.5 percentage points of macro-F1 (0.468 against 0.333), yet under 5-fold the two differ by only 0.7 points in favour of concatenation. With image-only inputs and Bi-LSTM pooling, concatenation is ahead in the single split (0.439 against 0.406), while self-attention is ahead by 8.0 points under 5-fold. Adding demographic data to the mean-pooling model changes the concatenation model by +10.8 points in the single split and by −3.6 points under 5-fold. Differences of five to thirteen percentage points between single runs therefore cannot be read as component effects, because a single training run and a test set with seven Moderada subjects are too noisy to support them.

**Table 3.** Three-class results under the single-split protocol (20 epochs, one run per configuration, 123 test subjects). The tabular baselines are those of the single-split Ex-3 notebook.

| Configuration | Fusion | Macro-F1 | Macro-AUC | Accuracy |
|:---|:---|:---|:---|:---|
| Image-only, mean pooling (Ex-1) | Concatenation | 0.333 | 0.659 | 0.455 |
|  | Self-attention | 0.468 | 0.750 | 0.593 |
| Image-only, Bi-LSTM pooling (Ex-1) | Concatenation | 0.439 | 0.742 | 0.569 |
|  | Self-attention | 0.406 | 0.697 | 0.569 |
| + Demographics, mean pooling (Ex-3) | Concatenation | 0.440 | 0.677 | 0.626 |
|  | Self-attention | 0.373 | 0.656 | 0.577 |
| + Demographics, Bi-LSTM pooling (Ex-3)† | Concatenation | 0.433 | 0.695 | 0.569 |
|  | Self-attention | 0.512 | 0.746 | 0.691 |
| Tabular logistic regression (features only) | — | 0.417 | 0.711 | 0.496 |
| Tabular random forest (features only) | — | 0.323 | 0.698 | 0.650 |

Figure 1 summarises the 5-fold macro-F1 values and adds a view of the Leve (mild) class. Panel (a) shows how closely the six comparable configurations sit together and how far the Bi-LSTM model with demographic data lies from them. Panel (b) shows why that distance is suspicious: its mild-class F1 is close to 1, while every other configuration, including the demographic-only logistic regression, stays below 0.5.

![Figure 1](fig_ablation_f1.png)

**Figure 1.** Five-fold test results. (a) Macro-F1 (bars: mean over folds; error bars: one standard deviation; dots: individual folds). (b) F1 of the Leve class from the fold-averaged predictions. The dashed line marks the tabular logistic regression. † See Section 1.4.

## 1.3 Analysis of Individual Components

### 1.3.1 Fusion method

**Table 4.** Self-attention compared with concatenation under the 5-fold protocol. The difference is the change in the mean fold score in percentage points. The p-value is the two-sided paired sign-flip permutation test (2000 iterations) reported by each notebook, applied to the five per-fold scores.

| Configuration | Metric | Difference (pp) | Folds favouring self-attention | p-value |
|:---|:---|:---|:---|:---|
| Image-only, mean pooling (Ex-1) | Macro-F1 | −0.7 | 3 / 5 | 0.948 |
| Image-only, Bi-LSTM (Ex-1) | Macro-F1 | +8.0 | 4 / 5 | 0.178 |
| + Demographics, mean pooling (Ex-3) | Macro-F1 | +0.6 | 3 / 5 | 0.702 |
| + Demographics, mean pooling (Ex-2, supplementary) | Macro-F1 | −1.1 | 2 / 5 | 0.875 |
| + Demographics, Bi-LSTM (Ex-3)† | Macro-F1 | +2.0 | 3 / 5 | 0.548 |
| Binary task, mean pooling (Ex-0) | F1 | +12.0 | 4 / 5 | 0.251 |
|  | ROC-AUC | −0.02 | 2 / 5 | 1.000 |

Fusion is the one component that every experiment compares, so it offers the most evidence (Table 4). Among the three comparable three-class settings, self-attention has the higher mean macro-F1 in 2 cases. Only one of these gains is sizeable: with image-only inputs and Bi-LSTM pooling, self-attention improves macro-F1 by 8.0 points and wins in four of five folds. In the other two settings the difference is below one point, and in the supplementary Ex-2 run concatenation is ahead by 1.1 points. In the binary task, self-attention raises the mean F1 by 12.0 points, but the mean ROC-AUC is unchanged.

None of these differences is statistically supported. The smallest p-value is 0.178. It is also worth noting a property of the test itself: with five folds there are only 32 possible sign patterns, so the smallest two-sided p-value that can be obtained is $2/32 = 0.0625$. A 5-fold comparison of this kind cannot reach the 0.05 level, however large the difference. The evidence therefore does not show that self-attention is better or worse than concatenation. It only suggests that self-attention may help when frame information is pooled by a Bi-LSTM, a pattern that a larger evaluation would need to confirm.

The per-class results in Table 7 add a detail. For the image-only Bi-LSTM model, self-attention raised the Leve F1 from 0.291 to 0.353 and was the only image-only model that recovered any Moderada subject (F1 0.222, which is one correct subject out of seven). For image-only mean pooling, self-attention raised the Leve F1 from 0.357 to 0.471 but still found no Moderada subject.

### 1.3.2 Temporal pooling

For the image-only models, replacing mean pooling with a Bi-LSTM had opposite effects for the two fusion methods (Table 5). Macro-F1 fell by 6.1 points with concatenation and rose by 2.6 points with self-attention. Both changes are smaller than the fold-to-fold standard deviation of the models involved (0.089 and 0.075). The Bi-LSTM therefore shows no consistent benefit over simple averaging in the comparable experiments.

A possible explanation is that the Bi-LSTM adds many parameters while the training data contain only a few hundred subjects. The training histories are consistent with this idea but do not prove it. In every clean configuration, the mean validation loss was lowest after the first or second epoch (about 0.98) and had risen to between 1.9 and 2.5 by epoch 10, while the training loss fell to between 0.15 and 0.24. This overfitting pattern appears with mean pooling as well, so it is not specific to the Bi-LSTM, and it also means that the epoch at which the best checkpoint is found varies widely between folds.

**Table 5.** Change in the 5-fold mean test macro-F1 (percentage points) when one component is changed and the other components are kept fixed. The last column states whether the comparison used the same image data in both runs. † See Section 1.4.

| Change made | Concatenation | Self-attention | Fair comparison |
|:---|:---|:---|:---|
| Bi-LSTM instead of mean pooling, image-only | −6.1 | +2.6 | Yes |
| Add demographic data, mean pooling | −3.6 | −2.3 | Yes |
| Bi-LSTM instead of mean pooling, with demographic data | +26.4 | +27.7 | No† |
| Add demographic data, Bi-LSTM pooling | +28.8 | +22.8 | No† |

### 1.3.3 Demographic and symptom features

Under mean pooling, where the comparison is fair, adding the twelve demographic and symptom features did not help. Macro-F1 changed by −3.6 points for concatenation and by −2.3 points for self-attention. The macro-AUC moved by +1.0 and −1.6 points, which is also within the fold spread. The supplementary Ex-2 run, which used the same inputs with a different optimiser, gave macro-F1 values of 0.427 and 0.417, close to the Ex-3 values of 0.404 and 0.410.

The tabular baselines help to explain this. Used alone, the demographic and symptom features give a macro-F1 of 0.407 (logistic regression) and 0.344 (random forest). The random forest reaches an accuracy of 0.699, essentially the majority-class rate, because it finds none of the Moderada subjects and few of the Leve subjects. These features are therefore weak predictors on their own, and the experiments give no sign that the deep model gains from them. This does not show that the features are useless in general, only that they added nothing measurable under this setup.

### 1.3.4 Binary versus three-class formulation

**Table 6.** Binary Normal-versus-anaemic results (Ex-0, mean pooling, no demographic data, unweighted loss). Sensitivity is the recall of the anaemic class. The 5-fold test set has 110 subjects (79 Normal, 31 anaemic).

| Protocol | Fusion | Accuracy | Sensitivity | Specificity | F1 | ROC-AUC |
|:---|:---|:---|:---|:---|:---|:---|
| Single split (test, 110 subjects) | Concatenation | 0.727 | 0.484 | 0.823 | 0.500 | 0.737 |
|  | Self-attention | 0.709 | 0.387 | 0.835 | 0.429 | 0.731 |
| 5-fold (mean ± SD) | Concatenation | 0.722 ± 0.038 | 0.200 ± 0.183 | 0.927 ± 0.027 | 0.263 ± 0.185 | 0.724 ± 0.067 |
|  | Self-attention | 0.684 ± 0.044 | 0.406 ± 0.248 | 0.792 ± 0.153 | 0.383 ± 0.144 | 0.724 ± 0.052 |

The binary experiment (Ex-0) gives a ROC-AUC of about 0.724 for both fusion methods under 5-fold, which is similar to the three-class macro-AUC values (Table 6). Its other metrics show a strong bias toward the majority class. With concatenation the mean sensitivity is only 0.200 and the specificity is 0.927, so the model finds about one anaemic subject in five. Its mean accuracy of 0.722 matches the share of Normal subjects in the test set (79/110 = 0.718). Self-attention is less conservative, with a mean sensitivity of 0.406 and an F1 of 0.383, but its fold-to-fold variation is large. The unweighted loss is a likely contributor to this behaviour, although the experiments did not test it.

Ex-0 cannot serve as the baseline against which the three-class models are ablated. It uses a different subject cohort (737 usable subjects against 808) and a different test set (110 against 123 subjects). It also reads the class and binary labels from the split file, whose class column the later notebooks report as misaligned for subject IDs 058 to 227 and replace with labels re-derived from the metadata file. The archive does not show whether this explains the 71 fewer usable subjects. Ex-0 is therefore reported as a reference for the binary formulation only, and no claim is made about whether a binary or a three-class formulation is easier.

## 1.4 Reliability of the Evidence

The largest effects in the archive belong to the Ex-3 Bi-LSTM configuration with demographic data. Under 5-fold it reaches macro-F1 values of 0.668 (concatenation) and 0.687 (self-attention), and mean macro-AUC values of 0.891 and 0.912. That is 26.4 to 27.7 points above the same model with mean pooling. Several observations argue against taking this result at face value.

First, this is the only 5-fold notebook that reads its images from different input folders. The other notebooks read the frames and nail images from one common input dataset. The Ex-3 Bi-LSTM notebook reads fingertip, palm and nail images from three separately named datasets. A line-by-line comparison of the two Ex-3 5-fold notebooks shows that the pooling mode, the image folders and the output folder name are the only differences. Second, the pattern of errors is unlike every other run. The fold-averaged predictions classify all 29 Leve subjects correctly with both fusion methods, giving a Leve F1 of 0.983 and 1.000 (Table 7). No other three-class configuration exceeds 0.471 on this class, and the demographic-only logistic regression reaches 0.377. Third, the Ex-3 Bi-LSTM single-split run, which used the common image folders, gave only 0.433 and 0.512 macro-F1 (Table 3), in line with the other single-split runs.

The archive does not reveal the cause. The preprocessing report mentions an earlier version of the frame extraction in which one class was saved at raw capture resolution instead of the target size, so a class-dependent difference in the stored images is one possible explanation. This is only a hypothesis. It cannot be checked from the files provided. Until the experiment is repeated on the common image folders, the 5-fold Ex-3 Bi-LSTM result is reported here but excluded from every conclusion about the effect of pooling or demographic data. It is marked with a dagger throughout.

**Table 7.** Per-class F1 on the test set (support: 87 Normal, 29 Leve, 7 Moderada) from the fold-averaged predictions of the 5-fold models. The last column gives the recall of the Moderada class. † See Section 1.4.

| Configuration | Fusion | Normal | Leve | Moderada | Moderada recall |
|:---|:---|:---|:---|:---|:---|
| Image-only, mean pooling (Ex-1) | Concatenation | 0.822 | 0.357 | 0.000 | 0.000 |
|  | Self-attention | 0.793 | 0.471 | 0.000 | 0.000 |
| Image-only, Bi-LSTM pooling (Ex-1) | Concatenation | 0.778 | 0.291 | 0.000 | 0.000 |
|  | Self-attention | 0.828 | 0.353 | 0.222 | 0.143 |
| + Demographics, mean pooling (Ex-3) | Concatenation | 0.815 | 0.245 | 0.000 | 0.000 |
|  | Self-attention | 0.783 | 0.240 | 0.167 | 0.143 |
| + Demographics, Bi-LSTM pooling (Ex-3)† | Concatenation | 0.925 | 0.983 | 0.143 | 0.143 |
|  | Self-attention | 0.929 | 1.000 | 0.400 | 0.571 |
| Tabular logistic regression | — | 0.631 | 0.377 | 0.213 | 0.714 |
| Tabular random forest | — | 0.816 | 0.216 | 0.000 | 0.000 |

Other limits on the evidence are smaller but real. The test set contains seven Moderada subjects, so one subject changes that class’s recall by 14 points. Runs of the same configuration also differ: the supplementary Ex-2 run and the Ex-3 mean-pooling run use the same inputs and differ only in the optimiser, yet their mean macro-F1 values differ by up to 2.3 points. Some result files also needed cleaning before use. The per-fold result files of the two Ex-3 self-attention runs contain extra rows from an earlier partial run (two in the Bi-LSTM file and three in the mean-pooling file). Each row was matched against the values printed in the notebook output, and the unmatched rows were removed. The remaining five rows per method reproduce the summary files exactly. The training-history files of the Ex-3 mean-pooling self-attention run contain the same duplication, so loss-curve statements in this section rely only on the clean configurations. Finally, the notebook descriptions of Ex-3 mention eight symptom flags and oxygen saturation, but the executed output lists twelve features without oxygen saturation, with seven symptom flags; the executed output was followed.

## 1.5 Summary of Ablation Findings

The ablation experiments support a cautious set of conclusions. Under the 5-fold protocol, six configurations that can be compared fairly produced mean macro-F1 values within about eight percentage points of each other, a range comparable to the fold-to-fold spread. Self-attention fusion was not shown to be better or worse than concatenation, replacing mean pooling by a Bi-LSTM had no consistent effect, and adding demographic and symptom features did not improve results where the comparison was fair. Single-split results often disagreed with 5-fold results, which shows that single runs on this test set are unreliable guides to component effects.

Two further points qualify these findings. The Moderada class was detected rarely or not at all in nearly every configuration, so the macro-F1 values mainly reflect the Normal and Leve classes. The one configuration with much higher scores, the Ex-3 Bi-LSTM model with demographic data, shows signs of a data or pipeline problem and cannot yet be used as evidence for or against any component.

## 1.6 Implications for Further Investigation

This section separates what the experiments showed from what they suggest and from what has still to be tested.

**Observed findings.** Among the configurations that can be compared fairly, the image-only model with Bi-LSTM pooling and self-attention fusion (Ex-1) has the highest mean 5-fold macro-F1 (0.459 ± 0.075). Its fold-averaged predictions also give the highest macro-F1 (0.468) and accuracy (0.707) of those configurations, with a macro-AUC of 0.756. It is the only comparable configuration that recovers any Moderada subject without also relying on demographic data (Moderada F1 0.222), although this rests on one correct subject out of seven. The margins are small. Its macro-F1 is 1.9 points above the simpler image-only mean-pooling concatenation model (0.440 ± 0.089), the permutation p-value against concatenation is 0.178, and the macro-AUC values of all comparable configurations lie within 4.6 points of each other. Its accuracy of 0.707 equals the 0.707 obtained by always predicting Normal, so accuracy gives no support for it.

**Research interpretation.** The evidence is inconclusive, and no configuration clearly dominates. If one configuration has to be chosen for the next stage, the image-only Bi-LSTM and self-attention model is a reasonable provisional candidate, for three reasons. It leads on the primary metric under both the per-fold and the fold-averaged views. It does not require demographic data, which gave no measurable benefit in the fair comparisons and which would complicate deployment. And it retains some ability to separate the minority classes, which a screening model needs. Its weaknesses are equally clear: the lead over several alternatives is within one standard deviation, its single-split run scored lower than its concatenation counterpart (0.406 against 0.439), the Moderada class is almost undetected, and the Leve F1 is only 0.353. The simpler mean-pooling concatenation model differs from it by less than one fold standard deviation and should be kept as a control. The research objective does not state which error is more costly, for example missed anaemia (sensitivity) versus false alarms, so the choice of macro-F1 as the priority is a modelling decision and not a clinical one. No claim is made here about computational efficiency, robustness or clinical suitability, since none of these was measured.

**Proposed future investigation (not yet performed).** First, the Ex-3 Bi-LSTM experiment should be repeated on the common image folders, and the stored images of that run should be compared class by class, for example by resolution and file properties, to see whether a class-dependent artefact exists. Second, the leading candidate and the mean-pooling control should be re-evaluated with several random seeds and with repeated cross-validation, because five folds cannot reach conventional significance levels. Third, the evaluation should add sensitivity-focused measures and class-specific thresholds once the intended use is defined. Fourth, the candidate should be benchmarked against other model frameworks under the same subject cohort, label source, split and metrics. Such a benchmark could also include the single-modality ResNet-18 baselines described in the project notes, which were run on a different task and protocol and would first have to be re-evaluated under the common one. Techniques not used so far, such as data augmentation or earlier stopping, may also be worth testing, given the early overfitting seen in the training histories.
