---
title: "OpenStreetMap test drive"
date: 2021-06-29
categories: [Engineering,OpenStreetMap]
tags: [osm]
---


# Download extract for Ukraine region

$ git clone git@github.com:openmaptiles/openmaptiles.git
$ cd openmaptiles
$ docker run -it --rm \
-u $(id -u ${USER}):$(id -g ${USER}) \
-v "${HOME}/workspace/source/osm/tileset:/tileset" \
openmaptiles/openmaptiles-tools \
/bin/bash 
# download-osm geofabrik europe/ukraine -s state.txt

