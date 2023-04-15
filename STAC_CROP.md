# Introduction
In this tutorial, we'll create an EO application package that crops a Sentinel-2 product using a Python script and CWL. The package will include a CWL workflow and a Python script that performs the actual cropping. We will also add a stage-in and stage-out step that use local STAC files to define the input and output.

## Prerequisites
To follow this tutorial, you'll need:

* A CWL runner installed on your machine. We recommend using cwltool, which can be installed using pip.
* Access to Sentinel-2 data. You can download Sentinel-2 data from the Copernicus Open Access Hub: https://scihub.copernicus.eu/dhus/#/home or directly from your Ellip account
* Basic knowledge of Python programming.

## Creating the Workflow
Let's start by creating the CWL workflow. The workflow will take a Sentinel-2 product as input and output the cropped product.

Here's what the workflow YAML file should look like:

````yaml
cwlVersion: v1.0
class: Workflow
inputs:
  input_stac:
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
  output_stac:
    type: File
    outputBinding:
      glob: $(inputs.input_stac.basename)_cropped.json
steps:
  stage_in:
    in:
      input_stac: input_stac
    out:
      input_product: input_product
    run:
      class: CommandLineTool
      baseCommand: python3
      inputs:
        script:
          type: File
          inputBinding:
            position: 1
          default: stage_in.py
        input_stac:
          type: File
          inputBinding:
            position: 2
      outputs:
        input_product:
          type: File
          outputBinding:
            glob: $(inputs.input_stac.basename).tif
  crop:
    in:
      input_product: stage_in/output/input_product
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
  stage_out:
    in:
      output_product: crop/output/output_product
    out:
      output_stac: output_stac
    run:
      class: CommandLineTool
      baseCommand: python3
      inputs:
        script:
          type: File
          inputBinding:
            position: 1
          default: stage_out.py
        output_product:
          type: File
          inputBinding:
            position: 2
      outputs:
        output_stac:
          type: File
          outputBinding:
            glob: $(inputs.output_product.basename)_cropped.json
````

Let's break down each part of the workflow:

 * We start by specifying the CWL version and the class of the workflow.
 * We define the input parameters of the workflow: `input_stac`, `xmin`, `ymin`, `xmax`, and `ymax`.
 * We define the output parameter of the workflow: `output_stac`.
 * We specify the `stage_in` step, which reads the Sentinel-2 product from the input STAC file and stages it in as a local file called `input_product.tif`.
 * We specify the `crop` step, which reads the input raster file and crops it using the specified bounding box. The output file is saved as `output_product_cropped.tif`.
 * We specify the `stage_out` step, which stages out the cropped Sentinel-2 product to an output STAC file.


## Creating the Python Scripts
Now that we have created the CWL workflow, let's create the Python scripts. We will create two Python scripts: `stage_in.py` and `stage_out.py`.

Here's what the `stage_in.py` Python script should look like:
````python
#!/usr/bin/env python3

import sys
import json
import shutil

# Get the input STAC file path from the command-line arguments
input_stac_path = sys.argv[1]

# Load the STAC file
with open(input_stac_path, 'r') as f:
    stac = json.load(f)

# Get the path to the Sentinel-2 product
asset = stac['assets']['data']
input_product_path = asset['href']

# Copy the Sentinel-2 product to the current directory
shutil.copy2(input_product_path, './input_product.tif')
````

This script reads the input STAC file and copies the Sentinel-2 product to the current directory as `input_product.tif`.

Here's what the `stage_out.py` Python script should look like:

```` python

#!/usr/bin/env python3

import sys
import json
import shutil

# Get the output STAC file path and the path to the cropped product from the command-line arguments
output_stac_path = sys.argv[1]
output_product_path = sys.argv[2]

# Load the input STAC file
with open(output_stac_path, 'r') as f:
    input_stac = json.load(f)

# Update the input STAC file with the path to the cropped product
asset = input_stac['assets']['data']
asset['href'] = output_product_path

# Save the updated input STAC file as the output STAC file
with open(output_stac_path, 'w') as f:
    json.dump(input_stac, f)
````

This script updates the input STAC file with the path to the cropped Sentinel-2 product and saves it as the output STAC file.

## Packaging the Application
Now that we have created the CWL workflow and the Python scripts, let's package them into an EO application package.

The EO application package should have the following structure:

````kotlin
.
├── cwl
│   ├── crop.cwl
│   ├── stage_in.py
│   └── stage_out.py
├── data
│   └── sentinel_product.json
└── README.md
````
* `cwl`: This directory contains the CWL files for the workflow and the Python scripts.
* `data`: This directory contains the input STAC file.
* `README.md`: This file contains instructions on how to use the EO application package.

Here's what the updated README.md file should look like:

````
# EO Application Package: Crop a Sentinel-2 Product
This EO application package crops a Sentinel-2 product using a Python script and CWL. The package includes a CWL workflow and a Python script that performs the actual cropping. We also added a stage-in and stage-out step that use local STAC files to define the input and output.

## Requirements
To run this application package, you will need:

* A CWL runner installed on your machine. We recommend using cwltool, which can be installed using pip.
* Access to Sentinel-2 data. You can download Sentinel-2 data from the Copernicus Open Access Hub: https://scihub.copernicus.eu/dhus/#/home or directly from your Ellip account
* Basic knowledge of Python programming.

## Usage
Follow these steps to use the EO application package to crop a Sentinel-2 product:

1. Download the EO application package from GitHub and extract it to a directory of your choice.
2. Change to the directory where you extracted the package.
3. Edit the sentinel_product.json file to specify the path to your Sentinel-2 product.
4. Run the following command to execute the workflow:

 `cwltool cwl/crop.cwl --input_stac data/sentinel_product.json --xmin 0 --ymin 0 --xmax 1000 --ymax 1000 --outdir .`

This will crop the Sentinel-2 product using a bounding box with `xmin` = 0, `ymin` = 0, `xmax` = 1000, and `ymax` = 1000. The cropped product will be saved as `input_product_cropped.tif`. The output STAC file will be saved as `input_product_cropped.json`.

You can also specify the output file names by using the --output_stac and --output_product options:
`cwltool cwl/crop.cwl --input_stac data/sentinel_product.json --xmin 0 --ymin 0 --xmax 1000 --ymax 1000 --output_stac output.json --output_product output.tif --outdir .`

This will crop the Sentinel-2 product using the same bounding box and save the cropped product as output.tif. The output STAC file will be saved as output.json.

Congratulations, you've successfully used an EO application package with stage-in and stage-out steps that use local STAC files to define the input and output!
````

## Conclusion

In this tutorial we've explored how to create an EO application package using Common Workflow Language (CWL) and the SpatioTemporal Asset Catalog (STAC) standard. With CWL, we created a workflow that can be executed on different platforms, and with STAC, we made it easy to define the input and output using local files. The resulting EO application package is portable, interoperable, and easy to use.

By creating EO application packages with CWL and STAC, we can share and reuse Earth observation processing workflows more easily. This makes it easier to collaborate and build upon existing work, accelerating progress in the field of Earth observation.

We hope this tutorial has been helpful and that it inspires you to create your own EO application packages. With CWL and STAC, the possibilities are endless!

