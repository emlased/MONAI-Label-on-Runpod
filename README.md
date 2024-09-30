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
## 1. Deploying a RunPod
1. Create a RunPod account: Sign up and log in to [RunPod](https://www.runpod.io/). You will need to request access to the UT Academic AI Team account and will receive a url to link this to your account.
2. Click **Pods** in the toolbar and then **+ Deploy**. Next you will configure the pod. Select the cloud type **Secure Cloud** and change to **Community Cloud**. Then select the location **Any** and change to **US - United States**. Then change the internet speed from **Med** to **High** or **Extreme**. Checl the box for **Public IP**. Finally you will select a GPU. It should have at least 12GB VRAM. The **RTX 4090** is a good option. Remember to stop the machine when not in use because the account will be charged by the hour.
![Image 1](images/tutorial_1.png)
3. When you scroll down, you will see a button **Edit Template**. A recommended starting point is 10GB for the **Container Disk** and 50GB for the **Volume Disk**. You can increase the volume disk later but once increased it cannot be decreased without restarting from scratch. The container disk is temporary disk space that is deleted any time the pod is stopped and restarted. The volume disk is permanent and stored in the /workspace directory. 
4. While on the **Edit Template** section, you want to expose port 8000 so that the pod can communicate with your computer. You can do this under **Expose TCP Ports** by replacing the text with 8000. You can delete the text under **Expose HTTP Ports** or leave the port 8888 exposed.
![Image 2](images/tutorial_2.png)
5. Click **Deploy On-Demand** to start the pod.
6. Under your RunPod, click the down arrow to expand. Click the **Connect** button. Then click the **Start Web Terminal** button and **Connect to Web Terminal**. Alternatively the SSH link provided by runpod can be used to access RunPod through a terminal on a users local computer. This will require you to generate an SSH key and upload through the RunPod Settings. 
![Image 3](images/tutorial_3.png)
![Image 4](images/tutorial_4.png)
