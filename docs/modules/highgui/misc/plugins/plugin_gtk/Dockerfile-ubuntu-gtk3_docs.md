# Documentation for `modules/highgui/misc/plugins/plugin_gtk/Dockerfile-ubuntu-gtk3`

## File Metadata

- **Full Path**: `modules/highgui/misc/plugins/plugin_gtk/Dockerfile-ubuntu-gtk3`
- **File Name**: `Dockerfile-ubuntu-gtk3`
- **File Size**: 403 bytes
- **File Type**: no extension
- **Link to Source**: [modules/highgui/misc/plugins/plugin_gtk/Dockerfile-ubuntu-gtk3](../../../../../modules/highgui/misc/plugins/plugin_gtk/Dockerfile-ubuntu-gtk3)

## Purpose and Role

This file is located in the `modules/highgui/misc/plugins/plugin_gtk` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
ARG VER
FROM ubuntu:$VER

RUN \
  apt-get update && \
  DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    pkg-config \
    cmake \
    g++ \
    ninja-build \
  && \
  rm -rf /var/lib/apt/lists/*

RUN \
  apt-get update && \
  DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    libgtk-3-dev \
  && \
  rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

```

## General Information

This file is part of the OpenCV repository infrastructure.

