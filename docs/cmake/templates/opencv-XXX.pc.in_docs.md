# Documentation for `cmake/templates/opencv-XXX.pc.in`

## File Metadata

- **Full Path**: `cmake/templates/opencv-XXX.pc.in`
- **File Name**: `opencv-XXX.pc.in`
- **File Size**: 301 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/opencv-XXX.pc.in](../../cmake/templates/opencv-XXX.pc.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
# Package Information for pkg-config

prefix=@prefix@
exec_prefix=@exec_prefix@
libdir=@libdir@
includedir=@includedir@

Name: OpenCV
Description: Open Source Computer Vision Library
Version: @OPENCV_VERSION_PLAIN@
Libs: @OPENCV_PC_LIBS@
Libs.private: @OPENCV_PC_LIBS_PRIVATE@
Cflags: -I${includedir}

```

## General Information

This file is part of the OpenCV repository infrastructure.

