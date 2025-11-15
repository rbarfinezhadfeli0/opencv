# Documentation for `platforms/winpack_dldt/2020.1/20200313-dldt-fix-binaries-location.patch`

## File Metadata

- **Full Path**: `platforms/winpack_dldt/2020.1/20200313-dldt-fix-binaries-location.patch`
- **File Name**: `20200313-dldt-fix-binaries-location.patch`
- **File Size**: 424 bytes
- **File Type**: .patch
- **Link to Source**: [platforms/winpack_dldt/2020.1/20200313-dldt-fix-binaries-location.patch](../../../platforms/winpack_dldt/2020.1/20200313-dldt-fix-binaries-location.patch)

## Purpose and Role

This file is located in the `platforms/winpack_dldt/2020.1` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
diff --git a/cmake/developer_package.cmake b/cmake/developer_package.cmake
index e59edb2..e42ac19 100644
--- a/cmake/developer_package.cmake
+++ b/cmake/developer_package.cmake
@@ -99,7 +99,7 @@ if(UNIX)
     SET(LIB_DL ${CMAKE_DL_LIBS})
 endif()
 
-set(OUTPUT_ROOT ${OpenVINO_MAIN_SOURCE_DIR})
+set(OUTPUT_ROOT ${CMAKE_BINARY_DIR})
 
 # Enable postfixes for Debug/Release builds
 set(IE_DEBUG_POSTFIX_WIN "d")

```

## General Information

This file is part of the OpenCV repository infrastructure.

