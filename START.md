# Getting Started with CWL

In this guide, we'll introduce the Common Workflow Language (CWL) and explained its role in creating reproducible and shareable workflows in Earth Observation applications. 

We'll walk you through the process of creating and running a simple EO application using CWL with a step-by-step guide on how to create CWL CommandLineTools and Workflows, with examples demonstrating how to crop and compute the NDVI of GeoTIFF images.

We'll start by creating a CommandLineTool for a basic EO operation, followed by a Workflow that combines multiple CommandLineTools to perform a more complex analysis.

## Prerequisites

Before diving into CWL, make sure you have the following software installed on your system:

1. **cwltool**: The reference implementation of CWL. You can install it using pip:
```
pip install cwltool
```

2. **GDAL**: A popular library for geospatial data manipulation. You can install it using the package manager for your operating system or follow the instructions on the [official GDAL website](https://gdal.org/download.html).

## Creating a CommandLineTool

Let's start by creating a CommandLineTool that performs a simple operation: cropping a GeoTIFF image using GDAL's `gdal_translate` utility.

Create a file named `crop_geotiff.cwl` with the following content:

```yaml
cwlVersion: v1.0
class: CommandLineTool

baseCommand: gdal_translate
requirements:
DockerRequirement:
 dockerPull: osgeo/gdal

inputs:
input_geotiff:
 type: File
 inputBinding:
   position: 1
output_geotiff:
 type: string
 inputBinding:
   position: 2
projwin:
 type: string[]
 inputBinding:
   prefix: -projwin
   position: 3
   itemSeparator: " "

outputs:
cropped_geotiff:
 type: File
 outputBinding:
   glob: $(inputs.output_geotiff)
```

This CommandLineTool takes three inputs: the input GeoTIFF file, the output file name, and the coordinates of the bounding box to crop. It uses the osgeo/gdal Docker image to ensure that the required GDAL utilities are available.

## Running the CommandLineTool

To run the CommandLineTool, create a YAML file named crop_geotiff_input.yml with the following content:

```yaml
input_geotiff:
  class: File
  path: input.tif
output_geotiff: output.tif
projwin: [xmin, ymin, xmax, ymax]
```

Replace input.tif with the path to your input GeoTIFF file and xmin, ymin, xmax, ymax with the desired bounding box coordinates.

Execute the CommandLineTool using cwltool:
```
cwltool crop_geotiff.cwl crop_geotiff_input.yml
```

If the process is successful, you should see a cropped GeoTIFF file named output.tif in your working directory.

## Creating a Workflow
Now, let's create a Workflow that combines two CommandLineTools: the crop_geotiff.cwl tool we just created and another tool that computes the NDVI (Normalized Difference Vegetation Index) of the cropped image.

First, create a CommandLineTool named compute_ndvi.cwl:

```yaml
cwlVersion: v1.0
class: CommandLineTool

baseCommand: [python, compute_ndvi.py]
requirements:
  DockerRequirement:
    dockerPull: python:3.8

inputs:
  input_geotiff:
    type: File
    inputBinding:
      position: 1

outputs:
  ndvi_geotiff:
    type: File
    outputBinding:
      glob: ndvi.tif
```

Next, create a Workflow file named `crop_and_compute_ndvi.cwl that uses both thecrop_geotiff.cwlandcompute_ndvi.cwl` CommandLineTools:

```yaml
cwlVersion: v1.0
class: Workflow

inputs:
  input_geotiff: File
  output_geotiff: string
  projwin: string[]
  ndvi_output: string

outputs:
  cropped_geotiff:
    type: File
    outputSource: crop_geotiff/cropped_geotiff
  ndvi_geotiff:
    type: File
    outputSource: compute_ndvi/ndvi_geotiff

steps:
  crop_geotiff:
    run: crop_geotiff.cwl
    in:
      input_geotiff: input_geotiff
      output_geotiff: output_geotiff
      projwin: projwin
    out: [cropped_geotiff]

  compute_ndvi:
    run: compute_ndvi.cwl
    in:
      input_geotiff: crop_geotiff/cropped_geotiff
    out: [ndvi_geotiff]
```

This workflow takes four inputs: the input GeoTIFF file, the output file name for the cropped image, the bounding box coordinates, and the output file name for the NDVI image. It first runs the crop_geotiff.cwl CommandLineTool and then passes the output to the compute_ndvi.cwl CommandLineTool.

## Running the Workflow
To run the Workflow, create a YAML file named crop_and_compute_ndvi_input.yml with the following content:

```yaml
input_geotiff:
  class: File
  path: input.tif
output_geotiff: cropped_output.tif
projwin: [xmin, ymin, xmax, ymax]
ndvi_output: ndvi_output.tif
```

Replace input.tif with the path to your input GeoTIFF file, xmin, ymin, xmax, ymax with the desired bounding box coordinates, and ndvi_output.tif with the desired NDVI output file name.

Execute the Workflow using cwltool:

```
cwltool crop_and_compute_ndvi.cwl crop_and_compute_ndvi_input.yml
```

If the process is successful, you should see a cropped GeoTIFF file named cropped_output.tif and an NDVI GeoTIFF file named ndvi_output.tif in your working directory.






