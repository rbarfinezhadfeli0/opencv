# Documentation for `modules/videoio/misc/plugin_ffmpeg/Dockerfile-ubuntu`

## File Metadata

- **Full Path**: `modules/videoio/misc/plugin_ffmpeg/Dockerfile-ubuntu`
- **File Name**: `Dockerfile-ubuntu`
- **File Size**: 350 bytes
- **File Type**: no extension
- **Link to Source**: [modules/videoio/misc/plugin_ffmpeg/Dockerfile-ubuntu](../../../../modules/videoio/misc/plugin_ffmpeg/Dockerfile-ubuntu)

## Purpose and Role

This file is located in the `modules/videoio/misc/plugin_ffmpeg` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
ARG VER
FROM ubuntu:$VER

RUN apt-get update && apt-get --no-install-recommends install -y \
        libavcodec-dev \
        libavfilter-dev \
        libavformat-dev \
        libavresample-dev \
        libavutil-dev \
        pkg-config \
        cmake \
        g++ \
        ninja-build \
    && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

```

## General Information

This file is part of the OpenCV repository infrastructure.

