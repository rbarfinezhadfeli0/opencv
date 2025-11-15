# Documentation for `docs/cmake/templates/OpenCVConfig-version.cmake.in_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/templates/OpenCVConfig-version.cmake.in_docs.md`
- **File Name**: `OpenCVConfig-version.cmake.in_docs.md`
- **File Size**: 1,075 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/templates/OpenCVConfig-version.cmake.in_docs.md](../../../docs/cmake/templates/OpenCVConfig-version.cmake.in_docs.md)

## Purpose and Role

This file is located in the `docs/cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/templates/OpenCVConfig-version.cmake.in`

## File Metadata

- **Full Path**: `cmake/templates/OpenCVConfig-version.cmake.in`
- **File Name**: `OpenCVConfig-version.cmake.in`
- **File Size**: 456 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/OpenCVConfig-version.cmake.in](../../cmake/templates/OpenCVConfig-version.cmake.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
set(OpenCV_VERSION @OPENCV_VERSION_PLAIN@)
set(PACKAGE_VERSION ${OpenCV_VERSION})

set(PACKAGE_VERSION_EXACT False)
set(PACKAGE_VERSION_COMPATIBLE False)

if(PACKAGE_FIND_VERSION VERSION_EQUAL PACKAGE_VERSION)
  set(PACKAGE_VERSION_EXACT True)
  set(PACKAGE_VERSION_COMPATIBLE True)
endif()

if(PACKAGE_FIND_VERSION_MAJOR EQUAL @OPENCV_VERSION_MAJOR@
   AND PACKAGE_FIND_VERSION VERSION_LESS PACKAGE_VERSION)
  set(PACKAGE_VERSION_COMPATIBLE True)
endif()

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

