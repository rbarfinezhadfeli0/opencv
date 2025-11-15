# Documentation for `docs/cmake/OpenCVExtraTargets.cmake_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/OpenCVExtraTargets.cmake_docs.md`
- **File Name**: `OpenCVExtraTargets.cmake_docs.md`
- **File Size**: 2,904 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/OpenCVExtraTargets.cmake_docs.md](../../docs/cmake/OpenCVExtraTargets.cmake_docs.md)

## Purpose and Role

This file is located in the `docs/cmake` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/OpenCVExtraTargets.cmake`

## File Metadata

- **Full Path**: `cmake/OpenCVExtraTargets.cmake`
- **File Name**: `OpenCVExtraTargets.cmake`
- **File Size**: 1,950 bytes
- **File Type**: .cmake
- **Link to Source**: [cmake/OpenCVExtraTargets.cmake](../cmake/OpenCVExtraTargets.cmake)

## Purpose and Role

This file is located in the `cmake` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
# ----------------------------------------------------------------------------
#   Uninstall target, for "make uninstall"
# ----------------------------------------------------------------------------
if(NOT TARGET uninstall)  # avoid conflicts with parent projects
  configure_file(
      "${OpenCV_SOURCE_DIR}/cmake/templates/cmake_uninstall.cmake.in"
      "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
      @ONLY
  )

  add_custom_target(uninstall
      COMMAND "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
  )

  if(ENABLE_SOLUTION_FOLDERS)
    set_target_properties(uninstall PROPERTIES FOLDER "CMakeTargets")
  endif()
endif()

# ----------------------------------------------------------------------------
# target building all OpenCV modules
# ----------------------------------------------------------------------------
add_custom_target(opencv_modules)
if(ENABLE_SOLUTION_FOLDERS)
  set_target_properties(opencv_modules PROPERTIES FOLDER "extra")
endif()


# ----------------------------------------------------------------------------
# targets building all tests
# ----------------------------------------------------------------------------
if(BUILD_TESTS)
  add_custom_target(opencv_tests)
  if(ENABLE_SOLUTION_FOLDERS)
    set_target_properties(opencv_tests PROPERTIES FOLDER "extra")
  endif()
endif()
if(BUILD_PERF_TESTS)
  add_custom_target(opencv_perf_tests)
  if(ENABLE_SOLUTION_FOLDERS)
    set_target_properties(opencv_perf_tests PROPERTIES FOLDER "extra")
  endif()
endif()

# Documentation
if(BUILD_DOCS)
  add_custom_target(opencv_docs)
  add_custom_target(install_docs DEPENDS opencv_docs
    COMMAND "${CMAKE_COMMAND}" -DCMAKE_INSTALL_COMPONENT=docs -P "${CMAKE_BINARY_DIR}/cmake_install.cmake")
endif()

# Samples
if(BUILD_EXAMPLES)
  add_custom_target(opencv_samples)
  if(ENABLE_SOLUTION_FOLDERS)
    set_target_properties(opencv_samples PROPERTIES FOLDER "extra")
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

