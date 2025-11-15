# Documentation for `cmake/templates/OpenCVConfig-IPPICV.cmake.in`

## File Metadata

- **Full Path**: `cmake/templates/OpenCVConfig-IPPICV.cmake.in`
- **File Name**: `OpenCVConfig-IPPICV.cmake.in`
- **File Size**: 245 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/OpenCVConfig-IPPICV.cmake.in](../../cmake/templates/OpenCVConfig-IPPICV.cmake.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
if(NOT TARGET ippicv)
  add_library(ippicv STATIC IMPORTED)
  set_target_properties(ippicv PROPERTIES
    IMPORTED_LINK_INTERFACE_LIBRARIES ""
    IMPORTED_LOCATION "${OpenCV_INSTALL_PATH}/@IPPICV_INSTALL_PATH_RELATIVE_CONFIGCMAKE@"
  )
endif()

```

## General Information

This file is part of the OpenCV repository infrastructure.

