## 6/28/2026 

There is a very specific format the dataset has to be in for any and all of the project code to work. I have listed it below. 

### Creating the Dataset, needs to be in this format:

<img width="512" height="366" alt="image" src="https://github.com/user-attachments/assets/60480a30-1247-4220-86d7-c22070cd1a6d" />

### Downloaded dataset currently looks like this: 

<img width="1928" height="244" alt="image" src="https://github.com/user-attachments/assets/e4f44f0a-b4ca-4082-b14e-5f8d9a1ce65a" />

* This is obviously not what I want but it is a little cool that there is the classification dataset, 2d segmentation and 3d segmentation, I need the 2d seg or just seg. I will just drag files into folders using my mouse, I'm sick of writing code to format files :)

## Workflow:
* Copy these into a folder that is not the zip folder (time estimated at ~2 hours)> upload to google drive > mount > and then unfortunately I will have to write python to properly format the files :(


---
## 6/29/2026

### Trying to get the dataset to work

* I have encountered so many issues with the dataset it is insane. If this were a project submitted to a professor I think it is safe to assume that they recieved a D as their grade for their work. The requirements for their formatting do not at all match what they give you and they do not readily supply a data splitting function. I will have to create one myself!
* I am currently downloading the dataset to google drive so that I can mount it, but the estimated time to download is hours, and I believe it will surely take that long.
* I tried downloading the dataset, it includes **82000** items. this was too much and I quickly ran out of space and had to delete everything. deleting everything is taking forever. Once I delete everything I think I am going to convert to a zip file, unfortunately I am not sure if that will be able to be downloaded either. This is a pretty hefty dataset.

### Example of my struggle:

<img width="802" height="388" alt="image" src="https://github.com/user-attachments/assets/e418d7d2-cc72-4557-87cc-ffccdb701ea9" />

* NOTE: So apparently, for segmentation, the data was in seg and seg3d (which would have been nice if it was explained in the readme). I suppose that because its technically 3 dimenional, not in an x y z sorta way but it just has another dimension to it even though it's really just a 2d image. It appears each file contains 8 slices of 369x369 pixels. The displayed central slices confirm this. The sample files VIBRANT+C2.npy from both the images and labels directories share the same name, strongly suggesting that files with identical names across these two folders represent corresponding image and mask pairs.

### Here's what BeastDM_DataInspecToProces does for me right now: 

* Unzipped Dataset: The BreastDMDS.zip file was unzipped to /content/BreastDMDS_unzipped.

* Dataset Reorganization Attempt (and correction): Initially, there was an attempt to classify files into 'B' and 'M' folders, which was later identified as not the user's primary goal for this task and also caused file overwrites. We then corrected this by reverting the DATASET path to the original unzipped location.

* NPY File Inspection: inspected sample .npy files from the seg3D/val/images and seg3D/val/labels directories. This revealed that the data was float64, ranged from 0.0 to 255.0, and consisted of 3D volumes (e.g., (369, 369, 8)), with images and masks sharing the same base filenames.

* Conversion and Reorganization to PNG: Based on the inspection, we implemented code to:
Create new /content/segmentation_dataset/images and /content/segmentation_dataset/masks directories.

* Iterate through all seg3D splits (train, val, test).

* Load each .npy file.

* Extract the central 2D slice from 3D volumes.

* Scale the data to 0-255 and convert it to uint8.

* Save the processed data as .png files using a unique naming convention (split_patientID_originalFileName.png) to prevent overwrites.
Verification: We confirmed that the new dataset structure is correct and that the files are appropriately organized, with 699 image files and 717 mask files created.

* The DATASET variable now points to /content/segmentation_dataset


---
## 6/30/2026

### Attempting to make the split function

* This is currently the structure of the file, but its not even all that we need because the corresponding mask data for segmentation task is in seg3d because it has the 8 layer dimension that I did not account for.

### Current structure of seg:
```
Depth = 2 

BreastDMDS_unzipped/
    seg3D/
        val/
        train/
        test/
    seg/
        val/
        train/
        test/
    cls/
        img9Se/
        GLCM/
        img17Se/
        LBP/

```
### Currently the code in this ipynb file does:

* Setup and Unzipping // It initializes by importing necessary libraries, mounts Google Drive, and unzips the main dataset from a .zip file.
* Initial Data Inspection // The code then looks at the directory structure of the unzipped dataset and loads a sample .npy file to understand its data type, shape, and value range. This makes it easy to see how far I am and how far I need to go to make it the desired structure.
* Classification Data Organization // Processes .npy files to organize them into distinct 'Benign' and 'Malignant' directories, preparing the data for a classification task.
* Segmentation Data Processing and Visualization // Identifies 3D image and mask .npy volumes from the segmentation part of the dataset, extracts a central 2D slice from each, and displays these slices for visual inspection.
* Segmentation Data Conversion // Finally, it converts these extracted 2D slices from .npy format into .png image files, scaling their pixel values, and saving them into separate 'images' and 'masks' directories for segmentation tasks.

### All the current cells listed: 

(copy and pasted)
```
Cell ID: EjrpEys8Bwzz, Title: Notebook Overview and Purpose
Cell ID: mpDxIsGgDMCB, Title: Import Libraries
Cell ID: cVA52J2CB3O5, Title: Mount Google Drive
Cell ID: 4n5PiRY7Cdgh, Title: Initial Dataset Path and Structure Exploration (Original Zip)
Cell ID: 5a77de7d, Title: Unzip Dataset
Cell ID: e40be580, Title: Directory Structure Inspection (Unzipped)
Cell ID: b1653db6, Title: Organize Classification Data (Benign/Malignant)
Cell ID: 579f06a3, Title: Verify Classification Dataset Structure
Cell ID: 3ce6ee7a, Title: Inspect Sample NPY File
Cell ID: cf4550ea, Title: Visualize Sample Segmentation Image and Mask Slices
Cell ID: 8645d641, Title: Convert Segmentation NPY to PNG (2D Slices)
Cell ID: ce174ae4, Title: Verify Converted Segmentation Dataset Structure
```
### We now have this structure for data: 
```
============================ New Segmentation Dataset Structure ============================
segmentation_dataset/
    masks/
        test_BreaDM-Be-2019_VIBRANT.png
        test_BreaDM-Be-2113_SUB2.png
        train_BreaDM-Be-1805_SUB2.png
        train_BreaDM-Be-1809_SUB2.png
        train_BreaDM-Ma-2111_VIBRANT.png
        ...(709) more files
    images/
        test_BreaDM-Be-2019_VIBRANT.png
        test_BreaDM-Be-2113_SUB2.png
        train_BreaDM-Be-1805_SUB2.png
        train_BreaDM-Be-1809_SUB2.png
        train_BreaDM-Ma-2111_VIBRANT.png
        ...(691) more files

Total image files: 696
Total mask files: 714
```
---
## 7/11/2026

### Trying to finish data preprocessing and building ResNet for Segmentation task
* I think there is an issue with the data splitting function.
* it comes from a inconsistency with the 3d-2d slivering
* I will try to find the severity of the issue, how to fix it and maybe find an alternative
* then I will properly reproduce! should be easy

### New workflow:

* I am going to start reformatting code in visual studio code
* all testing will probably be run on google colab for the processing power and speed of the t4 GPU (thank you google)

## 7/24/2026

#### Confirmed `img9Se` leakage

The released raw imagese image directories repeat six malignant training patients in the test folder:
The official repository was downloaded and extracted to:

- `BreaDM-Ma-1802`
- `BreaDM-Ma-1803`
- `BreaDM-Ma-1804`
- `BreaDM-Ma-1806`
- `BreaDM-Ma-1807`
- `BreaDM-Ma-1808`

These patients contribute 43 test images, all byte-identical to their training copies. The released GLCM and LBP feature folders do not contain these six extra test patients.

### Created a repeatable dataset audit

Run:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\audit_breastdm.ps1
```text
reference/extracted/Breast-cancer-dataset-master
```

The script checks:
I also created scripts that audit the dataset and generate training manifests:

- Total files and file extensions
- Patient and sequence counts
- Image/mask pairing
- Missing or orphan labels
- Patient overlap across train, validation, and test
- Classification experiment-group composition

It writes the machine-readable report to `outputs/dataset_audit.json`.

The human-readable interpretation is recorded in `AUDIT_FINDINGS.md`.

### Created immutable training manifests

Run:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\build_manifests.ps1
```text
scripts/audit_breastdm.ps1
scripts/build_manifests.ps1
```

This generates CSV manifests under `outputs/manifests` without moving, renaming, or modifying source images.

| Manifest | Rows | Train/test patient overlap |
|---|---:|---:|
| `seg2d_released.csv` | 29,512 | 3 patients |
| `seg2d_corrected.csv` | 29,274 | 0 |
| `seg3d.csv` | 696 | 0 |
| `classification_img17Se_released.csv` | 1,722 | 0 |
| `classification_img17Se_corrected.csv` | 1,722 | 0 |
| `classification_img9Se_released.csv` | 1,765 | 6 patients |
| `classification_img9Se_corrected.csv` | 1,722 | 0 |

Each manifest records the supplied split, patient ID, sequence or class, and paths relative to `data/BreastDM_clean`.

### Inspected the authors' U-Net implementation

The authors' U-Net folder pins:

- Python-era dependencies centered on PyTorch 1.10.0
- torchvision 0.11.1
- NumPy 1.21.3
- Pillow

The classification project separately pins PyTorch 1.8.0 and torchvision 0.9.0. These legacy dependencies should be isolated from any modern improvement environment.

The U-Net training script reports the following intended defaults:
## Dataset findings

- One tumor class plus background
- Batch size 64
- 100 epochs
- Initial learning rate 0.01
- SGD momentum 0.9
- Weight decay `1e-4`
- Input normalization mean `(0.709, 0.381, 0.224)`
- Input normalization standard deviation `(0.127, 0.079, 0.043)`
The dataset contains three main sections:

The repository includes saved U-Net training logs showing Dice and IoU values across epochs. However, its loader is still based on a DRIVE-dataset template: it expects `all_images`, `all_manual`, and BMP masks instead of the released BreastDM patient/sequence hierarchy. A BreastDM-specific loader is therefore required before training can be reproduced correctly.
```text
cls/      Classification data
seg/      Prepared 2D segmentation data
seg3D/    Prepared 3D segmentation data
```

## Reproduction policy
The clean extraction contains 70,809 files. The 3D segmentation data contains 696 image volumes, which matches 232 patients with three MRI sequences each.

Every experiment will be reported under one of two tracks:
Several problems were discovered in the released splits:

### Faithful track
- The 2D segmentation data repeats three patients across training and testing. This produces 238 identical image/mask pairs in both splits.
- The `img9Se` classification data repeats six malignant patients across training and testing, producing 43 identical images.
- The 3D training-label folder contains 18 extra masks belonging to six test patients. The 3D image splits themselves are patient-disjoint.
- The `img17Se` classification split is patient-disjoint and contains all 232 patients.

Use the authors' released folders and settings as closely as possible. This track determines whether the published values can be reproduced. Results must clearly disclose the confirmed duplicates and orphan labels.
Because of these issues, experiments will use two tracks:

### Corrected track
1. **Released track:** Uses the authors’ supplied splits to determine whether the published results can be reproduced.
2. **Corrected track:** Removes duplicated test patients and ignores orphan labels to produce scientifically valid results.

Use patient-disjoint manifests, ignore orphan labels, and evaluate without duplicate test patients. This track provides the scientifically valid comparison for future improvements.
## Generated manifests

Faithful and corrected values must never be combined in the same results column without a clear label.
CSV manifests were created under `outputs/manifests` so training code can load the data without moving or renaming files.

## Current workspace
Important manifests include:

```text
ok-s/
├── data/
│   └── BreastDM_clean/             # Verified clean dataset copy
├── outputs/
│   ├── dataset_audit.json          # Machine-readable audit
│   └── manifests/                  # Released and corrected CSV manifests
├── reference/
│   ├── Breast-cancer-dataset-master.zip
│   └── extracted/
│       └── Breast-cancer-dataset-master/
├── scripts/
│   ├── audit_breastdm.ps1
│   └── build_manifests.ps1
├── AUDIT_FINDINGS.md
├── README.md
└── .gitignore
seg2d_released.csv
seg2d_corrected.csv
seg3d.csv
classification_img17Se_corrected.csv
classification_img9Se_released.csv
classification_img9Se_corrected.csv
```

The `data/`, `reference/`, and generated `outputs/` contents are excluded from version control because they are large or reproducible artifacts.
The corrected manifests have no patient overlap between training and testing.

## 7/14/2026

### Reviewing and making sure everthing is in order 

* discovered duplicated patients across some training and testing splits
* created both original and corrected patient-disjoint manifests.

* ### Recreated Segmentation Dataset

* There were many issues with the segmentation dataset, many have already been described in earlier days.
* This time it should really be fixed, the segmentation dataset will now be placed in the data folder of the breast_dm project folder.

* Today was alot of organizing and finalizing the data

### One of the most important additions was the smoke testing

* the smoke test that I ran basically checked to see if each patient had a mask,
* I can fix the issue, but I won't know if its fixed or not, this is the major problem with the engineering component of the project
* I THINK that everything is in order now
* 
### The Dataset now exists as a zip file in my folder 
<img width="1028" height="634" alt="image" src="https://github.com/user-attachments/assets/e4c98ae4-16e5-47aa-b410-afa5f349ad93" />
(taken from my google drive)
--- 
## 7/16/2026
### Adding to the Data preproccessing README

* This is something I really need to stay on top of.
* I created a new split function because there were so many issues with the first one
1. there was a descrepency within the numbers
* the issue with this is obviously that I might garter innacurate results from this
2. The slicing that converted 3d to 2d had major issues
*  This is an issue because the masks wouldnt line up with the images, so we wouldn't even beable to run the segmentation
All of this is communicated in the readme
* new split function added (better and working)

* Now for UNET
---
## 7/17/2026

### Created Unet
* uses a simple 2D U-Net to identify and segment breast tumors in DCE-MRI slices. The model receives an MRI image and produces a binary mask showing the predicted tumor region.

### Model Architecture
The U-Net contains three main sections:
1. Encoder: Uses convolution blocks and pooling to learn image features while reducing spatial resolution.
2. Bottleneck: Learns the deepest representation of the image.
3. Decoder: Enlarges the feature maps and reconstructs the tumor mask.
Skip connections transfer detailed information from the encoder to the decoder. Each stage uses a reusable DoubleConv block containing two convolution layers, batch normalization, and ReLU activation.
### Input and Output
The DataLoader prepares grayscale images and masks at 256 × 256 resolution.
Input:  [batch_size, 1, 256, 256]
Output: [batch_size, 1, 256, 256]
Each output pixel represents the model’s confidence that the corresponding image pixel belongs to a tumor.
Training
The model was trained using:
Epochs: 10
Batch size: 8
Optimizer: Adam
Learning rate: 0.0001
Loss: Binary Cross-Entropy + Dice Loss
Binary cross-entropy evaluates individual pixel predictions, while Dice loss encourages overlap between the predicted and correct tumor masks.
Initial Results
The best validation performance occurred during epoch 3:
Training Dice:   0.8549
Validation Dice: 0.5846
Validation loss: 0.4277
The training Dice later increased above 0.91, but validation performance decreased. This indicates overfitting: the model continued learning the training images without improving on unseen patients.
The best checkpoint was automatically saved as:
/content/drive/MyDrive/BreastDM_Project/models/best_unet.pth
Comparison with the Paper
The BreastDM manuscript reported the following U-Net results:
Dice: 73.7%
mIoU: 80.3%
PPV:  83.6%
The current model’s validation Dice of 58.46% is a useful first baseline, but it cannot be compared perfectly with the paper because the dataset split, preprocessing, training duration, and evaluation procedure are different.
Next Steps
The next step is to load the best checkpoint, evaluate it once on the untouched test dataset, and display predicted masks beside the correct masks. Future improvements could include data augmentation, early stopping, learning-rate scheduling, and additional regularization.
### BreastDM specific data loader created 
* I wanted to create a better dataloader that i could use for both Unet and Unext
It:
* Reads your existing train, val, and test directories
* Pairs .jpg images with .png masks by filename stem
* Detects missing or duplicate pairs
* Returns three-channel C×H×W images for UNeXt
* Returns binary 1×H×W masks
* Supports Albumentations transforms
* Creates deterministic PyTorch loaders using seed 42
* Preserves the original UNeXt return format: image, mask, metadata
### Usage:
from dataloader import create_breastdm_loaders
```
loaders = create_breastdm_loaders(
    dataset_root="/content/segmentation_DS_7_14_2026",
    batch_size=8,
    num_workers=2,
    seed=42,
)

train_loader = loaders["train"]
val_loader = loaders["val"]
test_loader = loaders["test"]

images, masks, metadata = next(iter(train_loader))

print(images.shape)  # [8, 3, H, W]
print(masks.shape)   # [8, 1, H, W]
```
For binary UNeXt, instantiate the model with one output class:
```
model = UNext(num_classes=1, input_channels=3)
```

---
## 7/19/2026
### Made substantial progress on the UNEXT
The google colab inlcudes:
* Dataset integrity checks
* Patient-leakage verification
* Shape and finite-value assertions
* One-step gradient testing
* Tiny-dataset overfitting test
* Mixed-precision training
* Gradient clipping
* Early stopping
* Learning-rate scheduling
* Best-checkpoint selection
* Recovery after Colab disconnection
* CSV history logging
* Configuration recording
* Training-curve plots
* Prediction visualization
* Explicit separation of validation and test evaluation
* Explanatory comments and readable variable names

## Dataset

- Used the processed BreastDM 2D segmentation dataset.
- Created a reproducible `70/15/15` patient-level split.
- Removed duplicate samples and prevented patient leakage.
- Final dataset contains 29,274 image–mask pairs:

| Split | Patients | Samples |
|---|---:|---:|
| Train | 162 | 20,417 |
| Validation | 35 | 4,573 |
| Test | 35 | 4,284 |

## Data Pipeline

- Created a PyTorch dataset class.
- Paired JPG images with PNG masks.
- Converted images to `[3, 256, 256]`.
- Converted binary masks to `[1, 256, 256]`.
- Added resizing, flipping, and rotation augmentation.
- Created train, validation, and test loaders.
- Verified image/mask counts, patient separation, and batch shapes.

## UNeXt

- Recreated the complete UNeXt architecture in Colab.
- Used three convolutional encoder stages and two shifted-MLP stages.
- Configured one output channel for binary tumor segmentation.
- Verified the forward pass:

```text
Input:  [B, 3, 256, 256]
Output: [B, 1, 256, 256]
Mask:   [B, 1, 256, 256]
```

## Training Setup

- Defined BCE–Dice loss.
- Defined Dice and IoU metrics.
- Verified one backward optimization step.
- Tested overfitting on 16 tumor-containing samples.
- Prepared full training with:

```text
Optimizer: Adam
Learning rate: 1e-4
Batch size: 8
Maximum epochs: 100
Early stopping: 15 epochs
Checkpoint selection: Best validation Dice
```

## Saving and Evaluation

- Best model saves to Google Drive.
- Recovery checkpoint allows training to resume after Colab disconnects.
- Training history and curves are saved.
- After training, load the best validation checkpoint.
- Evaluate the untouched test set once.
- Report final test Dice and IoU.
- Visually inspect predicted tumor masks.
