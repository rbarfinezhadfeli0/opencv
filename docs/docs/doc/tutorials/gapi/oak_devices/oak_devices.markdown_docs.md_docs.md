# Documentation for `docs/doc/tutorials/gapi/oak_devices/oak_devices.markdown_docs.md`

## File Metadata

- **Full Path**: `docs/doc/tutorials/gapi/oak_devices/oak_devices.markdown_docs.md`
- **File Name**: `oak_devices.markdown_docs.md`
- **File Size**: 1,814 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/tutorials/gapi/oak_devices/oak_devices.markdown_docs.md](../../../../../docs/doc/tutorials/gapi/oak_devices/oak_devices.markdown_docs.md)

## Purpose and Role

This file is located in the `docs/doc/tutorials/gapi/oak_devices` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/tutorials/gapi/oak_devices/oak_devices.markdown`

## File Metadata

- **Full Path**: `doc/tutorials/gapi/oak_devices/oak_devices.markdown`
- **File Name**: `oak_devices.markdown`
- **File Size**: 1,151 bytes
- **File Type**: .markdown
- **Link to Source**: [doc/tutorials/gapi/oak_devices/oak_devices.markdown](../../../../doc/tutorials/gapi/oak_devices/oak_devices.markdown)

## Purpose and Role

This file is located in the `doc/tutorials/gapi/oak_devices` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
Using DepthAI Hardware / OAK depth sensors {#tutorial_gapi_oak_devices}
=======================================================================

@tableofcontents

@prev_tutorial{tutorial_gapi_face_beautification}

![Oak-D and Oak-D-Light cameras](pics/oak.jpg)

Depth sensors compatible with Luxonis DepthAI library are supported through OpenCV Graph API (or G-API) module. RGB image and some other formats of output can be retrieved by using familiar interface of G-API module.

In order to use DepthAI sensor with OpenCV you should do the following preliminary steps:
-#  Install Luxonis DepthAI library [depthai-core](https://github.com/luxonis/depthai-core).

-#  Configure OpenCV with DepthAI library support by setting `WITH_OAK` flag in CMake. If DepthAI library is found in install folders OpenCV will be built with depthai-core (see a status `WITH_OAK` in CMake log).

-#  Build OpenCV.

Source code
-----------

You can find source code how to process heterogeneous graphs in the `modules/gapi/samples/oak_basic_infer.cpp` of the OpenCV source code library.

@add_toggle_cpp
    @include modules/gapi/samples/oak_basic_infer.cpp
@end_toggle

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

