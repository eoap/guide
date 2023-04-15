# Introduction
In this tutorial, we'll create an EO application package that crops a Sentinel-2 product using a Python script and CWL. The package will include a CWL workflow and a Python script that performs the actual cropping.

# Prerequisites
To follow this tutorial, you'll need:

 * A CWL runner installed on your machine. We recommend using cwltool, which can be installed using pip.
 * Access to Sentinel-2 data. You can download Sentinel-2 data from the Copernicus Open Access Hub: https://scihub.copernicus.eu/dhus/#/home or directly from your Ellip account
 * Basic knowledge of Python programming.

# Creating the Workflow
Let's start by creating the CWL workflow. The workflow will take a Sentinel-2 product as input and output the cropped product.

Here's what the workflow YAML file should look like:


````yaml
cwlVersion: v1.0
class: Workflow
inputs:
  input_product:
    type: File
    inputBinding:
      position: 1
  xmin:
    type: int
    inputBinding:
      position: 2
  ymin:
    type: int
    inputBinding:
      position: 3
  xmax:
    type: int
    inputBinding:
      position: 4
  ymax:
    type: int
    inputBinding:
      position: 5
outputs:
  output_product:
    type: File
    outputBinding:
      glob: $(inputs.input_product.basename)_cropped.tif
steps:
  crop:
    in:
      input_product: input_product
      xmin: xmin
      ymin: ymin
      xmax: xmax
      ymax: ymax
    out:
      output_product: output_product
    run:
      class: CommandLineTool
      baseCommand: python3
      inputs:
        script:
          type: File
          inputBinding:
            position: 1
          default: crop.py
        input_product:
          type: File
          inputBinding:
            position: 2
        xmin:
          type: int
          inputBinding:
            position: 3
        ymin:
          type: int
          inputBinding:
            position: 4
        xmax:
          type: int
          inputBinding:
            position: 5
        ymax:
          type: int
          inputBinding:
            position: 6
      outputs:
        output_product:
          type: File
          outputBinding:
            glob: $(inputs.input_product.basename)_cropped.tif
````

Let's break down each part of the workflow:

 * `inputs`: This section specifies the input to the workflow. In this case, we have five inputs: `input_product` is the Sentinel-2 product in GeoTIFF format, `xmin`, `ymin`, `xmax`, and `ymax` specify the coordinates of the bounding box to crop.
 * `outputs`: This section specifies the output of the workflow. In this case, we have one output, `output_product`, which is the cropped Sentinel-2 product in GeoTIFF format.
 * `steps`: This section contains the steps of the workflow. In this case, we have one step, crop, which performs the cropping using the `crop.py` Python script.
 * `crop`: This step references the `crop.py` Python script and specifies the input and output of the step.

Note that the `crop.py` script is not included in this workflow. We'll create the script in the next step.

Creating the Python Script
Now let's create the crop.py Python script. This script will read the Sentinel-2 product and crop it using the specified bounding box.

Here's what the Python script should look like:

````python
#!/usr/bin/env python3

import sys
import rasterio

# Get the input and output file paths and the bounding box coordinates from the command-line arguments
input_path = sys.argv[1]
output_path = sys.argv[2]
xmin = int(sys.argv[3])
ymin = int(sys.argv[4])
xmax = int(sys.argv[5])
ymax = int(sys.argv[6])

# Open the input raster file
with rasterio.open(input_path) as src:
    # Calculate the window to read
    window = rasterio.windows.Window(xmin, ymin, xmax-xmin, ymax-ymin)
    # Read the data from the window
    data = src.read(window=window)
    # Get the metadata of the input raster file
    meta = src.meta
    # Update the metadata with the cropped window
    meta.update({
        'width': window.width,
        'height': window.height,
        'transform': rasterio.windows.transform(window, src.transform)
    })
    # Write the cropped data to the output raster file
    with rasterio.open(output_path, 'w', **meta) as dst:
        dst.write(data)
````

Let's break down each part of the script:

 * We start by importing the necessary modules: `sys` for accessing command-line arguments and `rasterio` for reading and writing raster files.
 * We get the input and output file paths and the bounding box coordinates from the command-line arguments.
 * We open the input raster file using `rasterio.open`.
 * We calculate the window to read using the `rasterio.windows.Window` function.
 * We read the data from the window using `src.read`.
 * We get the metadata of the input raster file using `src.meta`.
 * We update the metadata with the cropped window using `rasterio.windows.transform`.
 * Finally, we write the cropped data to the output raster file using `rasterio.open` and `dst.write`.

# Packaging the Application
Now that we've created the workflow and the Python script, let's package them into an EO application package.

The EO application package should have the following structure:

````kotlin
.
├── cwl
│   ├── crop.cwl
│   └── crop.py
├── data
│   └── sentinel_product.tif
└── README.md
````

* `cwl`: This directory contains the CWL files for the workflow and the Python script.
* `data`: This directory contains the Sentinel-2 product used as input.
* `README.md`: This file contains instructions on how to use the EO application package.

Here's what the `README.md` file could look like:

````
# EO Application Package for cropping a Sentinel-2 product

This EO application package crops a Sentinel-2 product using a Python script and CWL.

## Prerequisites

To run this EO application package, you'll need:

- A CWL runner installed on your machine. We recommend using `cwltool`, which can be installed using `pip`.
- Access to Sentinel-2 data. You can download Sentinel-2 data from the Copernicus Open Access Hub: https://scihub.copernicus.eu/dhus/#/home  or directly from your Ellip account
- Basic knowledge of Python programming.

## Usage

1. Download the EO application package from GitHub: https://github.com/yourusername/eo-app-crop-sentinel2
2. Extract the package to a directory of your choice.
3. Change to the directory where you extracted the package.
4. Run the following command to execute the workflow
`cwltool cwl/crop.cwl data/sentinel_product.tif xmin ymin xmax ymax`

Replace xmin, ymin, xmax, and ymax with the coordinates of the bounding box you want to crop. For example:
`cwltool cwl/crop.cwl data/sentinel_product.tif 0 0 1000 1000`

This will crop the Sentinel-2 product using a bounding box with xmin = 0, ymin = 0, xmax = 1000, and ymax = 1000. The cropped product will be saved as sentinel_product_cropped.tif.

## License
This EO application package is licensed under the MIT License. See the LICENSE file for more information.
````



## Conclusion

In this tutorial, we created an EO application package that crops a Sentinel-2 product using a Python script and CWL. We created a CWL workflow that references the Python script and specifies the input and output of the workflow. We also created the `crop.py` Python script that reads the input raster file and crops it using the specified bounding box. Finally, we packaged the workflow, the Python script, and the input raster file into an application package. The application package includes instructions on how to use the package to crop a Sentinel-2 product.
