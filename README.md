# Using MONAI Label and 3D Slicer to Annotate Imaging Studies
This document will discuss a workflow for using MONAI Label with 3D Slicer to annotate medical imaging studies. 
##Table of Contents
1. Setting Up MONAI Label on RunPod
2. Labeling Strategies
3. MONAI Review
4. Lung Nodule Example
5. Useful Resources
# Setting Up MONAI Label on RunPod
## Background
MONAI Label is a software tool for using machine learning to automate the image segmentation process. It is run on a server and the user interacts with the server using a client. 3D Slicer is a client that is run on the user's local machine and allows for visualization and segmentation of imaging data. RunPod is a service that allows you to build a temporary server called a pod with a high end gpu for performing machine learning computations. Orthanc is software that is used to create a dicom server that stores data for imaging studies. MONAI Label will communicate with an Orthanc dicom server to pull studies and push segmentation files. This tutorial describes how to build a MONAI Label server on a RunPod. 
