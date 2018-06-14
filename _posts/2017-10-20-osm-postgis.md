---
layout: post
title: "OSM Data in PostGIS on docker"
date: 2017-10-20
excerpt: "Loading OSM data into PostGIS on Docker"
tags: [sql, docker, spatial, geo, gis, postgres, data, db, database]
comments: true
---

How to get started with OpenStreetMap (OSM) data in Postgres/PostGIS databases using docker.

## Required tools/data
 * Download an OSM dataset for the region you are interested in from [http://download.geofabrik.de/](http://download.geofabrik.de/), This guide assumes you have downloaded the [australian-latest](http://download.geofabrik.de/australia-oceania/australia.html) data set but any region will work that fits inside your database storage
 * Have docker installed on your machine
 

## Installing PostGIS
 Using docker we can simplify the installation of Postgres & PostGIS and use the same installation on any operating system. The [mdillon/postgis](https://hub.docker.com/r/mdillon/postgis/) Docker image extends the official Postgres image with PostGIS extension’s to support spatial data queries.

Start the database with
```
docker run --name postgis-dev -p 5444:5432 -e POSTGRES_PASSWORD=mysecretpassword -d mdillon/postgis
```

## Create a GIS database to load the data into
Connect to the database
```
docker run -it --link postgis-dev:postgres --rm postgres sh -c 'exec psql -h "$POSTGRES_PORT_5432_TCP_ADDR" -p "$POSTGRES_PORT_5432_TCP_PORT" -U postgres'
```
Create the 'GIS' database and enable postgis extension for it
```
postgres=# CREATE DATABASE example_gis;
postgres=# \connect example_gis;
gis=# CREATE EXTENSION postgis;
gis=# \q
```

*Note: ``` \q ``` is used to exit the psql shell*

## Load the data
*If find yourself regularly loading data into PostGIS, using a native install of osm2pgsql may be more suitable and faster than the dockerized version shown below*

When following Docker instructions replace 172.17.0.2 with the ip address of the postgis container, this can be found with docker inspect
```bash
docker inspect postgis-dev
[
    {
        "Id": "f6090...",
        ...
            "MacAddress": "02:42:ac:11:00:AA",
            "Networks": {
                "bridge": {
                    ...
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    ...
                }
            }
        }
    }
]
```
Docker Windows: (Drive must be shared in Docker settings, assuming files are stored in C:\Data )

```bash
docker run -v c:/data/:/data -i -t --rm openfirmware/osm2pgsql -c 'osm2pgsql -cW /data/australia-latest.osm.bz2 --port 5432 --host 172.17.0.2 --username postgres'
```

Docker Linux: (assuming files are stored in /data/ )
```bash
docker run -v /data/:/data -i -t --rm openfirmware/osm2pgsql -c 'osm2pgsql -cW /data/australia-latest.osm.bz2 --port 5432 --host 172.17.0.2 --username postgres'
```

Native osm2pgsql:
```bash
osm2pgsql -cW /data/australia-latest.osm.bz2 --port 5444 --host localhost --username postgres
```

Expected Output:

```
osm2pgsql SVN version 0.87.2 (64bit id space)

Password:mysecretpassword
Using built-in tag processing pipeline
Using projection SRS 900913 (Spherical Mercator)
Setting up table: planet_osm_point
Setting up table: planet_osm_line
Setting up table: planet_osm_polygon
Setting up table: planet_osm_roads
Allocating memory for dense node cache
Allocating dense node cache in one big chunk
Allocating memory for sparse node cache
Sharing dense sparse
Node-cache: cache=800MB, maxblocks=102400*8192, allocation method=3
Mid: Ram, scale=100

Reading in file: /data/australia-latest.osm.bz2
Processing: Node(37987k 184.4k/s) Way(2540k 11.19k/s) Relation(0 0.00/s)
```


## Finished
You now have a local copy of the OSM maps data for your selected region, this data can be used with GIS systems such as [QGIS](http://www.qgis.org/en/site/) or visualized with tools like [OpenJump](http://www.openjump.org/) or used during analysis and software development.

Leave a comment if you would like me to continue this as a series of working with spatial data.

![Output visualised in OpenJump](/images/posts/osm-postgis/Australia.png) 
