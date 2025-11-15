# Documentation for `docs/3rdparty/zlib-ng/patches/zlib-ng-2.2.1-detect-intrinsics.patch_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/zlib-ng/patches/zlib-ng-2.2.1-detect-intrinsics.patch_docs.md`
- **File Name**: `zlib-ng-2.2.1-detect-intrinsics.patch_docs.md`
- **File Size**: 1,218 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/zlib-ng/patches/zlib-ng-2.2.1-detect-intrinsics.patch_docs.md](../../../../docs/3rdparty/zlib-ng/patches/zlib-ng-2.2.1-detect-intrinsics.patch_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/zlib-ng/patches` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/zlib-ng/patches/zlib-ng-2.2.1-detect-intrinsics.patch`

## File Metadata

- **Full Path**: `3rdparty/zlib-ng/patches/zlib-ng-2.2.1-detect-intrinsics.patch`
- **File Name**: `zlib-ng-2.2.1-detect-intrinsics.patch`
- **File Size**: 508 bytes
- **File Type**: .patch
- **Link to Source**: [3rdparty/zlib-ng/patches/zlib-ng-2.2.1-detect-intrinsics.patch](../../../3rdparty/zlib-ng/patches/zlib-ng-2.2.1-detect-intrinsics.patch)

## Purpose and Role

This file is located in the `3rdparty/zlib-ng/patches` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
diff --git a/3rdparty/zlib-ng/cmake/detect-intrinsics.cmake b/3rdparty/zlib-ng/cmake/detect-intrinsics.cmake
index 14f82fc..78e46e1 100644
--- a/3rdparty/zlib-ng/cmake/detect-intrinsics.cmake
+++ b/3rdparty/zlib-ng/cmake/detect-intrinsics.cmake
@@ -66,7 +66,7 @@ macro(check_armv6_compiler_flag)
             return __uqsub16(a, b);
         #endif
         }
-        int main(void) { return 0; }"
+        int main(void) { return f(1,2); }"
         HAVE_ARMV6_INTRIN
     )
     set(CMAKE_REQUIRED_FLAGS)

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

