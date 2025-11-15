# Documentation for `docs/cmake/templates/dllmain.cpp.in_docs.md`

## File Metadata

- **Full Path**: `docs/cmake/templates/dllmain.cpp.in_docs.md`
- **File Name**: `dllmain.cpp.in_docs.md`
- **File Size**: 1,697 bytes
- **File Type**: .md
- **Link to Source**: [docs/cmake/templates/dllmain.cpp.in_docs.md](../../../docs/cmake/templates/dllmain.cpp.in_docs.md)

## Purpose and Role

This file is located in the `docs/cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `cmake/templates/dllmain.cpp.in`

## File Metadata

- **Full Path**: `cmake/templates/dllmain.cpp.in`
- **File Name**: `dllmain.cpp.in`
- **File Size**: 1,151 bytes
- **File Type**: .in
- **Link to Source**: [cmake/templates/dllmain.cpp.in](../../cmake/templates/dllmain.cpp.in)

## Purpose and Role

This file is located in the `cmake/templates` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef _WIN32
#error "Build configuration error"
#endif
#ifndef CVAPI_EXPORTS
#error "Build configuration error"
#endif

#pragma warning(disable:4447) // Disable warning 'main' signature found without threading model

#define WIN32_LEAN_AND_MEAN
#include <windows.h>

#define OPENCV_MODULE_S "@the_module@"

namespace cv {
extern __declspec(dllimport) bool __termination;  // Details: #12750
}

#ifdef _WIN32_WCE
#define DLL_MAIN_ARG0 HANDLE
#else
#define DLL_MAIN_ARG0 HINSTANCE
#endif

extern "C"
BOOL WINAPI DllMain(DLL_MAIN_ARG0, DWORD fdwReason, LPVOID lpReserved);

extern "C"
BOOL WINAPI DllMain(DLL_MAIN_ARG0, DWORD fdwReason, LPVOID lpReserved)
{
    if (fdwReason == DLL_THREAD_DETACH || fdwReason == DLL_PROCESS_DETACH)
    {
        if (lpReserved != NULL) // called after ExitProcess() call
        {
            //printf("OpenCV: terminating: " OPENCV_MODULE_S "\n");
            cv::__termination = true;
        }
    }
    return TRUE;
}

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

