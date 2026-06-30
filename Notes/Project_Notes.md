6/28/2026

**Creating the Dataset
**

The dataset given from the link is very obscure and difficult to format. I wonder why they didnt just give me the correct useable dataset. I need the dataset to be in this format: 

<img width="512" height="366" alt="image" src="https://github.com/user-attachments/assets/60480a30-1247-4220-86d7-c22070cd1a6d" />

Downloaded dataset currently looks like this 

<img width="1928" height="244" alt="image" src="https://github.com/user-attachments/assets/e4f44f0a-b4ca-4082-b14e-5f8d9a1ce65a" />

This is obviously not what I want but it is a little cool that there is the classification dataset, 2d segmentation and 3d segmentation, I need the 2d seg or just seg. I will just drag files into folders using my mouse, I'm sick of writing code to format files :)

Workflow:
Copy these into a folder that is not the zip folder (time estimated at ~2 hours)> upload to google drive > mount > and then unfortunately I will have to write python to properly format the files :(



6/29/2026

**Trying to get the dataset to work**

I have encountered so many issues with the dataset it is insane. If this were a project submitted to a professor I think it is safe to assume that they recieved a D as their grade for their work. The requirements for their formatting do not at all match what they give you and they do not readily supply a data splitting function. I will have to create one myself!
I am currently downloading the dataset to google drive so that I can mount it, but the estimated time to download is hours, and I believe it will surely take that long.
I tried downloading the dataset, it includes 82000 items. this was too much and I quickly ran out of space and had to delete everything. deleting everything is taking forever. Once I delete everything I think I am going to convert to a zip file, unfortunately I am not sure if that will be able to be downloaded either. This is a pretty hefty dataset. 

<img width="802" height="388" alt="image" src="https://github.com/user-attachments/assets/e418d7d2-cc72-4557-87cc-ffccdb701ea9" />

So apparently, for segmentation, the data was in seg and seg3d (which would have been nice if it was explained in the readme). I suppose that because its technically 3 dimenional, not in an x y z sorta way but it just has another dimension to it even though it's really just a 2d image. It appears each file contains 8 slices of 369x369 pixels. The displayed central slices confirm this. The sample files VIBRANT+C2.npy from both the images and labels directories share the same name, strongly suggesting that files with identical names across these two folders represent corresponding image and mask pairs.

Here's what BeastDM_DataInspecToProces does for me right now: 

Unzipped Dataset: The BreastDMDS.zip file was unzipped to /content/BreastDMDS_unzipped.

Dataset Reorganization Attempt (and correction): Initially, there was an attempt to classify files into 'B' and 'M' folders, which was later identified as not the user's primary goal for this task and also caused file overwrites. We then corrected this by reverting the DATASET path to the original unzipped location.

NPY File Inspection: inspected sample .npy files from the seg3D/val/images and seg3D/val/labels directories. This revealed that the data was float64, ranged from 0.0 to 255.0, and consisted of 3D volumes (e.g., (369, 369, 8)), with images and masks sharing the same base filenames.

Conversion and Reorganization to PNG: Based on the inspection, we implemented code to:
Create new /content/segmentation_dataset/images and /content/segmentation_dataset/masks directories.

Iterate through all seg3D splits (train, val, test).

Load each .npy file.

Extract the central 2D slice from 3D volumes.

Scale the data to 0-255 and convert it to uint8.

Save the processed data as .png files using a unique naming convention (split_patientID_originalFileName.png) to prevent overwrites.
Verification: We confirmed that the new dataset structure is correct and that the files are appropriately organized, with 699 image files and 717 mask files created.

The DATASET variable now points to /content/segmentation_dataset



6/30/2026

**Attempting to make the split function**

This is currently the structure of the file, but its not even all that we need because the corresponding mask data for segmentation task is in seg3d because it has the 8 layer dimension that I did not account for. That being said this is the current structure of seg: 

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

Currently the code in this ipynb file does:

Setup and Unzipping // It initializes by importing necessary libraries, mounts Google Drive, and unzips the main dataset from a .zip file.
Initial Data Inspection // The code then looks at the directory structure of the unzipped dataset and loads a sample .npy file to understand its data type, shape, and value range. This makes it easy to see how far I am and how far I need to go to make it the desired structure.
Classification Data Organization // Processes .npy files to organize them into distinct 'Benign' and 'Malignant' directories, preparing the data for a classification task.
Segmentation Data Processing and Visualization // Identifies 3D image and mask .npy volumes from the segmentation part of the dataset, extracts a central 2D slice from each, and displays these slices for visual inspection.
Segmentation Data Conversion // Finally, it converts these extracted 2D slices from .npy format into .png image files, scaling their pixel values, and saving them into separate 'images' and 'masks' directories for segmentation tasks.

All the current cells listed: 
(copy and pasted)
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

We now have this structure for data: 

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
