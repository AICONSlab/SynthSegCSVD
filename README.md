# **SynthSegCSVD**
## Update : 11/21/2023

**SynthSegCSVD<sub>WMH</sub> :** release candidate now available with full support and regular updates

**SynthSegCSVD<sub>PVS</sub> :** beta version now available with limited support


## **i) Overview**
* **SynthSegCSVD Overview**: SynthSegCSVD is a CNN-based tool for segmentation of white matter hyperintensities (WMH) on FLAIR MRI and perivascular spaces (PVS) on T1 or T2 MRI 
* **Purpose and Development**: SynthSegCSVD was developed to improve segmentation accuracy for large-scale, heterogenous imaging datasets with varying degrees of cerebrovascular disease (CVSD) burden
* **Technical Foundation**: SynthSegCSVD was developed using patient MRI and leverages FreeSurfer's SynthSeg tool (https://surfer.nmr.mgh.harvard.edu/fswiki/SynthSeg), which was developed using synthetic data derived from a generative model conditioned on label maps from full-brain segmentations without PVS/WMH labels
* **Availability**: SynthSegCSVD is distributed as a docker/singularity image
* **Citation Request**: Users of SynthSegCSVD in their research are kindly requested to cite the following in their work:

    * **SynthSeg:** B Billot, DN Greve, O Puonti, A Thielscher, K Van Leemput, B Fischl, AV Dalca, JE Iglesias.Segmentation of brain MRI scans of any contrast and resolution without retraining. Medical Image Analysis, 83, 102789 (2023). 
    * **RORPO:** O Merveille, H Talbot, L Najman, N Passat. Ranking orientation responses of path operators: Motivations, choices and algorithmics. International Symposium on Mathematical Morphology (ISMM), 2015, Reykjavik, Iceland. pp.633-644, 10.1007/978-3-319-18720-4_53. hal- 01168732
    * **SynthSegCSVD:** E Gibson, J Ramirez, LA Woods, R Sommers, N M Ghahjaverestan, CJM Scott, F Gao, AE Lang, C Marras, DP Breen, MC Tartaglia, MA Binns, R B, S Symons, RH Swartz, M Masellis, SE Black, A Moody, ONDRI Investigators, CAIN Investigators, Colleagues from the Foundation Leducq Transatlantic Network of Excellence, Andrew SP Lim, M Goubran. Examining perivascular spaces (PVS) in cerebral small vessel disease (CSVD) using a novel T1-based automated PVS segmentation tool. The International Society of Vascular Behavioural and Cognitive Disorder (VASCOG) Conference, 2023, Gothenburg, Sweden.
    * **SynthSegCSVD<sub>WMH</sub>:** [TODO] Coming soon
    * **SynthSegCSVD<sub>PVS</sub>:** [TODO] Coming soon

 <br>

![Example](synthsegcsvd_example.png)

<br>
 
## **ii) Installation**
>**Singularity Users**
>* download synthsegcsvd_rc05.sif from: 
> https://s3.us-east.cloud-object-storage.appdomain.cloud//cloud-synthsegcsvdrc05/synthsegcsvd_rc05.sif
>
>**Docker Users**
>* download synthsegcsvd_rc05.tar.gz from:
>  https://s3.us-east.cloud-object-storage.appdomain.cloud//cloud-synthsegcsvdrc05/synthsegcsvd_rc05.tar.gz
>* install: `sudo docker load -i synthsegcsvd_rc05.tar.gz`

<br>
 
## **iii) Input Requirements** ##
>**WMH Segmentation:**
>* \<required\> : FLAIR image
>* \<required\> : FreeSurfer's SynthSeg output (version 2.0 with CSF)
>
>**PVS Segmentation:**
>* \<required\> : T1 (or T2) image with isotropic voxel dimensions
>* \<required\> : FreeSurfer's SynthSeg output (version 2.0 with CSF)
>* \<required\> : WMH segmentation (can be an empty image for populations without WMHs)
>* [optional] : coregistered FLAIR 
>* [optional] : RORPO mask 
 

## **iv) RUNNING SynthSegCSVD**
> **Overview**:
>* define variables -- described in the variable setup sections below
>* execute singularity run command -- can be copy/pasted without modification once variables are defined
>* for PVS segmentation, three run commands are provided to generate the optional RORPO input which vary in how WMHs are handled -- see section 3 for details
>
>**Typical workflow:**
>  * coregister FLAIR and T1/T2 images (if using T1/T2)
>  * generate synthseg image (i.e. run: `mri_synthseg --i <FLAIR.nii.gz> --o <synthseg.nii.gz>`) 
>  * run WMH segmentation (section 1)
>  * run RORPO (section 2)
>  * run PVS segmentation (section 3)


<br>
 

# **1. SynthSegCSVD<sub>WMH</sub>**
> ## **1.1 Variable Setup**
```bash
in_dir=$(pwd)
out_dir=${in_dir}
flair_fn=FLAIR.nii.gz
synth_fn=synthseg.nii.gz
sif=${HOME}/synthsegcsvd_rc05.sif
out_fn=seg_wmh.nii.gz
seg_wmh_thr=0.35
skip_mask_and_bias=true
cleanup=true
```

where:

> **in_dir** : full path to input data, or $(pwd) for current directory
>
> **out_dir** : full path to the output data
>
> **flair_fn** : FLAIR input filename 
>
> **synth_fn** : FreeSurfer synthseg input filename (output from "mri_synthseg")
>
> **sif** : full path + filename of singulairty file downloaded above
>
> **out_fn** : wmh segmentation output filename (with file extension)
> 
> **seg_wmh_thr** : threshold for binarizing WMH segmentation output 
>
> **skip_mask_and_bias** : true | false (true if FLAIR has been masked and bias corrected, otherwise false)
>
> **cleanup** : true | false (true to remove temporary files, otherwise false)

<br>

> ## **1.2. Run Command (copy/paste)**
```bash
# FOR SINGULARITY USERS:
singularity run \
  --bind ${in_dir}:/indir,${out_dir}:/outdir  --pwd /  ${sif}  segment_wmh  \
  /indir/${flair_fn} \
  /indir/${synth_fn}  \
  /outdir/${out_fn}  \
  1  \
  "96,128"  \
  ${seg_wmh_thr} \
  1 \
  ${skip_mask_and_bias} \
  ${cleanup} 

# OR, FOR DOCKER USERS:
sudo docker run \
  -v ${in_dir}:/indir \
  -v ${out_dir}:/outdir \
  -w / \
  synthsegcsvd_rc05 \
  segment_wmh \
  /indir/${flair_fn} \
  /indir/${synth_fn} \
  /outdir/${out_fn} \
  1 \
  "96,128" \
  ${seg_wmh_thr} \
  1 \
  ${skip_mask_and_bias} \
  ${cleanup}
```
<br>

> ## **1.3. Output**

> * seg_wmh.nii.gz : unthresholded WMH segmentation [0,1]
> * thr_seg_wmh.nii.gz : thresholded binarized WMH segmentation

# **2. Generate RORPO Image**
* The RORPO filter is included in the SynthSegCSVD container and can be used to extract small tubular structures from MR images and improves the PVS segmentation result in the following section, but requires some user-based choices for the parameters that control filter performance and how/if WMHs are handled (recommendations below)
* Three run commands are provided varying in how WMHs are handled:
  1. **RORPO + WMH exclusion and WMH recovery**
      * requires T1 + FLAIR + WMH segmentation
      * excludes non-PVS voxels within WMHs and includes true-PVS voxels within WMHs
  2. **RORPO + WMH exclustion**
      * requires T1 + WMH segmentation
      * excludes non-PVS voxels within WMHs
  3. **RORPO only**
      * requires T1
      * assumes no WMHs present


> ## **2.1. Variable Setup**
```bash
in_dir=$(pwd)
out_dir=${in_dir}
in_fn=T1.nii.gz
out_fn=T1_rorpo.nii.gz
sif=${HOME}/synthsegcsvd_rc05.sif
in_adf="0.025 1 25"
invert_contrast=1
rorpo_params="--scaleMin=1 --factor=2 --nbScales=4 --dilationSize=0 --verbose --uint8 --nbCores 8" 
in_rorpo_thresh=18
cleanup=true
seg_wmh_fn=thr_seg_wmh.nii.gz
flair_fn=FLAIR.nii.gz
in_flair_adf="0.03 2 25"
in_flair_rorpo_thresh=22
```

**WHERE:**

> **in_dir** : full path to input data, or $(pwd) to use current directory
>
> **out_dir** : full path to output data
>
> **in_fn** : input image filename (T1 or potentially T2, masked and bias corrected)
>
> **out_fn** : output image filname
>
> **sif** : full path + filename of singulairty file downloaded above
>
> **in_adf** : anisotropic diffusion filter settings for input image
>
> **invert_contrast** : 1 (for T1) or 0 (for T2)
>
> **rorpo_params** : RORPO filter settings
>
> **in_rorpo_thresh** : input RORPO threshold
>
> **cleanup** : (remove) cleanup=true or (keep) cleanup=false (temporary files)
>
> **seg_wmh_fn** : segmenation image filename with 1s for WMHs and 0s for non-WMHs
>
> **flair_fn** : FLAIR filename
>
> **in_flair_adf** : anisotropic diffusion filter settings for input-FLAIR image
>
> **in_flair_rorpo_thresh** : input-FLAIR rorpo thresh

**NOTES**:

"rorpo_params" controls rorpo filter (for details, see online documentation: https://github.com/path-openings/RORPO)
*  the "--core" option controls multicore performace
*  setting "--factor" between 1.8 and 2 and "--nbScales" between 4-5 should be appropriate for PVS detection

"in_adf" controls image smoothing
*  the first value controls the time step and should not need to be varied
*  the second value controls the conductance parameter and should be set approximately equal to the voxel size
*  the third value controls the number of iterations is dependent upon image characteristics:
    * e.g. ~20-25 for T1 images, or lower if images are blurry, for example after resampling
    * e.g. ~1 for T2 images [TODO (still validating)]

"cleanup"
* can be set to false to troubleshoot or refine above settings


<br>

> ## **2.2. Run Commands (copy/paste)**

### **i) RORPO + WMH exclusion and WMH recovery (recommended approach)**
```bash
# FOR SINGULARITY USERS:
singularity run  --bind ${in_dir}:/indir,${out_dir}:/outdir \
  --pwd / \
  ${sif} \
  get_rorpo \
  /indir/${out_fn} \
  /indir/${in_fn} \
  "${in_adf}" \
  ${invert_contrast} \
  "${rorpo_params}" \
  ${in_rorpo_thresh} \
  ${cleanup} \
  /indir/${seg_wmh_fn} \
  /indir/${flair_fn} \
  "${in_flair_adf}" \
  ${in_flair_rorpo_thresh}

# OR, FOR DOCKER USERS:
sudo docker run \
  -v ${in_dir}:/indir \
  -v ${out_dir}:/outdir \
  -w / \
  synthsegcsvd_rc05 \
  get_rorpo \
  /indir/${out_fn} \
  /indir/${in_fn} \
  "${in_adf}" \
  ${invert_contrast} \
  "${rorpo_params}" \
  ${in_rorpo_thresh} \
  ${cleanup} \
  /indir/${seg_wmh_fn} \
  /indir/${flair_fn} \
  "${in_flair_adf}" \
  ${in_flair_rorpo_thresh}
``` 
### **ii) RORPO + WMH exclusion**
```bash
# FOR SINGULARITY USERS:
singularity run --bind ${in_dir}:/indir,${out_dir}:/outdir \
  --pwd / \
  ${sif} \
  get_rorpo \
  /indir/${out_fn} \
  /indir/${in_fn} \
  "${in_adf}" \
  ${invert_contrast} \
  "${rorpo_params}" \
  ${in_rorpo_thresh} \
  ${cleanup} \
  /indir/${seg_wmh_fn}

  # OR, FOR DOCKER USERS:
  sudo docker run \
    -v ${in_dir}:/indir \
    -v ${out_dir}:/outdir \
    -w / \
    synthsegcsvd_rc05 \
    get_rorpo \
    /indir/${out_fn} \
    /indir/${in_fn} \
    "${in_adf}" \
    ${invert_contrast} \
    "${rorpo_params}" \
    ${in_rorpo_thresh} \
    ${cleanup} \
    /indir/${seg_wmh_fn} 
``` 
### **iii) RORPO only**
```bash
# FOR SINGULARITY USERS:
singularity run --bind ${in_dir}:/indir,${out_dir}:/outdir \
  --pwd / \
  ${sif} \
  get_rorpo \
  /indir/${out_fn} \
  /indir/${in_fn} \
  "${in_adf}" \
  ${invert_contrast} \
  "${rorpo_params}" \
  ${in_rorpo_thresh} \
  ${cleanup} 

# OR, FOR DOCKER USERS:
sudo docker run \
  -v ${in_dir}:/indir \
  -v ${out_dir}:/outdir \
  -w / \
  synthsegcsvd_rc05 \
  get_rorpo \
  /indir/${out_fn} \
  /indir/${in_fn} \
  "${in_adf}" \
  ${invert_contrast} \
  "${rorpo_params}" \
  ${in_rorpo_thresh} \
  ${cleanup}
``` 
<br>


# **3. Generate PVS Segmentation**
> ## **3.1. Variable Setup**
```bash
in_dir=$(pwd)
out_dir=${in_dir}
t1_fn=T1.nii.gz
rorpo_fn=T1_rorpo.nii.gz
seg_wmh_fn=thr_seg_wmh.nii.gz
synth_fn=synthseg.nii.gz
out_pfx=seg
sif=${HOME}/synthsegcsvd_rc05.sif
cleanup=true
skip_mask_and_bias=true
```

**WHERE:**

> **in_dir** : full path to input data, or $(pwd) to use current directory
>
> **out_dir** : full path to output data
>
> **t1_fn** : T1 filename 
>
> **rorpo_fn** : RORPO filename (or "" to omit with a reduction PVS segmentation accuracy)
> 
> **sif** : full path + filename of singulairty file downloaded above
>
> **seg_wmh_fn** : WMH segmentation filename (or empty image to skip)
> 
> **out_pfx** : output prefix (no extension, will append _pvs.nii.gz / _pvs_no_rorpo.nii.gz | _pvs_with_rorpo.nii.gz)
> 
> **skip_mask_and_bias** : true | false (true if FLAIR has been masked and bias corrected, otherwise false)
>
> **cleanup** : true | false (true to remove temporary files, otherwise false)

> ## **3.2. Run Command (copy/paste)**
```bash
# FOR SINGULARITY USERS:
singularity run --bind ${in_dir}:/indir,${out_dir}:/outdir \
  --pwd / \
  ${sif} \
  segment_pvs \
  /indir/${t1_fn} \
  /indir/${synth_fn} \
  /indir/${rorpo_fn} \
  /indir/${seg_wmh_fn} \
  /outdir/${out_pfx} \
  1 0 \
  ${skip_mask_and_bias} \
  ${cleanup}

# OR, FOR DOCKER USERS:
sudo docker run \
  -v ${in_dir}:/indir \
  -v ${out_dir}:/outdir \
  -w / \
  synthsegcsvd_rc05 \
  segment_pvs \
  /indir/${t1_fn} \
  /indir/${synth_fn} \
  /indir/${rorpo_fn} \
  /indir/${seg_wmh_fn} \
  /outdir/${out_pfx} \
  1 0 \
  ${skip_mask_and_bias} \
  ${cleanup}

```

> ## **3.3. Output**
>
> pvs_seg.nii.gz : unthresholded PVS segmentation using T1 and T1-RORPO [0,2]
> 
> pvs_seg_no_rorpo.nii.gz : unthreshold PVS segmentation using T1 only [0,1]
>
> pvs_seg_with_rorpo.nii.gz : unthreshold PVS segmentation T1-RORPO only [0,1]