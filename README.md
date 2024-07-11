# Using MONAI Label with 3D Slicer
## Notes
MONAI Label is a software tool for using machine learning to automate the image segmentation process. It is run on a server and the user interacts with the server using a client. The client we will use is a program called 3D Slicer. 3D Slicer allows the user to visualize studies and perform segmentation tasks. Imaging data can either be stored locally and uploaded to the MONAI Label server using 3D Slicer or the MONAI Label server can pull data from a DICOM web server. In this guide, we set up a virtual MONAI Label server using the RunPod service and access the server from 3D Slicer installed on a local machine.
## Step by step
1. Set up RunPod
2. Set up MONAI Label Server
3. Set up 3D Slicer
4. Active Learning with MONAI Label
## Set up RunPod
1. Create a RunPod account: Sign up and log in to [RunPod](https://www.runpod.io/).
2. Create an SSH key on your local computer and add this to RunPod. [This tutorial](https://sftptogo.com/blog/how-to-create-ssh-keys-on-windows-10/?gad_source=2&gclid=EAIaIQobChMIypyRk8efhwMVVyvUAR2lyAuDEAAYASAAEgJhG_D_BwE) explains how to create a key in windows 10. In RunPod, navigate to Settings -> SSH Public Keys and paste the SSH key.
3. Deploy a pod. Make sure that you have SSH terminal access checked. Under Edit Template, change Expose HTTP Ports from "8888," to "8888,8000". This allows you to connect to the server with 3D Slicer. You may need to increase your pod storage capacity depending on the size of the data set you are annotating. Changing this later will reset the pod.
4. In a local terminal, login to RunPod using ssh

```
ssh <your_pod_username>@<your_pod_ip>
```
You can find <your_pod_username>@<your_pod_ip> under the "Connect" tab in your pod. This will take you to a section of code you can copy and run in your terminal.
## Set up MONAI Label Server
[This tutorial](https://www.youtube.com/watch?v=8y1OBQs2wis&list=PLtoSVSQ2XzyD4lc-lAacFBzOdv5Ou-9IA&index=1) reviews several methods for installing MONAI Label on a local machine. 
1. Within RunPod instance, create and activate a python virtual environment. This helps prevent issues with software dependencies when installing MONAI Label.

```
python3 -m venv venv
source venv/bin/activate
```

2. Install monailabel. Install radiology app and monaibundle app.

```
pip install monailabel
monailabel apps
monailabel apps --download --name radiology --output apps
monailabel apps --download --name monaibundle --output apps
```

3. Run a monailabel server.

Here is the general format for the command to start a server.

```
monailabel start_server --app ChooseApp --studies dataset --conf model ChooseModel
```

To use the DeepEdit model from the Radiology app, run the following. If you will be uploading dicom files directly though 3DSlicer you can leave the location for the studies command as "dataset". 

```
monailabel start_server --app radiology --studies dataset --conf models deepedit
```

If you are using a DICOMweb server then you should first login using SSH. Then you can pass the data to MONAI Label using this command.

```
monailabel start_server --app radiology --studies http://127.0.0.1:8042/dicom-web --conf models deepedit
```

The radiology app also supports DeepGrow and Segmentation models. The monai bundle app hosts its own models including the lung_nodule_ct_detection model.

```
monailabel start_server --app monaibundle --studies dataset --conf models lung_nodule_ct_detection
```

## Set Up 3D Slicer
1. Install and run 3D Slicer. Install the MONAI Label plugin for 3D Slicer. [This tutorial](https://www.youtube.com/watch?v=KjwuFx0pTXU&list=PLtoSVSQ2XzyD4lc-lAacFBzOdv5Ou-9IA&index=2) shows how to complete the installation and provides an overview of the MONAI Label plugin.
3. To use 3D Slicer with MONAI Label, you first need to connect your local machine to the MONAI Label server running on the RunPod instance. To do this go to the Connect button on the RunPod and select "Connect to HTTP Service [Port 8000]". This will open the MONAI Label App in a web browser.
4. Hit the Refresh button on 3D Slicer to connect to the MONAI Label Server.
5. To load a study, hit the Next Sample button.
## Active Learning with MONAI Label
This guide will use active learning to fine-tune the [DeepEdit model](https://arxiv.org/pdf/2305.10655). 
1. You want to begin by using the Segment Editor tab to define labels for each segment you want to create. 
2. Now you either use a pretrained model (like the lung_nodule_ct_detection model) to auto segment the study or you manually create a segmentation mask. Here we will start by manually creating a mask and use the DeepEdit model to refinethe mask. This can be done using a paintbrush or basic tools built in to 3D Slicer; however MONAI Label provides the scribbles machine learning algorithm to speed this up. [This tutorial](https://www.youtube.com/watch?v=Wxmo7MVc7hI&list=PLtoSVSQ2XzyD4lc-lAacFBzOdv5Ou-9IA&index=4) shows how to use scribbles algorithm. First you select the region of interest in each anatomic plane. Then you draw lines or "scribble" on the object of interest (foreground) and around the object (background). At any point you can use the Show 3D button to see a volumetric rendering of the annotation but this feature should be used sparingly as it slows down 3D Slicer. You can continue adding scribbles and hitting the Update button to improve segmentation. After you finish annotating a study, hit the Submit Label button to submit the segmentation to the server.
3. After submitting your first annotation, you can begin fine tuning a model to automate segmentation of the rest of your data. [This tutorial](https://www.youtube.com/watch?v=3HTh2dqZqew&list=PLtoSVSQ2XzyD4lc-lAacFBzOdv5Ou-9IA&index=3) shows how to use Active Learning to train a new model. First select the DeepEdit model in the Active Learning tab. Hit the Train button under the Active Learning section. Then you can continue annotation of additional studies while the model is being trained. As you submit labels, the model will improve and the accuracy bar in the Active Learning section will increase. If you change the Strategy dropdown from "Random" or "First" to "Epistemic", MONAI Label will change the order so that studies the model has the hardest time with (models with most uncertainty) will be annotated sooner, speeding up the improvement in model accuracy.
4. As the model accuracy improves, you can run the fine-tuned DeepEdit model under the Auto Segmentation tab to automatically create a mask on a new study using your model 
