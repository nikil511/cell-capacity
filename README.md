# `cellxm`

# Introduction

A CLI tool to identify the optimum location of **WeatherXM** stations in a given country

# Installation

## Prerequisites

The tool requires
- [gdal](https://gdal.org/index.html) library
- [python 3.10+](https://www.python.org/downloads/)

## Application
The recomended way to install cellxm is with [poetry](https://python-poetry.org/docs/#installing-with-the-official-installer).
For Debian or Ubuntu it can be installed using:

```console
sudo apt-get install libgdal-dev
```

You also need to get the version of the installed gdal (`x.y.z`):

```console
gdal-config --version
```

1. Install Python 3.10+
2. Install poetry `pip install poetry`
3. Navigate into the source directory
4. Run `poetry add gdal==x.y.z`
5. Run `poetry install`
6. Spawn a shell with the virtual environment `poetry shell`

# Usage

```console
python cellxm/main.py locate [OPTIONS] [COUNTRY_CODE] [CONFIG]

Example:
python cellxm/main.py locate GR --out-folder ./data/outputs/GR
```

The secondary command `combine` can be used to concatenate multiple results in a single file per output type

```console
python cellxm/main.py combine [IN_FOLDER] [OUT_FOLDER]

Example:
python cellxm/main.py combine --in-folder ./data/outputs/countries --out-folder ./data/outputs/countries-all
```

**Arguments**:

- `[COUNTRY_CODE]`: The country code to run the application. Should follow the ISO 3166-1 alpha-2 specification [default: GR]
- `[CONFIG]`: The location of the config file. The required options are marked inside the file [default: ./config.toml]

**Options**:

**_Directly passing an option overrides the corresponding value in the config file_**

- `--h3-resolution INTEGER`: The resolution for the H3 grid. Should be between 0 and 15
- `--elev-diff-thresh FLOAT`: The elevation difference inside a cell to trigger the heuristic
- `--cell-buffer FLOAT`: Buffer distance around a cell where stations cannot be placed.
- `--min-area-green FLOAT`: The minimum area to consider a green area as individual element
- `--min-area-urban FLOAT`: The minimum area to consider an urban area as individual element
- `--min-aspect-zone-area-cell-perc FLOAT` : The minimum area of an aspect zone as a percentage of the average h3 cell area"
- `--largest-urban` : Boolean to specify which urban areas to keep. If True only the largest urban area for each cell will be exported else all
- `--largest-green`: Boolean to specify which green areas to keep. If True only the largest green area for each urban region will be exported else all
- `--include-buildings` : Use buildings to infer urban areas. This reduces the likelihood of not accurately capturing urban areas due to missing information in OSM. However this comes at increased computational cost (~1.5 longer processing time)
- `--export-zones`: Boolean to specify whether the zones for each area should be exported or not
- `--out-folder TEXT`: Location to export the results [default: ./data/outputs]
- `--ncpus INTEGER`: The number of cpus to be used
- `--help`: Show this message and exit.

**Outputs**

All files are in `.gpkg` format (QGIS is the easiest tool to open them). More information about the output is described in the methodology.

- cells.gpkg
  - The cells covering the analysed country (only above land)
- zones.gpkg
  - The zones which can host a station
  - characterised by 8 types:
    - topo (required by topography)
      1. High (topo-h)
      2. North (topo-n)
      3. East (topo-e)
      4. South (topo-s)
      5. West (topo-w)
    - landuse (required by land use)
      1. Urban (lu-urban)
      2. Green (lu-green)
      3. Coastline (lu-coast)
- stations.gpkg
  - Indicative points (usually centroids) within the above zones
  - Same types as for zones

**Notes**

Recommended specs (for a complete world run):

- 8 CPU cores
- 32GB RAM

**_If you experience problems (e.g. unexpected crashes), reduce the ncpus parameter_**

The process:

- Downloads OSM data from the [geofabrik website](https://www.geofabrik.de/).
- Uses the [Copernicus GLO-90 Digital Elevation Model](https://portal.opentopography.org/raster?opentopoID=OTSDEM.032021.4326.1).
- Creates a `./.cache` folder to store OSM and DEM data. The folder can be safely deleted but subsuquent runs will be delayed. As long as OSM data are available in the `./.cache/osm` folder no check for more recent OSM data will take place.
- The `run.sh` is a script to automate the exection of the process for multiple countries. In this file the complete list of country codes is also available
- Has not been tested on Windows and is very likely to not work on this platform. However, it has been tested with Linux and MacOS for both x86 and arm64 architectures.
