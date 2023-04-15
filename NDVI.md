# Introduction
The Normalized Difference Vegetation Index (NDVI) is a simple but powerful index used to assess vegetation cover, biomass, and health. The NDVI is calculated from the reflectance values of two wavelengths, typically the red and near-infrared bands.

In this tutorial, we'll create an EO application package that calculates the NDVI from Sentinel-2 data using CWL. The package will include a CWL workflow and a CommandLineTool that performs the actual calculation.

# Prerequisites
To follow this tutorial, you'll need:

 * A CWL runner installed on your machine. We recommend using `cwltool`, which can be installed using `pip`.
 * Access to Sentinel-2 data. You can download Sentinel-2 data from the Copernicus Open Access Hub: https://scihub.copernicus.eu/dhus/#/home or directly from your Ellip account 

# Creating the Workflow

Let's start by creating the CWL workflow. The workflow will take a Sentinel-2 scene as input and output the NDVI raster.
Here's what the workflow YAML file should look like:

````yaml
cwlVersion: v1.0
class: Workflow
inputs:
  input_scene:
    type: File
    inputBinding:
      position: 1
outputs:
  ndvi_raster:
    type: File
    outputBinding:
      glob: $(inputs.input_scene.basename).ndvi.tif
steps:
  ndvi_calculation:
    in:
      input_scene: input_scene
    out:
      ndvi_raster: ndvi_raster
    run:
      class: CommandLineTool
      baseCommand: gdal_calc.py
      inputs:
        input_raster:
          type: File
          inputBinding:
            position: 1
        ndvi_raster:
          type: File
          outputBinding:
            glob: $(inputs.input_raster.basename).ndvi.tif
      arguments:
        - --outfile=$(inputs.ndvi_raster.path)
        - --calc="(B05 - B04) / (B05 + B04)"
        - --type=Float32
````

Let's break down each part of the workflow:

 * `inputs`: This section specifies the input to the workflow. In this case, we have one input, `input_scene`, which is a Sentinel-2 scene in GeoTIFF format.
 * `outputs`: This section specifies the output of the workflow. In this case, we have one output, `ndvi_raster`, which is the NDVI raster in GeoTIFF format.
 * `steps`: This section contains the steps of the workflow. In this case, we have one step, `ndvi_calculation`, which performs the NDVI calculation using the `gdal_calc.py` CommandLineTool.
 * `ndvi_calculation`: This step references the gdal_calc.py CommandLineTool and specifies the input and output of the step.

Note that the `gdal_calc.py` CommandLineTool is not included in this workflow. We'll create the CommandLineTool in the next step.

## Creating the CommandLineTool
Now let's create the gdal_calc.py CommandLineTool. This CommandLineTool will perform the actual NDVI calculation using the GDAL library.

Here's what the CommandLineTool YAML file should look like:

````yaml
cwlVersion: v1.0
class: CommandLineTool
baseCommand: gdal_calc.py
inputs:
  input_raster:
    type: File
    inputBinding:
      position: 1
  ndvi_raster:
    type: File
    outputBinding:
      glob
outputs:
  ndvi_raster:
    type: File
    outputBinding:
      glob: $(inputs.ndvi_raster.basename).tif
arguments:
  - $(inputs.input_raster.path)
  - --outfile=$(inputs.ndvi_raster.path)
  - --calc="(B05 - B04) / (B05 + B04)"
  - --type=Float32
````

Let's break down each part of the CommandLineTool:

 * `inputs`: This section specifies the input to the CommandLineTool. In this case, we have two inputs, input_raster and ndvi_raster. input_raster is the Sentinel-2 scene in GeoTIFF format, and ndvi_raster is the output NDVI raster in GeoTIFF format.
 * `outputs`: This section specifies the output of the CommandLineTool. In this case, we have one output, ndvi_raster, which is the NDVI raster in GeoTIFF format.
 * `arguments`: This section specifies the arguments to the gdal_calc.py command. We pass the input raster, specify the output file using the --outfile option, specify the NDVI calculation using the --calc option, and set the output data type to Float32 using the --type option.

Packaging the Application
Now that we've created the workflow and the CommandLineTool, let's package them into an EO application package.

The EO application package should have the following structure:

````kotlin
.
├── cwl
│   ├── ndvi_calculation.cwl
│   └── gdal_calculation.cwl
├── data
│   └── sentinel_scene.tif
└── README.md
````

 * `cwl`: This directory contains the CWL files for the workflow and the CommandLineTool.
 * `data`: This directory contains the Sentinel-2 scene used as input.
 * `README.md`: This file contains instructions on how to use the EO application package.

Here's what the README.md file could look like:

````yaml
# EO Application Package for calculating the NDVI from Sentinel-2

This EO application package calculates the NDVI from Sentinel-2 data using the Common Workflow Language (CWL).

## Prerequisites

To run this EO application package, you'll need:

- A CWL runner installed on your machine. We recommend using `cwltool`, which can be installed using `pip`.
- Access to Sentinel-2 data. You can download Sentinel-2 data from the Copernicus Open Access Hub: https://scihub.copernicus.eu/dhus/#/home or directly from your Ellip account 

## Usage

1. Download the EO application package from GitHub: https://github.com/yourusername/eo-app-ndvi-sentinel2
2. Extract the package to a directory of your choice.
3. Change to the directory where you extracted the package.
4. Run the following command to execute the workflow:
  `cwltool cwl/ndvi_calculation.cwl data/sentinel_scene.tif`

This will calculate the NDVI from the `sentinel_scene.tif` input and output the NDVI raster as `sentinel_scene.ndvi.tif`.

# License

This EO application package is licensed under the MIT License. See the LICENSE file for more information.

````

## Conclusion
In this tutorial, we created an EO application package that calculates the NDVI from Sentinel-2 data using CWL. We created a CWL workflow and a CommandLineTool that performs the actual calculation, and packaged them into an application package. The application package includes instructions on




