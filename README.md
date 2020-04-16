QWC Map Viewer v2
=================

Provide a [QWC2 Web Client](https://github.com/qgis/qwc2-demo-app) application using QWC services.

**v2** (WIP): add support for multitenancy and replace QWC Config service and QWC2 config files with static config and permission files.

**Note:**: Custom viewers have been replaced by tenants in v2.

**Note:** Requires a QWC OGC service or QGIS server running on `ogc_service_url`. Additional QWC Services are optional.


Setup
-----

Copy your QWC2 files from a production build (see [QWC2 Quick start](https://github.com/qgis/qwc2-demo-app/blob/master/doc/QWC2_Documentation.md#quick-start)):

    SRCDIR=path/to/qwc2-app/prod/ DSTDIR=$PWD
    mkdir $DSTDIR/qwc2 && mkdir $DSTDIR/qwc2/dist
    cd $SRCDIR && \
    cp -r assets $DSTDIR/qwc2 && \
    cp -r translations $DSTDIR/qwc2/translations && \
    cp dist/QWC2App.js $DSTDIR/qwc2/dist/ && \
    cp index.html $DSTDIR/qwc2/ && \
    cp config.json $DSTDIR/qwc2/config.json && \
    cd -

Copy your QWC2 themes config file:

    cp themesConfig.json $DSTDIR/qwc2/


Configuration
-------------

The static config and permission files are stored as JSON files in `$CONFIG_PATH` with subdirectories for each tenant,
e.g. `$CONFIG_PATH/default/*.json`. The default tenant name is `default`.


### Map Viewer config

* File location: `$CONFIG_PATH/<tenant>/mapViewerConfig.json`

Example:
```jsonc
{
  "service": "map-viewer",
  "config": {
    // path to QWC2 files
    "qwc2_path": "qwc2/",
    // QWC OGC service (required)
    "ogc_service_url": "http://localhost:5013/",
    // some optional QWC services
    "auth_service_url": "http://localhost:5017/",
    "data_service_url": "http://localhost:5012/"
  },
  "resources": {
    "qwc2_config": {
      // contents from QWC2 config.json
    },
    "qwc2_themes": {
      // themes configuration
      "items": [
        {
          "name": "qwc_demo",
          "title": "Demo",
          "url": "/ows/qwc_demo",
          // ...
          "sublayers": [
            // ...
          ]
        }
      ],
      "backgroundLayers": [
        // ...
      ],
      // ...
    }
  }
}
```

All `config` options may be overridden by setting corresponding upper-case environment variables, e.g. `OGC_SERVICE_URL` for `ogc_service_url`.

Main optional QWC services:
 * `auth_service_url`: QWC Auth Service URL
 * `data_service_url`: QWC Data Service URL
 * `elevation_service_url`: QWC Elevation Service URL
 * `info_service_url`: QWC FeatureInfo Service URL
 * `legend_service_url`: QWC Legend Service URL
 * `permalink_service_url`: QWC Permalink Service URL
 * `print_service_url`: QWC Print Service URL
 * `proxy_service_url`: Proxy Service URL
 * `search_service_url`: QWC Search Service URL

`qwc2_config` contains the QWC2 application config, corresponding to the contents of your standalone `config.json` file (see [Documentation](https://github.com/qgis/qwc2-demo-app/blob/master/doc/QWC2_Documentation.md#application-configuration-the-configjson-and-jsappconfigjs-files)).

`qwc2_themes` contains the full themes configuration, mostly corresponding to `themes` in the `themes.json` collected from `themesConfig.json`.

Add new themes to your `themesConfig.json` (see [Documentation](https://github.com/qgis/qwc2-demo-app/blob/master/doc/QWC2_Documentation.md#theme-configuration-qgis-projects-and-the-themesconfigjson-file)) and put any theme thumbnails into `$QWC2_PATH/assets/img/mapthumbs/`.
The `themesConfig.json` file is used to collect the full themes configuration using GetProjectSettings.


Usage
-----

Set the `CONFIG_PATH` environment variable to the path containing the service config and permission files when starting this service (default: `config`).

Base URL:

    http://localhost:5030/

Sample requests:

    curl 'http://localhost:5030/config.json'
    curl 'http://localhost:5030/themes.json'


Docker usage
------------

### Run docker image

To run this docker image you will need the following three additional services:

* qwc-postgis
* qwc-qgis-server
* qwc-config-service
* qwc-ogc-service
* qwc-data-service

Those services can be found under https://github.com/qwc-services/. The following steps explain how to download those services and how to run the `qwc-map-viewer` with `docker-compose`.

**Step 1: Clone qwc-docker**

    git clone https://github.com/qwc-services/qwc-docker
    cd qwc-docker

**Step 2: Create docker-compose.yml file**

    cp docker-compose-example.yml docker-compose.yml

**Step 3: Choose between a version of the qwc-map-viewer**

#### qwc-map-viewer-demo

This is the demo version used in the `docker-compose-example.yml` file. With this version, the docker image comes with a preinstalled version of the latest qwc2-demo-app build and the python application for the viewer. Use this docker image, if you don't have your own build of the QWC2 app.

#### qwc-map-viewer-base

If you want to use your own QWC2 build then this is the docker image that you want to use. This docker image comes with only the python application installed on. Here is an example, on how you can add you own QWC2 build to the docker image:

```
qwc-map-viewer:
    image: sourcepole/qwc-map-viewer-base
    environment:
        - CONFIG_SERVICE_URL=http://qwc-config-service:9090/
        - QWC2_PATH=/qwc2/
        - QWC2_CONFIG=/qwc2/config.json
        - OGC_SERVICE_URL=/ows/
        - DATA_SERVICE_URL=/api/v1/data/
    ports:
        - "127.0.0.1:5030:9090"
    # Here you mount your own QWC2 build
    volumes:
    - /PATH_TO_QWC2_BUILD/:/qwc2:ro
    depends_on:
      - qwc-config-service
      - qwc-ogc-service
      - qwc-data-service
```
**Step 4: Build docker containers**

    docker-compose build

**Step 5: Start docker containers**

    docker-compose up qwc-map-viewer

For more information please visit: https://github.com/qwc-services/qwc-docker


Development
-----------

Create a virtual environment:

    virtualenv --python=/usr/bin/python3 --system-site-packages .venv

Without system packages:

    virtualenv --python=/usr/bin/python3 .venv

Activate virtual environment:

    source .venv/bin/activate

Install requirements:

    pip install -r requirements.txt

Start local service:

    CONFIG_PATH=/PATH/TO/CONFIGS/ python server.py
