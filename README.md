## BreastDM-Reproduction


## Project Overview

This repository investigates the reproducibility of **BreastDM: A DCE-MRI Dataset for Breast Tumor Image Segmentation and Classification**. The project reproduces and evaluates selected segmentation and classification experiments from the original paper using the publicly released BreastDM dataset and code.

The repository includes data-preprocessing notebooks, dataset-integrity checks, segmentation experiments using FCN, U-Net, and UNeXt, and a reconstructed implementation of the paper’s Local-Global Cross Attention Fusion Network (LG-CAFN). Experiments are performed with patient-level separation to reduce data leakage and produce more reliable estimates of model generalization.

During this reproduction, several issues were identified in the released materials. The original dataset contains duplicated patients across certain training and test directories, while important LG-CAFN preprocessing and evaluation details are not fully documented. In particular, the released 9- and 17-channel MRI arrays are incompatible with the three-channel inputs expected by the provided model code, and no conversion procedure is supplied.

To address these limitations, this project introduces leakage-free patient splits, dataset manifests, synchronized multichannel preprocessing, explicit patient-level evaluation, and a documented temporal-channel adapter. Because some implementation decisions had to be reconstructed, the classification models are identified as **LG-CAFN-R** rather than exact reproductions of the original architecture.

The initial LG-CAFN-R models showed substantial overfitting and limited test generalization. Improved versions using class weighting and differential fine-tuning increased patient-level test AUC from **0.7510 to 0.8059** for the 9-channel model and from **0.7627 to 0.8176** for the 17-channel model. The improved 17-channel model achieved **76.60% patient-level test accuracy**, compared with **63.83%** for its baseline.

This repository is intended to provide a transparent record of the reproduction process, including successful experiments, unresolved ambiguities, negative findings, methodological changes, and differences from the paper-reported results. It is an independent research reproduction and is not intended for clinical use.

## Classification Results

All values below are patient-level test results at a decision threshold of
`0.5`.

| Experiment | Accuracy | Balanced accuracy | Sensitivity | Specificity | ROC AUC |
|---|---:|---:|---:|---:|---:|
| LG-CAFN-R9 baseline | 63.83% | 65.29% | 60.00% | 70.59% | 0.7510 |
| LG-CAFN-R17 baseline | 63.83% | 50.00% | 100.00% | 0.00% | 0.7627 |
| LG-CAFN-R9+ | 74.47% | **78.73%** | 63.33% | **94.12%** | 0.8059 |
| LG-CAFN-R17+ | **76.60%** | 77.84% | **73.33%** | 82.35% | **0.8176** |

### Patient-Level Bootstrap Confidence Intervals

Uncertainty in the main classification results was estimated using 10,000
stratified patient-level bootstrap samples with seed `20260805`. Each bootstrap
replicate sampled 17 benign and 30 malignant test patients with replacement.
Values below show the observed result followed by the percentile 95% confidence
interval.

| Experiment | Accuracy (95% CI) | Balanced Accuracy (95% CI) | Sensitivity (95% CI) | Specificity (95% CI) | ROC AUC (95% CI) |
|---|---:|---:|---:|---:|---:|
| LG-CAFN-R9 baseline | 63.83% (51.06–76.60) | 65.29% (51.47–79.12) | 60.00% (43.33–76.67) | 70.59% (47.06–88.24) | 0.7510 (0.5745–0.9000) |
| LG-CAFN-R17 baseline | 63.83% (63.83–63.83) | 50.00% (50.00–50.00) | 100.00% (100.00–100.00) | 0.00% (0.00–0.00) | 0.7627 (0.6157–0.8882) |
| LG-CAFN-R9+ | 74.47% (61.70–85.11) | 78.73% (67.84–88.33) | 63.33% (46.67–80.00) | 94.12% (82.35–100.00) | 0.8059 (0.6686–0.9176) |
| LG-CAFN-R17+ | 76.60% (63.83–87.23) | 77.84% (65.29–89.12) | 73.33% (56.67–90.00) | 82.35% (64.71–100.00) | 0.8176 (0.6784–0.9373) |

The R17 baseline has zero-width intervals for its threshold-dependent metrics
because it classified every patient as malignant and every stratified bootstrap
sample retained the same 30 malignant and 17 benign patients. Its AUC interval
remains variable because AUC measures probability ranking rather than the fixed
classification decision.

Class weighting, differential learning rates, and gradual fine-tuning improved
both reconstructed models. R17+ produced the highest observed accuracy and ROC AUC, while R9+ produced the highest observed balanced accuracy and specificity. However, every paired
R17+-versus-R9+ bootstrap interval included zero, so this analysis does not
establish that either improved model is definitively superior overall. The improved
R17 model also corrected the baseline model's collapse to predicting every test
patient as malignant.

For context, the paper reports accuracies of 88.20% and 83.93% with ROC AUC
values of 0.9154 and 0.8826. These values are not directly equivalent to the
reconstructed results because the paper groups cannot be mapped confidently to
R9 and R17, and this project uses documented reconstruction assumptions and a
leakage-corrected patient split.

## Segmentation Results

The validation-selected U-Net, FCN-ResNet50, and improved UNeXt checkpoints were evaluated on the same test set using one shared evaluator, a fixed threshold of `0.5`, and consistent metric definitions. The test set contains 4,284 images from 35 patients, with no patient overlap between the training, validation, and test splits.

| Model | Global foreground Dice | Global foreground IoU | Mean patient Dice | Mean patient IoU | Macro mIoU |
|---|---:|---:|---:|---:|---:|
| UNeXt improved | **66.27%** | **49.55%** | 56.11% | 43.21% | **74.69%** |
| FCN-ResNet50 | 59.61% | 42.46% | 46.43% | 34.67% | 71.13% |
| U-Net first trial | 59.31% | 42.15% | **58.07%** | **45.77%** | 70.99% |

These are test-set results rather than validation-only results. The shared evaluator, executed notebook, metric configuration, dataset and checkpoint hashes, and direct comparison table are preserved in the refactored repository.

The detailed per-image and per-patient prediction exports were written to ephemeral Colab storage because of an output-path error and could not be recovered after the runtime ended. Therefore, the aggregate comparison is preserved, but the requested detailed segmentation prediction files are not available.

### Matched Segmentation Test Evaluation

The validation-selected checkpoints were evaluated on the same held-out test
split containing 4,284 images from 35 patients. All models used a fixed
foreground threshold of 0.5 and the same metric implementation. Predictions
were resized to the original ground-truth mask resolution before evaluation.

| Model | Selected epoch | Global Dice | Global foreground IoU | Mean patient Dice | Mean patient foreground IoU | Macro mIoU |
|---|---:|---:|---:|---:|---:|---:|
| Improved UNeXt | 23 | **66.27%** | **49.55%** | 56.11% | 43.21% | **74.69%** |
| FCN-ResNet50 | 10 | 59.61% | 42.46% | 46.43% | 34.67% | 71.13% |
| U-Net first trial | 3 | 59.31% | 42.15% | **58.07%** | **45.77%** | 70.99% |

Improved UNeXt achieved the highest globally pooled foreground overlap, while
U-Net achieved the highest mean patient-level Dice and IoU. This difference
shows that model ranking depends on aggregation: global metrics give greater
weight to patients and slices containing more tumor pixels, whereas the mean
patient metrics give every patient equal weight.

Macro mIoU includes the background class and is therefore higher than
foreground IoU. It should not be interpreted as tumor IoU.

## Main Conclusions

- Patient-level dataset auditing is essential: the released R9 split contains
  directly observable train-test leakage.
- The reconstructed LG-CAFN models did not match the paper-reported
  performance under the documented, corrected evaluation pipeline.
- The improved training strategy increased test AUC by approximately 0.055 for
  both channel configurations and substantially reduced the train-test accuracy
  gap.
- Validation AUC was consistently higher than test AUC, highlighting the
  uncertainty created by a validation set of only 19 patients.
- The results remain preliminary single-run findings. Patient-level bootstrap
  confidence intervals have been calculated for the main classification
  results, but repeated-seed experiments and external validation remain future work.
  
---

## Improved-Model Disclosure

The R9+ and R17+ configurations were developed after examining baseline test
behavior. Test samples were not used for gradient training, checkpoint
selection, or threshold optimization; however, the improved configurations were
indirectly informed by earlier test observations. Their results should therefore
be interpreted as exploratory rather than as evaluations on a completely
untouched final holdout set.

## Repository Contents

- [`Experiments/`](Experiments/) contains preprocessing, segmentation, and
  classification notebooks.
- [`Results/Appreviated-Results.md`](Results/Appreviated-Results.md) provides a
  concise summary of the main quantitative findings.
- [`Results/Full-Results.md`](Results/Full-Results.md) contains the detailed
  analysis and interpretation.
- [`Results/Limitations-of-Repro.md`](Results/Limitations-of-Repro.md)
  documents dataset leakage and methodological ambiguities.
- [`Results/All-tables.md`](Results/All-tables.md) preserves the supporting
  result tables and provenance records.
- [`Notes/`](Notes/) records the research and development process.

## Limitations

These experiments use one canonical split, one recorded run per configuration,
and a test set of 47 patients. The paper does not fully specify every
preprocessing, architecture, and evaluation decision, so direct numerical
comparison must be interpreted cautiously. These results do not establish
clinical usefulness or generalization to other institutions, scanners, or
patient populations.
