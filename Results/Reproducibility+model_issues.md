# BreastDM Paper and Reproduction Issues

## Overview

This repository documents an independent attempt to reproduce parts of the BreastDM study:

> Zhao, X. et al. “BreastDM: A DCE-MRI Dataset for Breast Tumor Image Segmentation and Classification.” *Computers in Biology and Medicine*, Volume 164, 2023, Article 107255.
(copy and paste)

During the reproduction, two separate problems were identified:

1. **The segmentation evaluation is under-specified and contains metric results that require clarification.**
2. **The released LG-CAFN classification archive contains patient-level train/test leakage in its raw 9-sequence branch.**

The segmentation problem prevents the exact published scores from being interpreted or reproduced confidently without additional information from the authors.

The classification problem is a directly observable error in the released dataset structure. Six malignant patients assigned to training also appear in the raw `img9Se` test directory. This creates patient-level data leakage unless those noncanonical test copies are removed or excluded.

These findings do not prove that the authors fabricated their results. They also do not establish that every experiment in the paper is invalid or that the BreastDM dataset is unusable. They do show that the published paper and released files cannot be treated as a completely specified, immediately reproducible benchmark.

---

# Part I: Segmentation Evaluation Problems

## 1. Dice and IoU

Dice and Intersection over Union, or IoU, both measure overlap between a predicted tumor mask and its ground-truth mask.

For a binary tumor mask:

- `TP` is the number of tumor pixels correctly predicted as tumor.
- `FP` is the number of background pixels incorrectly predicted as tumor.
- `FN` is the number of tumor pixels missed by the model.

Foreground Dice and foreground IoU are calculated as:

```text
Dice = 2TP / (2TP + FP + FN)

IoU = TP / (TP + FP + FN)
```

When Dice and IoU are calculated from the same binary prediction and ground-truth mask, they have an exact mathematical relationship:

```text
Dice = (2 × IoU) / (1 + IoU)

IoU = Dice / (2 − Dice)
```

This is not merely an approximate relationship or general expectation. It is an algebraic identity.

Therefore, when the two metrics use the same predictions and evaluation procedure, Dice must always be greater than or equal to IoU. Except when both values are exactly `0` or exactly `1`, Dice is strictly greater than IoU.

Examples include:

| IoU | Corresponding Dice |
|---:|---:|
| 20% | 33.3% |
| 40% | 57.1% |
| 60% | 75.0% |
| 80% | 88.9% |
| 100% | 100% |

This mathematical relationship applies only when Dice and IoU use the same:

- predicted masks;
- ground-truth masks;
- test samples;
- foreground or class definition;
- probability threshold;
- handling of empty masks;
- smoothing convention; and
- aggregation method.

If any of these conditions differ, the reported values may no longer form a directly matched Dice/IoU pair.

## 2. The Warning in the Paper’s Results

The BreastDM paper reports UNeXt with:

- **70.1% Dice**
- **78.5% mIoU**

If `78.5%` represents foreground IoU calculated from the same masks and aggregation as the Dice score, its corresponding Dice would be approximately `88.0%`:

```text
Dice = (2 × 0.785) / (1 + 0.785)
     = 0.8796
     = approximately 88.0%
```

Conversely, a Dice score of `70.1%` corresponds to a foreground IoU of approximately `54.0%`, not `78.5%`:

```text
IoU = 0.701 / (2 − 0.701)
    = approximately 0.540
```

Similar relationships appear in other reported results.

| Model | Paper-reported Dice | Paper-reported mIoU |
|---|---:|---:|
| UNeXt | 70.1% | 78.5% |
| Random Forest | 66.2% | 74.5% |
| FCN-50 | 71.1% | 77.7% |
| FCN-101 | 71.4% | 79.0% |

If these mIoU values were intended to be foreground IoU values calculated from the same predictions and aggregation as Dice, the pairs would be mathematically incompatible.

At least one of the following would then have to be true:

- The Dice and IoU labels were reversed.
- One or more values were copied incorrectly.
- One of the metric implementations contained an error.
- Dice and IoU used different prediction thresholds.
- The metrics came from different model checkpoints.
- The metrics were calculated using different test samples.
- One metric used probabilities while the other used binary masks.
- Empty masks were handled differently.
- Dice and IoU used different aggregation procedures.
- The metrics did not actually describe the same class.

This does not mean that a model somehow performed unusually well on IoU but poorly on Dice. That interpretation would be mathematically impossible for matched foreground Dice and foreground IoU.

## 3. The Important mIoU Complication

The paper labels the second metric as **mIoU**, not necessarily foreground IoU.

This distinction matters.

If Dice describes only the tumor class while mIoU is the average of tumor IoU and background IoU, mIoU can legitimately be higher than tumor Dice.

For example:

```text
Tumor IoU:      50.0%
Background IoU: 95.0%

mIoU = (50.0% + 95.0%) / 2
     = 72.5%

Tumor Dice:     66.7%
```

In this example, mIoU is higher than tumor Dice because mIoU includes the easily classified background, while Dice measures only tumor overlap.

The large background region can raise macro mIoU substantially.

Therefore, it would be too strong to say that every reported `mIoU > Dice` pair is automatically mathematically impossible. The correct conclusion is:

- If the reported mIoU is actually matched foreground IoU, the result is mathematically inconsistent.
- If mIoU averages tumor and background IoU while Dice is tumor-only, the result may be possible.
- The paper does not explain the class definitions and aggregation clearly enough to determine which interpretation produced the table.

Even if the second interpretation is correct, foreground Dice and background-inclusive mIoU measure different things. Presenting them together without clearly defining the classes and averaging procedure makes the results difficult to interpret and reproduce.

## 4. The Dice Aggregation Ambiguity

The paper provides the standard Dice definition for an individual prediction. However, defining Dice for one mask is not the same as defining how thousands of predictions are combined into one dataset-level result.

The BreastDM segmentation data contains many 2D MRI slices from multiple patients. The paper does not clearly state how those slices were reduced to one reported Dice score.

Several valid approaches are possible.

### Mean Per-Slice Dice

Dice is calculated independently for every 2D slice. The individual slice scores are then averaged.

This gives every slice equal influence, regardless of tumor size. A slice containing a tiny tumor has the same influence as a slice containing a large tumor.

This method can also be strongly affected by slices containing no annotated tumor.

### Global Dice

True-positive, false-positive, and false-negative pixels are pooled over the complete test set before calculating one Dice score.

This gives greater influence to slices and patients with larger tumors because they contribute more foreground pixels.

### Mean Per-Patient Dice

All slices from one patient are combined into a 3D volume. Dice is calculated separately for every patient, and patient scores are averaged.

This gives every patient equal influence. It may also be more clinically meaningful than treating every correlated 2D slice as an independent observation.

### Mean Per-Batch Dice

Intersection and union are pooled within each evaluation mini-batch. One Dice score is calculated for each batch, and the batch scores are averaged.

This can make the final result depend on:

- evaluation batch size;
- sample ordering;
- which images appear together;
- whether slices from the same patient appear in the same batch; and
- the size of the final incomplete batch.

These approaches are not mathematically equivalent.

Exactly the same predictions can produce different final scores depending on the chosen aggregation method. Therefore, the paper’s exact Dice values cannot be reproduced unless the evaluation unit and aggregation procedure are specified.

## 5. Empty-Mask Cases

Some MRI slices may contain no annotated tumor. These slices require an explicit evaluation policy.

There are four important situations:

1. The ground-truth mask is empty and the prediction is empty.
2. The ground-truth mask is empty but the prediction contains a false-positive tumor.
3. The ground truth contains a tumor but the prediction is empty.
4. Both masks contain tumor pixels.

When the prediction and ground truth are both empty, the unsmoothed Dice formula becomes:

```text
0 / 0
```

The result is undefined.

A study must decide how to handle this case. Common policies include:

- treating the empty prediction as perfectly correct and assigning Dice `1`;
- excluding empty/empty slices from the average;
- assigning the slice Dice `0`;
- using a smoothing constant;
- or reporting tumor-bearing and tumor-free slices separately.

These choices can substantially change mean per-slice Dice when the test set contains many tumor-free slices.

A smoothing value can also introduce a hidden empty-mask policy by turning an undefined calculation into a finite result.

The BreastDM manuscript does not clearly state:

- whether tumor-free slices were included;
- how empty ground truths were handled;
- how empty predictions were handled;
- or how smoothing affected empty cases.

## 6. What the Released UNeXt Code Appears to Do

The released UNeXt `metrics.py` provides some information about one evaluation implementation.

The implementation appears to:

1. Apply sigmoid to the model output.
2. Threshold predictions at `0.5`.
3. Pool intersection and union over every image in the current mini-batch.
4. Calculate one foreground IoU value for the batch.
5. Calculate Dice directly from that IoU.
6. Add the batch result to an average weighted by the number of images.

The essential calculation is:

```python
intersection = (output_ & target_).sum()
union = (output_ | target_).sum()

iou = (intersection + smooth) / (union + smooth)
dice = (2 * iou) / (iou + 1)
```

This implementation has two important consequences.

First, Dice must be greater than or equal to the paired IoU within every batch because Dice is calculated directly from IoU.

Second, the final result is batch-dependent. It is not necessarily equal to:

- averaging Dice independently across all slices;
- calculating one global Dice over the complete test set;
- or averaging Dice across patients.

The released validation loop appears to average batch results while weighting them by the number of images. However, weighting by the number of images does not make this equivalent to one global pixel-level calculation because the nonlinear metric has already been calculated separately inside each batch.

The manuscript does not clearly disclose this batch-based aggregation method.

The released code also does not establish that:

- this exact implementation produced the paper’s segmentation table;
- every segmentation baseline used this evaluator;
- every baseline used the same batch size;
- every baseline used the same sample order;
- or the reported mIoU was produced by this foreground-only function.

The code therefore provides one plausible evaluation implementation but does not fully resolve the published table.

## 7. Why the Segmentation Results Cannot Be Reproduced Exactly

A reproducible experiment must provide enough information for another researcher to calculate the same result from the same predictions.

The BreastDM segmentation evaluation leaves several details missing or uncertain:

- whether evaluation was performed per slice, patient, volume, batch, or complete test set;
- how Dice was aggregated;
- how IoU or mIoU was aggregated;
- whether Dice and mIoU used the same predictions;
- whether Dice and mIoU used the same test samples;
- which classes were included in mIoU;
- whether Dice was foreground-only;
- whether tumor-free slices were included;
- how empty masks were scored;
- the smoothing value;
- the exact patient-level training, validation, and test split;
- the evaluation batch size;
- the order of evaluation samples;
- the prediction threshold;
- model normalization;
- the loss function;
- the random seed;
- the checkpoint-selection rule;
- and whether every baseline used the same evaluation code.


Exact reproduction requires clarification from the authors or a complete evaluation script that can be directly connected to the results table.

## 8. Why the Segmentation Ranking Is Not Secure

A claim that one model outperforms another requires all models to be evaluated under matched conditions.

The available materials do not establish that every model used:

- the same patient-level split;
- the same training and test images;
- the same preprocessing;
- the same resolution;
- the same threshold;
- the same empty-mask policy;
- the same metric definitions;
- the same aggregation procedure;
- the same checkpoint-selection rule;
- or the same evaluation implementation.

If aggregation and evaluation were not standardized across the experiment, the reported models cannot be compared empirically with high confidence.

This issue also applies to the paper’s proposed model. Without a shared and clearly documented evaluator, the table is not sufficient to demonstrate that the proposed model is genuinely superior to every baseline under matched conditions.

The severity depends on what actually happened:

- If Dice and IoU used the same masks and aggregation, IoU above Dice indicates a calculation, labeling, or transcription error.
- If the same masks were used but the metrics had different aggregation methods, the values may be possible, but presenting them without explaining the difference is misleading.
- If different thresholds, checkpoints, or test samples were used, that would represent a more serious methodological problem if the values were presented as one experiment.
- If the underlying evaluation was consistent but important details were omitted, the experiment may still be valid, but its reproducibility is weak.

The most defensible conclusion is that the segmentation evaluation contains an unresolved reporting, implementation, class-definition, or aggregation inconsistency.

## 9. What the Segmentation Finding Proves

The available evidence supports the following conclusions:

- The segmentation evaluation is under-specified.
- The reported mIoU-above-Dice pairs require an explicit explanation.
- Matched foreground IoU cannot be higher than matched foreground Dice.
- A background-inclusive macro mIoU may be higher than tumor Dice.
- The exact published segmentation scores cannot be reproduced confidently from the manuscript alone.
- The released UNeXt code uses a batch-dependent evaluation method.
- The UNeXt code derives Dice directly from IoU.
- The reported issue may involve aggregation, implementation, class definition, labeling, or transcription.
- The segmentation table should be treated cautiously until the protocol is clarified.



---

# Part II: LG-CAFN Classification Data Leakage

## 10. The Released Split Anomaly

The classification archive contains a split anomaly specifically in the raw:

```text
cls/img9Se
```

branch.

The following five branches agree on a canonical, nonoverlapping patient split:

- `cls/img17Se`;
- `cls/GLCM/img9Se`;
- `cls/GLCM/img17Se`;
- `cls/LBP/img9Se`; and
- `cls/LBP/img17Se`.

Their shared split contains:

| Split | Patients |
|---|---:|
| Training | 166 |
| Validation | 19 |
| Testing | 47 |
| **Total** | **232** |

No patient belongs to more than one split in this canonical allocation.

In contrast, the raw `cls/img9Se` branch physically contains 53 test patients instead of 47.

The six additional patients are malignant patients that are already assigned to training.

## 11. The Six Duplicated Patients

The affected patients are:

| Patient | Canonical assignment | Noncanonical test files |
|---|---|---:|
| `BreaDM-Ma-1802` | Train | 13 |
| `BreaDM-Ma-1803` | Train | 9 |
| `BreaDM-Ma-1804` | Train | 4 |
| `BreaDM-Ma-1806` | Train | 5 |
| `BreaDM-Ma-1807` | Train | 8 |
| `BreaDM-Ma-1808` | Train | 4 |
| **Total** | **Train** | **43** |

Their `.npy` files occur underneath both:

```text
cls/img9Se/train/Malignant/<patient-name>/
```

and:

```text
cls/img9Se/test/Malignant/<patient-name>/
```

This is not merely an unusual directory arrangement. It creates patient-level train/test leakage if the physical folder structure is used without correction.

## 12. Why This Is Patient-Level Leakage

MRI samples belonging to the same patient are correlated.

ROIs or slices from one patient may share:

- anatomy;
- lesion appearance;
- enhancement behavior;
- acquisition characteristics;
- scanner artifacts;
- patient-specific preprocessing signatures;
- and other characteristics unrelated to general disease classification.

If a model sees samples from a patient during training and then sees other samples from the same patient during testing, the test patient is not genuinely unseen.

The model may learn patient-specific information rather than generalizable differences between benign and malignant tumors.

This can artificially increase:

- accuracy;
- AUC;
- sensitivity;
- specificity;
- balanced accuracy;
- and other test metrics.

Because all six affected patients are malignant, the leak also introduces class-specific distortion rather than a neutral duplication across both classes.

The exact amount of inflation cannot be calculated without evaluating the same original trained checkpoint with and without the leaked test samples. However, any evaluation that includes these patients in both training and testing should not be treated as a valid estimate of unseen-patient generalization.

## 13. Possible Causes of the Archive Error

The released files do not reveal the exact cause of the anomaly.

Possible explanations include:

- an archive-packaging mistake;
- accidental copying of patient folders;
- an incomplete update to the raw `img9Se` branch;
- stale files left behind when the split was revised;
- or one branch being generated from an older version of the split.

The consistent 166/19/47 allocation in the other five branches strongly supports treating that allocation as canonical.

However, the released materials do not prove which packaging event caused the six patients to appear in the R9 test directory.

## 14. What Was Done to Correct the Classification Data

The reproduction made the smallest defensible correction while preserving an audit trail.

### 14.1 The Consistent Split Was Treated as Canonical

The 166/19/47 patient allocation shared by the unaffected branches was treated as the canonical split.

The six duplicated malignant patients remain assigned to training.

They were not reassigned to testing because doing so would conflict with the split consistently used by the other branches.

### 14.2 Only the Noncanonical Test Copies Were Excluded

The 43 files belonging to the six training patients were excluded from:

```text
cls/img9Se/test/Malignant/
```

Their legitimate training copies were retained.

The correction therefore removes the contaminated test assignments without removing valid training data.

### 14.3 The Original Archive Was Preserved

The source archive was not silently rewritten.

Instead, the correction was represented through explicit manifests and exclusion records. This preserves the evidence and allows another researcher to see exactly:

- which files were excluded;
- why they were excluded;
- which patients were affected;
- and what their canonical split should be.

### 14.4 Explicit Manifests Were Created

The corrected data inventory is recorded through:

- `canonical_patient_split.csv`;
- `excluded_files.csv`;
- `img9Se` manifests; and
- `img17Se` manifests.

`canonical_patient_split.csv` records one split assignment for each patient.

`excluded_files.csv` lists all 43 rejected paths and identifies the reason as:

```text
noncanonical_split_assignment
```

The R9 and R17 manifests define exactly which samples belong to each reconstructed experiment.

### 14.5 Fail-Fast Leakage Checks Were Added

The data loader collects patient IDs for training, validation, and testing and checks every pair of splits:

```python
overlaps = {
    "train/val": train_ids & val_ids,
    "train/test": train_ids & test_ids,
    "val/test": val_ids & test_ids,
}
```

If any overlap is found, loading stops with a `ValueError`.

This prevents the experiment from silently continuing with contaminated splits.

A leakage error is therefore treated as an invalid experiment rather than as a warning that can be ignored.

### 14.6 Classification Was Evaluated at the Patient Level

The dataset contains multiple ROI samples per patient.

Using every ROI as an independent test observation would allow patients with many ROIs to have more influence than patients with only a few. It would also overstate the effective test sample size because samples from the same patient are correlated.

The corrected evaluation therefore:

1. generates probabilities for individual ROIs;
2. aggregates ROI probabilities within each patient; and
3. calculates headline classification metrics from patient-level predictions.

This gives every test patient one contribution to the final evaluation.

### 14.7 Model Selection Was Kept Separate From Test Evaluation

Model checkpoints and optional decision thresholds were selected using training and validation data.

After selection:

- the checkpoint was frozen;
- the threshold was frozen; and
- the corrected 47-patient test set was evaluated.

The test set was not used for model tuning or threshold optimization.

### 14.8 Segmentation Assumptions Were Made Explicit

Where the paper did not provide enough information, the reproduction documented its own choices, including:

- preprocessing;
- image resolution;
- probability threshold;
- checkpoint rule;
- metric definitions;
- and aggregation method.

Foreground Dice, foreground IoU, and background-inclusive mIoU were named separately.

The resulting experiments are described as reconstructions under documented assumptions rather than claimed to be bit-for-bit replications of the paper.

---

# Part III: Consequences for Reproduction

## 15. Why the Published Results Cannot Be Reproduced Directly

The segmentation and classification problems affect reproducibility in different ways.

### Segmentation

The segmentation table cannot be reproduced exactly because the paper does not completely specify:

- metric class definitions;
- aggregation;
- empty-mask handling;
- the exact split;
- evaluation batch size;
- threshold;
- smoothing;
- checkpoint selection;
- and whether all models shared one evaluator.

### Classification

The classification archive cannot safely be used exactly as physically released because the raw R9 branch contains training patients in its test directory.

A researcher who loads samples directly from those folders may unknowingly evaluate on leaked patients.

Even after correcting the leak, an exact LG-CAFN reproduction still requires information about:

- the split used for the published result;
- patient-level or ROI-level aggregation;
- preprocessing;
- initialization;
- augmentation;
- checkpoint selection;
- and threshold selection.

## 16. Impact on Model Comparisons

The published results do not provide enough evidence to guarantee that all models were compared under the same conditions.

For segmentation, unclear or inconsistent metric procedures can change model rankings.

For classification, evaluating one model with patient leakage and another without leakage would make the results fundamentally unfair.

A claim that one model is better than another requires:

- the same patient split;
- the same test patients;
- the same preprocessing;
- the same evaluation unit;
- the same metric definitions;
- the same aggregation;
- and the same rules for model and threshold selection.

Without those controls, a numerical ranking may reflect differences in evaluation rather than differences in model quality.

## 17. What the Combined Evidence Supports

The available evidence supports the following conclusions:

- The segmentation evaluation is under-specified.
- Reported mIoU-above-Dice pairs require an explicit class definition.
- Matched foreground IoU cannot be higher than matched foreground Dice.
- Background-inclusive macro mIoU can be higher than tumor Dice.
- The released UNeXt evaluator uses batch-dependent foreground overlap.
- The UNeXt evaluator derives Dice directly from IoU.
- The exact segmentation values cannot be recovered confidently from the manuscript alone.
- The raw R9 classification branch contains 43 noncanonical test files.
- Those files belong to six patients assigned to training.
- Using those files in both training and testing creates patient leakage.
- The other five classification branches consistently support a leakage-free 166/19/47 split.
- The safest correction is to retain the six patients in training and exclude their noncanonical R9 test copies.
- Patient-overlap checks should be mandatory before training or evaluation.
- Classification metrics should be reported at the patient level.
- All reconstruction assumptions should be documented explicitly.



# Part IV: Questions That Still Need Answers

## 18. Segmentation Questions for the Authors

The following questions are necessary for an exact segmentation reproduction:

1. Were segmentation metrics calculated per slice, per patient, per volume, per mini-batch, or globally?
2. Was Dice calculated only for the tumor class?
3. Which classes were included in mIoU?
4. Was background included in mIoU?
5. Were Dice and mIoU calculated from exactly the same predictions?
6. Were they calculated from the same model checkpoint?
7. Were they calculated from the same test samples?
8. Did they use the same probability threshold?
9. Were any Dice and mIoU labels or table values reversed?
10. Were slices without annotated tumors included?
11. How were empty prediction and ground-truth masks scored?
12. What smoothing value was used?
13. What evaluation batch size was used?
14. What was the evaluation sample order?
15. Was the split created by patient or by individual image?
16. Did every segmentation baseline use the same split?
17. Did every baseline use the same metric implementation?
18. What loss function was used for each model?
19. What image normalization was used?
20. What random seed was used?
21. How was the final checkpoint selected?
22. What threshold was used to convert probabilities into binary masks?

## 19. Classification Questions for the Authors

The following questions are necessary for an exact LG-CAFN reproduction:

1. Which patient split produced the published LG-CAFN results?
2. Were the six duplicated R9 patients present in the test set used for the paper?
3. Are the raw, GLCM, and LBP branches intended to share one canonical split?
4. Is the 166/19/47 allocation the intended official split?
5. Was the raw `img9Se` directory packaged incorrectly?
6. Were predictions evaluated per ROI or per patient?
7. If evaluated per patient, how were multiple ROI probabilities aggregated?
8. How do the paper’s classification groups correspond to the R9 and R17 inputs?
9. What preprocessing was applied?* 
10. How were model weights initialized?
11. What augmentations were used?
12. What checkpoint-selection rule was used?
13. What classification threshold was used?
14. Was the threshold selected from validation data?
15. Were the reported metrics produced from the currently released archive or another internal dataset version?

Until these questions are answered, the appropriate goal is a transparent reproduction under explicit assumptions, not a claim of exact replication, and I think you can see that by the discrepancy between my model scores and theirs!

---

# Part V: Recommended Evaluation Procedure

## 20. Recommended Segmentation Procedure

A careful reproduction should save one set of test predictions and calculate every segmentation metric from those same predictions.

The evaluation should report:

- mean per-image foreground Dice;
- mean per-image foreground IoU;
- global foreground Dice;
- global foreground IoU;
- mean per-patient foreground Dice;
- mean per-patient foreground IoU;
- tumor IoU;
- background IoU;
- two-class macro mIoU; and
- a separate batch-based result matching the released UNeXt evaluator when historical comparison is necessary.

For every result, record:

- the patient-level split;
- the evaluated checkpoint;
- the probability threshold;
- the smoothing value;
- the empty-mask policy;
- whether tumor-free slices were included;
- the aggregation method;
- the evaluation batch size;
- and the evaluation sample order.

Every matched foreground Dice/IoU pair should be checked automatically:

```text
Dice ≈ (2 × IoU) / (1 + IoU)

Dice ≥ IoU
```

If the check fails, evaluation should stop and report the inconsistency.

## 21. Recommended Classification Procedure

A careful LG-CAFN reproduction should:

1. Establish one canonical patient split.
2. Verify that patient IDs are disjoint across training, validation, and testing.
3. Place every ROI from one patient in only one split.
4. Fail immediately when overlap is detected.
5. Select checkpoints using training and validation data only.
6. Select any probability threshold using validation data only.
7. Freeze the checkpoint and threshold before test evaluation.
8. Aggregate ROI predictions within each patient.
9. Calculate headline metrics at the patient level.
10. Report AUC, accuracy, balanced accuracy, sensitivity, and specificity.
11. Preserve an explicit manifest of included files.
12. Preserve an explicit list of excluded files and exclusion reasons.
13. Never silently modify the source archive.
14. Report uncertainty using patient-level confidence intervals.
15. Repeat experiments with multiple random seeds.

## 22. Recommended Statistical Unit

The patient—not the ROI or image slice—should be the primary independent unit for classification analysis and uncertainty estimation.

For segmentation, patient-level aggregation or patient-level bootstrap sampling should be reported in addition to slice- and pixel-level results.

Resampling ROIs or slices independently would ignore within-patient correlation and could produce confidence intervals that are too narrow.

---

# Final Conclusion

The central segmentation problem is not the standard mathematical definition of Dice. The problem is that the paper does not explain how many slice and patient predictions were combined into the final reported score.

In addition, the paper reports mIoU values above Dice. This is mathematically impossible if both are matched foreground overlap metrics calculated from the same masks and aggregation. It may be possible if Dice is tumor-only while mIoU averages tumor and background IoUs. The paper and released code do not provide enough information to determine confidently which procedure generated every reported value.

The central classification problem is a concrete patient-level split anomaly in the released raw `img9Se` branch. Six malignant training patients also appear in the test directory, contributing 43 noncanonical test files. Using those patients in both training and testing creates data leakage and makes the test set an invalid measure of unseen-patient performance.

The reproduction corrected this problem by:

- treating the consistent 166/19/47 allocation as canonical;
- retaining the six affected patients in training;
- excluding their 43 noncanonical test copies;
- preserving the original archive;
- recording every exclusion;
- creating explicit manifests;
- enforcing disjoint patient IDs;
- evaluating classification at the patient level;
- and separating validation-based model selection from final test evaluation.

The most defensible overall conclusion is that the BreastDM materials contain:

1. an unresolved segmentation reporting, implementation, class-definition, or aggregation inconsistency; and
2. a directly observable patient-level classification split error in the raw R9 archive.

The paper’s reported results should therefore be interpreted cautiously.

A useful reproduction is still possible, but it must use a leakage-free patient split, clearly named metrics, one standardized evaluator, explicit aggregation rules, and a complete record of every assumption.

---
