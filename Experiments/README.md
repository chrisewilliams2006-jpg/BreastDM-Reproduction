## UNEXT info
---
# BreastDM UNeXt Segmentation Reproduction

## Overview

This project reproduces the UNeXt breast-tumor segmentation workflow using the BreastDM dataset.

UNeXt is an efficient medical-image segmentation architecture combining convolutional encoding stages with shifted multilayer perceptron blocks.

This reproduction includes:

- Patient-level dataset splitting
- Duplicate-sample detection
- Image–mask matching
- Dataset-integrity verification
- PyTorch datasets and data loaders
- Medical-image augmentation
- A complete UNeXt implementation
- BCE–Dice loss
- Dice and IoU metrics
- Forward-pass verification
- Backward-pass verification
- Tiny-subset overfitting
- Mixed-precision training
- Best-model checkpointing
- Colab recovery checkpoints
- Learning-rate scheduling
- Early stopping
- Final test-set evaluation
- Training curves
- Prediction visualization

## Reference Implementations

The UNeXt architecture was reproduced from the following repository:

- [Breast-cancer-dataset UNeXt implementation](https://github.com/smallboy-code/Breast-cancer-dataset/tree/master/Segmentation%20task/UNeXt-pytorch-main)
- [Original UNeXt repository](https://github.com/jeya-maria-jose/UNeXt-pytorch)
- [UNeXt paper](https://arxiv.org/abs/2203.04967)

## BreastDM Dataset

BreastDM is a dynamic contrast-enhanced MRI dataset for breast-tumor classification and segmentation.

The released two-dimensional segmentation data contains MRI slices and their corresponding tumor masks.

### Original Dataset Issue

The referenced UNeXt repository did not include a dataset-splitting function suitable for the BreastDM dataset.

The released BreastDM splits also contained duplicate samples. Therefore, a new deterministic patient-level splitting workflow was developed for this reproduction.

## Dataset-Splitting Procedure

The custom splitting notebook performs the following operations:

1. Reads the original BreastDM ZIP archive.
2. Extracts only the two-dimensional `seg` dataset.
3. Excludes the `seg3D` dataset.
4. Locates each JPG MRI image.
5. Locates the corresponding PNG tumor mask.
6. Identifies every sample using:

   - Patient identifier
   - MRI sequence
   - Slice name

7. Detects duplicate samples across the released splits.
8. Retains duplicates only when both files are byte-identical.
9. Stops with an error if duplicate filenames contain conflicting data.
10. Creates a new patient-level split.
11. Stratifies patients by benign or malignant diagnosis.
12. Verifies that no patient appears in multiple splits.
13. Creates unique output filenames.
14. Saves a `split_manifest.csv` file.
15. Packages the processed dataset into a new ZIP archive.

### Split Configuration

The dataset uses the following split:

| Split | Percentage |
|---|---:|
| Training | 70% |
| Validation | 15% |
| Testing | 15% |

A fixed random seed ensures reproducibility:

```python
SEED = 42
```

### Split Results

| Split | Patients | Benign | Malignant | Image–mask pairs |
|---|---:|---:|---:|---:|
| Training | 162 | 59 | 103 | 20,417 |
| Validation | 35 | 13 | 22 | 4,573 |
| Testing | 35 | 13 | 22 | 4,284 |
| Total | 232 | 85 | 147 | 29,274 |

Patient overlap between the splits was verified to be zero.

## Processed Dataset Structure

The splitting process produces the following structure:

```text
segmentation_DS_7_14_2026/
├── train/
│   ├── images/
│   └── masks/
├── val/
│   ├── images/
│   └── masks/
├── test/
│   ├── images/
│   └── masks/
└── split_manifest.csv
```

Each image and mask uses the following naming convention:

```text
patient__sequence__slice.jpg
patient__sequence__slice.png
```

The image and its corresponding mask share the same filename stem.

## Google Colab Setup

This reproduction was designed to run from top to bottom in Google Colab.

### Recommended Runtime

Use a GPU runtime:

```text
Runtime → Change runtime type → T4 GPU
```

### Dataset Location

The processed dataset ZIP is expected at:

```text
/content/drive/MyDrive/BreastDM_Project/data/segmentation_DS_7_14_2026.zip
```

The dataset is extracted to Colab’s temporary local storage:

```text
/content/segmentation_DS_7_14_2026
```

Local Colab storage is used during training because repeatedly reading thousands of files directly from Google Drive is slower.

## Dataset Class

A custom `BreastDMSegmentationDataset` class was created using PyTorch’s `Dataset` interface.

### Dataset-Class Responsibilities

The class:

- Supports `train`, `val`, and `test` splits
- Finds supported image files
- Finds supported mask files
- Pairs images and masks by filename stem
- Detects duplicate filename stems
- Detects missing images or masks
- Loads MRI images as three-channel tensors
- Loads tumor masks as single-channel tensors
- Verifies matching image and mask dimensions
- Applies paired image–mask transformations
- Converts images to `float32`
- Scales image values to the range `0–1`
- Converts masks into binary values
- Returns useful sample metadata

### Dataset Output

Each dataset sample returns:

```python
image, mask, metadata
```

The image shape is:

```text
[3, height, width]
```

The mask shape is:

```text
[1, height, width]
```

The metadata includes:

```python
{
    "img_id": "...",
    "image_path": "...",
    "mask_path": "...",
    "split": "train"
}
```

## Individual Dataset Objects

Three separate dataset objects are created:

```python
train_dataset = BreastDMSegmentationDataset(
    dataset_root=DATASET_ROOT,
    split="train",
    transform=train_transform,
)

val_dataset = BreastDMSegmentationDataset(
    dataset_root=DATASET_ROOT,
    split="val",
    transform=eval_transform,
)

test_dataset = BreastDMSegmentationDataset(
    dataset_root=DATASET_ROOT,
    split="test",
    transform=eval_transform,
)
```

The training dataset receives random augmentation.

The validation and test datasets receive deterministic preprocessing only.

## Image Preprocessing

### Initial Input Resolution

The initial experiments use:

```python
IMAGE_HEIGHT = 256
IMAGE_WIDTH = 256
```

The original UNeXt example uses `512 × 512`, but `256 × 256` was selected initially to reduce Colab memory usage and simplify debugging.

A later controlled experiment can compare `256 × 256` and `512 × 512`.

### Training Transformations

The training transformations include:

- Resize
- Random horizontal flip
- Random vertical flip
- Random 90-degree rotation

Example:

```python
train_transform = A.Compose([
    A.Resize(
        height=256,
        width=256,
    ),
    A.HorizontalFlip(p=0.5),
    A.VerticalFlip(p=0.5),
    A.RandomRotate90(p=0.5),
])
```

### Evaluation Transformations

Validation and testing use only deterministic resizing:

```python
eval_transform = A.Compose([
    A.Resize(
        height=256,
        width=256,
    ),
])
```

Random augmentation is not applied to validation or test samples.

## Dataset Verification

Before creating UNeXt, the data pipeline was checked for:

- Equal image and mask counts
- Missing image–mask pairs
- Duplicate filename stems
- Patient overlap
- Image and mask spatial agreement
- Correct tensor dimensions
- Correct tensor data types
- Image values between `0` and `1`
- Binary mask values
- Correct data-loader batch dimensions
- Visual image–mask alignment

### Expected Batch Shapes

With a batch size of eight:

```text
Images: [8, 3, 256, 256]
Masks:  [8, 1, 256, 256]
```

The full-batch verification passed before UNeXt was created.

## UNeXt Architecture

UNeXt combines convolutional stages with tokenized shifted-MLP stages.

### High-Level Architecture

```text
Input image
    ↓
Convolutional encoder stage 1
    ↓
Convolutional encoder stage 2
    ↓
Convolutional encoder stage 3
    ↓
Tokenized shifted-MLP stage
    ↓
Shifted-MLP bottleneck
    ↓
Shifted-MLP decoder stage
    ↓
Shifted-MLP decoder stage
    ↓
Convolutional decoder stages
    ↓
One-channel segmentation logits
```

### Encoder

The encoder contains three convolutional stages:

| Stage | Input channels | Output channels |
|---|---:|---:|
| Encoder 1 | 3 | 8 |
| Encoder 2 | 8 | 16 |
| Encoder 3 | 16 | 32 |

Each encoder stage uses:

- `3 × 3` convolution
- Batch normalization
- Max pooling
- ReLU activation

### Tokenized MLP Stages

The convolutional features are passed into two overlapping patch-embedding stages:

```text
32 channels → 64 channels
64 channels → 128 channels
```

These stages use shifted MLP blocks to learn spatial relationships efficiently.

### Shifted MLP

The shifted MLP:

1. Converts image tokens back into spatial feature maps.
2. Divides channels into groups.
3. Shifts channel groups vertically.
4. Applies a linear projection.
5. Applies depthwise convolution.
6. Applies GELU activation.
7. Shifts channel groups horizontally.
8. Applies a second linear projection.
9. Adds a residual connection.

### Decoder

The decoder reduces channels through the following stages:

```text
128 → 64 → 32 → 16 → 8 → 8
```

Bilinear interpolation restores spatial resolution.

Additive skip connections transfer encoder features into the decoder.

### Final Segmentation Layer

The final layer is a `1 × 1` convolution:

```python
nn.Conv2d(
    in_channels=8,
    out_channels=1,
    kernel_size=1,
)
```

The model produces raw logits with this shape:

```text
[batch_size, 1, height, width]
```

No sigmoid function is included inside the model.

## Forward-Pass Verification

Before training, a batch was passed through UNeXt.

Expected shapes:

```text
Input:  [2, 3, 256, 256]
Output: [2, 1, 256, 256]
Mask:   [2, 1, 256, 256]
```

The verification checked that:

- The output shape matched the mask shape
- The output contained only finite values
- No NaN values were produced
- No infinite values were produced

The UNeXt forward-pass verification passed.

## BCE–Dice Loss

The training objective combines binary cross-entropy and soft Dice loss.

### Binary Cross-Entropy

Binary cross-entropy evaluates individual tumor and background pixels.

`binary_cross_entropy_with_logits` is used because UNeXt produces raw logits.

### Soft Dice Loss

Soft Dice loss directly optimizes overlap between predicted and true masks.

For prediction \(P\) and target \(T\):

```text
Dice = (2 × intersection + smooth) /
       (predicted pixels + target pixels + smooth)
```

Dice loss is:

```text
Dice loss = 1 − Dice
```

### Combined Loss

The combined objective is:

```text
Total loss = 0.5 × BCE loss + Dice loss
```

The smoothing constant prevents division by zero:

```python
smooth = 1e-5
```

## Evaluation Metrics

### Dice Score

Dice measures overlap between the predicted and ground-truth masks:

```text
Dice = 2 × intersection /
       (predicted pixels + target pixels)
```

Dice ranges from zero to one:

- `0` indicates no overlap.
- `1` indicates perfect overlap.

### Intersection over Union

IoU is calculated as:

```text
IoU = intersection / union
```

IoU is stricter than Dice and also ranges from zero to one.

### Prediction Threshold

Raw logits are converted into probabilities:

```python
probabilities = torch.sigmoid(logits)
```

Binary predictions use:

```python
predictions = probabilities >= 0.5
```

## Optimization-Step Verification

Before full training, one batch was used to verify:

- Forward propagation
- BCE–Dice loss calculation
- Backpropagation
- Finite gradients
- Gradient clipping
- Optimizer updates
- Actual model-parameter changes

The one-step optimization verification passed.

## Tiny-Subset Overfitting Test

Before full training, a separate UNeXt model was trained on 16 positive-mask samples.

### Purpose

A segmentation model should be capable of memorizing a very small deterministic dataset.

If it cannot, there may be a problem with:

- Image–mask alignment
- Loss calculation
- Gradient flow
- Model implementation
- Tensor dimensions
- Mask processing
- Optimizer configuration

### Tiny-Dataset Configuration

```text
Samples: 16
Batch size: 4
Transformations: deterministic
Optimizer: Adam
Learning rate: 0.001
Maximum epochs: 100
```

Only samples containing positive tumor pixels were selected.

### Success Criteria

A healthy overfitting test should show:

- Substantially decreasing loss
- Increasing Dice
- Increasing IoU
- Ideally, Dice approaching `0.90` or higher

A completely fresh model is created after this test. The overfitted debugging model is not used for full training.

## Full Training Configuration

The full experiment uses:

| Setting | Value |
|---|---:|
| Random seed | 42 |
| Input channels | 3 |
| Output channels | 1 |
| Input height | 256 |
| Input width | 256 |
| Batch size | 8 |
| Optimizer | Adam |
| Initial learning rate | `1e-4` |
| Weight decay | `1e-5` |
| Loss | BCE–Dice |
| Maximum epochs | 100 |
| Early-stopping patience | 15 |
| Prediction threshold | 0.5 |
| Gradient clipping | 5.0 |
| Mixed precision | Enabled on GPU |

## Optimizer

Adam is used for model optimization:

```python
optimizer = torch.optim.Adam(
    model.parameters(),
    lr=1e-4,
    weight_decay=1e-5,
)
```

## Learning-Rate Scheduler

`ReduceLROnPlateau` monitors validation loss.

If validation loss stops improving, the learning rate is multiplied by `0.5`.

```python
scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
    optimizer,
    mode="min",
    factor=0.5,
    patience=5,
    min_lr=1e-7,
)
```

## Mixed-Precision Training

Mixed precision is enabled when a CUDA GPU is available.

It reduces GPU-memory consumption and can improve training speed.

The workflow uses:

- `torch.amp.autocast`
- `torch.amp.GradScaler`

## Gradient Clipping

Gradient norm is limited to:

```python
GRADIENT_CLIP_NORM = 5.0
```

Gradient clipping reduces the risk of unstable parameter updates.

## Model Selection

The model is selected using validation Dice.

Whenever validation Dice improves, the checkpoint is saved as:

```text
best_model.pt
```

The test set is not used for model selection.

## Early Stopping

Training stops if validation Dice does not improve for 15 consecutive epochs:

```python
EARLY_STOPPING_PATIENCE = 15
```

This helps prevent unnecessary training and overfitting.

## Colab Recovery Checkpoints

At the end of every completed epoch, the current training state is saved as:

```text
recovery_checkpoint.pt
```

The recovery checkpoint contains:

- Current epoch
- Model parameters
- Optimizer state
- Scheduler state
- Mixed-precision scaler state
- Best validation Dice
- Early-stopping state
- Training history
- Data-loader generator state
- Training configuration

To resume after a Colab disconnection:

```python
RESUME_TRAINING = True
```

The notebook then loads the last completed epoch.

## Saved Experiment Files

Experiment outputs are stored in:

```text
/content/drive/MyDrive/BreastDM_Project/UNeXt_runs/BreastDM_UNeXt_256/
```

The directory contains:

```text
BreastDM_UNeXt_256/
├── best_model.pt
├── recovery_checkpoint.pt
├── training_config.json
├── training_history.csv
├── training_curves.png
├── test_results.json
└── test_predictions.png
```

## Training Output

Each epoch reports:

```text
Train | Loss | Dice | IoU
Val   | Loss | Dice | IoU
```

Example:

```text
Epoch 1/100
Train | Loss: ... | Dice: ... | IoU: ...
Val   | Loss: ... | Dice: ... | IoU: ...
Best validation Dice: ...
```

## Training Curves

The workflow saves plots for:

- Training and validation loss
- Training and validation Dice
- Training and validation IoU

These plots help identify:

- Successful learning
- Underfitting
- Overfitting
- Training instability
- Divergence between training and validation results

## Final Test Evaluation

After training:

1. The best validation checkpoint is loaded.
2. The model is switched to evaluation mode.
3. The untouched test set is evaluated once.
4. Test loss, Dice, and IoU are saved.
5. Prediction visualizations are generated.

The test set is not used during training, hyperparameter adjustment, early stopping, or checkpoint selection.

## Experimental Results

Complete this section after training finishes.

### Best Validation Result

| Metric | Result |
|---|---:|
| Best epoch | `TBD` |
| Validation loss | `TBD` |
| Validation Dice | `TBD` |
| Validation IoU | `TBD` |

### Final Test Result

| Metric | Result |
|---|---:|
| Test loss | `TBD` |
| Test Dice | `TBD` |
| Test IoU | `TBD` |
| Prediction threshold | 0.5 |

## General Dice Interpretation

The following ranges provide only a general interpretation:

| Dice score | General interpretation |
|---:|---|
| `0.90–1.00` | Excellent |
| `0.80–0.90` | Strong |
| `0.70–0.80` | Moderate |
| `0.60–0.70` | Weak |
| Below `0.60` | Requires investigation |

These are not universal clinical-performance thresholds.

Model quality should be evaluated using quantitative metrics and visual inspection.

## Prediction Visualization

The final notebook displays:

1. Original MRI image
2. Ground-truth tumor mask
3. Predicted tumor probability
4. Predicted-mask overlay

Visual inspection is important because a single average score can hide:

- Missed small tumors
- False-positive regions
- Poor boundary quality
- Failures on unusual MRI sequences
- Incorrect behavior on empty masks

## Important Reproduction Limitation

The original authors’ exact dataset partition was unavailable.

This project therefore reproduces the UNeXt architecture and training procedure using a newly created deterministic patient-level split.

As a result, this work should be described as an independent reproduction rather than an exact replication of the authors’ numerical results.

A suitable statement is:

> We reproduced the published UNeXt architecture and training workflow. Because the original BreastDM dataset partition was unavailable and duplicate samples were present in the released splits, we created a deterministic, diagnosis-stratified patient-level partition. Consequently, our results constitute an independent reproduction rather than an exact replication of the originally reported scores.

## Reproducibility Notes

For reproducibility, this project records:

- Random seed
- Dataset split
- Patient assignments
- Dataset manifest
- Input resolution
- Batch size
- Model architecture
- Loss configuration
- Optimizer configuration
- Learning-rate schedule
- Best checkpoint
- Training history
- Test metrics
- Prediction threshold
- Output visualizations

## Future Experiments

Potential follow-up experiments include:

- Training at `512 × 512`
- Multiple random-seed experiments
- Reporting mean and standard deviation
- Separate MRI-sequence evaluation
- Benign-versus-malignant subgroup analysis
- Small-tumor performance analysis
- Empty-mask performance analysis
- Comparison with U-Net
- Comparison with U-Net++
- Comparison with attention-based segmentation models
- Alternative loss functions
- Test-time augmentation
- Cross-validation

## Citation

If using UNeXt, cite the original paper:

```bibtex
@article{valanarasu2022unext,
  title={UNeXt: MLP-based Rapid Medical Image Segmentation Network},
  author={Valanarasu, Jeya Maria Jose and Patel, Vishal M.},
  journal={arXiv preprint arXiv:2203.04967},
  year={2022}
}
```

## Acknowledgments

This reproduction is based on the UNeXt architecture developed by Jeya Maria Jose Valanarasu and Vishal M. Patel.

The BreastDM dataset and associated reference implementation provided the medical-image segmentation task used in this project.
