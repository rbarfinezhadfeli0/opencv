# Documentation for `docs/platforms/winpack_dldt/2020.4/20201005-dldt-fix-cldnn-compilation.patch_docs.md`

## File Metadata

- **Full Path**: `docs/platforms/winpack_dldt/2020.4/20201005-dldt-fix-cldnn-compilation.patch_docs.md`
- **File Name**: `20201005-dldt-fix-cldnn-compilation.patch_docs.md`
- **File Size**: 1,229 bytes
- **File Type**: .md
- **Link to Source**: [docs/platforms/winpack_dldt/2020.4/20201005-dldt-fix-cldnn-compilation.patch_docs.md](../../../../docs/platforms/winpack_dldt/2020.4/20201005-dldt-fix-cldnn-compilation.patch_docs.md)

## Purpose and Role

This file is located in the `docs/platforms/winpack_dldt/2020.4` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `platforms/winpack_dldt/2020.4/20201005-dldt-fix-cldnn-compilation.patch`

## File Metadata

- **Full Path**: `platforms/winpack_dldt/2020.4/20201005-dldt-fix-cldnn-compilation.patch`
- **File Name**: `20201005-dldt-fix-cldnn-compilation.patch`
- **File Size**: 486 bytes
- **File Type**: .patch
- **Link to Source**: [platforms/winpack_dldt/2020.4/20201005-dldt-fix-cldnn-compilation.patch](../../../platforms/winpack_dldt/2020.4/20201005-dldt-fix-cldnn-compilation.patch)

## Purpose and Role

This file is located in the `platforms/winpack_dldt/2020.4` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
diff --git a/inference-engine/thirdparty/clDNN/kernel_selector/common/tensor_type.h b/inference-engine/thirdparty/clDNN/kernel_selector/common/tensor_type.h
index 3dbdfd0b..6b04b910 100644
--- a/inference-engine/thirdparty/clDNN/kernel_selector/common/tensor_type.h
+++ b/inference-engine/thirdparty/clDNN/kernel_selector/common/tensor_type.h
@@ -15,6 +15,7 @@
 
 #pragma once
 
+#include <stdexcept>
 #include "common_types.h"
 #include "common_tools.h"
 #include <vector>

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

