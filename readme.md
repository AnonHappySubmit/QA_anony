# QA Processing tool

This repository includes the QA processing for medical image sessions. There are some experiences and suggestions to deal with the files downloaded from *****. The function modules are located in *QA_tool.py*, and the examples in *test_example.py*. 

**The respository is currently anonymous. More details will be provided after the work get published**.

## 1. Big Picture

<img src="/example_image/ProblemFig.png" width="600">
Above is the problem figure: Potentially misleading interpretation of a low-quality image. (a) The complete image with a pulmonary nodule highlighted within the red circle. (b) The same image with a few slices missing. The image from (b) can be misinterpreted as lacking a pulmonary nodule to human reviewers and AI algorithms alike.

<img src="/example_image/Framework.png" width="600">
We introduce this QA framework, which includes objective assessment on DICOM and NIfTI, subjective assessment with batch.

<img src="/example_image/MiniStudy.png" width="800">
We also show a mini machine learning study here. The comparison of predicted cancer probability of complete scan and slices-lost scan. The scans are all from cancer patients. The slices-lost is caused by data transfer problems, and re-transfer make the scan complete.

## 2. Steps for QA

** Here should the instructions downloading data from our database, which are not shown in anonymous submission **

The following steps code can be found at *test_example.py*. 

**Step 1**: Check the instance number of DICOMs (need to read DICOM header) if the instance number can match the number of DICOMs for the session. 

![Instance Number Failing case](./example_image/InstanceCheck.png)
<p align="center"> Fig. 1 Instance Number Failing case (click this image can see details)  </p>


**Step 2**: Check the slice distance if DICOMs (need to read DICOM header) to avoid those sessions lose slices. 

![Slice Distance Failing case](./example_image/SliceDistance.png)
<p align="center"> Fig. 2 Slice Distance Failing case (click this image can see details)  </p>



**Step 3**: filter some sessions with very limited slices (e.g. 1 or 2) but still can pass the instance number check. 

![Few Slices case](./example_image/FewSlices.png)
<p align="center"> Fig. 3 Few Slices case (click this image can see details)  </p>

**Step 4**: Find out scans unreasonablely extend Region of Interest. 

![Out of ROI case](./example_image/PhysicalLength.png)
<p align="center"> Fig. 4 Out of ROI case (click this image can see details)  </p>


**Step 5**: Use the dcm2niix tool to convert DICOMs to NIFTI by:


> dcm2niix -m n -z y -o *output_folder* *DICOM_folder*

if set the -m as n doesn't work, set the -m as y. However, here should be *very careful* to check if generated NIFTI is what you want. 


A example (2290718171-20100301) that the image is good, but dcm2niix -m n cannot successfully convert. Should use -m y.

**Step 6**: use the slicedir tool to visualize a batch of NIFTI files to double check, or use MIPAV to check one by one (time consuming). 

**Step 7**: NIFTI Orientation Check and Resolution filtering.

![Orientation Check](./example_image/orientationCheck.png)
<p align="center"> Fig. 5 Orientation Check failing case (click this image can see details)  </p>

**Step 8**: use the slicedir tool to visualize a batch of NIFTI files to double check. Or use MIPAV / ITKSNAP to check one by one (not recommended, time consuming). 


> slicesdir *NIFTI1_path* *NIFTI2_path* …… *NIFTIn_path*

For *** lab member, you can find the binary slicesdir at /usr/share/fsl/5.0/bin/slicesdir. 

![Orientation Check](./example_image/slicesdirCheck.png)
<p align="center"> Fig. 5 Slicesdir double Check failing case (click this image can see details)  </p>

**FYI**: 


(1) when we check the slice distance in step 2, there is a parameter <slice distance difference> we should define. We can not define the difference to 0, because I find there are good images with slice distance difference. My suggestion is define the tolerance slice distance difference smaller than slice distance but larger than 0. 

(2) Maybe not all the above steps are necessary to judge a session is good or not. For example, based on my experience, I find all the sessions can pass the slice distance check (described in step 1) can always pass the instance number check (described in step 2). However, each function can provide different information for DICOM or NIFTI, and the instance number check would not take much time. 

(3) In my QA experinece, a session is good when one scan is with good quality of this session. 

## 3. Suggestions and Experiences dealing with the file from ***

1. Some file names contain  invalid symbol (such as ')', ',' ) which might cause i/o error when process those images. 

   **My suggestion** is renaming those files before processing. Please see the function  *QA_tool/dcm2nii_project*  for example. 

2. In most of time, the DICOM file under a folder with the name "DICOM". However, in few case, DICOM files are under the folder with name "secondary". 

   **My suggestion** is renaming those files before processing. Please see the function  *QA_tool/dcm_fold*  for example.   

3. Some sessions from *** would have more that one folder that contains DICOM files. What we need is if only there is one folder that contains "legal" DICOM files. 

   **My suggestion** is pick the folder with largest number of DICOMs to process in the first round, and keep the record of the session if it has multiple DICOM folders or not. If yes, manually check other folders if the first round didn't pass the QA. Please see  the function  *QA_tool/dcm_fold*  for example.   

    

