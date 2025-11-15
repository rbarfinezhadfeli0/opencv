# Documentation for `docs/cmake/templates/opencv_abi.xml.in_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/templates/opencv_abi.xml.in_docs.md`
- **File Name**: `opencv_abi.xml.in_docs.md`
- **File Size**: 1,493 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/templates/opencv_abi.xml.in_docs.md](../../../docs/cmake/templates/opencv_abi.xml.in_docs.md)

## Purpose and Role

This file is located in the `docs/cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/templates/opencv_abi.xml.in`

## File Metadata

- **Full Path**: `cmake/templates/opencv_abi.xml.in`
- **File Name**: `opencv_abi.xml.in`
- **File Size**: 934 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/opencv_abi.xml.in](../../cmake/templates/opencv_abi.xml.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
<?xml version="1.0" encoding="utf-8"?>

<!--

    This file is auto-generated

-->

<descriptor>

<version>
    @OPENCV_ABI_VERSION@
</version>

<headers>
    @OPENCV_ABI_HEADERS@
</headers>

<libs>
    @OPENCV_ABI_LIBRARIES@
</libs>

<skip_headers>
    opencv2/core/hal/intrin*
    opencv2/core/hal/*macros.*
    opencv2/core/hal/*.impl.*
    opencv2/core/cuda*
    opencv2/core/opencl*
    opencv2/core/parallel/backend/*
    opencv2/core/private*
    opencv2/core/*quaternion*
    opencv/cxeigen.hpp
    opencv2/core/eigen.hpp
    opencv2/flann/hdf5.h
    opencv2/imgcodecs/imgcodecs_c.h
    opencv2/imgcodecs/ios.h
    opencv2/imgcodecs/macosx.h
    opencv2/videoio/videoio_c.h
    opencv2/videoio/cap_ios.h
    opencv2/xobjdetect/private.hpp
    @OPENCV_ABI_SKIP_HEADERS@
</skip_headers>

<skip_libs>
    @OPENCV_ABI_SKIP_LIBRARIES@
</skip_libs>

<gcc_options>
 -std=c++11
 @OPENCV_ABI_GCC_OPTIONS@
</gcc_options>

</descriptor>

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

