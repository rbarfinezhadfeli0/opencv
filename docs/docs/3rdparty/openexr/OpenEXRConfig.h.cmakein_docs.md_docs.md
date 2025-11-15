# Documentation for `docs/3rdparty/openexr/OpenEXRConfig.h.cmakein_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openexr/OpenEXRConfig.h.cmakein_docs.md`
- **File Name**: `OpenEXRConfig.h.cmakein_docs.md`
- **File Size**: 2,580 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openexr/OpenEXRConfig.h.cmakein_docs.md](../../../docs/3rdparty/openexr/OpenEXRConfig.h.cmakein_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openexr` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openexr/OpenEXRConfig.h.cmakein`

## File Metadata

- **Full Path**: `3rdparty/openexr/OpenEXRConfig.h.cmakein`
- **File Name**: `OpenEXRConfig.h.cmakein`
- **File Size**: 1,979 bytes
- **File Type**: .cmakein
- **Link to Source**: [3rdparty/openexr/OpenEXRConfig.h.cmakein](../../3rdparty/openexr/OpenEXRConfig.h.cmakein)

## Purpose and Role

This file is located in the `3rdparty/openexr` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
//
// Define and set to 1 if the target system supports a proc filesystem
// compatible with the Linux kernel's proc filesystem.  Note that this
// is only used by a program in the IlmImfTest test suite, it's not
// used by any OpenEXR library or application code.
//

#cmakedefine OPENEXR_IMF_HAVE_LINUX_PROCFS

//
// Define and set to 1 if the target system is a Darwin-based system
// (e.g., OS X).
//

#cmakedefine OPENEXR_IMF_HAVE_DARWIN

//
// Define and set to 1 if the target system has a complete <iomanip>
// implementation, specifically if it supports the std::right
// formatter.
//

#cmakedefine OPENEXR_IMF_HAVE_COMPLETE_IOMANIP

//
// Define and set to 1 if the target system has support for large
// stack sizes.
//

#cmakedefine OPENEXR_IMF_HAVE_LARGE_STACK

//
// Define if we can support GCC style inline asm with AVX instructions
//

#cmakedefine OPENEXR_IMF_HAVE_GCC_INLINE_ASM_AVX

//
// Define if we can use sysconf(_SC_NPROCESSORS_ONLN) to get CPU count
//

#cmakedefine OPENEXR_IMF_HAVE_SYSCONF_NPROCESSORS_ONLN

//
// Current internal library namepace name
//
#define OPENEXR_IMF_INTERNAL_NAMESPACE_CUSTOM @OPENEXR_IMF_INTERNAL_NAMESPACE_CUSTOM@
#define OPENEXR_IMF_INTERNAL_NAMESPACE @OPENEXR_IMF_INTERNAL_NAMESPACE@

//
// Current public user namepace name
//

#define OPENEXR_IMF_NAMESPACE_CUSTOM @OPENEXR_IMF_NAMESPACE_CUSTOM@
#define OPENEXR_IMF_NAMESPACE @OPENEXR_IMF_NAMESPACE@

//
// Version string for runtime access
//

#define OPENEXR_VERSION_STRING @OPENEXR_VERSION_STRING@
#define OPENEXR_PACKAGE_STRING @OPENEXR_PACKAGE_STRING@

#define OPENEXR_VERSION_MAJOR @OPENEXR_VERSION_MAJOR@
#define OPENEXR_VERSION_MINOR @OPENEXR_VERSION_MINOR@
#define OPENEXR_VERSION_PATCH @OPENEXR_VERSION_PATCH@

// Version as a single hex number, e.g. 0x01000300 == 1.0.3
#define OPENEXR_VERSION_HEX ((OPENEXR_VERSION_MAJOR << 24) | \
                             (OPENEXR_VERSION_MINOR << 16) | \
                             (OPENEXR_VERSION_PATCH <<  8))


```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

