# BreastDM Dataset Processing

## Overview

This repository contains the data-processing workflow used to prepare the **BreastDM breast MRI dataset** for classification and segmentation tasks.

The main notebook handles the initial dataset setup, inspection, organization, visualization, and conversion of MRI data stored as NumPy files.

The notebook creates two processed datasets:

1. A **classification dataset** organized into benign and malignant classes.
2. and A **segmentation dataset** containing MRI images and their corresponding tumor masks as PNG files.

---

## Main Features

The notebook performs the following steps:

### 1. Environment Setup

The notebook imports the libraries required for file management, numerical processing, image visualization, and image conversion.

The main libraries used are:

* os for navigating directories and constructing file paths
* NumPy for loading and processing numpy arrays
* Matplotlib for displaying MRI images and segmentation masks
* Google Colab drive for accessing the dataset from Google Drive
* zipfile for extracting the downloaded dataset
* shutil for moving and organizing files
* Pillow for converting NumPy arrays into PNG images

---

### 2. Google Drive and Dataset Loading

notebook mounts Google Drive and accesses the compressed BreastDM dataset using the following path:

/content/drive/MyDrive/BreastDMDS.zip 

Using Colab's local storage makes later file-processing operations faster than repeatedly accessing files directly from Google Drive. I found this out the hard way.

---

### 3. Dataset Structure Inspection

dir_structure() walks through the dataset directories and prints a limited view of the folder structure.

This makes it possible to inspect the organization of the dataset without printing every file.

The original dataset contains several major sections, including:

```text
BreastDMDS_unzipped/
├── cls/
├── seg/
└── seg3D/
```

The seg3d directory is divided into training, validation, and testing subsets containing MRI images and segmentation labels.

```text
seg3D/
├── train/
│   ├── images/
│   └── labels/
├── val/
│   ├── images/
│   └── labels/
└── test/
    ├── images/
    └── labels/
```

---

## Classification Dataset Processing

### Benign and Malignant Organization

The notebook searches through the unzipped dataset for NumPy files associated with benign and malignant cases.

Files are organized into the following output structure:

```text
/content/processed_dataset/
├── B/
└── M/
```

Where:

* B contains benign samples
* M contains malignant samples

This structure prepares the data for a binary classification model that predicts whether a breast MRI case is benign or malignant.

The notebook also prints file counts and displays the resulting directory structure to verify that the files were organized successfully.

### Classification Output

```text
processed_dataset/
├── B/
│   ├── sample_1.npy
│   ├── sample_2.npy
│   └── ...
└── M/
    ├── sample_1.npy
    ├── sample_2.npy
    └── ...
```

---

## NumPy Data Inspection

Before converting the data, the notebook loads sample numpy files and reports important information such as:

* NumPy data type
* Array dimensions
* Array shape
* Minimum pixel value
* Maximum pixel value
* Example pixel values

This inspection helps determine whether a file represents:

* A two-dimensional image
* A three-dimensional MRI volume
* A segmentation mask
* Another unsupported type of array

* this was the hardest part because I had no idea what was going on with dataset. Only with extensive testing was I actually able to figure it out :)
  
For example, a sample 3D MRI volume may have a shape similar to:

```text
(369, 369, 8)
```

This represents an MRI volume with eight two-dimensional slices.

---

## Segmentation Data Inspection and Visualization

The notebook searches the seg3d directories for a sample MRI image and its corresponding segmentation label.

For three-dimensional arrays, it selects a central slice from the volume and displays it using Matplotlib.
The central slice is best because it shows the same everytime, and the anatomy for these pics are centered. Also it easily converts it to 2d from 3d. 

MRI images are displayed using a grayscale colormap, while masks may be displayed using a separate colormap to make the segmented tumor region easier to inspect.

This step helps verify that:

* Image files can be loaded correctly
* Mask files can be loaded correctly
* Images and masks have compatible dimensions
* The segmentation labels contain visible tumor regions
* The correct slice axis is being used

---

## Segmentation Dataset Conversion

The segmentation-processing section converts the NumPy MRI volumes into standard PNG image files.

The output directories are:

```text
/content/segmentation_dataset/
├── images/
└── masks/
```

Where:

* images contains MRI slices
* masks contains the corresponding segmentation masks

---

### Central Slice Extraction

When the notebook encounters a three-dimensional array, it selects the central slice along the final dimension:

```python
central_slice_idx = data.shape[2] // 2
processed_data = data[:, :, central_slice_idx]
```

If the input is already two-dimensional, the complete array is used.

Arrays with unsupported dimensions are skipped.

---

### Pixel-Value Scaling

Before saving an array as a PNG image, the notebook scales its values to the range 0-255.

The converted array is then stored using the unsigned 8-bit integer format required for standard grayscale PNG images.

The scaling process is equivalent to:

```python
scaled_data = (
    (processed_data - processed_data.min())
    / (processed_data.max() - processed_data.min())
    * 255
).astype(np.uint8)
```

---

### Unique File Naming

Each converted file is given a unique filename containing:

* Dataset split
* Patient identifier
* Original MRI sequence name

Example filenames may look like:

```text
train_BreaDM-Ma-2018_VIBRANT.png
train_BreaDM-Ma-2018_SUB2.png
val_BreaDM-Be-2112_VIBRANT+C2.png
```

This naming system prevents samples from different patients or dataset splits from being confused with one another.

It also preserves useful information about where each image originated.

---

## MRI Sequences

The dataset contains multiple MRI sequence types, including:

* `VIBRANT`
* `SUB2`
* `VIBRANT+C2`

The processing code retains the original sequence name in each output filename.

This makes it possible to later train a model using one sequence or compare model performance across multiple MRI sequences.

---

## Final Segmentation Structure

After processing, the segmentation dataset has the following general organization:

```text
segmentation_dataset/
├── images/
│   ├── train_BreaDM-Ma-2018_VIBRANT.png
│   ├── train_BreaDM-Ma-2018_SUB2.png
│   ├── val_BreaDM-Be-2112_VIBRANT.png
│   └── ...
└── masks/
    ├── train_BreaDM-Ma-2018_VIBRANT.png
    ├── train_BreaDM-Ma-2018_SUB2.png
    ├── val_BreaDM-Be-2112_VIBRANT.png
    └── ...
```



---

## How to Run the Notebook

### 1. Upload the Dataset

Place the ZIP file in Google Drive at:

```text
MyDrive/BreastDMDS.zip
```

### 2. Open the Notebook in Google Colab

Open:

```text
BreastDM_DataProcessing.ipynb
```

### 3. Run the Cells in Order

---

## Requirements

The notebook requires Python 3 and the following packages:

```text
numpy
matplotlib
Pillow
```

When running in Google Colab, most of these packages are already installed.

The notebook also uses Colab's built-in Google Drive integration:

```python
from google.colab import drive
```

---

