# openstreetmap-tile-server

This container allows you to easily set up an OpenStreetMap PNG tile server given a `.osm.pbf` file. It is based on the [latest Ubuntu 18.04 LTS guide](https://switch2osm.org/manually-building-a-tile-server-18-04-lts/) from [switch2osm.org](https://switch2osm.org/) and therefore uses the default OpenStreetMap style.

## Setting up the server

First create a Docker volume to hold the PostgreSQL database that will contain the OpenStreetMap data:

    docker volume create openstreetmap-data

Tiles that have already been rendered will be stored in `/var/lib/mod_tile`. To make sure that this data survives container restarts, you should create another volume for it:

    docker volume create openstreetmap-rendered-tiles

Next, download an .osm.pbf extract from geofabrik.de for the region that you're interested in. You can then start importing it into PostgreSQL by running a container and mounting the file as `/data.osm.pbf`. For example:

    docker run -v /absolute/path/to/luxembourg.osm.pbf:/data.osm.pbf -v openstreetmap-data:/var/lib/postgresql/10/main cogitoweb/openstreetmap-tile-server import

If the container exits without errors, then your data has been successfully imported and you are now ready to run the tile server.

## Running the server (with performance tuning and preserving tiles)

Run the server like this:

    docker run --shm-size 8g -e THREADS=8 -p 80:80 -p 443:443 -v openstreetmap-data:/var/lib/postgresql/10/main -v /etc/ssl/cogito:/etc/ssl/cogito -v openstreetmap-rendered-tiles:/var/lib/mod_tile cogitoweb/openstreetmap-tile-server:latest run

Your tiles will now be available at http://localhost:80/tile/{z}/{x}/{y}.png. If you open `leaflet-demo.html` in your browser, you should be able to see the tiles served by your own machine. Note that it will initially quite a bit of time to render the larger tiles for the first time.

## Performance tuning

The import and tile serving processes use 4 threads by default, but this number can be changed by setting the `THREADS` environment variable. For example:

    docker run -p 80:80 -e THREADS=24 -v openstreetmap-data:/var/lib/postgresql/10/main -d cogitoweb/openstreetmap-tile-server run

## Preserving rendered tiles

Tiles that have already been rendered will be stored in `/var/lib/mod_tile`. To make sure that this data survives container restarts, you should create another volume for it:

    docker volume create openstreetmap-rendered-tiles
    docker run -p 80:80 -v openstreetmap-data:/var/lib/postgresql/10/main -v openstreetmap-rendered-tiles:/var/lib/mod_tile -d cogitoweb/openstreetmap-tile-server run

## Debugging the service mode

Just substitute the 'run' command with 'debug' like so:

    docker run --shm-size 14g -p 80:80 -v osm-europe-data:/var/lib/postgresql/10/main -v /home/ubuntu/osm:/etc/postgresql/10/main/conf.d cogitoweb/openstreetmap-tile-server:latest debug

## License

```
Copyright 2018 Cogito

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
