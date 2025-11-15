# Documentation for `docs/platforms/winpack_dldt/2021.2/20201218-dldt-vs-version.patch_docs.md`

## File Metadata

- **Full Path**: `docs/platforms/winpack_dldt/2021.2/20201218-dldt-vs-version.patch_docs.md`
- **File Name**: `20201218-dldt-vs-version.patch_docs.md`
- **File Size**: 1,438 bytes
- **File Type**: .md
- **Link to Source**: [docs/platforms/winpack_dldt/2021.2/20201218-dldt-vs-version.patch_docs.md](../../../../docs/platforms/winpack_dldt/2021.2/20201218-dldt-vs-version.patch_docs.md)

## Purpose and Role

This file is located in the `docs/platforms/winpack_dldt/2021.2` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `platforms/winpack_dldt/2021.2/20201218-dldt-vs-version.patch`

## File Metadata

- **Full Path**: `platforms/winpack_dldt/2021.2/20201218-dldt-vs-version.patch`
- **File Name**: `20201218-dldt-vs-version.patch`
- **File Size**: 754 bytes
- **File Type**: .patch
- **Link to Source**: [platforms/winpack_dldt/2021.2/20201218-dldt-vs-version.patch](../../../platforms/winpack_dldt/2021.2/20201218-dldt-vs-version.patch)

## Purpose and Role

This file is located in the `platforms/winpack_dldt/2021.2` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
diff --git a/cmake/vs_version/vs_version.cmake b/cmake/vs_version/vs_version.cmake
index d857e2e4..453903fd 100644
--- a/cmake/vs_version/vs_version.cmake
+++ b/cmake/vs_version/vs_version.cmake
@@ -22,9 +22,9 @@ if(IE_VS_VER_HAS_VERSION)
 endif()
 
 set(IE_VS_VER_PRODUCTVERSION_STR "${CI_BUILD_NUMBER}")
-set(IE_VS_VER_PRODUCTNAME_STR "OpenVINO toolkit")
+set(IE_VS_VER_PRODUCTNAME_STR "OpenVINO toolkit (for OpenCV Windows package)")
 set(IE_VS_VER_COPYRIGHT_STR "Copyright (C) 2018-2020, Intel Corporation")
-set(IE_VS_VER_COMMENTS_STR "https://docs.openvinotoolkit.org/")
+set(IE_VS_VER_COMMENTS_STR "https://github.com/opencv/opencv/wiki/Intel%27s-Deep-Learning-Inference-Engine-backend")
 
 #
 # ie_add_vs_version_file(NAME <name>

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

