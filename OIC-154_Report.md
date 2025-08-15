# OIC-154 Microglia Quantification and Morphology Analysis

Total Hours: 22.5

GitHub Repository: <https://github.com/vaioic/OIC-154_Microglia_Quantification/tree/main>

## Authorship and Methods

Research supported by the Optical Imaging Core should be acknowledged and considered for authorship. Please refer to our [SharePoint page](https://vanandelinstitute.sharepoint.com/sites/optical/SitePages/Acknowledgements-and-Authorship.aspx) for guidelines.

Please include our RRID in the methods section for any research supported by the OIC. RRID:SCR_021968

### Sample Acknowledgement

>We thank the Van Andel Institute Optical Imaging Core (RRID:SCR_021968), especially [staff name], for their assistance with [technique/technology]. This research was supported in part by the Van Andel Institute Optical Imaging Core (RRID:SCR_021968) (Grand Rapids, MI).

## Summary of Request

From Request:
>We aim to quantify microglial cells in both control and C9 protein morpholino knockdown conditions and analyze their morphology using the same analysis pipeline previously employed for project OIC-136.

### Brief summary of analysis pipeline

Python-based analysis pipeline that normalized images using [py-clesperanto](https://github.com/clEsperanto/pyclesperanto/tree/main) package, a custom trained model using [CellPose v3](https://github.com/MouseLand/cellpose/tree/v3.1.1.2) to segment microglia and [SciKit-Image](https://scikit-image.org) to measure fluorescence intensities and morphology metrics (volume, surface area, and sphericity).

## Data

3D images of zebrafish brains were collected on the OIC's Andor Spinning Disk Confocal Dragonfly 620.

Images from the Dragonfly are in the Imaris format (.ims). They are a proprietary type of HDF5 image with a pyramidal structure. These images can be read in directly using the [`imaris_ims_file_reader`](https://pypi.org/project/imaris-ims-file-reader/) python package.

**Groups**: Control MO (n=16), 40uM C9 MO (n=14), 80uM C9 MO (n=18), 120uM C9 MO (n=17)

**Image Dimensions**

|Dimension|Pixel Size|
|:--------:|:---:|
|Z| 4.5 um|
|X| 0.3 um|
|Y| 0.3 um|

**Challenges of dataset**: large z-spacing gives boxy objects in z that are less representative of the microglia; high background in some images gives a low signal to noise ratio that is challenging to correct for.

## Analysis Pipeline

Imaris images were read in using `imaris-ims-file-reader` as a NumPy compatible array. Background and illumination were corrected per slice using a correction factor (mean intensity of slice/mean intensity of whole image) and a top hat filter (radius=10). Preprocessed images were fed into CellPose and segmented using the custom trained model in 2D with post-segmentation stitching to create the 3D microglia objects. Objects touching the border of the image were removed prior to collecting measurements.

Fluorescence intensity from the original images (mean, min, max), volume, and surface area calculated using [SciKit-Image](https://scikit-image.org). Sphericity was calculated with the following equation:

($\pi$ ^1/3^ ( ( 6*Volume )^2/3^) ) / Surface Area

The numerator is the surface area of a perfect sphere with the same volume as the object, the denominator is the surface area of the object. The more "wrinkled" the surface of the object, the larger the surface area leading to a smaller ratio value. Sphericity = 1 is a perfect sphere, closer to 0 means less perfect.

## Output

The preprocessed images, segmentation results, and csv files of collected measurements were exported and saved with the following naming convention: normalized_*image name*.tif, filtered_masks_*image name*.tif, and measurements_*image name*.tif

Example of measurement table:

|    |   Unnamed: 0 |   label |    area |   intensity_mean |   intensity_min |   intensity_max |   Surface_Area (um^2) |   Sphericity |
|---:|-------------:|--------:|--------:|-----------------:|----------------:|----------------:|----------------------:|-------------:|
|  0 |            0 |       2 | 2288.73 |          494.819 |             122 |            1937 |              2377.26  |     0.353294 |
|  1 |            1 |       3 | 3173.38 |          545.724 |             133 |            1125 |              2784.87  |     0.374997 |
|  2 |            2 |       4 | 2264.82 |          690.209 |             112 |            2590 |              1950.13  |     0.427671 |
|  3 |            3 |       5 |  446.45 |          406.641 |             140 |             740 |               573.653 |     0.492436 |
|  4 |            4 |       6 | 2949.13 |          964.903 |             133 |            3729 |              2275.81  |     0.436995 |

- Unnamed is an indexing value and can be ignored.
- Label - Object ID
- area - volume of the object in um^3
- intensity_mean/min/max - fluorescence intensity measurement of pixels within the object, from original image
- Surface area - in um^2
- Sphericity - measure of how perfectly round the object in on a scale of 0-1.

## Notes

Cellpose training worked very well to clean up and improve the segmentation results. When finer Z-resolution data is collect, I suspect that a model trained on that data will yield very informal shape measurements for grouping microglia into their different states.

### Optional Analyses - what other information could you get from this data

- A Sholl analysis could be added to assess branching of microglia
- Nearest neighbor distances could be calculated to assess clustering and distribution
