# Documentation for `docs/3rdparty/openexr/IlmBaseConfig.h.cmakein_docs.md`

## File Metadata

- **Full Path**: `docs/3rdparty/openexr/IlmBaseConfig.h.cmakein_docs.md`
- **File Name**: `IlmBaseConfig.h.cmakein_docs.md`
- **File Size**: 2,712 bytes
- **File Type**: .md
- **Link to Source**: [docs/3rdparty/openexr/IlmBaseConfig.h.cmakein_docs.md](../../../docs/3rdparty/openexr/IlmBaseConfig.h.cmakein_docs.md)

## Purpose and Role

This file is located in the `docs/3rdparty/openexr` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `3rdparty/openexr/IlmBaseConfig.h.cmakein`

## File Metadata

- **Full Path**: `3rdparty/openexr/IlmBaseConfig.h.cmakein`
- **File Name**: `IlmBaseConfig.h.cmakein`
- **File Size**: 2,111 bytes
- **File Type**: .cmakein
- **Link to Source**: [3rdparty/openexr/IlmBaseConfig.h.cmakein](../../3rdparty/openexr/IlmBaseConfig.h.cmakein)

## Purpose and Role

This file is located in the `3rdparty/openexr` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#cmakedefine PLATFORM_WINDOWS

//
// Define and set to 1 if the target system has c++11/14 support
// and you want IlmBase to NOT use it's features
//

#cmakedefine01 ILMBASE_FORCE_CXX03
#if ILMBASE_FORCE_CXX03 == 0
#undef ILMBASE_FORCE_CXX03
#endif

//
// Define and set to 1 if the target system has POSIX thread support
// and you want IlmBase to use it for multithreaded file I/O.
//

#cmakedefine01 HAVE_PTHREAD

//
// Define and set to 1 if the target system supports POSIX semaphores
// and you want OpenEXR to use them; otherwise, OpenEXR will use its
// own semaphore implementation.
//

#cmakedefine01 HAVE_POSIX_SEMAPHORES


#cmakedefine HAVE_UCONTEXT_H


//
// Dealing with FPEs
//
#cmakedefine01 ILMBASE_HAVE_CONTROL_REGISTER_SUPPORT


//
// Define and set to 1 if the target system has support for large
// stack sizes.
//

#cmakedefine ILMBASE_HAVE_LARGE_STACK

//
// Current (internal) library namepace name and corresponding public
// client namespaces.
//
#define ILMBASE_INTERNAL_NAMESPACE_CUSTOM @ILMBASE_INTERNAL_NAMESPACE_CUSTOM@
#define IMATH_INTERNAL_NAMESPACE @IMATH_INTERNAL_NAMESPACE@
#define IEX_INTERNAL_NAMESPACE @IEX_INTERNAL_NAMESPACE@
#define ILMTHREAD_INTERNAL_NAMESPACE @ILMTHREAD_INTERNAL_NAMESPACE@

#define ILMBASE_NAMESPACE_CUSTOM @ILMBASE_NAMESPACE_CUSTOM@
#define IMATH_NAMESPACE @IMATH_NAMESPACE@
#define IEX_NAMESPACE @IEX_NAMESPACE@
#define ILMTHREAD_NAMESPACE @ILMTHREAD_NAMESPACE@


//
// Define and set to 1 if the target system has support for large
// stack sizes.
//

#cmakedefine ILMBASE_HAVE_LARGE_STACK


//
// Version information
//
#define ILMBASE_VERSION_STRING @ILMBASE_VERSION_STRING@
#define ILMBASE_PACKAGE_STRING @ILMBASE_PACKAGE_STRING@

#define ILMBASE_VERSION_MAJOR @ILMBASE_VERSION_MAJOR@
#define ILMBASE_VERSION_MINOR @ILMBASE_VERSION_MINOR@
#define ILMBASE_VERSION_PATCH @ILMBASE_VERSION_PATCH@

// Version as a single hex number, e.g. 0x01000300 == 1.0.3
#define ILMBASE_VERSION_HEX ((ILMBASE_VERSION_MAJOR << 24) | \
                             (ILMBASE_VERSION_MINOR << 16) | \
                             (ILMBASE_VERSION_PATCH <<  8))



```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

