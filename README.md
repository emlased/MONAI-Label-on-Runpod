# Using MONAI Label with 3D Slicer

## Steps
1. Set up RunPod
2. Set up MONAI Label Server
3. Set Up 3D Slicer
4. 
## Set up RunPod
1. Create a RunPod account: Sign up and log in to [RunPod](https://www.runpod.io/).
2. Deploy a pod. 
3. In a local terminal, login to RunPod using ssh

```
ssh <your_pod_username>@<your_pod_ip>
```
You can find <your_pod_username>@<your_pod_ip> under the "Connect" tab.
## Set up MONAI Label Server
1. Within RunPod instance, create and activate a python virtual environment. This helps prevent issues with software dependencies.

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

To use the DeepEdit model from the Radiology app, run the following. If you will be uploading dicom files directly to 3DSlicer you can leave the location for the studies command as "dataset". 

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
1. Install and run 3D Slicer.
2. Install the MONAI Label plugin for 3D Slicer. 
3. To use 3D Slicer with MONAI Label, you first need to connect your local machine to the MONAI Label server running on the RunPod instance. To do this go to the Connect button on the RunPod and select "Connect to HTTP Service [Port 8000]". This will open the MONAI Label App in a web browser.
4. Hit the Refresh button on 3D Slicer to connect to the MONAI Label Server.
5. To load a study, hit the NExt Sample button.

## Active Learning with MONAI Label
1. You want to begin by using the Segment Editor tab to define labels for each segment you want to create. 
2. You must begin labelling data from scratch. This can be done using a paintbrush or basic tools built in to 3D Slicer; however MONAI Label provides a scribbles machine learning algorithm to speed this up. [Here](https://www.youtube.com/watch?v=Wxmo7MVc7hI&list=PLtoSVSQ2XzyD4lc-lAacFBzOdv5Ou-9IA&index=4) is a video tutorial for using the scribbles algorithm. After you finish annotating a study, hit the Submit Label button to submit the segmentation to the server.
3. After submitting your first annotation, you can begin training a model to automate segmentation of the rest of your data. [Here](https://www.youtube.com/watch?v=3HTh2dqZqew&list=PLtoSVSQ2XzyD4lc-lAacFBzOdv5Ou-9IA&index=3) is a tutorial for using Active Learning within Monai Label. Hit the Train button under the Active Learning section. Then you can continue annotation of additional studies while the model is being trained. As you submit labels, the model will improve and the accuracy bar in the Active Learning section will increase. If you change the Strategy dropdown from "Random" or "First" to "Epistemic", MONAI Label will change the order so that studies the model has the hardest time with will be annotated sooner, speeding up the improvement in model accuracy.  
