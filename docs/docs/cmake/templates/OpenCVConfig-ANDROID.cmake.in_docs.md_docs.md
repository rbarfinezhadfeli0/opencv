# Documentation for `docs/cmake/templates/OpenCVConfig-ANDROID.cmake.in_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/templates/OpenCVConfig-ANDROID.cmake.in_docs.md`
- **File Name**: `OpenCVConfig-ANDROID.cmake.in_docs.md`
- **File Size**: 1,190 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/templates/OpenCVConfig-ANDROID.cmake.in_docs.md](../../../docs/cmake/templates/OpenCVConfig-ANDROID.cmake.in_docs.md)

## Purpose and Role

This file is located in the `docs/cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/templates/OpenCVConfig-ANDROID.cmake.in`

## File Metadata

- **Full Path**: `cmake/templates/OpenCVConfig-ANDROID.cmake.in`
- **File Name**: `OpenCVConfig-ANDROID.cmake.in`
- **File Size**: 571 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/OpenCVConfig-ANDROID.cmake.in](../../cmake/templates/OpenCVConfig-ANDROID.cmake.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
# Android API level from which OpenCV has been compiled is remembered
set(OpenCV_ANDROID_NATIVE_API_LEVEL "@OpenCV_ANDROID_NATIVE_API_LEVEL_CONFIGCMAKE@")

# ==============================================================
#  Check OpenCV availability
# ==============================================================
if(OpenCV_ANDROID_NATIVE_API_LEVEL GREATER ANDROID_NATIVE_API_LEVEL)
  if(NOT OpenCV_FIND_QUIETLY)
    message(WARNING "Minimum required by OpenCV API level is android-${OpenCV_ANDROID_NATIVE_API_LEVEL}")
  endif()
  set(OpenCV_FOUND 0)
  return()
endif()

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

