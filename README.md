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

Class weighting, differential learning rates, and gradual fine-tuning improved
both reconstructed models. R17+ produced the highest accuracy and ROC AUC,
while R9+ produced the highest balanced accuracy and specificity. The improved
R17 model also corrected the baseline model's collapse to predicting every test
patient as malignant.

For context, the paper reports accuracies of 88.20% and 83.93% with ROC AUC
values of 0.9154 and 0.8826. These values are not directly equivalent to the
reconstructed results because the paper groups cannot be mapped confidently to
R9 and R17, and this project uses documented reconstruction assumptions and a
leakage-corrected patient split.

## Segmentation Results

The available segmentation histories support the following validation results:

| Experiment | Best validation Dice | Best-Dice epoch | Additional validation result |
|---|---:|---:|---|
| U-Net | **67.57%** | 10 | Tumor IoU 51.02%; mIoU 75.45% |
| FCN | 62.73% | 23 | IoU 52.59% |
| UNeXt | 54.55% | 3 | Best recorded IoU 45.05% at epoch 11 |

An executed Colab experiment previously recorded a UNeXt test Dice of 58.84%,
compared with the paper-reported 70.10%. That result is retained as a legacy
Colab result because the current result bundle does not contain the associated
test-prediction CSV. The segmentation models should not be ranked conclusively
until all saved checkpoints are evaluated with one shared test evaluator and
identical metric definitions.

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
- The results are preliminary single-run findings and should be followed by
  repeated seeds, patient-level confidence intervals, and external validation.

## Repository Contents

- [`Experiments/`](Experiments/) contains preprocessing, segmentation, and
  classification notebooks.
- [`Results/Appreviated-Results.md`](Results/Appreviated-Results.md) provides a
  concise summary of the main quantitative findings.
- [`Results/Full-Results.md`](Results/Full-Results.md) contains the detailed
  analysis and interpretation.
- [`Results/Reproducibility+model_issues.md`](Results/Reproducibility+model_issues.md)
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
