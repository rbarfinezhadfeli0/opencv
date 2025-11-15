# Documentation for `docs/cmake/OpenCVDetectApacheAnt.cmake_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/OpenCVDetectApacheAnt.cmake_docs.md`
- **File Name**: `OpenCVDetectApacheAnt.cmake_docs.md`
- **File Size**: 2,073 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/OpenCVDetectApacheAnt.cmake_docs.md](../../docs/cmake/OpenCVDetectApacheAnt.cmake_docs.md)

## Purpose and Role

This file is located in the `docs/cmake` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/OpenCVDetectApacheAnt.cmake`

## File Metadata

- **Full Path**: `cmake/OpenCVDetectApacheAnt.cmake`
- **File Name**: `OpenCVDetectApacheAnt.cmake`
- **File Size**: 1,104 bytes
- **File Type**: .cmake
- **Link to Source**: [cmake/OpenCVDetectApacheAnt.cmake](../cmake/OpenCVDetectApacheAnt.cmake)

## Purpose and Role

This file is located in the `cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
set(OPENCV_JAVA_SOURCE_VERSION "" CACHE STRING "Java source version (javac Ant target)")
set(OPENCV_JAVA_TARGET_VERSION "" CACHE STRING "Java target version (javac Ant target)")

file(TO_CMAKE_PATH "$ENV{ANT_DIR}" ANT_DIR_ENV_PATH)
file(TO_CMAKE_PATH "$ENV{ProgramFiles}" ProgramFiles_ENV_PATH)

if(CMAKE_HOST_WIN32)
  set(ANT_NAME ant.bat)
else()
  set(ANT_NAME ant)
endif()

find_host_program(ANT_EXECUTABLE NAMES ${ANT_NAME}
  PATHS "${ANT_DIR_ENV_PATH}/bin" "${ProgramFiles_ENV_PATH}/apache-ant/bin"
  NO_DEFAULT_PATH
  )

find_host_program(ANT_EXECUTABLE NAMES ${ANT_NAME})

if(ANT_EXECUTABLE)
  execute_process(COMMAND ${ANT_EXECUTABLE} -version
    RESULT_VARIABLE ANT_ERROR_LEVEL
    OUTPUT_VARIABLE ANT_VERSION_FULL
    OUTPUT_STRIP_TRAILING_WHITESPACE)
  if (ANT_ERROR_LEVEL)
    unset(ANT_EXECUTABLE)
    unset(ANT_EXECUTABLE CACHE)
  else()
    string(REGEX MATCH "[0-9]+.[0-9]+.[0-9]+" ANT_VERSION "${ANT_VERSION_FULL}")
    set(ANT_VERSION "${ANT_VERSION}" CACHE INTERNAL "Detected ant version")

    message(STATUS "Found apache ant: ${ANT_EXECUTABLE} (${ANT_VERSION})")
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

