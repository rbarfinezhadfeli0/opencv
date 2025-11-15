# Documentation for `docs/samples/cpp/CMakeLists.txt_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/CMakeLists.txt_docs.md`
- **File Name**: `CMakeLists.txt_docs.md`
- **File Size**: 3,867 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/CMakeLists.txt_docs.md](../../../docs/samples/cpp/CMakeLists.txt_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/CMakeLists.txt`

## File Metadata

- **Full Path**: `samples/cpp/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 2,932 bytes
- **File Type**: .txt
- **Link to Source**: [samples/cpp/CMakeLists.txt](../../samples/cpp/CMakeLists.txt)

## Purpose and Role

This file is located in the `samples/cpp` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
ocv_install_example_src(cpp *.cpp *.hpp CMakeLists.txt)

set(OPENCV_CPP_SAMPLES_REQUIRED_DEPS
  opencv_core
  opencv_imgproc
  opencv_flann
  opencv_imgcodecs
  opencv_videoio
  opencv_highgui
  opencv_ml
  opencv_video
  opencv_objdetect
  opencv_photo
  opencv_features2d
  opencv_calib3d
  opencv_stitching
  opencv_dnn
  opencv_gapi
  ${OPENCV_MODULES_PUBLIC}
  ${OpenCV_LIB_COMPONENTS})
ocv_check_dependencies(${OPENCV_CPP_SAMPLES_REQUIRED_DEPS})

if(NOT BUILD_EXAMPLES OR NOT OCV_DEPENDENCIES_FOUND)
  return()
endif()

set(DEPS_example_snippet_imgproc_segmentation opencv_core opencv_imgproc)
set(DEPS_example_cpp_intelligent_scissors opencv_core opencv_imgproc opencv_imgcodecs opencv_highgui)

project(cpp_samples)
ocv_include_modules_recurse(${OPENCV_CPP_SAMPLES_REQUIRED_DEPS})
file(GLOB_RECURSE cpp_samples RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.cpp)
if(NOT HAVE_opencv_cudaarithm OR NOT HAVE_opencv_cudafilters)
  ocv_list_filterout(cpp_samples "/gpu/")
endif()
ocv_list_filterout(cpp_samples "real_time_pose_estimation/")
ocv_list_filterout(cpp_samples "parallel_backend/")
foreach(sample_filename ${cpp_samples})
  set(package "cpp")
  if(sample_filename MATCHES "tutorial_code/snippet")
    set(package "snippet")
  elseif(sample_filename MATCHES "tutorial_code")
    set(package "tutorial")
  endif()
  ocv_define_sample(tgt ${sample_filename} ${package})
  set(deps ${OPENCV_CPP_SAMPLES_REQUIRED_DEPS})
  if(DEFINED DEPS_${tgt})
    set(deps ${DEPS_${tgt}})
  endif()
  ocv_target_link_libraries(${tgt} PRIVATE ${OPENCV_LINKER_LIBS} ${deps})
  if(sample_filename MATCHES "/gpu/" AND HAVE_opencv_cudaarithm AND HAVE_opencv_cuda_filters)
    ocv_target_link_libraries(${tgt} PRIVATE opencv_cudaarithm opencv_cudafilters)
  endif()
  if(sample_filename MATCHES "/viz/")
    ocv_target_link_libraries(${tgt} PRIVATE ${VTK_LIBRARIES})
    target_compile_definitions(${tgt} PRIVATE -DUSE_VTK)
  endif()
  if(HAVE_OPENGL AND sample_filename MATCHES "detect_mser")
    target_compile_definitions(${tgt} PRIVATE HAVE_OPENGL)
    ocv_target_link_libraries(${tgt} PRIVATE "${OPENGL_LIBRARIES}")
  endif()
  if(sample_filename MATCHES "simd_")
    # disabled intentionally - demonstration purposes only
    #target_include_directories(${tgt} PRIVATE "${CMAKE_CURRENT_LIST_DIR}")
    #target_compile_definitions(${tgt} PRIVATE OPENCV_SIMD_CONFIG_HEADER=opencv_simd_config_custom.hpp)
    #target_compile_definitions(${tgt} PRIVATE OPENCV_SIMD_CONFIG_INCLUDE_DIR=1)
    #target_compile_options(${tgt} PRIVATE -mavx2)
  endif()
endforeach()

include("tutorial_code/calib3d/real_time_pose_estimation/CMakeLists.txt" OPTIONAL)

# Standalone samples only
if(OpenCV_FOUND AND NOT CMAKE_VERSION VERSION_LESS "3.1")
  add_subdirectory("example_cmake")
endif()
if(OpenCV_FOUND AND NOT CMAKE_VERSION VERSION_LESS "3.9"
    AND NOT OPENCV_EXAMPLES_SKIP_PARALLEL_BACKEND
)
  add_subdirectory("tutorial_code/core/parallel_backend")
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

