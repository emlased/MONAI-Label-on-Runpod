# Image Segmentation with MONAI Label and 3D Slicer

## Table of Contents

## MONAI Label Installation

### Background
- MONAI Label is a software tool that uses machine learning to automate the image segmentation process. It is run on a server, and the user interacts with the server using a client.
- 3D Slicer is a client that is run on the user's local machine and allows for visualization and segmentation of imaging data.
- RunPod is a service that allows you to build a temporary server with a high end GPU for performing machine learning computations. A server or “pod” can be attached to a network volume so that stored data persists after the pod is terminated. 

### Deploying a Network Volume
When starting a new project, the first step will be to deploy a network volume for the project. After installing MONAI Label on the network volume and uploading imaging data, the project will be accessible whenever a pod is deployed. 

1. **Create a RunPod account**: Sign up and log in to RunPod. You will need to request access to the UT Academic AI Team account and will receive a URL to link this to your account.

2. **Deploy the Network Volume**: 
   - Click Pods in the toolbar and then + Deploy. 
   - Select the cloud type Secure Cloud. 
   - Select Network Volume and click + Network Volume. 
   - Select a location in the US with a high availability of RTX 4090s. 
   - Name the Volume and increase Volume size based on need.

4. **Deploy the Pod**:
   - Select the Network Volume created in the previous step.
   - Select an appropriate GPU like the RTX 4090.
   - For Instance Pricing, use On-Demand Non-Interruptable.
   - Select Edit Template and expose port 8000 under Expose TCP Ports.
   - Click Deploy On-Demand to start the pod.
   - Note that after creating a pod, RunPod will continue to charge credits until the pod is terminated. Remember to terminate pods when not in use. Another pod can be quickly created and data will persist on the Network Volume.

6. **Connect to the Pod**: 
   - Click the down arrow to expand your RunPod. 
   - Click the Connect button, then Start Web Terminal, and Connect to Web Terminal. 
   - Alternatively, use the SSH link provided by RunPod to access through a terminal on your local computer.

### Installing MONAI Label
1. **Set Up Environment**: Run the following in the RunPod terminal to create a virtual environment and install MONAI Label. This will also create a script deepgrow.sh that can be used to start a MONAI Label server with the DeepGrow model loaded. Information on how to load alternative models will be included in the appendix.

    ```bash
    # Create and activate python virtual environment in permanent workspace directory
    cd /workspace &&
    python -m venv venv &&
    source /workspace/venv/bin/activate &&

    # Install monailabel and radiology app inside environment
    cd /workspace/venv &&
    pip install monailabel &&
    monailabel apps --name radiology --download --output . &&
    mkdir /workspace/venv/dataset &&

    # Create script deepgrow.sh
    cd /workspace &&
    echo -e 'source /workspace/venv/bin/activate\nmonailabel start_server --app /workspace/venv/radiology --studies /workspace/venv/dataset --conf models "deepgrow_2d,deepgrow_3d"' > deepgrow.sh
    ```

2. **Edit the Default Labels**: Use the following command to change the default labels to nodule_1, nodule_2, nodule_3. DeepGrow uses a 2D and 3D model so two config files must be changed. 

    ```
    # Define the new labels
    NEW_LABELS='        self.labels = [\n            "nodule_1",\n            "nodule_2",\n            "nodule_3"\n        ]'

    # Update the first config file
    CONFIG_FILE="/workspace/venv/radiology/lib/configs/deepgrow_3d.py"
    sed -i "/self.labels = \[/,/]/c\\$NEW_LABELS" "$CONFIG_FILE"

    # Update the second config file
    CONFIG_FILE="/workspace/venv/radiology/lib/configs/deepgrow_2d.py"
    sed -i "/self.labels = \[/,/]/c\\$NEW_LABELS" "$CONFIG_FILE"
    ```
## Data Preparation

At this point, you have MONAI Label installed on the Network Volume and a script prepared to start the server. There are two ways that your MONAI Label server can access imaging studies: via local storage or remote DICOM storage. Which method is best will depend on the current formatting of your imaging data and if you plan to use MONAI Review.

### Local Storage
Notice the deepgrow.sh script points to the /workspace/venv/dataset directory for imaging studies. This method requires that imaging studies are loaded onto the network volume. MONAI Label will only recognize locally stored data in the following formats: 
`
.nii.gz .nii .nrrd
`
Based on our DeepGrow script, data should be placed into the dataset directory as previously mentioned. If segmentation files are available, they should have the same file name as their corresponding image file and they should be placed into the directory /workspace/venv/dataset/images/final. Using locally stored data is advantageous because the lag in loading studies into 3D Slicer is substantially lower and the MONAI Review platform does not work with studies stored on a DICOM server. 

### Remote Storage
In the start server script after the --studies modifier, the web address of a DICOM server can be included instead of a local directory. The following is an example of how to link a DICOM server in the start server command.
```
-- studies http://20.55.49.33/dicom-web
```
Studies can be uploaded to a DICOM server (such as an Orthanc server) in DICOM format. When MONAI Label creates segmentation files, they will be uploaded to the DICOM server as DICOM segmentation files. 

### Converting DICOM to NIFTI
If you have a DICOM data set and you want to store data locally, you will need to convert to NIFTI files. This is easy to do with the [dcm2niix tool](https://github.com/rordenlab/dcm2niix). After downloading the tool, just use the dcm2niix followed by the directory you wish to convert from DICOM to NIFTI.
```
dcm2niix dicom_directory
```

### Data transfer
Transferring data to and from RunPod requires the [runpodctl tool](https://github.com/runpod/runpodctl). After downloading this tool on your local machine, you can easily transfer files to and from a pod. If you are transferring a directory titled dataset, use the following code and the machine hosting the data:
```
runpodctl send dataset
```
The output will look something like this: 
```
Sending 'dataset' (5 GB)
Code is: 8338-galileo-collect-fidel
On the other computer run

runpodctl receive 8338-galileo-collect-fidel
```
Simply navigate to the directory in which you want the data to be placed on the runpod and run the last line.
```
runpodctl receive 8338-galileo-collect-fidel
```

### Model Portability
Now that you have installed MONAI LAbel and prepared you data locally or on a DICOM server, you are ready to start a server. You can start labelling with a generic pretrained model provided by MONAI Label like deepgrow or deepedit. In the case where you have either fine tuned or trained from scratch your own model, you can save that model and upload it to future pods using the runpodctl tool. MONAI Label stores models in the directory /workspace/venv/radiology/model. MONAI Label also includes performance metrics like a DICE score in this directory. 

## Starting the MONAI Label Server
Starting the MONAI Label server simply requires running the bash command beginning with
```
monailabel start_server
```
There is more detail on the start_server command in the appendix. Earlier we wrote the script deepgrow.sh to contain the bash code to start the server. Run the following to execure the script.
```
bash /workspace/deepgrow.sh
```

### Accessing the MONAI Label API
- **Find the API URL**: Click the RunPod Connect button and then TCP Port Mappings to get the URL. Use `http://` + `Public IP` + `:` + `External port`. For example, `http://69.145.85.93:30135`. Searching this in a browser while the server is running will bring up the API page. This is the url that will be input into 3D Slicer.

### Installing MONAI Label Plugin for 3D Slicer
1. **Install 3D Slicer**: Download and run 3D Slicer on your local machine.

2. **Install MONAI Label Plugin**: Navigate to the Extensions Manager in 3D Slicer and install MONAILabel.

3. **Add MONAI Label Plugin to Favorites**: From the Edit tab in 3D Slicer, navigate to Application Settings and click the Modules tab. Drag the MONAI Label module into the Favorite Modules section to add it to the toolbar at the top of 3D Slicer.

4. **Configure MONAI Label Plugin**: Click the MONAI Label tab in Application Settings. Change Client/User-ID to a username of your choice. Check the box for Developer Mode.

5. **Connect to MONAI Label Server**: Start the MONAI Label plugin and paste the API URL into the MONAI Label Server box. Click Refresh. This will load the model specified by your RunPod MONAI Label server.

6. **Load Studies**: Use the Next Sample button to load studies from the DICOM server or MONAI Label server.

### Re-deploying a RunPod
Since everything we installed will remain on the volume network, if you done using MONAI Label, terminate the pod. When you need to use MONAI Label again, create a  new pod and connect the same network volume. Everything you have set up will be available.

## Labeling Strategies

### Manual Labelling in 3D Slicer
- **Tools**: Slicer has many built in tools to help create segmentation masks but two that are especially useful are the threshold and smooth. 

- **Tresholding**: Use the threshold tool to identify lung nodule tissue for segmentation based on Hounsfield unit windowing. The option titled Use for masking allows you to combine basic tools like a paintbrush or perimeter drawing with the threshold tool. For the segmentation of simple lung nodules from scratch, the paintbrush with the Sphere brush option selected can be used. This will automatically segment all tissue inside of a sphere within the threshold window. For more complex nodules, the draw tool can be applied to each axial slice just outside the perimeter of the nodule so that all of the nodule is included in the segmentation. Segmentation is usually easiest to complete in axial views but it is useful to review sagittal and coronal views for quality of segmentation.

- **Smoothing**: The smooth tool is useful for closing small holes in the segmentation left be the threshold tool or smoothing out rough edges of at the perimeter of the segmentation. 

### Labelling with MONAI Label
- **Automated and Semi-automated Segmentation**: MONAI Label works well out of the box for some specific labelling tasks including spleen segmentation, vertebra segmentation, and various organ segmentations found in the total body segmenter model. When choosing a model for a new labeling task, two good options are DeepGrow and DeepEdit. Both can function as semiautomatic segmentation strategies that generate segmentation masks in response to clicks in the region of interest. DeepEdit has the additional advantage of performing fully autonomous segmentation. While both models come pretrained on medical imaging data, DeepEdit is specifically pretrained to label the spleen, kidneys, liver, stomach, aorta, and IVC. For a new labeling task, the pretrained DeepEdit expects these 7 labels and therefore must be trained from scratch. Training from scratch requires substantially more data than simply fine tuning. DeepGrow requires 10-20 annotated studies to produce excellent output while DeepEdit will require several times that. One approach to labeling a large dataset could be using a fine tuned DeepGrow initially to expedite the labeling process and after labeling 100 or so studies, training a fully autonomous model like DeepEdit from scratch.

- **Active Learning**: As noted earlier, DeepGrow and DeepEdit can both be fine tuned. DeepGrow is much faster to fine tune. It is recommended to train a model every 10 studies with DeepGrow or every 20-30 studies with DeepEdit. With each batch there should be a significant improvement in the DICE score reported in the RunPod MONAI Label server log. Model run statistics can be found at /workspace/venv/radiology/model. Depending on how accurate the initial model is, it may be more productive to complete several batches of initial segmentations by hand rather than making corrections with Slicer tools. Training parameters like the number of epochs and the training/vallidation data split can be changed in the Options tab.

## Examples

### Lung Nodule Segmentation

This example uses a DeepGrow model that was trained on 30 annotated studies. First a study was loaded using the Next Sample button. Then under the SmartEdit/Deepgrow tab, the model was changed to deepgrow_pipeline. The deepgrow pipeline is a combination of the DeepGrow 2D and 3D models. Since the pipeline relies on both models, both models were separately trained. When the study was loaded, a foreground point was added to the approximate center of the nodule. 
![DeepGrow 1](images/DeepGrow_1.png)

Here is the initial output of the DeepGrow model. Notice how the model had difficulty annotating the superior aspect of the nodule. A strength of DeepGrow is that model predictions can be improved with the addition of more foreground or background points. Here a point was added to the superior part of the nodule. 
![DeepGrow 2](images/DeepGrow_2.png)

As you can see, the model has done a better job including parts of the nodule that were previously missed. However, it is also adding unwanted segmentation to the vasculature. A good tool that can be used in the 3D viewer is the scissors tool. This will let you draw a shape that is extruded through the 3D space. You can select options like Erase Inside or Erase Outside. Here the scissors were used with Erase Inside to cut out bits of nodule expanding into vasculature.
![DeepGrow 3](images/DeepGrow_3.png)

Finally, afer applying a smoothing filter, this is the output. It includes only the main lung nodule and using this method took approximately 1 to 2 minutes to segment. 
![DeepGrow 4](images/DeepGrow_4.png)

Since this output is acceptable, it can be submitted to the server using the Submit Label button. After submitting 10 or so labels, the model can be fine tuned. First select DeepGrow 2D in the models section under the Active Learning tab. Then select the Train button. The MONAI Label server logs on RunPod will output the progress of training and the DICE score after each training epoch. Then do the same with the DeepGrow 3D model.

## Quality Control with MONAI Review

### For Labelers
After labelling a set of studies, navigate to the MONAI Review plugin. Select Basic Mode. The Server Url should autopopulate the MONAI Label server URL from the main plugin, then hit Connect. Next navigate to Data Set Explorer, and select your username. Click Load to load studies that you have labeled. For each annotation, you will mark the study as Easy, Medium, or Hard difficulty. These ratings will be used to help guide the reviewers. After rating the study, hit Next to load another.

### For Reviewers
Under the Reviewer's mode tab, you first need to type out a reviewer name. After you enter a name, clock Connect. Next navigate to Data Set Explorer and select an Annotator to review their studies. You would think that you could select "All" for the Annotator but that will not work. Since you are reviewing studies that have been segmented but not approved, make sure the segmented box is selected and the approved box is not selected. Press Load. A labeled study should populate and the progress bar should show how many studies are currently loaded. You can review the study in slicer and determine if the study needs revision. If it is acceptable, press Approve under Data Evaluation. If it needs revision, press the Start Label Edit button. This will allow you to use the Slicer segmentation toolbar to make edits to the label. You can also add comments to the label, change the difficulty level assigned to the label, or flag the label. When you have completed your changes, press Save as New Version to save the changed label. This will save the new label in the directory /workspace/venv/dataset/labels/version_1. 

## Appendix

### MONAI Label Server Configuration Options

The MONAI Label platform allows for the use of several powerful deep learning tools. These tools are either fully automated or interactive. Examples of fully automated tools include the total body segmenter, the spleen segmenter, and the vertebrae segmenter. DeepGrow is an interactive tool that will automatically select tissues of interest after the user adds one or more clicks to the area. DeepEdit is a model that both automatically identifies tissues and can be refined with the addition of clicks by the user like DeepGrow. DeepEdit comes pre-trained to segment select organs; however an untrained model can be used and refined to meet a user’s needs. To use these tools in Slicer, they must be specified in the command used to start the MONAI Label server. Earlier, we created a script that imports the model deepgrow using the following code.

```
echo -e "source /workspace/venv/bin/activate\nmonailabel start_server --app /workspace/venv/radiology --studies http://20.55.49.33/dicom-web --conf models deepgrow_2d,deepgrow_3d" > deepgrow.sh
```

This code creates a script containing the following text

`monailabel start_server --app /workspace/venv/radiology --studies http://20.55.49.33/dicom-web --conf models deepgrow_2d,deepgrow_3d`

Notice that after the start_server command there are multiple configuration variables including --app, --studies, and --conf models. These values can be modified to change how the server functions. --app defines what MONAI Label application will be used. This tutorial uses the radiology application, but MONAI Label has additional models available in the pathology and MONAI Bundle applications. --studies defines the location of the imaging data. The included url points to a DICOM server but you can also point to a directory on RunPod local to the MONAI Label server. --conf lets you specify what models to upload or specific model parameters. The following tables list the key parameters used when starting the server and their arguments as well as the models available for use in the radiology application. [This](https://github.com/Project-MONAI/MONAILabel/tree/main/sample-apps/radiology) is a good resource for viewing the available settings for each model.

| Config        | Value                  | Description                                          |
|---------------|------------------------|------------------------------------------------------|
| --app         | path/to/radiology      | location of radiology app directory on server        |
| --studies     | path/to/studies        | location of studies on server or URL for DICOM server|
| --conf models | model_name1,model_name2| imports listed models separated by commas            |

| Models                                                        | Description                                    |
|---------------------------------------------------------------|------------------------------------------------|
| deepedit                                                      | imports DeepEdit interactive/automated model   |
| deepgrow_2d, deepgrow_3d                                      | imports DeepGrow interactive segmentation model|
| segmentation                                                  | imports segmentation automated model           |
| segmentation_spleen                                           | imports model for spleen segmentation          |
| localization_spine, localization_vertebra, segmentation_vertebra | imports vertebral segmentation model           |

### Troubleshooting
- **The MONAI Label server times out**: Press the refresh button to reconnect. If a study is loaded, continue segmentation after refreshing.

- **MONAI Label plugin glitches**: Restart the MONAI Label server to resolve random errors.

### Other Resources
- [MONAI Label Tutorial PDF](https://help.rc.ufl.edu/mediawiki/images/d/da/2022-Feb-MONAILabel-Tutorial-UF.pdf)
