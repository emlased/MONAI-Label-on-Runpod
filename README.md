# MONAI Label
## Background and definitions
MONAI Label is a software tool for using machine learning to automate the image segmentation process. It is run on a server and the user interacts with the server using a client. 3D Slicer is a client that is run on the user's local machine. OHIF is an alternate client that is run on the MONAI Label server and accessed through the user's browser. RunPod is a service that allows you to build a temporary server called a pod with a high end gpu for performing machine learning computations. Orthanc is a software that is used to create a dicom server that stores data for imaging studies. MONAI Label will communicate with an Orthanc dicom server to pull studies and push segmentation files. This tutorial describes how to build a MONAI Label server on a RunPod. 
## Step by step
1. Set up RunPod
2. Set up MONAI Label Server
3. Accessing the MONAI Label API
4. Set up 3D Slicer
6. Active Learning with MONAI Label
## 1. Set up RunPod
1. Create a RunPod account: Sign up and log in to [RunPod](https://www.runpod.io/). You will need to request access to the UT Academic AI Team account and will receive a url to link this to your account.
2. Next, you can configure a pod for deployment. Click **Pods** in the toolbar and then **+ Deploy**. Next you will configure the pod. Select the cloud type **Secure Cloud** and change to **Community Cloud**. Then select the location **Any** and change to **US - United States**. Then change the internet speed from **Med** to **High** or **Extreme**. Checl the box for **Public IP**. Finally you will select a GPU. It should have at least 12GB VRAM. The **RTX 4090** is a good option. Remember to stop the machine when not in use because the account will be charged by the hour.
3. When you scroll down, you will see a button **Edit Template**. A recommended starting point is 10GB for the **Container Disk** and 50GB for the **Volume Disk**. You can increase the volume disk later but once increased it cannot be decreased without restarting from scratch. The container disk is temporary disk space that is deleted any time the pod is stopped and restarted. The volume disk is permanent and stored in the /workspace directory. 
4. While on the **Edit Template** section, you want to expose port 8000 so that the pod can communicate with your computer. You can do this under **Expose TCP Ports** by replacing the text with 8000. You can delete the text under **Expose HTTP Ports** or leave the port 8888 exposed.
5. Click **Deploy On-Demand** to start the pod.
6. Under your RunPod, click the down arrow to expand. Click the **Connect** button. Then click the **Start Web Terminal** button and **Connect to Web Terminal**. Alternatively the SSH link provided by runpod can be used to access RunPod through a terminal on a users local computer. This will require you to generate an SSH key and upload through the RunPod Settings. 
## 2. Set up MONAI Label Server
The following text text can be copied and run in the RunPod terminal. This will create a virtual environment and install MONAI Label along with its radiology application in the /workspace directory. After running this text once, the RunPod can be stopped and started without needing to complete this step again. It will also create a script named start_server.sh containing all the commands to run the MONAI Label server upon starting the RunPod. 
```
#create and activate python virtual environment in permanent workspace directory
cd /workspace &&
python -m venv venv &&
source /workspace/venv/bin/activate &&

#install monailabel and radiology app inside environment
cd /workspace/venv &&
pip install monailabel &&
monailabel apps --name radiology --download --output . &&

#create script start_server.sh
cd /workspace &&
echo -e "source /workspace/venv/bin/activate\nmonailabel start_server --app /workspace/venv/radiology --studies http://20.55.49.33/dicom-web --conf models deepgrow_2d,deepgrow_3d --conf use_pretrained_model false" > start_server.sh
```
Use the following code to run the script start_server.sh.
```
bash /workspace/start_server.sh
```
The script start_server.sh can be modified when you run the code to initialize the server. The echo command is used to write the script. The command **--studies** is used to define to location of our imaging studies. **http://20.55.49.33/dicom-web** is the location of an Orthanc web server. The command **--conf models** lets you define what models MONAI Label will import for data segmentation. Currently it is set to **deepgrow_2d,deepgrow_3d** which will allow you to use the [deepgrow model](https://github.com/Project-MONAI/MONAILabel/tree/main/sample-apps/radiology#deepgrow). Other models included in the radiology application include deepedit and segmentation. 
## 3. Accessing the MONAI Label API
Before you can connect to the MONAI Label server using your client (3D Slicer or OHIF), you need to know the url for the API. In step 6 of the RunPod setup, you click the **Connect** button. If you click the **TCP Port Mappings** button this will give you the url. You will use **http://** + **Public IP** + **: (colon)** + **External (port)**. An example is **http://38.80.153.61:31333**. This url will not be active until you run the code to start the MONAI Label server. Then your RunPod will connect to its internal port 8000 and send data to the url at http://38.80.153.61:31333. If you paste this url into your browser after starting the server, you will see the MONAI Label API page. 
## 4. Set up 3D Slicer
1. Install and run 3D Slicer. Install the MONAI Label plugin for 3D Slicer. [This tutorial](https://www.youtube.com/watch?v=KjwuFx0pTXU&list=PLtoSVSQ2XzyD4lc-lAacFBzOdv5Ou-9IA&index=2) shows how to complete the installation and provides an overview of the MONAI Label plugin.
3. .....
5. To load a study, hit the **Next Sample** button.
## Active Learning with MONAI Label

......

This guide will use active learning to fine-tune the [DeepEdit model](https://arxiv.org/pdf/2305.10655). 
1. You want to begin by using the Segment Editor tab to define labels for each segment you want to create. 
2. Now you either use a pretrained model (like the lung_nodule_ct_detection model) to auto segment the study or you manually create a segmentation mask. Here we will start by manually creating a mask and use the DeepEdit model to refinethe mask. This can be done using a paintbrush or basic tools built in to 3D Slicer; however MONAI Label provides the scribbles machine learning algorithm to speed this up. [This tutorial](https://www.youtube.com/watch?v=Wxmo7MVc7hI&list=PLtoSVSQ2XzyD4lc-lAacFBzOdv5Ou-9IA&index=4) shows how to use scribbles algorithm. First you select the region of interest in each anatomic plane. Then you draw lines or "scribble" on the object of interest (foreground) and around the object (background). At any point you can use the **Show 3D** button to see a volumetric rendering of the annotation but this feature should be used sparingly as it slows down 3D Slicer. You can continue adding scribbles and hitting the **Update** button to improve segmentation. After you finish annotating a study, hit the **Submit Label** button to submit the segmentation to the server.
3. After submitting your first annotation, you can begin fine tuning a model to automate segmentation of the rest of your data. [This tutorial](https://www.youtube.com/watch?v=3HTh2dqZqew&list=PLtoSVSQ2XzyD4lc-lAacFBzOdv5Ou-9IA&index=3) shows how to use Active Learning to train a new model. First select the _DeepEdit_ model in the **Active Learning** tab. Hit the **Train** button. Then you can continue annotation of additional studies while the DeepEdit model is being fine-tuned. As you submit labels, the model will improve and the accuracy bar in the Active Learning section will increase. If you change the **Strategy** dropdown from _Random_ or _First_ to _Epistemic_, MONAI Label will change the order so that studies the model has the hardest time with (models with most uncertainty) will be annotated sooner, speeding up the improvement in model accuracy.
4. As the model accuracy improves, you can run the fine-tuned DeepEdit model under the **Auto Segmentation** tab to automatically create a mask on a new study using your model 
