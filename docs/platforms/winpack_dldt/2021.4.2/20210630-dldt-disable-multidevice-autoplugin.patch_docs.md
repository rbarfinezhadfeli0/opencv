# Documentation for `platforms/winpack_dldt/2021.4.2/20210630-dldt-disable-multidevice-autoplugin.patch`

## File Metadata

- **Full Path**: `platforms/winpack_dldt/2021.4.2/20210630-dldt-disable-multidevice-autoplugin.patch`
- **File Name**: `20210630-dldt-disable-multidevice-autoplugin.patch`
- **File Size**: 451 bytes
- **File Type**: .patch
- **Link to Source**: [platforms/winpack_dldt/2021.4.2/20210630-dldt-disable-multidevice-autoplugin.patch](../../../platforms/winpack_dldt/2021.4.2/20210630-dldt-disable-multidevice-autoplugin.patch)

## Purpose and Role

This file is located in the `platforms/winpack_dldt/2021.4.2` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
diff --git a/inference-engine/src/CMakeLists.txt b/inference-engine/src/CMakeLists.txt
index 0ba0dd78..7d34e7cb 100644
--- a/inference-engine/src/CMakeLists.txt
+++ b/inference-engine/src/CMakeLists.txt
@@ -26,9 +26,9 @@ endif()
 
 add_subdirectory(hetero_plugin)
 
-add_subdirectory(auto_plugin)
+#add_subdirectory(auto_plugin)
 
-add_subdirectory(multi_device)
+#add_subdirectory(multi_device)
 
 add_subdirectory(transformations)
 

```

## General Information

This file is part of the OpenCV repository infrastructure.

