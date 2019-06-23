# Multiple-ML-Models-on-HiRes-Images
## Environment Setup for Linux
### Install Project Dependencies
The required packages for this project can be installed using the following command within a virtual environment.
```
pip install -r requirements.txt
```
Alternatively the packages can be installed individually using the following commands.
```
pip install numpy
pip install tensorflow==2.0.0a0
pip install opencv-python
pip install scikit-image
pip install pyyaml
```
### DeepLabv3+
The relevent files 'model.py', 'load_weights.py' and 'extract_weights.py' are included in this repository for convenience. However, the full source code for the DeepLabv3+ implementation used in this project can be found here:  <br />
https://github.com/bonlime/keras-deeplab-v3-plus
### AlexNet using Caffe framework
The AlexNet pretrained model used in this project is available on the Caffe deeplearning framework which can be found here: <br />
https://github.com/BVLC/caffe<br />
Once the caffe repository is cloned the file 'Makefile.Config' included in this repository can be copied in.
```
cp PATH_TO_Multi-ML-Models-on-HiRes-Images/Makefile.Config PATH_TO_CAFFE/Makefile.Config
```
Modifications can be made if needed. <br />
To install pycaffe change to the caffe directory and use the following command.
```
make pycaffe
```
Finally the CAFFE_ROOT variable in 'classifyAlexNet.py' will need to be changed to match the path to your caffe installation.
Open the 'classifyAlexNet.py' file and change line 13 from:
```
> CAFFE_ROOT = ""
```
to
```
< CAFFE_ROOT = PATH_TO_CAFFE
```
where PATH_TO_CAFFE could be for example '/home/USER/caffe/'.
The project implementation can now be run with AlexNet but further downloads for pre-trained weights and label names will be required. The program will prompt for these downloads when run with the AlexNet option.
## Run
Usage: 
```
$ python main.py [options] [src-img-path] [des-img-path]
```
### Options
#### Segmentation Method
-s SEGMETHOD, --segMethod=SEGMETHOD
* Segmentation Method: Method used during segmentation stage.
* Choice of [cedd] or [legt].
#### Classification Model
-c CLASSIFYMODEL, --classifyModel=CLASSIFYMODEL
* Classification Model: Model used during the classification stage. 
* Choice of [alexnet] or [deeplab].
#### maxRes
-m MAXRES, --maxRes=MAXRES
* maxRes: The maximimum resolution used for each ROI image during the classification stage.
#### Test Option
-t TEST, --test=TEST
* Test type: Determines test type for autotesting or dynamic testing. 
* Choice of [single], [folder] or [dynamic].
#### Display Option
-d DISPLAY, --display=DISPLAY
* Display: Display intermediate image processing steps in the segmentation stage. 
* Choice of [none], [output], [segment] or [all].
### Argurments
#### src-img-path
The source image path is the first argument and specifies where images are read from. The src-img-path should point to a directory if test option 'folder' or 'dynamic' is chosen, otherwise if test option 'single' is chosen it should point to a file.
### des-img-path
The destination path is the second argument and specifies the location where output image files are saved. It should always point to a directory.
### Demo
#### Running with DeepLabv3+
```
python main.py -c deeplab src-img-path des-img-path
```
#### Running with AlexNet
```
python main.py -c alexnet src-img-path des-img-path
```
#### Using the Local Entropy and Global Thresholding (LEGT) segmentation method
```
python main.py -s legt src-img-path des-img-path
```
#### Using the Canny Edge Detection and Dilation (CEDD) segmentation method
```
python main.py -s cedd src-img-path des-img-path
```
#### Autotest a full directory
```
python main.py -t folder src-img-path des-img-path
```
#### Autotest a single file
```
python main.py -t single src-img-file-path des-img-path
```
#### Hide output file display
```
python main.py -d none src-img-file-path des-img-path
```
#### Display segmentation steps only
```
python main.py -d segment src-img-file-path des-img-path
```
#### Display segmentation steps and output
```
python main.py -d all src-img-file-path des-img-path
```

