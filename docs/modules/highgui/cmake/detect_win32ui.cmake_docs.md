# Documentation for `modules/highgui/cmake/detect_win32ui.cmake`

## File Metadata

- **Full Path**: `modules/highgui/cmake/detect_win32ui.cmake`
- **File Name**: `detect_win32ui.cmake`
- **File Size**: 471 bytes
- **File Type**: .cmake
- **Link to Source**: [modules/highgui/cmake/detect_win32ui.cmake](../../../modules/highgui/cmake/detect_win32ui.cmake)

## Purpose and Role

This file is located in the `modules/highgui/cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
#--- Win32 UI ---
ocv_clear_vars(HAVE_WIN32UI)
if(WITH_WIN32UI)
  try_compile(HAVE_WIN32UI
    "${CMAKE_CURRENT_BINARY_DIR}"
    "${OpenCV_SOURCE_DIR}/cmake/checks/win32uitest.cpp"
    CMAKE_FLAGS "-DLINK_LIBRARIES:STRING=user32;gdi32")
  if(HAVE_WIN32UI)
    set(__libs "user32" "gdi32")
    if(OpenCV_ARCH STREQUAL "ARM64")
      list(APPEND __libs "comdlg32" "advapi32")
    endif()
    ocv_add_external_target(win32ui "" "${__libs}" "HAVE_WIN32UI")
  endif()
endif()

```

## Purpose

This configuration file is used to control build settings, dependencies, or runtime behavior of the OpenCV library.

## Key Settings

Configuration files in OpenCV typically control:
- Build system configuration (CMake)
- Compiler flags and options
- Feature enablement/disablement
- Path specifications
- Version information
- Dependency management

## Usage

This file is processed during the build configuration phase or at runtime to customize OpenCV behavior.

