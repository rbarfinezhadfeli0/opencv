# Documentation for `platforms/winpack_dldt/2020.1/20200313-ngraph-disable-tests-examples.patch`

## File Metadata

- **Full Path**: `platforms/winpack_dldt/2020.1/20200313-ngraph-disable-tests-examples.patch`
- **File Name**: `20200313-ngraph-disable-tests-examples.patch`
- **File Size**: 595 bytes
- **File Type**: .patch
- **Link to Source**: [platforms/winpack_dldt/2020.1/20200313-ngraph-disable-tests-examples.patch](../../../platforms/winpack_dldt/2020.1/20200313-ngraph-disable-tests-examples.patch)

## Purpose and Role

This file is located in the `platforms/winpack_dldt/2020.1` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
diff --git a/CMakeLists.txt b/CMakeLists.txt
index 631465f..723153b 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -567,7 +567,7 @@ if (NGRAPH_ONNX_IMPORT_ENABLE)
     endif()
 endif()
 
-include(cmake/external_gtest.cmake)
+#include(cmake/external_gtest.cmake)
 if(NGRAPH_JSON_ENABLE)
     include(cmake/external_json.cmake)
 endif()
@@ -623,8 +623,8 @@ endif()
 
 add_subdirectory(src)
 
-add_subdirectory(test)
-add_subdirectory(doc/examples)
+#add_subdirectory(test)
+#add_subdirectory(doc/examples)
 
 if (NGRAPH_DOC_BUILD_ENABLE)
     add_subdirectory(doc)
 

```

## General Information

This file is part of the OpenCV repository infrastructure.

