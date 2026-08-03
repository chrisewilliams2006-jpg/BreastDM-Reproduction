## Abbreviated Results 
these are all taken from the csv files which were created in the code. 

---
### Test-set reuse disclosure
It is important to be careful with using the same test-set. There are certain rules that I followed for myself, but for the purposes of scientific integrity, I will disclose that:

The baseline R9 and R17 models were evaluated on the canonical test set
before the improved R9+ and R17+ configurations were developed. Observations
from the baseline test evaluations, including the R17 baseline's tendency to
predict all patients as malignant at a threshold of 0.5, informed subsequent
model-development decisions such as class weighting and gradual fine-tuning.

The test samples were never used for gradient-based training, checkpoint
selection, or threshold optimization. However, because the improved
experimental design was informed by earlier results on the same test set,
R9+ and R17+ should be interpreted as exploratory rather than as performance
estimates from a completely untouched final test set.

---

| Comparison | Accuracy gap | AUC gap |
|---|---:|---:|
| R17+ vs. Paper Group 1 | −11.60 pp | −0.0978 |
| R17+ vs. Paper Group 2 | −7.33 pp | −0.0650 |

These gaps should be interpreted cautiously. The paper groups have not been assumed to map directly to R9 and R17, and the reconstructed experiments use documented preprocessing assumptions and a corrected patient-level split.

## Classification Training Summary

| Experiment | Epochs recorded | Best validation AUC | Best epoch | Final recorded training accuracy | Test AUC |
|---|---:|---:|---:|---:|---:|
| R9 baseline | 45 | 0.8810 | 25 | 98.56% | 0.7510 |
| R17 baseline | 22 | 0.9405 | 2 | 95.86% | 0.7627 |
| R9+ | 31 | 0.9881 | 11 | 93.92% | 0.8059 |
| R17+ | 39 | 0.9762 | 19 | 98.14% | 0.8176 |

The small 19-patient validation set produced substantially higher AUC estimates than the 47-patient test set. This validation-to-test gap is consistent with checkpoint-selection variance and overfitting. It is also why further hyperparameter tuning should not use the test set.

## Segmentation Training Summaries

The supplied segmentation CSVs contain training and validation histories but do not contain test-mask predictions or a common final test-summary file. The table therefore reports only values that can be verified from those histories.

| Experiment | Epochs recorded | Best validation Dice | Epoch of best Dice | Additional best validation metric |
|---|---:|---:|---:|---|
| U-Net first trial | 8 completed | **58.46%** | 3 | Recoverable `best_unet.pth`; run interrupted during epoch 9 |
| Legacy U-Net history | 14 | **67.57%** | 10 | Tumor IoU 51.02%; mIoU 75.45%; corresponding checkpoint unavailable/unconfirmed |
| FCN run | 43 | 62.73% | 23 | IoU 52.59% |
| UNeXt run | 18 | 54.55% | 3 | Best recorded IoU 45.05% at epoch 11 |

The UNeXt experiment was previously summarized with a reproduced test Dice of **58.84%**, compared with the paper's reported **70.10%**. That test value comes from the executed Colab experiment record rather than a test-prediction CSV in this bundle.

### Paper-Reported FCN Results

| Model | DSC | Reported mIoU | PPV |
|---|---:|---:|---:|
| FCN-50 | 71.10% | 77.70% | 77.00% |
| FCN-101 | 71.40% | 79.00% | 76.20% |

The paper does not fully disclose the loss function, threshold, normalization, random seed, or checkpoint rule for these FCN experiments. The reconstruction notebook makes those choices explicit, so any later test comparison should be described as a reproduction under documented assumptions rather than an exact rerun.

> [!NOTE]
> Dice and IoU values are only directly comparable when they use the same class definition, sample set, empty-mask policy, and aggregation method. Foreground Dice, foreground IoU, and macro mIoU should not be treated as interchangeable.

## Main Findings

1. **Patient-level splitting materially improves experimental validity.** The audit found 43 R9 files from six training patients physically located in the released test directory.
2. **The 17-channel baseline was poorly calibrated at the default threshold.** It ranked patients above chance (`AUC = 0.7627`) but classified every test patient as malignant.
3. **The improved training strategy corrected class collapse.** R17+ raised balanced accuracy from 50.00% to 77.84% and specificity from 0.00% to 82.35%.
4. **R17+ is the strongest general-purpose classification result.** It achieved the highest accuracy (`76.60%`) and AUC (`0.8176`).
5. **R9+ is the most conservative classifier.** It achieved the highest specificity (`94.12%`) and balanced accuracy (`78.73%`) at threshold `0.5`.
6. **Validation estimates are optimistic.** All classification runs reached much higher validation AUC than test AUC, emphasizing the need for repeated seeds and confidence intervals.
7. **The improved models remain below the paper-reported results.** This is a useful negative reproduction result, particularly because the reconstruction uses explicit assumptions and leakage-corrected evaluation.

## Recommended Interpretation

The results support the following conclusion:

> Under a documented, patient-level evaluation pipeline, the LG-CAFN reconstructions did not match the paper-reported performance. Class weighting, differential learning rates, and gradual fine-tuning produced meaningful improvements, corrected the R17 baseline's all-malignant predictions, and increased test AUC by approximately 0.055 for both input branches. R17+ offered the best overall discrimination, while R9+ offered the strongest specificity.

These experiments should be considered preliminary because they use one canonical split, one recorded run per configuration, and a relatively small test set.

## Limitations

- The test set contains only 47 patients.
- The current tables describe single recorded runs rather than multi-seed averages.
- Confidence intervals have not yet been reported.
- The original paper does not fully specify every preprocessing and evaluation decision.
- Paper groups cannot yet be mapped confidently to the R9 and R17 reconstructions.
- Threshold selection can change sensitivity and specificity substantially; it does not change AUC.
- Medical performance claims require external validation and should not be inferred from this reproduction alone.


## Result Provenance

| Evidence type | Files |
|---|---|
| Canonical patient split | `canonical_patient_split.csv` |
| Leakage audit | `excluded_files.csv` |
| R9/R17 manifests | `img9Se_*.csv`, `img17Se_*.csv` |
| Classification summaries | `LG_CAFN_R9_R17_comparison.csv`, `LG_CAFN_author_comparison.csv`, `baseline_improved_paper_comparison.csv` |
| Patient-level predictions | `test_patient_predictions*.csv`, `validation_patient_predictions*.csv` |
| ROI-level predictions | `test_roi_predictions*.csv`, `validation_roi_predictions*.csv` |
| Classification histories | `history*.csv` |
| Segmentation histories | `training_history*.csv` |

## Reproducibility Principles

- Split by patient before model training.
- Never select hyperparameters or thresholds using the test set.
- Aggregate correlated ROI predictions to the patient level.
- Report accuracy together with AUC, balanced accuracy, sensitivity, and specificity.
- Preserve baseline experiments before introducing improvements.
- Record the seed, preprocessing, checkpoint rule, and metric definitions for every run.
- Label results by provenance: `paper-reported`, `legacy Colab result`, `reconstruction`, or `verified with the reusable pipeline`.


