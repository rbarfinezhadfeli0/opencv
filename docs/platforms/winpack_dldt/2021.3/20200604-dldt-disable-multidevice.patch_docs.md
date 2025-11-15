# Documentation for `platforms/winpack_dldt/2021.3/20200604-dldt-disable-multidevice.patch`

## File Metadata

- **Full Path**: `platforms/winpack_dldt/2021.3/20200604-dldt-disable-multidevice.patch`
- **File Name**: `20200604-dldt-disable-multidevice.patch`
- **File Size**: 383 bytes
- **File Type**: .patch
- **Link to Source**: [platforms/winpack_dldt/2021.3/20200604-dldt-disable-multidevice.patch](../../../platforms/winpack_dldt/2021.3/20200604-dldt-disable-multidevice.patch)

## Purpose and Role

This file is located in the `platforms/winpack_dldt/2021.3` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
diff --git a/inference-engine/src/CMakeLists.txt b/inference-engine/src/CMakeLists.txt
index 0ba0dd78..7d34e7cb 100644
--- a/inference-engine/src/CMakeLists.txt
+++ b/inference-engine/src/CMakeLists.txt
@@ -26,7 +26,7 @@ endif()
 
 add_subdirectory(hetero_plugin)
 
-add_subdirectory(multi_device)
+#add_subdirectory(multi_device)
 
 add_subdirectory(transformations)
 

```

## General Information

This file is part of the OpenCV repository infrastructure.

