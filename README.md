# Overview 

This repository contains the code and data for a deep learning pipeline designed to perform slice-to-volume (S2V) 
registration between intraoperative Ultrasound (US) and pre-operative Magnetic Resonance Imaging (MRI) volumes. The primary 
goal is to predict the 3D position and orientation (Z-translation, X-rotation, Y-rotation) of a given 2D US slice within the 
coordinate system of a 3D MRI volume.

This registration is crucial for applications like image-guided neurosurgery, where compensating for brain shift during surgery 
is necessary to maintain the accuracy of navigation systems based on pre-operative scans.

# Contents

- **Raw Data (RawData/):** Contains raw 3D Ultrasound (US) and 3D MRI (FLAIR) volumes in NIfTI format (.nii.gz) from RESECT database [1]. 
Each pair represents a single patient case.

- **Pre-processing script (Required_preprocessing.ipynb):** The  

- **Pre-processed Data (INPUT/):** The output of the pre-processing script. This consists of 3D US and MRI pairs that are co-registered and cropped 
to (50x50x50).

- **Script for attempt on slice-to-volume registration (Final-slice-to-volume-registration-US-MRI-script.ipynb):** The main Python script 
implementing the SVR pipeline & training.

[1] Y. Xiao, M. Fortin, G. Unsgård, H. Rivaz, and I. Reinertsen, “REtroSpective Evaluation of Cerebral Tumors (RESECT): A clinical database of pre-operative 
MRI and intra-operative ultrasound in low-grade glioma surgeries,” Med Phys, vol. 44, no. 7, pp. 3875–3882, July 2017, doi: 10.1002/mp.12268.


# Methodology
The core of this project is a multi-stage deep learning pipeline designed for progressive refinement of the registration parameters:

- **Stage 1: Coupled 3DOF Prediction:** A model (CoupledPredictor) takes the 2D US slice and the full 3D MRI volume as input. 
It uses 2D (for US) and 3D (for MRI) encoders to extract features, fuses them, and performs probabilistic regression to predict an 
initial estimate of the (Z, X, Y) parameters and their uncertainty.

- **Stage 2: Z-Slice Refinement:** A classification model (ZRefiner) takes the US slice and predicts the most likely Z-slice index within the MRI 
volume. In the "guided" pipeline, the search space is dynamically narrowed based on the prediction and uncertainty from Stage 1.

- **Stage 3: X-Rotation Refinement:** A pair-wise comparison model (RotationRefiner) with separate US/MRI feature extractors takes the US 
slice and the candidate MRI slice (identified by Stage 2). It predicts the best X-rotation angle from a discrete set (e.g., 0°-20°). 
This stage uses a hybrid loss (CrossEntropy + NCC) for stable training and can employ "active guidance" during inference by only testing angles 
close to the Stage 1 prediction.

- **Stage 4: Y-Rotation Refinement:** Uses the same architecture (RotationRefiner) and hybrid loss as Stage 3. It takes the US slice and the MRI 
slice (now corrected for Z and X) to predict the final Y-rotation angle.

This cascaded approach breaks down the complex 3DOF problem into manageable steps, allowing for both coupled and decoupled attention.


# Contact

Olive Schonfeldt / scholi016@myuct.ac.za / Department of Electrical and Electronic Engineering, University of Cape Town


