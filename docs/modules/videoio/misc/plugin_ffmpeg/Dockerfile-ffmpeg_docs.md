# Documentation for `modules/videoio/misc/plugin_ffmpeg/Dockerfile-ffmpeg`

## File Metadata

- **Full Path**: `modules/videoio/misc/plugin_ffmpeg/Dockerfile-ffmpeg`
- **File Name**: `Dockerfile-ffmpeg`
- **File Size**: 903 bytes
- **File Type**: no extension
- **Link to Source**: [modules/videoio/misc/plugin_ffmpeg/Dockerfile-ffmpeg](../../../../modules/videoio/misc/plugin_ffmpeg/Dockerfile-ffmpeg)

## Purpose and Role

This file is located in the `modules/videoio/misc/plugin_ffmpeg` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
FROM ubuntu:18.04

RUN apt-get update && apt-get --no-install-recommends install -y \
        pkg-config \
        cmake \
        g++ \
        ninja-build \
        make \
        nasm \
    && \
    rm -rf /var/lib/apt/lists/*

ARG VER

ADD ffmpeg-${VER}.tar.xz /ffmpeg/

WORKDIR /ffmpeg/ffmpeg-${VER}
RUN ./configure \
    --enable-avresample \
    --prefix=/ffmpeg-shared \
    --enable-shared \
    --disable-static \
    --disable-programs \
    --disable-doc \
    --disable-avdevice \
    --disable-postproc \
    && make -j8 install \
    && make clean \
    && make distclean

RUN ./configure \
    --enable-avresample \
    --prefix=/ffmpeg-static \
    --disable-shared \
    --enable-static \
    --enable-pic \
    --disable-programs \
    --disable-doc \
    --disable-avdevice \
    --disable-postproc \
    && make -j8 install \
    && make clean \
    && make distclean

WORKDIR /tmp

```

## General Information

This file is part of the OpenCV repository infrastructure.

