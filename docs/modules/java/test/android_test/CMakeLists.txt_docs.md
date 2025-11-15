# Documentation for `modules/java/test/android_test/CMakeLists.txt`

## File Metadata

- **Full Path**: `modules/java/test/android_test/CMakeLists.txt`
- **File Name**: `CMakeLists.txt`
- **File Size**: 1,695 bytes
- **File Type**: .txt
- **Link to Source**: [modules/java/test/android_test/CMakeLists.txt](../../../../modules/java/test/android_test/CMakeLists.txt)

## Purpose and Role

This file is located in the `modules/java/test/android_test` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
project(opencv_test_android)

set(OPENCV_ANDROID_TEST_DIR "${OpenCV_BINARY_DIR}/android_test" CACHE INTERNAL "")
file(REMOVE_RECURSE "${OPENCV_ANDROID_TEST_DIR}")

set(ANDROID_TESTS_SRC_DIRS
"'${CMAKE_CURRENT_SOURCE_DIR}/src', \
'${OpenCV_SOURCE_DIR}/modules/java/test/common_test/src', \
'${CMAKE_BINARY_DIR}/modules/java_bindings_generator/gen/test'" CACHE INTERNAL "")

set(ANDROID_TESTS_RES_DIR "'${OpenCV_SOURCE_DIR}/modules/java/test/common_test/res'" CACHE INTERNAL "")

list(APPEND TEST_PROJECT_FILES "CMakeLists.txt" "gradle.properties" "settings.gradle")
foreach(TEST_PROJECT_FILE ${TEST_PROJECT_FILES})
    file(COPY "${CMAKE_CURRENT_SOURCE_DIR}/${TEST_PROJECT_FILE}" DESTINATION "${OPENCV_ANDROID_TEST_DIR}")
endforeach()
file(COPY "${CMAKE_CURRENT_SOURCE_DIR}/tests_module/AndroidManifest.xml" DESTINATION "${OPENCV_ANDROID_TEST_DIR}/tests_module")
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/tests_module/build.gradle.in" "${OPENCV_ANDROID_TEST_DIR}/tests_module/build.gradle" @ONLY)
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/build.gradle.in" "${OPENCV_ANDROID_TEST_DIR}/build.gradle" @ONLY)

file(COPY "${OpenCV_SOURCE_DIR}/platforms/android/gradle-wrapper/gradlew" DESTINATION "${OPENCV_ANDROID_TEST_DIR}")
file(COPY "${OpenCV_SOURCE_DIR}/platforms/android/gradle-wrapper/gradlew.bat" DESTINATION "${OPENCV_ANDROID_TEST_DIR}")
file(COPY "${OpenCV_SOURCE_DIR}/platforms/android/gradle-wrapper/gradle/wrapper/gradle-wrapper.jar" DESTINATION "${OPENCV_ANDROID_TEST_DIR}/gradle/wrapper")

configure_file("${OpenCV_SOURCE_DIR}/platforms/android/gradle-wrapper/gradle/wrapper/gradle-wrapper.properties.in" "${OPENCV_ANDROID_TEST_DIR}/gradle/wrapper/gradle-wrapper.properties" @ONLY)

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

