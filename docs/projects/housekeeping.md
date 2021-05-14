# Basic Housekeeping

For most users data usage often boils down to the software you use to analyze and manipulate images and how you are going to work with them. So here are going to do some housekeeping and setup depending on which tools and setup you are most comfortable with

#### 1) Google Earth Engine(GEE) Command Line Interface(CLI) Setup
This assumes that you have registered for a Google Earth Engine account but also installed its client. Incase you have missed it go to their [main reference page for installation of their python client](https://developers.google.com/earth-engine/command_line). Since you can consume Earth Engine using both Javascript(in browser) and Python(locally) the interaction would depend on the scale of your tasks and what you wish your achieve as your end result. Once installed make sure you authenticate your earth engine client and then your CLI should give you the following options

<center>![Google Earth Engine](/images/ee_cli.JPG)</center>

#### 2) Adding additional Images
For the simplest users getting images into GEE begins with the Image upload tool located inside GEE. Once you have added the filename you can edit additional metadata such as start time, cloud cover information if you have that from the metadata file among other things. This tool does not have a way for you to ingest any metadata automatically so it has to be fed manually.

The image name is automatically filled in with the filename that you select when uploading.

![upload_gee](https://user-images.githubusercontent.com/6677629/118185789-8ccb4c80-b402-11eb-8ce8-0adc27d4e5fd.gif)

Note you cannot select more than one image and upload as a single image if they overlap each other. To handle which we have the concept of image collections. Where you can upload many images. To import images into collections, you have to either import them manually as images first and then copy them into the collection one by one or for now use an external tool to help such as using the Google Earth Engine CLI.

Incase you have a Google Cloud Storage bucket you can also push images automatically to be ingested into GEE. Though this requires interaction with gsutil and starting ingestion function for each image. The GEE guide for image ingestion can be [found here](https://developers.google.com/earth-engine/image_upload)
