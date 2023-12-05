# Standardize heightmap export with Python

QGIS provides the ability to save scripts written in Python that run many of the UI commands normally used for export. To standardize the export process (i.e., not have to type the command line parameters in Translate (Convert Format...) dialog every time) save the following as a script:

```python
from osgeo import gdal
import os

input_file = \path\to\dem.tif

def name_output_file(input_file):
    return os.path.splitext(input_file)[0] + '-heightmap.png'

translate_options = {
    "destName" : name_output_file(input_file),
    "srcDS": gdal.Open(input_file),
    "format": 'PNG',
    "outputType": gdal.GDT_UInt16,
    "scaleParams": [[0, 4096, 0, 65535]],
    "width": 4096,
    "height": 4096,
    "resampleAlg": 'cubic'
}

def main():
    gdal.Translate(**translate_options)
    translate_options['srcDS'] = None

main()
```

## Translate options explanation
- `input_file` is the path to the DEM to be used for export.
- `destName` is a generated heightmap file name based on the input file name.
- `format` is set to PNG, which should not be changed.
- `outputType` is set to UInt16, which should not be changed.
- `scaleParams` tells `gdal.Translate` how to map elevation to grayscale. The first two numbers (0, 4096) represent elevation, in meters, of the DEM. The range of these two numbers should be the same as the Height Scale set in game. The last two numbers (0, 65535) is the range of a 16-bit unsigned integer, and represents the grayscale values used by the game to display terrain.
- `width` and `height` are the dimensions, in pixels, of the output heightmap. These should not be changed, [see Terrain and heightmap resolution](docs\vanilla-map-information\terrain-and-heightmap-resolution.md) for more details.
- `resampleAlg` is the algorithm to be used for resampling (e.g., going from a 1m resolution to 3.5m for playable area). There are various values supported by QGIS for this option. The list of these options can be found [here](https://docs.qgis.org/3.28/en/docs/user_manual/working_with_raster/georeferencer.html#define-the-resampling-method). A high level summary of these resampling methods can be found [here](https://gisgeography.com/raster-resampling/). An in-depth discussion of QGIS-specific resampling algorithms can be found [here](https://gis.stackexchange.com/a/14361). It is recommended to review these resources and understand the tradeoffs before choosing a resampling algorithm.