## Full results (not abbreviated)

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

Compared with R9+, R17+ correctly detects three additional malignant patients but incorrectly labels two additional benign patients as malignant. Its slightly lower balanced accuracy reflects this trade-off, while its higher AUC indicates the best threshold-independent ranking.

### Improvement Over Each Reconstruction Baseline

The following table compares each improved model with its corresponding
baseline using patient-level test results at a fixed threshold of `0.5`.

| Comparison | Accuracy change | Balanced-accuracy change | Sensitivity change | Specificity change | AUC change |
|---|---:|---:|---:|---:|---:|
| R9+ vs. R9 baseline | +10.64 pp | +13.43 pp | +3.33 pp | +23.53 pp | +0.0549 |
| R17+ vs. R17 baseline | +12.77 pp | +27.84 pp | −26.67 pp | +82.35 pp | +0.0549 |

R9+ improved every reported classification metric relative to the R9
baseline. Its largest improvement was in specificity, which increased from
70.59% to 94.12%.

R17+ corrected the R17 baseline's all-malignant classification behavior.
Specificity increased from 0.00% to 82.35%, while balanced accuracy increased
from 50.00% to 77.84%. Sensitivity decreased from 100.00% to 73.33% because
the baseline's perfect sensitivity resulted from predicting every patient as
malignant, not from meaningful class separation.

The validation-selected threshold changes the operating point but not AUC, because AUC evaluates ranking across thresholds. For R17+, the Youden threshold may be useful when false negatives are considered substantially more costly, but the fixed threshold generalized better as a balanced default.

## Patient-Level Bootstrap Confidence Intervals

The main fixed-threshold classification metrics were evaluated using 10,000
stratified patient-level bootstrap samples. Each replicate sampled 17 benign
and 30 malignant patients with replacement. The bootstrap used seed `20260805`
and percentile 95% confidence intervals.

| Experiment | Accuracy (95% CI) | Balanced Accuracy (95% CI) | Sensitivity (95% CI) | Specificity (95% CI) | ROC AUC (95% CI) |
|---|---:|---:|---:|---:|---:|
| LG-CAFN-R9 baseline | 63.83% (51.06–76.60) | 65.29% (51.47–79.12) | 60.00% (43.33–76.67) | 70.59% (47.06–88.24) | 0.7510 (0.5745–0.9000) |
| LG-CAFN-R17 baseline | 63.83% (63.83–63.83) | 50.00% (50.00–50.00) | 100.00% (100.00–100.00) | 0.00% (0.00–0.00) | 0.7627 (0.6157–0.8882) |
| LG-CAFN-R9+ | 74.47% (61.70–85.11) | 78.73% (67.84–88.33) | 63.33% (46.67–80.00) | 94.12% (82.35–100.00) | 0.8059 (0.6686–0.9176) |
| LG-CAFN-R17+ | 76.60% (63.83–87.23) | 77.84% (65.29–89.12) | 73.33% (56.67–90.00) | 82.35% (64.71–100.00) | 0.8176 (0.6784–0.9373) |

### Exact Validation-Selected Thresholds

| Experiment | Validation-selected threshold | Validation sensitivity | Validation specificity | Validation Youden J |
|---|---:|---:|---:|---:|
| R9+ | 0.3200 | 91.67% | 100.00% | 0.9167 |
| R17+ | 0.1204 | 100.00% | 85.71% | 0.8571 |

Both selected thresholds are below `0.5`, especially for R17+. This explains the increase in test sensitivity and decrease in specificity. The validation set contains only seven benign and 12 malignant patients, so a threshold can appear highly effective after being selected from a very small number of cases and then generalize less favorably.

## Validation-to-Test Generalization

| Experiment | Validation AUC | Test AUC | AUC decrease | Validation accuracy at 0.5 | Test accuracy at 0.5 |
|---|---:|---:|---:|---:|---:|
| R9 baseline | 0.8810 | 0.7510 | −0.1300 | 73.68% | 63.83% |
| R17 baseline | 0.9405 | 0.7627 | −0.1777 | 63.16% | 63.83% |
| R9+ | 0.9881 | 0.8059 | −0.1822 | 84.21% | 74.47% |
| R17+ | 0.9762 | 0.8176 | −0.1585 | 89.47% | 76.60% |

Every experiment performs worse on test AUC than on validation AUC. The decrease ranges from `0.1300` to `0.1822`. This is a central result rather than a minor detail: with only 19 validation patients, checkpoint selection and threshold selection can fit idiosyncrasies of that small group.

R17 baseline validation accuracy does not reveal its strong validation AUC because it predicts every validation patient malignant at `0.5`, just as it does on the test set. This further demonstrates that threshold-independent ranking and threshold-dependent classification answer different questions.

## Patient Probability Analysis

### Test Probability Distributions

| Experiment | True class | Mean malignancy probability | Median | Minimum | Maximum |
|---|---|---:|---:|---:|---:|
| R9 baseline | Benign | 0.3099 | 0.1043 | 0.0131 | 0.9265 |
| R9 baseline | Malignant | 0.5836 | 0.6116 | 0.0805 | 0.9959 |
| R17 baseline | Benign | 0.9998 | 0.9999 | 0.9983 | 0.9999 |
| R17 baseline | Malignant | 0.9999 | 0.9999 | 0.9996 | 1.0000 |
| R9+ | Benign | 0.1755 | 0.1028 | 0.0078 | 0.6891 |
| R9+ | Malignant | 0.5270 | 0.6182 | 0.0292 | 0.9957 |
| R17+ | Benign | 0.2206 | 0.0581 | 0.0003 | 0.9508 |
| R17+ | Malignant | 0.6692 | 0.7656 | 0.0058 | 0.9999 |

R17+ has the largest separation between the class means: `0.6692 − 0.2206 = 0.4486`. R9+ has a mean separation of `0.3515`, while the baseline R9 separation is `0.2737`. The R17 baseline class means are nearly identical because its outputs are saturated near one.

The wide minimum-to-maximum ranges also show substantial overlap. Some malignant patients receive very low probabilities, and a small number of benign patients receive very high probabilities. Therefore, no single threshold perfectly separates the test classes.

### R17+ Misclassified Patients at Threshold 0.5

| Patient | True class | Malignancy probability | ROI count | Error type |
|---|---|---:|---:|---|
| `BreaDM-Be-2102` | Benign | 0.9508 | 9 | False positive |
| `BreaDM-Be-1826` | Benign | 0.8584 | 2 | False positive |
| `BreaDM-Be-1810` | Benign | 0.8218 | 2 | False positive |
| `BreaDM-Ma-2126` | Malignant | 0.4490 | 7 | False negative |
| `BreaDM-Ma-1915` | Malignant | 0.3095 | 16 | False negative |
| `BreaDM-Ma-1921` | Malignant | 0.2049 | 5 | False negative |
| `BreaDM-Ma-2038` | Malignant | 0.1850 | 2 | False negative |
| `BreaDM-Ma-2122` | Malignant | 0.1233 | 5 | False negative |
| `BreaDM-Ma-2141` | Malignant | 0.0338 | 5 | False negative |
| `BreaDM-Ma-1909` | Malignant | 0.0172 | 3 | False negative |
| `BreaDM-Ma-2109` | Malignant | 0.0058 | 5 | False negative |

Several errors are confident rather than borderline. For example, three benign patients receive probabilities above `0.82`, and three malignant patients receive probabilities below `0.04`. These cases are useful candidates for qualitative review of masks, lesion appearance, channel construction, and possible label or acquisition differences.

The number of ROIs alone does not explain the errors. Misclassified patients range from two to 16 ROIs. Patient `BreaDM-Ma-1915` remains a false negative despite having 16 ROIs, while some two-ROI patients are confidently misclassified in both directions.

### Comparison With Paper-Reported Results

| Comparison | Accuracy gap | AUC gap |
|---|---:|---:|
| R17+ vs. Paper Group 1 | −11.60 pp | −0.0978 |
| R17+ vs. Paper Group 2 | −7.33 pp | −0.0650 |

R17+ remained below both paper-reported LG-CAFN results. These comparisons
should be interpreted cautiously because the paper's two experimental groups
cannot be mapped confidently to the R9 and R17 reconstructions. The
reconstructed experiments also use a corrected patient-level split and
documented preprocessing decisions that were not fully specified in the paper.

Because R17+ was developed after the baseline test results had been observed,
this comparison is exploratory and does not constitute an independent
confirmatory evaluation.

The small 19-patient validation set produced substantially higher AUC estimates than the 47-patient test set. This validation-to-test gap is consistent with checkpoint-selection variance and overfitting. It is also why further hyperparameter tuning should not use the test set.

### Training Behavior

| Experiment | Best validation epoch | Recorded stopping epoch | Epochs after best checkpoint | Interpretation |
|---|---:|---:|---:|---|
| R9 baseline | 25 | 45 | 20 | Validation performance peaked well before training ended |
| R17 baseline | 2 | 22 | 20 | Very early peak suggests rapid overfitting or unstable calibration |
| R9+ | 11 | 31 | 20 | Early stopping preserved the strongest validation checkpoint |
| R17+ | 19 | 39 | 20 | Later peak than R17 baseline, consistent with more controlled fine-tuning |

All runs contain 20 recorded epochs after the best validation-AUC epoch, which is consistent with an early-stopping patience of 20 epochs. The selected checkpoint—not the last training epoch—should be used for final evaluation.

The high final training accuracies do not imply high generalization:

| Experiment | Final training accuracy | Test accuracy | Train–test gap |
|---|---:|---:|---:|
| R9 baseline | 98.56% | 63.83% | 34.73 pp |
| R17 baseline | 95.86% | 63.83% | 32.03 pp |
| R9+ | 93.92% | 74.47% | 19.45 pp |
| R17+ | 98.14% | 76.60% | 21.54 pp |

The improved runs reduce the train–test accuracy gap relative to the baselines, although R17+ still fits the training set very strongly. Because training accuracy is calculated on correlated ROIs and test accuracy is calculated on patients, this gap is descriptive rather than a perfectly like-for-like comparison.

## Segmentation Training Summaries

The supplied segmentation CSVs contain training and validation histories but do not contain test-mask predictions or a common final test-summary file. The table therefore reports only values that can be verified from those histories.
> [!NOTE]
> Dice and IoU values are only directly comparable when they use the same class definition, sample set, empty-mask policy, and aggregation method. Foreground Dice, foreground IoU, and macro mIoU should not be treated as interchangeable.

### Understanding the Segmentation Metrics

| Metric | Question answered | Important limitation |
|---|---|---|
| Dice similarity coefficient | How strongly do predicted and true tumor regions overlap? | Can vary depending on empty-mask handling and per-image vs. global aggregation |
| Tumor IoU | What fraction of the union of predicted and true tumor pixels is correctly overlapped? | Numerically lower than Dice for the same predictions |
| Mean IoU | What is the mean overlap across included classes? | May be inflated by the dominant background class |
| Precision / PPV | Of all pixels predicted as tumor, how many are truly tumor? | Does not measure missed tumor pixels |
| Sensitivity / recall | Of all true tumor pixels, how many were detected? | Does not measure false-positive regions |

For the same binary foreground predictions, Dice and foreground IoU are related by:

```text
Dice = 2 × IoU / (1 + IoU)
IoU  = Dice / (2 − Dice)
```

This relationship does not necessarily hold between foreground Dice and a separately averaged macro mIoU value. Metric definitions must therefore be documented before comparing tables from different implementations.

### Segmentation Generalization Caveat

The best validation metrics identify promising checkpoints but are not substitutes for final test metrics. The segmentation histories use different column schemas, suggesting that the runs did not all record exactly the same definitions. A fair final table should rerun every saved checkpoint through one shared evaluator with:

- the same test patients and masks;
- the same image resolution;
- the same probability threshold;
- identical empty-mask handling;
- foreground Dice and foreground IoU;
- the same macro/background policy; and
- bootstrap confidence intervals at the patient level.

## Main Findings

1. **The released `cls/img9Se` split contains patient-level leakage.**
   Forty-three files belonging to six malignant training patients also appear
   in the released test directory. The corrected experiments exclude these
   noncanonical test assignments.

2. **The R17 baseline collapsed at the default threshold.**
   Although it achieved a test AUC of 0.7627, it classified all 47 test
   patients as malignant at a threshold of `0.5`. Its 63.83% accuracy therefore
   matched the malignant prevalence of the test set.

3. **The improved training strategy corrected the R17 class collapse.**
   R17+ increased specificity from 0.00% to 82.35% and balanced accuracy from
   50.00% to 77.84%.

4. **R17+ achieved the strongest overall discrimination.**
   Among the reconstructed classification experiments, R17+ achieved the
   highest patient-level test accuracy at 76.60% and the highest ROC AUC at
   0.8176.

5. **R9+ produced the strongest benign-class performance.**
   At a threshold of `0.5`, R9+ achieved the highest specificity at 94.12% and
   the highest balanced accuracy at 78.73%.

6. **Validation performance was consistently optimistic.**
   Test AUC was lower than validation AUC in all four classification
   experiments. The decreases ranged from 0.1300 to 0.1822, which is
   consistent with uncertainty from the 19-patient validation set and
   single-run checkpoint selection.

7. **The improved models did not reach the paper-reported performance.**
   This is a useful negative reproduction result, but the comparison is not
   exact because the original preprocessing and evaluation procedures are
   incompletely documented.

8. **The improved-model results are exploratory.**
   R9+ and R17+ were developed after baseline results on the same test split
   had been observed. Test images were not used for gradient-based training,
   checkpoint selection, or threshold optimization, but the improved
   experimental design was test-informed.

These findings are preliminary because they come from one canonical split,
one recorded run per configuration, and a test set of only 47 patients.
## What the Results Do and Do Not Show

### Supported by the supplied data

- The canonical R9 and R17 manifests contain the same 232 patients and 1,722 ROIs.
- Forty-three R9 files from six training patients were physically located in the released test directory.
- R9+ and R17+ outperform their corresponding reconstruction baselines in accuracy, balanced accuracy, and AUC.
- R17 baseline collapses to all-malignant predictions at threshold `0.5`.
- R17+ has the strongest test AUC and accuracy among the reconstructed classification experiments.
- R9+ has the strongest test specificity and balanced accuracy at threshold `0.5`.
- Validation performance is consistently more optimistic than test performance.
- The validation-derived thresholds increase sensitivity at the cost of specificity on the test set.

### Not established by the current data

- That any reconstructed model is clinically useful.
- That the improved configuration will outperform the baselines across random seeds or alternative patient splits.
- That R17 is universally superior to R9.
- That the source paper's results are wrong; the evaluation protocols are not directly equivalent.
- That the segmentation models can be ranked fairly from validation histories alone.
- That the reported probabilities are calibrated estimates of real-world malignancy risk.
- That performance will transfer to another institution, scanner, acquisition protocol, or patient population.

## Limitations

- The classification test set contains only 47 patients: 17 benign and 30
  malignant.
- The validation set contains only 19 patients, making checkpoint and
  threshold selection sensitive to a small number of cases.
- Each configuration is represented by one recorded training run rather than
  a multi-seed average.
- Confidence intervals have not yet been calculated.
- The experiments use one canonical patient split, so performance may depend
  on the particular patient assignment.
- The original paper and released code do not fully specify the conversion
  from 9- and 17-channel MRI arrays to the three-channel model input.
- Several architectural, preprocessing, and patient-aggregation decisions had
  to be reconstructed.
- The paper's two LG-CAFN experimental groups cannot be mapped confidently to
  the R9 and R17 reconstructions.
- R9+ and R17+ were developed after baseline test results had been observed,
  so their test results are exploratory rather than confirmatory.
- The segmentation histories do not include a shared final test-evaluation
  file using identical metric definitions.
- The results have not been validated using data from another institution,
  scanner, acquisition protocol, or patient population.
- These experiments do not establish clinical usefulness.

## Proposed Statistical Reporting

Future result tables should report uncertainty rather than only point estimates.

| Metric | Recommended uncertainty method | Sampling unit |
|---|---|---|
| AUC | Stratified patient bootstrap | Patient |
| Accuracy | Patient bootstrap or exact binomial interval | Patient |
| Sensitivity | Exact binomial or stratified bootstrap | Malignant patient |
| Specificity | Exact binomial or stratified bootstrap | Benign patient |
| Balanced accuracy | Stratified patient bootstrap | Patient |
| Dice / IoU | Patient-level bootstrap after aggregating slices | Patient |

The patient—not the ROI or image slice—should be the resampling unit. Resampling ROIs independently would violate the independence assumption and produce confidence intervals that are too narrow.

For multi-seed experiments, report both sources of variability:

```text
Metric across seeds: mean ± standard deviation
Patient uncertainty: 95% bootstrap confidence interval
```
