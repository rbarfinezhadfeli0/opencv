# Documentation for `3rdparty/tbb/version_string.ver.cmakein`

## File Metadata

- **Full Path**: `3rdparty/tbb/version_string.ver.cmakein`
- **File Name**: `version_string.ver.cmakein`
- **File Size**: 429 bytes
- **File Type**: .cmakein
- **Link to Source**: [3rdparty/tbb/version_string.ver.cmakein](../../3rdparty/tbb/version_string.ver.cmakein)

## Purpose and Role

This file is located in the `3rdparty/tbb` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#define __TBB_VERSION_STRINGS(N) \
#N": BUILD_PACKAGE	OpenCV @OPENCV_VERSION@" ENDL \
#N": BUILD_HOST 	@CMAKE_HOST_SYSTEM_NAME@@TBB_HOST_VERSION@ @CMAKE_HOST_SYSTEM_PROCESSOR@" ENDL \
#N": BUILD_TARGET	@CMAKE_SYSTEM_NAME@ @CMAKE_SYSTEM_VERSION@ @CMAKE_SYSTEM_PROCESSOR@" ENDL \
#N": BUILD_COMPILER	@CMAKE_CXX_COMPILER@ (ver @CMAKE_CXX_COMPILER_VERSION@)" ENDL \
#N": BUILD_COMMAND	use cv::getBuildInformation() for details" ENDL

```

## General Information

This file is part of the OpenCV repository infrastructure.

