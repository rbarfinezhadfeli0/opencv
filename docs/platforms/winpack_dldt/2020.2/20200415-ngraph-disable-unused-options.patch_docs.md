# Documentation for `platforms/winpack_dldt/2020.2/20200415-ngraph-disable-unused-options.patch`

## File Metadata

- **Full Path**: `platforms/winpack_dldt/2020.2/20200415-ngraph-disable-unused-options.patch`
- **File Name**: `20200415-ngraph-disable-unused-options.patch`
- **File Size**: 590 bytes
- **File Type**: .patch
- **Link to Source**: [platforms/winpack_dldt/2020.2/20200415-ngraph-disable-unused-options.patch](../../../platforms/winpack_dldt/2020.2/20200415-ngraph-disable-unused-options.patch)

## Purpose and Role

This file is located in the `platforms/winpack_dldt/2020.2` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
diff --git a/CMakeLists.txt b/CMakeLists.txt
index edf8233f..addac6cd 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -78,8 +78,7 @@ function(build_ngraph)
     if (NOT ANDROID)
         ngraph_set(NGRAPH_UNIT_TEST_ENABLE TRUE)
         ngraph_set(NGRAPH_UNIT_TEST_OPENVINO_ENABLE TRUE)
-        # ngraph_set(NGRAPH_ONNX_IMPORT_ENABLE TRUE)
-        set(NGRAPH_ONNX_IMPORT_ENABLE TRUE CACHE BOOL "" FORCE)
+        ngraph_set(NGRAPH_ONNX_IMPORT_ENABLE TRUE)
     else()
         ngraph_set(NGRAPH_UNIT_TEST_ENABLE FALSE)
         ngraph_set(NGRAPH_TEST_UTIL_ENABLE FALSE)

```

## General Information

This file is part of the OpenCV repository infrastructure.

