# Documentation for `docs/cmake/templates/OpenCVConfig-IPPIW.cmake.in_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/templates/OpenCVConfig-IPPIW.cmake.in_docs.md`
- **File Name**: `OpenCVConfig-IPPIW.cmake.in_docs.md`
- **File Size**: 850 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/templates/OpenCVConfig-IPPIW.cmake.in_docs.md](../../../docs/cmake/templates/OpenCVConfig-IPPIW.cmake.in_docs.md)

## Purpose and Role

This file is located in the `docs/cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/templates/OpenCVConfig-IPPIW.cmake.in`

## File Metadata

- **Full Path**: `cmake/templates/OpenCVConfig-IPPIW.cmake.in`
- **File Name**: `OpenCVConfig-IPPIW.cmake.in`
- **File Size**: 241 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/OpenCVConfig-IPPIW.cmake.in](../../cmake/templates/OpenCVConfig-IPPIW.cmake.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
if(NOT TARGET ippiw)
  add_library(ippiw STATIC IMPORTED)
  set_target_properties(ippiw PROPERTIES
    IMPORTED_LINK_INTERFACE_LIBRARIES ""
    IMPORTED_LOCATION "${OpenCV_INSTALL_PATH}/@IPPIW_INSTALL_PATH_RELATIVE_CONFIGCMAKE@"
  )
endif()

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

