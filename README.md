# cogeo-mosaic

Create mosaics of Cloud Optimized GeoTIFF based on [mosaicJSON](https://github.com/developmentseed/mosaicjson-spec) specification.

[![PyPI pyversions](https://img.shields.io/pypi/pyversions/cogeo-mosaic.svg)](https://pypi.python.org/pypi/ansicolortags/)
[![Packaging status](https://badge.fury.io/py/cogeo-mosaic.svg)](https://badge.fury.io/py/cogeo-mosaic)
[![CircleCI](https://circleci.com/gh/developmentseed/cogeo-mosaic.svg?style=svg)](https://circleci.com/gh/developmentseed/cogeo-mosaic)
[![codecov](https://codecov.io/gh/developmentseed/cogeo-mosaic/branch/master/graph/badge.svg)](https://codecov.io/gh/developmentseed/cogeo-mosaic)

![cogeo-mosaic](https://user-images.githubusercontent.com/10407788/73185274-c41dc900-40eb-11ea-8b67-f79c0682c3b0.jpg)

**Read the official announcement https://medium.com/devseed/cog-talk-part-2-mosaics-bbbf474e66df**

This python module provide a CLI to help create mosaicJSON.

## Install (python >=3)
```bash
$ pip install pip -U
$ pip install cogeo-mosaic

# Or from source

$ pip install git+http://github.com/developmentseed/cogeo-mosaic
```

**Notes**: 
- Starting with version 2.0, pygeos has replaced shapely and thus makes `libgeos` a requirement.
- **pygeos** hosted on pypi migth not compile on certain machine. This has been fixed in the master branch and can be installed with `pip install git+https://github.com/pygeos/pygeos.git`

# CLI
```
$ cogeo-mosaic --help
Usage: cogeo-mosaic [OPTIONS] COMMAND [ARGS]...

  cogeo_mosaic cli.

Options:
  --version  Show the version and exit.
  --help     Show this message and exit.

Commands:
  create     Create mosaic definition from list of files
  footprint  Create geojson from list of files
  overview   [EXPERIMENT] Create COG overviews for a mosaic
```

### Create Mosaic definition
```bash
$ cogeo-mosaic create --help
Usage: cogeo-mosaic create [OPTIONS] [INPUT_FILES]

  Create mosaic definition file.

Options:
  -o, --output PATH       Output file name
  --minzoom INTEGER       An integer to overwrite the minimum zoom level derived from the COGs.
  --maxzoom INTEGER       An integer to overwrite the maximum zoom level derived from the COGs.
  --quadkey-zoom INTEGER  An integer to overwrite the quadkey zoom level used for keys in the MosaicJSON.
  --min-tile-cover FLOAT  Minimum % overlap
  --tile-cover-sort       Sort files by covering %
  --threads INTEGER       threads
  -q, --quiet             Remove progressbar and other non-error output.
  --help                  Show this message and exit.
 ```

`[INPUT_FILES]` must be a list of valid Cloud Optimized GeoTIFF.

```
$ cogeo-mosaic create list.txt -o mosaic.json

# or 

$ cat list.txt | cogeo-mosaic create - | gzip > mosaic.json.gz
```

#### Example: create a mosaic from OAM

```bash 
# Create Mosaic
$ curl https://api.openaerialmap.org/user/5d6a0d1a2103c90007707fa0 | jq -r '.results.images[] | .uuid' | cogeo-mosaic create - | gzip >  5d6a0d1a2103c90007707fa0.json.gz

# Create Footprint (optional)
$ curl https://api.openaerialmap.org/user/5d6a0d1a2103c90007707fa0 | jq -r '.results.images[] | .uuid' | cogeo-mosaic footprint | gist -p -f test.geojson
```

# API 
## Mosaic Storage Backend

Starting in version `3.0.0`, we introduced specific backend storage for:
- **File** (default, `file:///`)
- **HTTP/HTTPS/FTP** (`https://`, `https://`, `ftp://`)
- **AWS S3** (`s3://`)
- **AWS DynamoDB*** (`dynamodb://{region}/{table_name}`)

More info on Backend can be found in [/docs/backends.md](/docs/backends.md)

##### Methods & properties
- **mosaic_def**: MosaicJSON data (pydantic model)
- **metadata**: mosaic metadata (all set values except `tiles`)
- **tile(x, y, z)**: find assets for a specific tile
- **point(lng, lat)**: find assets for a specific point
- **write**: Write mosaicJSON doc to the backend
- **update**: Update mosaicJSON data

##### Read
```python
# MosaicBackend is the top level backend and will distribute to the
# correct backend by checking the path/url schema.
from cogeo_mosaic.backends import MosaicBackend

with MosaicBackend("s3://mybucket/amosaic.json") as mosaic:
    assets: List = mosaic.tile(1, 2, 3) # get assets for mercantile.Tile(1, 2, 3)
```

##### Write
```python
from cogeo_mosaic.utils import create_mosaic
from cogeo_mosaic.backends import MosaicBackend

mosaicdata = create_mosaic(["1.tif", "2.tif"])

with MosaicBackend("s3://mybucket/amosaic.json", mosaic_def=mosaicdata) as mosaic:
    mosaic.write() # trigger upload to S3
```

#### In Memory
```python
from cogeo_mosaic.utils import create_mosaic
from cogeo_mosaic.backends import MosaicBackend

mosaicdata = create_mosaic(["1.tif", "2.tif"])

with MosaicBackend(None, mosaic_def=mosaicdata) as mosaic:
    assets: List = mosaic.tile(1, 2, 3) # get assets for mercantile.Tile(1, 2, 3)
```

# Associated Modules
- [**cogeo-mosaic-tiler**](http://github.com/developmentseed/cogeo-mosaic-tiler): A serverless stack to serve and vizualized tiles from Cloud Optimized GeoTIFF mosaic.

- [**cogeo-mosaic-viewer**](http://github.com/developmentseed/cogeo-mosaic-viewer): A local Cloud Optimized GeoTIFF mosaic viewer based on [rio-viz](http://github.com/developmentseed/rio-viz).

# Contribution & Development

Issues and pull requests are more than welcome.

**Dev install & Pull-Request**

```
$ git clone http://github.com/developmentseed/cogeo-mosaic.git
$ cd cogeo-mosaic
$ pip install -e .[dev]
```


**Python >=3.6 only**

This repo is set to use `pre-commit` to run *flake8*, *pydocstring* and *black* ("uncompromising Python code formatter") when committing new code.

```
$ pre-commit install
$ git add .
$ git commit -m'my change'
black....................................................................Passed
Flake8...................................................................Passed
Verifying PEP257 Compliance..............................................Passed
$ git push origin
```

## About
Created by [Development Seed](<http://developmentseed.org>)
