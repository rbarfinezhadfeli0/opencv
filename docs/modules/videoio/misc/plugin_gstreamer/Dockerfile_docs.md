# Documentation for `modules/videoio/misc/plugin_gstreamer/Dockerfile`

## File Metadata

- **Full Path**: `modules/videoio/misc/plugin_gstreamer/Dockerfile`
- **File Name**: `Dockerfile`
- **File Size**: 309 bytes
- **File Type**: no extension
- **Link to Source**: [modules/videoio/misc/plugin_gstreamer/Dockerfile](../../../../modules/videoio/misc/plugin_gstreamer/Dockerfile)

## Purpose and Role

This file is located in the `modules/videoio/misc/plugin_gstreamer` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
FROM ubuntu:18.04

RUN apt-get update && apt-get --no-install-recommends install -y \
        libgstreamer-plugins-base1.0-dev \
        libgstreamer-plugins-good1.0-dev \
        libgstreamer1.0-dev \
        cmake \
        g++ \
        ninja-build \
    && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

```

## General Information

This file is part of the OpenCV repository infrastructure.

