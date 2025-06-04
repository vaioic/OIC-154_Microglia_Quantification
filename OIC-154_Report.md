# OIC-154 Microglia Quantification and Morphology Analysis
GitHub Repository: https://github.com/vaioic/OIC-136_Microglia/tree/main

## Summary of Request
From Request:
>We aim to quantify microglial cells in both control and C9 protein morpholino knockdown conditions and analyze their morphology using the same analysis pipeline previously employed for project OIC-136. 

### Brief summary of analysis pipeline
Python-based analysis pipeline that normalized images using [py-clesperanto](https://github.com/clEsperanto/pyclesperanto/tree/main) package, [CellPose v3](https://github.com/MouseLand/cellpose/tree/v3.1.1.2) to segment microglia and [SciKit-Image](https://scikit-image.org) to measure fluorescence intensities and morphology metrics (volume, surface area, and sphericity). 

Custom CellPose model to improve segmentation and skeleton length measurements will be added once data collection approach is fine tuned.

## Data
3D images of zebrafish brains were collected on the OIC's Andor Spinning Disk Confocal Dragonfly 620.

Images from the Dragonfly are in the Imaris format (.ims). They are a proprietary type of HDF5 image with a pyramidal structure. These images were converted into multi-page tiff files with ImageJ prior to analysis (open in ImageJ and then save as a tiff).

### Metadata - *should this section be included?*

|Dimension|Pixel Size|
|:--------:|:---:|
|Z| 4.5 um|
|X| 0.3 um|
|Y| 0.3 um|

*Other metadata here?*

**Groups**: Control MO (n=16), 40uM C9 MO (n=14), 80uM C9 MO (n=18), 120uM C9 MO (n=17)

**Challenges of dataset**: large z-spacing gives boxy objects in z that are less representative of the microglia; high background in some images gives a low signal to noise ratio that is challenging to correct for.

## Analysis Pipeline
Update to this pipeline: Found [imaris-ims-file-reader](https://pypi.org/project/imaris-ims-file-reader/) python package that easily opens ims files in NumPy compatible format.

```
from imaris_ims_file_reader.ims import ims
all_files = sorted(glob('./**/*.ims')) #recursively look for ims files in sub folders
all_imgs = [ims(img,ResolutionLevelLock=0) for img in all_files] #open each image at full resolution
```

To correct changes in illumination in Z and to increase the signal to background ratio, images were normalized by the mean intensity of the image and then a white top hat filter with `radius=10` was applied.

*Add in some examples*

Normalized images