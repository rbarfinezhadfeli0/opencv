# Documentation for `3rdparty/zlib/patches/20190330-ununitialized-use-state-check.diff`

## File Metadata

- **Full Path**: `3rdparty/zlib/patches/20190330-ununitialized-use-state-check.diff`
- **File Name**: `20190330-ununitialized-use-state-check.diff`
- **File Size**: 502 bytes
- **File Type**: .diff
- **Link to Source**: [3rdparty/zlib/patches/20190330-ununitialized-use-state-check.diff](../../../3rdparty/zlib/patches/20190330-ununitialized-use-state-check.diff)

## Purpose and Role

This file is located in the `3rdparty/zlib/patches` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
diff --git a/3rdparty/zlib/inflate.c b/3rdparty/zlib/inflate.c
index ac333e8c2e..19a2cf2ed8 100644
--- a/3rdparty/zlib/inflate.c
+++ b/3rdparty/zlib/inflate.c
@@ -228,6 +228,7 @@ int stream_size;
     state->strm = strm;
     state->window = Z_NULL;
     state->mode = HEAD;     /* to pass state test in inflateReset2() */
+    state->check = 1L;      /* 1L is the result of adler32() zero length data */
     ret = inflateReset2(strm, windowBits);
     if (ret != Z_OK) {
         ZFREE(strm, state);

```

## General Information

This file is part of the OpenCV repository infrastructure.

