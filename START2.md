# Getting Started with CWL: EO Applications Examples

In this section, we will demonstrate how to create a simple Earth Observation (EO) application using the Common Workflow Language (CWL) with the Workflow class. This example does not involve combining multiple CWL files, making it easy to understand the structure and workflow of a CWL Workflow.

## Example: Calculating NDVI from Sentinel-2 Imagery

The Normalized Difference Vegetation Index (NDVI) is a widely used remote sensing metric for assessing vegetation health. We will create a simple CWL workflow to calculate NDVI from Sentinel-2 imagery using two bands: the red band (B4) and the near-infrared band (B8).

### Step 1: Prepare the Python script
Create a Python script named compute_ndvi.py that reads the red and NIR bands from input GeoTIFF files, calculates the NDVI, and writes the result to an output GeoTIFF file.

````python
import rasterio
import numpy as np

def compute_ndvi(red_band, nir_band, output_file):
    with rasterio.open(red_band) as red_src:
        red = red_src.read(1).astype(np.float32)

    with rasterio.open(nir_band) as nir_src:
        nir = nir_src.read(1).astype(np.float32)

    ndvi = (nir - red) / (nir + red)

    profile = red_src.profile
    profile.update(dtype=rasterio.float32, count=1)

    with rasterio.open(output_file, 'w', **profile) as dst:
        dst.write(ndvi.astype(rasterio.float32), 1)

if __name__ == '__main__':
    import argparse

    parser = argparse.ArgumentParser(description='Compute NDVI from Sentinel-2 red and NIR bands')
    parser.add_argument('red_band', help='Input GeoTIFF file with the red band (B4)')
    parser.add_argument('nir_band', help='Input GeoTIFF file with the NIR band (B8)')
    parser.add_argument('output_file', help='Output GeoTIFF file with the NDVI')

    args = parser.parse_args()

    compute_ndvi(args.red_band, args.nir_band, args.output_file)

````
### Step 2: Create the CWL Workflow
Create a CWL file named compute_ndvi_workflow.cwl with the following content:

````yaml
cwlVersion: v1.0
class: Workflow
label: Compute NDVI from Sentinel-2 red and NIR bands
doc: This workflow computes the NDVI from Sentinel-2 red and NIR bands using a Python script

inputs:
  red_band:
    type: File
    doc: Input GeoTIFF file with the red band (B4)
  nir_band:
    type: File
    doc: Input GeoTIFF file with the NIR band (B8)

outputs:
  ndvi_image:
    type: File
    outputSource: compute_ndvi/output_file
    doc: Output GeoTIFF file with the NDVI

steps:
  compute_ndvi:
    run:
      class: CommandLineTool
      baseCommand: [python3, compute_ndvi.py]
      inputs:
        red_band: File
        nir_band: File
        output_file:
          type: string
          default: ndvi_image.tif
      outputs:
        output_file:
          type: File
          outputBinding:
            glob: ndvi_image.tif
    in:
      red_band: red_band
      nir_band: nir_band
    out: [output_file]
````

This CWL file defines a simple workflow that uses the CommandLineTool class to execute the 

**... to be continued **

````yaml

````
