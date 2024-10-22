---
title: "Using gst-shark tool"
date: 2024-10-20
categories: [Engineering,Linux]
tags: [video,audio,gstremer]
---
# Install

Clone:
```shell
$ git clone https://github.com/RidgeRun/gst-shark.git
$ cd gst-shark
$ git checkout -b v0.8.1 v0.8.1
```

Building:
```shell
$ meson setup builddir --prefix=/usr -Denable-plotting=disabled
$ cd builddir
$ meson compile
```
Install:
```shell
$ sudo meson install
or
$ meson install --destdir=$HOME/gst-shark
```
Check if new plugins are available:
```shell
$ gst-inspect-1.0 --no-colors --plugin | grep shark
sharktracers:  bitrate (GstTracerFactory)
sharktracers:  buffer (GstTracerFactory)
sharktracers:  cpuusage (GstTracerFactory)
sharktracers:  framerate (GstTracerFactory)
sharktracers:  graphic (GstTracerFactory)
sharktracers:  interlatency (GstTracerFactory)
sharktracers:  proctime (GstTracerFactory)
sharktracers:  queuelevel (GstTracerFactory)
sharktracers:  scheduletime (GstTracerFactory)
```
# Using

Using `cpuusage` tracer:
```shell
$ GST_DEBUG="GST_TRACER:7" GST_TRACERS="cpuusage" GST_SHARK_LOCATION="$HOME/cpuusage" \
    gst-launch-1.0 videotestsrc num-buffers=10000 ! 'video/x-raw, format=(string)YUY2, width=(int)640, height=(int)480, framerate=(fraction)30/1' ! videoconvert ! queue ! avenc_h263p ! queue ! avimux ! fakesink
```

Plotting:
```shell
$ cd $HOME/gst-shark/scripts/graphics
$ ./gstshark-plot $HOME/cpuusage -s pdf -l outside
```
