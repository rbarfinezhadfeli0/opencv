# Documentation for `docs/cmake/templates/vs_version.rc.in_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/templates/vs_version.rc.in_docs.md`
- **File Name**: `vs_version.rc.in_docs.md`
- **File Size**: 1,730 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/templates/vs_version.rc.in_docs.md](../../../docs/cmake/templates/vs_version.rc.in_docs.md)

## Purpose and Role

This file is located in the `docs/cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/templates/vs_version.rc.in`

## File Metadata

- **Full Path**: `cmake/templates/vs_version.rc.in`
- **File Name**: `vs_version.rc.in`
- **File Size**: 1,174 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/vs_version.rc.in](../../cmake/templates/vs_version.rc.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#include <winver.h>

VS_VERSION_INFO         VERSIONINFO
  FILEVERSION           @OPENCV_VS_VER_FILEVERSION_QUAD@
  PRODUCTVERSION        @OPENCV_VS_VER_PRODUCTVERSION_QUAD@
  FILEFLAGSMASK         VS_FFI_FILEFLAGSMASK
#ifdef _DEBUG
  FILEFLAGS             1
#else
  FILEFLAGS             0
#endif
  FILEOS                VOS__WINDOWS32
  FILETYPE              VFT_DLL
  FILESUBTYPE           0
BEGIN
  BLOCK "StringFileInfo"
  BEGIN
    BLOCK "040904E4"
    BEGIN
      VALUE "FileDescription",	"@OPENCV_VS_VER_FILEDESCRIPTION_STR@\0"
      VALUE "FileVersion",	"@OPENCV_VS_VER_FILEVERSION_STR@\0"
      VALUE "InternalName",	"@OPENCV_VS_VER_INTERNALNAME_STR@\0"
#if @OPENCV_VS_VER_HAVE_COPYRIGHT_STR@
      VALUE "LegalCopyright",	"@OPENCV_VS_VER_COPYRIGHT_STR@\0"
#endif
      VALUE "OriginalFilename",	"@OPENCV_VS_VER_ORIGINALFILENAME_STR@\0"
      VALUE "ProductName",	"@OPENCV_VS_VER_PRODUCTNAME_STR@\0"
      VALUE "ProductVersion",	"@OPENCV_VS_VER_PRODUCTVERSION_STR@\0"
#if @OPENCV_VS_VER_HAVE_COMMENTS_STR@
      VALUE "Comments",		"@OPENCV_VS_VER_COMMENTS_STR@\0"
#endif
    END
  END
  BLOCK "VarFileInfo"
  BEGIN
    VALUE "Translation", 0x0409, 1252
  END
END

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

