# Documentation for `docs/platforms/winpack_dldt/2020.3.0/20200413-dldt-fix-binaries-location.patch_docs.md`

## File Metadata

- **Full Path**: `docs/platforms/winpack_dldt/2020.3.0/20200413-dldt-fix-binaries-location.patch_docs.md`
- **File Name**: `20200413-dldt-fix-binaries-location.patch_docs.md`
- **File Size**: 1,214 bytes
- **File Type**: .md
- **Link to Source**: [docs/platforms/winpack_dldt/2020.3.0/20200413-dldt-fix-binaries-location.patch_docs.md](../../../../docs/platforms/winpack_dldt/2020.3.0/20200413-dldt-fix-binaries-location.patch_docs.md)

## Purpose and Role

This file is located in the `docs/platforms/winpack_dldt/2020.3.0` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `platforms/winpack_dldt/2020.3.0/20200413-dldt-fix-binaries-location.patch`

## File Metadata

- **Full Path**: `platforms/winpack_dldt/2020.3.0/20200413-dldt-fix-binaries-location.patch`
- **File Name**: `20200413-dldt-fix-binaries-location.patch`
- **File Size**: 462 bytes
- **File Type**: .patch
- **Link to Source**: [platforms/winpack_dldt/2020.3.0/20200413-dldt-fix-binaries-location.patch](../../../platforms/winpack_dldt/2020.3.0/20200413-dldt-fix-binaries-location.patch)

## Purpose and Role

This file is located in the `platforms/winpack_dldt/2020.3.0` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
diff --git a/cmake/developer_package.cmake b/cmake/developer_package.cmake
index bed73503..5124795a 100644
--- a/cmake/developer_package.cmake
+++ b/cmake/developer_package.cmake
@@ -137,7 +137,7 @@ if("${CMAKE_BUILD_TYPE}" STREQUAL "")
     set(CMAKE_BUILD_TYPE "Release")
 endif()
 
-set(OUTPUT_ROOT ${OpenVINO_MAIN_SOURCE_DIR})
+set(OUTPUT_ROOT "${CMAKE_BINARY_DIR}")
 
 # Enable postfixes for Debug/Release builds
 set(IE_DEBUG_POSTFIX_WIN "d")

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

