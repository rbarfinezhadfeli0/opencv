# Documentation for `docs/samples/opengl/CMakeLists.txt_docs.md`

## File Metadata

- **Full Path**: `docs/samples/opengl/CMakeLists.txt_docs.md`
- **File Name**: `CMakeLists.txt_docs.md`
- **File Size**: 2,339 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/opengl/CMakeLists.txt_docs.md](../../../docs/samples/opengl/CMakeLists.txt_docs.md)

## Purpose and Role

This file is located in the `docs/samples/opengl` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/opengl/CMakeLists.txt`

## File Metadata

- **Full Path**: `samples/opengl/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 1,389 bytes
- **File Type**: .txt
- **Link to Source**: [samples/opengl/CMakeLists.txt](../../samples/opengl/CMakeLists.txt)

## Purpose and Role

This file is located in the `samples/opengl` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
if(APPLE)
  return()
endif()

if(UNIX)
  find_package(X11 QUIET)
endif()

find_package(PkgConfig QUIET)
pkg_search_module(EPOXY QUIET epoxy)

SET(OPENCV_OPENGL_SAMPLES_REQUIRED_DEPS
  opencv_core
  opencv_imgproc
  opencv_imgcodecs
  opencv_videoio
  opencv_highgui)
ocv_check_dependencies(${OPENCV_OPENGL_SAMPLES_REQUIRED_DEPS})

if(BUILD_EXAMPLES AND OCV_DEPENDENCIES_FOUND)
  project(opengl_samples)
  ocv_include_modules_recurse(${OPENCV_OPENGL_SAMPLES_REQUIRED_DEPS})
  file(GLOB all_samples RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.cpp)
  if(NOT X11_FOUND)
    ocv_list_filterout(all_samples "opengl_interop")
  endif()
  if(NOT EPOXY_FOUND)
    ocv_list_filterout(all_samples "opengl3_2")
  endif()
  foreach(sample_filename ${all_samples})
    ocv_define_sample(tgt ${sample_filename} opengl)
    ocv_target_link_libraries(${tgt} PRIVATE "${OPENGL_LIBRARIES}" "${OPENCV_OPENGL_SAMPLES_REQUIRED_DEPS}")
    if(sample_filename STREQUAL "opengl_interop.cpp")
      ocv_target_link_libraries(${tgt} PRIVATE ${X11_LIBRARIES})
      ocv_target_include_directories(${tgt} ${X11_INCLUDE_DIR})
    endif()
    if(sample_filename STREQUAL "opengl3_2.cpp")
      ocv_target_link_libraries(${tgt} PRIVATE ${EPOXY_LIBRARIES})
      ocv_target_include_directories(${tgt} PRIVATE ${EPOXY_INCLUDE_DIRS})
    endif()
  endforeach()
endif()

ocv_install_example_src(opengl *.cpp *.hpp CMakeLists.txt)

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

