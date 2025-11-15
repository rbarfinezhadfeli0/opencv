# Documentation for `platforms/winpack_dldt/2021.4.2/20220118-dldt-fix-msvs-compilation-21469.patch`

## File Metadata

- **Full Path**: `platforms/winpack_dldt/2021.4.2/20220118-dldt-fix-msvs-compilation-21469.patch`
- **File Name**: `20220118-dldt-fix-msvs-compilation-21469.patch`
- **File Size**: 410 bytes
- **File Type**: .patch
- **Link to Source**: [platforms/winpack_dldt/2021.4.2/20220118-dldt-fix-msvs-compilation-21469.patch](../../../platforms/winpack_dldt/2021.4.2/20220118-dldt-fix-msvs-compilation-21469.patch)

## Purpose and Role

This file is located in the `platforms/winpack_dldt/2021.4.2` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
diff --git a/inference-engine/src/plugin_api/caseless.hpp b/inference-engine/src/plugin_api/caseless.hpp
index d8ce739..0dd8886 100644
--- a/inference-engine/src/plugin_api/caseless.hpp
+++ b/inference-engine/src/plugin_api/caseless.hpp
@@ -12,6 +12,7 @@
 #include <algorithm>
 #include <cctype>
 #include <functional>
+#include <iterator>
 #include <map>
 #include <set>
 #include <unordered_map>

```

## General Information

This file is part of the OpenCV repository infrastructure.

