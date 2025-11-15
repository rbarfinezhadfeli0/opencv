# Documentation for `docs/samples/CMakeLists.example.in_docs.md`

## File Metadata

- **Full Path**: `docs/samples/CMakeLists.example.in_docs.md`
- **File Name**: `CMakeLists.example.in_docs.md`
- **File Size**: 1,831 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/CMakeLists.example.in_docs.md](../../docs/samples/CMakeLists.example.in_docs.md)

## Purpose and Role

This file is located in the `docs/samples` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/CMakeLists.example.in`

## File Metadata

- **Full Path**: `samples/CMakeLists.example.in`
- **File Name**: `CMakeLists.example.in`
- **File Size**: 1,293 bytes
- **File Type**: .in
- **Link to Source**: [samples/CMakeLists.example.in](../samples/CMakeLists.example.in)

## Purpose and Role

This file is located in the `samples` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
# cmake needs this line
cmake_minimum_required(VERSION 3.5)

if(NOT DEFINED EXAMPLE_NAME)
  message(FATAL_ERROR "Invalid build script: missing EXAMPLE_NAME")
endif()
if(NOT DEFINED EXAMPLE_FILE)
  message(FATAL_ERROR "Invalid build script: missing EXAMPLE_FILE")
endif()

file(TO_CMAKE_PATH "${EXAMPLE_FILE}" EXAMPLE_FILE)
message(STATUS "Project: ${EXAMPLE_NAME}")
message(STATUS "File   : ${EXAMPLE_FILE}")

# Define project name
project(${EXAMPLE_NAME})

# Find OpenCV, you may need to set OpenCV_DIR variable
# to the absolute path to the directory containing OpenCVConfig.cmake file
# via the command line or GUI
find_package(OpenCV REQUIRED)

# If the package has been found, several variables will
# be set, you can find the full list with descriptions
# in the OpenCVConfig.cmake file.
# Print some message showing some of them
message(STATUS "OpenCV library status:")
message(STATUS "    config: ${OpenCV_DIR}")
message(STATUS "    version: ${OpenCV_VERSION}")
message(STATUS "    libraries: ${OpenCV_LIBS}")
message(STATUS "    include path: ${OpenCV_INCLUDE_DIRS}")

# Declare the executable target built from your sources
add_executable(${EXAMPLE_NAME} "${EXAMPLE_FILE}")

# Link your application with OpenCV libraries
target_link_libraries(${EXAMPLE_NAME} PRIVATE ${OpenCV_LIBS})

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

