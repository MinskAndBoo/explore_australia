# explore_australia

Access to Australia-wide public data for OZ Minerals/Unearthed Explorer challenge - the jumpstarter repo!

## Installation

This is a basic Python package but it makes use of a number of system libraries to read/write geospatial data. We recommed using Anaconda to manage these libraries otherwise you're likely to get burnt with strange C++ exceptions & dependency conflicts. Download Anaconda for your system [here](https://www.anaconda.com/distribution/). Either the full distribution or [Miniconda](https://docs.conda.io/en/latest/miniconda.html) is fine.

We've provided a conda environment file to install all the required dependencies - `environment.yml`. You can create the environment and install the dependencies with:

```bash
$ cd /path/to/explore_australia

$ conda env create --file environment.yml
Collecting package metadata/|\-
# ...snip output

$ conda activate explore_australia

(explore_australia) $ # you should see the prompt change
```

Then you can install the package with:

```bash
(explore_australia) $ python setup.py install
running install
# ...snip output
Successfully installed explore_australia
```

This should install the python package and also the `get_coverages` CLI tool.

```bash
$ which get_coverages
# should show where this is installed, probably in .../conda/bin
```

## Targets: cleaned deposit locations

As targets, we've provided 3034 deposit locations gleaned from Geoscience Australia's [Identified Mineral Resources database](http://www.ga.gov.au/scientific-topics/minerals/mineral-resources-and-advice/aimr). They're available as `data/deposit_locations.csv` and `data/deposit_locations.geo.json` respectively. Geoscience Australia also has a nice [poster showing these locations around Australia](http://www.ga.gov.au/webtemp/image_cache/GA6886.pdf).

To make it easier to target commodity types and remove deposits that are unlikely to be useful targets we've done the following:
- Remove very rare deposits like diamond or 'uninteresting' deposits like opal or coal (who cares about silica or carbon?)
- Concatenated the commodity types into some larger groups for prediction purposes (e.g. illmenite -> Ti, hematite -> Fe). Also combined rare-earth (Sc, Y, La, Ce, Pr, Nd, Pm, Sm, Eu, Gd, Tb, Dy, Ho, Er, Yb, Lu, REO, REE, Xen, Mnz) and platinum-group element (PGE, Pt, Pd, Os, Rh, In, Ir, Re, Ru, Nb) resources into `REE` and `PGE` resources respectively.
- Streamlined the commodity type labels into a semicolon-delimited list (;)

and provided the latitude and longitude of these deposits in WGS84 longitude/latitude (epsg:4326).

![Deposit locations across Australia](https://github.com/jesserobertson/explore_australia/blob/master/resources/deposit_locations.png?raw=true)

## Covariates: getting geophysical coverage data

Most of the geophysical data for all of Australia is pretty big so we've created a couple of Python functions to pull the data from their [web coverage service endpoints](http://nci.org.au/services/nci-national-research-data-collection/geosciences/) - basically a little wrapper around [owslib](https://github.com/geopython/OWSLib).

All of the endpoints are stored in `explorer_australia/endpoints.py` (note you can also load these in any decent GIS package as well as see them in [nationalmap.gov.au](https://nationalmap.gov.au)). We've provided endpoints for continent-wide magnetics (TMI and VRTP), gravity (isostatic residual and bouger anomaly), a number of ASTER products (which map surface mineralogy at a 30 m scale), and radiometric data (K, Th, U and total dose).

![Coverage examples](https://github.com/jesserobertson/explore_australia/blob/master/resources/layer_examples.png?raw=true)

### `get_coverages` CLI

You can use the CLI to pull out aligned coverages for any piece of Australia that you'd like (for example over deposit locations). This should be useful for generating test and train datasets for building your models.

If you've got a particular area that you'd like to look at (e.g. over a known deposit), then you can pull out a box of (roughly) size `distance` using:

```bash
$ get_coverages --help
Usage: get_coverages [OPTIONS] NAME

  Get coverages for a given centre and angle

Options:
  --lat FLOAT         latitude
  --lon FLOAT         longitude
  --distance INTEGER  scale in km
  --angle FLOAT       angle to rotate
  --help              Show this message and exit.

$ get_coverages --lon=122.169999 --lat=-32.42 --angle=239 test_output
# will loop through and grab tifs from WCS

# Show all the downloaded geotiffs
$ ls test_output/**/*
test_output/geophysics/gravity:
bouger_gravity_anomaly.tif  isostatic_residual_gravity_anomaly.tif

test_output/geophysics/magnetics:
total_magnetic_intensity.tif  variable_reduction_to_pole.tif

test_output/geophysics/radiometrics:
filtered_potassium_pct.tif  filtered_terrestrial_dose.tif  filtered_thorium_ppm.tif  filtered_uranium_ppm.tif

test_output/remote_sensing/aster:
aloh_groun_content.tif      ferric_oxide_content.tif  ferrous_iron_index.tif  mgoh_group_content.tif  thermal_infrared_gypsum_index.tif  tir_quartz_index.tif
aloh_group_composition.tif  ferrous_iron_content.tif  kaolin_group_index.tif  opaque_index.tif        thermal_infrared_silica_index.tif
```

The CLI code is in `explore_australia/cli.py` if you want to see the regridding and alignment process (using [rasterio](https://github.com/mapbox/rasterio)).

We've also provided a Jupyter notebook showing you how to use rasterio to read the data once you've downloaded it.

## Covariates: Geological mapping data

Geoscience Australia provides national coverages of surface geology. The data is available on [data.gov.au](https://data.gov.au/dataset/ds-dga-48fe9c9d-2f10-49d2-bd24-ac546662c4ec/details), with a direct download [here](https://d28rz98at9flks.cloudfront.net/74619/74619_1M_shapefiles.zip) (about a Gb of shape files so we haven't just included here). You can see a rendered version [here](https://ecat.ga.gov.au/geonetwork/srv/eng/catalog.search#/metadata/73360). There are seperate layers for geological unit polygons, linear features (e.g. faults, shear zones and dykes) and contacts. There are a lot of attributes to dig into, including age, lithology, history and some chemistry. The geological data comes with extensive documentation on the meanings of terms, and semantic identifiers (via GeoSciML and other standards) for ages, lithologies and other geological aspects.

Some of the state surveys (GSWA in particular) provide more detailed geology (down to 1:100k scale) with the covering rocks removed, seperate regolith (cover) maps, as well as extra information about tectonic history etc. You might be interested in using these datasets for the deposits in a single state to see whether they improve your predictions or change your certainty of existing predictions. If this sounds like you, head over to the WA state data portal: https://catalogue.data.wa.gov.au/dataset?q=geology.

We've provided a jupyter notebook which you can use to pull out the geology for a particular bounding box. [GeoPandas](https://github.com/geopandas/geopandas) and [shapely](https://github.com/Toblerity/Shapely) make this very easy in Python.

![Geology polygons](https://github.com/jesserobertson/explore_australia/blob/master/resources/geology_polygons.png?raw=true)

## Covariates: Other data?

There's nothing that's stopping you from using other data to train or validate your models if you think it will make for a better outcome or submission. Make sure you also take a look at the data portals of the other state and federal geological surveys for tons of useful data. For starters, try:
- [Geological Survey of South Australia](https://map.sarig.sa.gov.au)
- [Geoscience Australia Data and Publications](http://www.ga.gov.au/data-pubs)
- [Geological Survey of Western Australia](http://www.dmp.wa.gov.au/Geological-Survey/Geological-Survey-262.aspx)
- [National map](https://nationalmap.gov.au) - you can use this to find other Web Coverage Services (WCS) to plug into the data getter.

## Any issues?

Any problems, ask a question on the forum or in the Unearthed community slack. Feel free to submit pull requests if you find a bug in this repo.
