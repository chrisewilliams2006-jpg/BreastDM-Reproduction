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
