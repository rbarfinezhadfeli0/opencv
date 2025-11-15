# Documentation for `docs/3rdparty/openjpeg/openjp2/libopenjp2.pc.cmake.in_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openjpeg/openjp2/libopenjp2.pc.cmake.in_docs.md`
- **File Name**: `libopenjp2.pc.cmake.in_docs.md`
- **File Size**: 972 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openjpeg/openjp2/libopenjp2.pc.cmake.in_docs.md](../../../../docs/3rdparty/openjpeg/openjp2/libopenjp2.pc.cmake.in_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openjpeg/openjp2` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openjpeg/openjp2/libopenjp2.pc.cmake.in`

## File Metadata

- **Full Path**: `3rdparty/openjpeg/openjp2/libopenjp2.pc.cmake.in`
- **File Name**: `libopenjp2.pc.cmake.in`
- **File Size**: 335 bytes
- **File Type**: .in
- **Link to Source**: [3rdparty/openjpeg/openjp2/libopenjp2.pc.cmake.in](../../../3rdparty/openjpeg/openjp2/libopenjp2.pc.cmake.in)

## Purpose and Role

This file is located in the `3rdparty/openjpeg/openjp2` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
prefix=@CMAKE_INSTALL_PREFIX@
bindir=@bindir@
mandir=@mandir@
docdir=@docdir@
libdir=@libdir@
includedir=@includedir@

Name: openjp2
Description: JPEG2000 library (Part 1 and 2)
URL: http://www.openjpeg.org/
Version: @OPENJPEG_VERSION@
Libs: -L${libdir} -lopenjp2
Libs.private: -lm
Cflags: -I${includedir}
Cflags.private: -DOPJ_STATIC

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

