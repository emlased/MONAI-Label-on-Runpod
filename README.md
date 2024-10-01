# Using MONAI Label and 3D Slicer to Annotate Imaging Studies
This document will discuss a workflow for using MONAI Label with 3D Slicer to annotate medical imaging studies. 
## Table of Contents
1. Setting Up MONAI Label on RunPod
2. Labeling Strategies
3. MONAI Review
4. Lung Nodule Example
5. Troubleshooting
6. Useful Resources
# Setting Up MONAI Label on RunPod
## Background
* **MONAI Label** is a software tool that uses machine learning to automate the image segmentation process. It is run on a server, and the user interacts with the server using a client. 
* **3D Slicer** is a client that is run on the user's local machine and allows for visualization and segmentation of imaging data. 
* **RunPod** is a service that allows you to build a temporary server called a pod with a high end gpu for performing machine learning computations. 
* **Orthanc** is software that is used to create a dicom server that stores data for imaging studies. 
* MONAI Label will communicate with an Orthanc dicom server to pull studies and push segmentation files. This tutorial describes how to build a MONAI Label server on a RunPod. 
## Step by step
1. Deploying a RunPod
2. Initializing MONAI Label Server
3. Accessing the MONAI Label API
4. Installing MONAI Label Plugin for 3D Slicer
5. Re-depolying a Runpod
## 1. Deploying a RunPod
1. Create a RunPod account: Sign up and log in to [RunPod](https://www.runpod.io/). You will need to request access to the UT Academic AI Team account and will receive a url to link this to your account.
2. Click **Pods** in the toolbar and then **+ Deploy**. Next you will configure the pod. Select the cloud type **Secure Cloud** and change to **Community Cloud**. Then select the location **Any** and change to **US - United States**. Then change the internet speed from **Med** to **High** or **Extreme**. Checl the box for **Public IP**. Finally you will select a GPU. It should have at least 12GB VRAM. The **RTX 4090** is a good option. Remember to stop the machine when not in use because the account will be charged by the hour.
![Image 1](images/tutorial_1.png)
3. When you scroll down, you will see a button **Edit Template**. A recommended starting point is 10GB for the **Container Disk** and 30GB for the **Volume Disk**. You can increase the volume disk later but once increased it cannot be decreased without restarting from scratch. The container disk is temporary disk space that is deleted any time the pod is stopped and restarted. The volume disk is permanent and stored in the /workspace directory. 
4. While on the **Edit Template** section, you want to expose port 8000 so that the pod can communicate with your computer. You can do this under **Expose TCP Ports** by replacing the text with 8000. You can delete the text under **Expose HTTP Ports** or leave the port 8888 exposed.
![Image 2](images/tutorial_2.png)
5. Click **Deploy On-Demand** to start the pod.
6. Under your RunPod, click the down arrow to expand. Click the **Connect** button. Then click the **Start Web Terminal** button and **Connect to Web Terminal**. Alternatively the SSH link provided by runpod can be used to access RunPod through a terminal on a users local computer. This will require you to generate an SSH key and upload through the RunPod Settings. 
![Image 3](images/tutorial_3.png)
![Image 4](images/tutorial_4.png)
## 2. Initializing MONAI Label Server
1. The following text text can be copied and run in the RunPod terminal. This will create a virtual environment and install MONAI Label along with its radiology application in the /workspace directory. After running this text once, the RunPod can be stopped and started without needing to complete this step again. It will also create a script named deepgrow.sh containing all the commands to run the MONAI Label server upon starting the RunPod. 
```
#create and activate python virtual environment in permanent workspace directory
cd /workspace &&
python -m venv venv &&
source /workspace/venv/bin/activate &&

#install monailabel and radiology app inside environment
cd /workspace/venv &&
pip install monailabel &&
monailabel apps --name radiology --download --output . &&
monailabel apps --name monaibundle --download --output . &&

#create script deepgrow.sh
cd /workspace &&
echo -e "source /workspace/venv/bin/activate\nmonailabel start_server --app /workspace/venv/radiology --studies http://20.55.49.33/dicom-web --conf models deepgrow_2d,deepgrow_3d" > deepgrow.sh
```
![Image 5](images/tutorial_5.png)
2. Use the following code to run the script deepgrow.sh.
```
bash /workspace/deepgrow.sh
```
3. When the server is up an running you should see this at the end of the page.
![Image 7](images/tutorial_7.png)
## 3. Accessing the MONAI Label API
Before you can connect to the MONAI Label server using your client (3D Slicer or OHIF), you need to know the url for the API. In step 6 of the RunPod setup, you click the **Connect** button to open up the following window. If you click the **TCP Port Mappings** button this will give you the url. You will use **http://** + **Public IP** + **: (colon)** + **External (port)**. The below example would use **http://69.145.85.93:30135**. This url will not be active until you run the code to start the MONAI Label server. Then your RunPod will connect to its internal port 8000 and send data to the url at http://69.145.85.93:30135. If you paste this url into your browser after starting the server, you will see the MONAI Label API page. 
![Image 6](images/tutorial_6.png)
![Image 8](images/tutorial_8.png)
## 4. Installing MONAI Label Plugin for 3D Slicer
1. Install and run [3D Slicer](https://download.slicer.org/).
2. Install the MONAI Label plugin for 3D Slicer. [This tutorial](https://www.youtube.com/watch?v=KjwuFx0pTXU&list=PLtoSVSQ2XzyD4lc-lAacFBzOdv5Ou-9IA&index=2) shows how to complete the installation and provides an overview of the MONAI Label plugin.
3. Navigate to the MONAI Label plugin and paste the API url from step 4 into the box titled **MONAI Label Server**. Click the refresh button. This should load the model specified by your deepgrow.sh script.
4. To load a study from the DICOM server, hit the **Next Sample** button. Alternatively, you can load samples to your MONAI Label server directly through the DICOM module in 3D Slicer.
## 5. Re-depolying a Runpod
After you create the MONAI Label server on a RunPod, it will persist. Next time you need to use the server all you need to do is run the script deepgrow.sh using the bash command as described previously. RunPod will charge an hourly rate for GPU usage as well as a daily flat rate for storage, so be mindful of keeping many inactive RunPods on the account.
# Labeling Strategies
There are 2 strategies for labeling data. Slicer has a library of labeling tools built in that use simple algorithms for efficient labeling. The MONAI Label plugin adds capabilities for several deep learning tools that can be fine tuned to improve labeling efficiency. 
## Labeling in Slicer
The Slicer segmentation toolbar has a number of built in segmentation methods but 2 that are especially useful are threshold and smoothing. The thresholding tool will select tissues within a specified hounsfield unit window. Especially useful is the threshold masking feature where the user can create a sphere shaped paintbrush where only tissues selected by the threshold tool will be marked. This tool is useful for correcting an automated segmentation that missed part of the object of interest. 
<images>
After using the thresholding tool, the output can have an irregular border so the smoothing tool can help create a cleaner output. 
<images>
## Labeling with MONAI LAbel
The MONAI Label platform allows for the use of several powerful deep learning tools. These tools are either fully automated or interactive. Examples of fully automated tools include the total body segmenter, the spleen segmenter, and the vertebrae segmenter. Deepgrow is an interactive tool that will automatically select tissues of interest after the user adds one or more clicks to the area. Deepedit is a model that both automatically identifies tissues and can be refined with the addition of clicks by the user like deepgrow. Deepedit comes pretrained to segment select organs; however an untrained model can be used an refined to meet a user’s needs.
