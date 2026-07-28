
The data loader constructs patient-ID sets for train, validation, and test and checks every pair:

```python
overlaps = {
    "train/val": train_ids & val_ids,
    "train/test": train_ids & test_ids,
    "val/test": val_ids & test_ids,
}
```

If any overlap exists, loading stops with a `ValueError`. This converts a subtle validity problem into an explicit failure instead of allowing a contaminated experiment to continue.

#### 5. Evaluated classification at the patient level

ROI probabilities are aggregated within each patient before headline metrics are calculated. This prevents a patient with many ROIs from having more influence than a patient with only a few and keeps the statistical unit aligned with the question of generalization to unseen patients.

#### 6. Kept model selection separate from test evaluation

Checkpoints and optional decision thresholds are selected from the training/validation data. The selected model and threshold are then frozen before evaluation on the corrected 47-patient test set. The test set is not used to tune the model.

#### 7. Made segmentation assumptions explicit

The reproduction notebooks state their chosen threshold, preprocessing, checkpoint rule, and metric definitions. Foreground Dice, foreground IoU, and background-inclusive mIoU are named separately. Where exact paper settings are unavailable, results are labeled as reconstructions under documented assumptions rather than claimed as exact replications.

### What Can and Cannot Be Concluded

The evidence supports these conclusions:

- the segmentation evaluation in the paper is under-specified;
- reported mIoU-above-Dice pairs need an explicit class and aggregation definition;
- matched foreground Dice and foreground IoU cannot have IoU above Dice;
- the released UNeXt evaluator uses batch-dependent foreground overlap and derives Dice from IoU;
- the exact published segmentation values cannot be recovered confidently from the manuscript alone;
- the raw R9 classification branch contains 43 noncanonical test files from six training patients;
- using those files in testing creates patient leakage; and
- the leakage-free 166/19/47 split is consistently supported by the other released branches.

The evidence does **not** establish:

- that the authors fabricated any result;
- that every experiment in the paper is invalid;
- that the classification leak caused a known numerical amount of score inflation;
- that the classification issue necessarily affected branches other than raw `img9Se`;
- that the paper's classification table used the leaked archive exactly as released;
- that mIoU above Dice is inherently impossible when mIoU includes background;
- that the dataset has no research value; or
- that any model in this repository is clinically validated.

### Information Still Needed for an Exact Reproduction

Even after correcting the observable leakage, an exact reproduction requires clarification or missing artifacts from the authors.

For segmentation:

- Was the evaluation unit a slice, mini-batch, patient, volume, or complete dataset?
- Was Dice foreground-only?
- Which classes were included in mIoU?
- Were Dice and mIoU calculated from exactly the same checkpoint, predictions, masks, threshold, and test samples?
- Were any table labels or values reversed?
- Were tumor-free slices included?
- How were empty/empty masks scored?
- What smoothing value was used?
- What evaluation batch size and sample order were used?
- Was the split made by patient or individual image?
- Did every baseline use the same split and metric code?
- What loss, normalization, seed, threshold, and checkpoint-selection rule was used for each model?

For classification:

- Which patient split was used to produce the paper's LG-CAFN results?
- Were the six duplicated R9 patients present in the test set used for the reported scores?
- Are the raw, GLCM, and LBP branches intended to share one canonical split?
- How were multiple ROI predictions combined into one patient prediction?
- Do the paper's groups correspond to R9/R17 inputs or to another grouping?
- Which preprocessing, initialization, augmentation, checkpoint, and threshold settings generated the reported values?

Until those questions are answered, the appropriate goal is a transparent reproduction under explicit assumptions, not a claim of bit-for-bit replication.

### Recommended Standard for Future Segmentation Evaluation

A robust re-evaluation should save one set of test predictions and calculate every metric from those same predictions. It should report, with unambiguous names:

- mean per-image foreground Dice and foreground IoU;
- global foreground Dice and foreground IoU;
- mean per-patient foreground Dice and foreground IoU;
- two-class macro mIoU with tumor and background IoUs shown separately; and
- a separate batch-based result matching the released UNeXt code, if historical comparison is needed.

Each result should record the patient split, checkpoint, threshold, smoothing constant, empty-mask policy, inclusion or exclusion of tumor-free slices, aggregation method, evaluation batch size, and sample order. A consistency test should verify for every matched foreground pair that:

```text
Dice approximately equals 2 * IoU / (1 + IoU)
Dice >= IoU
```

If the test fails, the evaluator should stop and identify the mismatch rather than publish the pair.

### Evidence and Source Links

- Paper: Zhao, X. et al., “BreastDM: A DCE-MRI Dataset for Breast Tumor Image Segmentation and Classification,” *Computers in Biology and Medicine*, Volume 164, 2023, Article 107255. [DOI](https://doi.org/10.1016/j.compbiomed.2023.107255)
- [Original BreastDM repository](https://github.com/smallboy-code/Breast-cancer-dataset)
- [Released UNeXt metric implementation](https://github.com/smallboy-code/Breast-cancer-dataset/blob/master/Segmentation%20task/UNeXt-pytorch-main/metrics.py)
- [Released UNeXt validation implementation](https://github.com/smallboy-code/Breast-cancer-dataset/blob/master/Segmentation%20task/UNeXt-pytorch-main/val.py)
- [DataSplitter_LG-CAFN Colab documenting the split anomaly](https://colab.research.google.com/drive/1nmTf4lwkT2qb_VnZhznYJwRh9OvepVn-#scrollTo=Tdu7ND_YsqKV)

## Project Scope

| Task | Model family | Role in this project | Current evidence |
