# Documentation for `modules/core/src/gl_core_3_1.cpp`

## File Metadata

- **Full Path**: `modules/core/src/gl_core_3_1.cpp`
- **File Name**: `gl_core_3_1.cpp`
- **File Size**: 127,080 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/src/gl_core_3_1.cpp](../../../modules/core/src/gl_core_3_1.cpp)

## Purpose and Role

This file is located in the `modules/core/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of the copyright holders may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#include "precomp.hpp"
#include "gl_core_3_1.hpp"

#ifdef HAVE_OPENGL

    #ifdef __APPLE__
        #include <dlfcn.h>

        static void* AppleGLGetProcAddress (const char* name)
        {
            static bool initialized = false;
            static void * handle = NULL;
            if (!handle)
            {
                if (!initialized)
                {
                    initialized = true;
                    const char * const path = "/System/Library/Frameworks/OpenGL.framework/Versions/Current/OpenGL";

                    handle = dlopen(path, RTLD_LAZY | RTLD_GLOBAL);
                }
                if (!handle)
                    return NULL;
            }
            return dlsym(handle, name);
        }
    #endif // __APPLE__

    #if defined(__sgi) || defined (__sun)
        #include <dlfcn.h>
        #include <stdio.h>

        static void* SunGetProcAddress (const char* name)
        {
            typedef void* (func_t*)(const GLubyte*);

            static void* h = 0;
            static func_t gpa = 0;

            if (!h)
            {
                h = dlopen(NULL, RTLD_LAZY | RTLD_LOCAL);
                if (!h)
                    return 0;
                gpa = (func_t) dlsym(h, "glXGetProcAddress");
            }

            return gpa ? gpa((const GLubyte*) name) : dlsym(h, name);
        }
    #endif // __sgi || __sun

    #if defined(_WIN32)
        #ifdef _MSC_VER
            #pragma warning(disable: 4055)
            #pragma warning(disable: 4054)
        #endif

        static int TestPointer(const PROC pTest)
        {
            if(!pTest)
                return 0;

            ptrdiff_t iTest = (ptrdiff_t) pTest;

            if (iTest == 1 || iTest == 2 || iTest == 3 || iTest == -1)
                return 0;

            return 1;
        }

        static PROC WinGetProcAddress(const char* name)
        {
            PROC pFunc = wglGetProcAddress((LPCSTR) name);
            if (TestPointer(pFunc))
                return pFunc;

            HMODULE glMod = GetModuleHandleA("OpenGL32.dll");
            return (PROC) GetProcAddress(glMod, (LPCSTR) name);
        }
    #endif // _WIN32

    #if defined(_WIN32)
        #define CV_GL_GET_PROC_ADDRESS(name) WinGetProcAddress(name)
    #elif defined(__APPLE__)
        #define CV_GL_GET_PROC_ADDRESS(name) AppleGLGetProcAddress(name)
    #elif defined(__sgi) || defined(__sun)
        #define CV_GL_GET_PROC_ADDRESS(name) SunGetProcAddress(name)
    #else // GLX
        #include <GL/glx.h>

        #define CV_GL_GET_PROC_ADDRESS(name) glXGetProcAddressARB((const GLubyte*) name)
    #endif

    static void* IntGetProcAddress(const char* name)
    {
        void* func =  (void*) CV_GL_GET_PROC_ADDRESS(name);
        if (!func)
        {
            CV_Error(cv::Error::OpenGlApiCallError, cv::format("Can't load OpenGL extension [%s]", name) );
        }
        return func;
    }
#else
#if defined(_MSC_VER)
    #pragma warning(disable : 4702)  // unreachable code
#endif
    static void* IntGetProcAddress(const char*)
    {
        CV_Error(cv::Error::OpenGlNotSupported, "The library is compiled without OpenGL support");
    }
#endif

namespace gl
{
    //////////////////////////////////////////////
    // Function pointer types

    // Extension: 1.1
    typedef void (CODEGEN_FUNCPTR *PFNCULLFACEPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNFRONTFACEPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNHINTPROC)(GLenum , GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNLINEWIDTHPROC)(GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNPOINTSIZEPROC)(GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNPOLYGONMODEPROC)(GLenum , GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNSCISSORPROC)(GLint , GLint , GLsizei , GLsizei );
    typedef void (CODEGEN_FUNCPTR *PFNTEXPARAMETERFPROC)(GLenum , GLenum , GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNTEXPARAMETERFVPROC)(GLenum , GLenum , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNTEXPARAMETERIPROC)(GLenum , GLenum , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNTEXPARAMETERIVPROC)(GLenum , GLenum , const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNTEXIMAGE1DPROC)(GLenum , GLint , GLint , GLsizei , GLint , GLenum , GLenum , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNTEXIMAGE2DPROC)(GLenum , GLint , GLint , GLsizei , GLsizei , GLint , GLenum , GLenum , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNDRAWBUFFERPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNCLEARPROC)(GLbitfield );
    typedef void (CODEGEN_FUNCPTR *PFNCLEARCOLORPROC)(GLfloat , GLfloat , GLfloat , GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNCLEARSTENCILPROC)(GLint );
    typedef void (CODEGEN_FUNCPTR *PFNCLEARDEPTHPROC)(GLdouble );
    typedef void (CODEGEN_FUNCPTR *PFNSTENCILMASKPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNCOLORMASKPROC)(GLboolean , GLboolean , GLboolean , GLboolean );
    typedef void (CODEGEN_FUNCPTR *PFNDEPTHMASKPROC)(GLboolean );
    typedef void (CODEGEN_FUNCPTR *PFNDISABLEPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNENABLEPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNFINISHPROC)();
    typedef void (CODEGEN_FUNCPTR *PFNFLUSHPROC)();
    typedef void (CODEGEN_FUNCPTR *PFNBLENDFUNCPROC)(GLenum , GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNLOGICOPPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNSTENCILFUNCPROC)(GLenum , GLint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNSTENCILOPPROC)(GLenum , GLenum , GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNDEPTHFUNCPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNPIXELSTOREFPROC)(GLenum , GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNPIXELSTOREIPROC)(GLenum , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNREADBUFFERPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNREADPIXELSPROC)(GLint , GLint , GLsizei , GLsizei , GLenum , GLenum , GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNGETBOOLEANVPROC)(GLenum , GLboolean *);
    typedef void (CODEGEN_FUNCPTR *PFNGETDOUBLEVPROC)(GLenum , GLdouble *);
    typedef GLenum (CODEGEN_FUNCPTR *PFNGETERRORPROC)();
    typedef void (CODEGEN_FUNCPTR *PFNGETFLOATVPROC)(GLenum , GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNGETINTEGERVPROC)(GLenum , GLint *);
    typedef const GLubyte * (CODEGEN_FUNCPTR *PFNGETSTRINGPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNGETTEXIMAGEPROC)(GLenum , GLint , GLenum , GLenum , GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNGETTEXPARAMETERFVPROC)(GLenum , GLenum , GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNGETTEXPARAMETERIVPROC)(GLenum , GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETTEXLEVELPARAMETERFVPROC)(GLenum , GLint , GLenum , GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNGETTEXLEVELPARAMETERIVPROC)(GLenum , GLint , GLenum , GLint *);
    typedef GLboolean (CODEGEN_FUNCPTR *PFNISENABLEDPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNDEPTHRANGEPROC)(GLdouble , GLdouble );
    typedef void (CODEGEN_FUNCPTR *PFNVIEWPORTPROC)(GLint , GLint , GLsizei , GLsizei );
    typedef void (CODEGEN_FUNCPTR *PFNDRAWARRAYSPROC)(GLenum , GLint , GLsizei );
    typedef void (CODEGEN_FUNCPTR *PFNDRAWELEMENTSPROC)(GLenum , GLsizei , GLenum , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNGETPOINTERVPROC)(GLenum , GLvoid* *);
    typedef void (CODEGEN_FUNCPTR *PFNPOLYGONOFFSETPROC)(GLfloat , GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNCOPYTEXIMAGE1DPROC)(GLenum , GLint , GLenum , GLint , GLint , GLsizei , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNCOPYTEXIMAGE2DPROC)(GLenum , GLint , GLenum , GLint , GLint , GLsizei , GLsizei , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNCOPYTEXSUBIMAGE1DPROC)(GLenum , GLint , GLint , GLint , GLint , GLsizei );
    typedef void (CODEGEN_FUNCPTR *PFNCOPYTEXSUBIMAGE2DPROC)(GLenum , GLint , GLint , GLint , GLint , GLint , GLsizei , GLsizei );
    typedef void (CODEGEN_FUNCPTR *PFNTEXSUBIMAGE1DPROC)(GLenum , GLint , GLint , GLsizei , GLenum , GLenum , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNTEXSUBIMAGE2DPROC)(GLenum , GLint , GLint , GLint , GLsizei , GLsizei , GLenum , GLenum , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNBINDTEXTUREPROC)(GLenum , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNDELETETEXTURESPROC)(GLsizei , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNGENTEXTURESPROC)(GLsizei , GLuint *);
    typedef GLboolean (CODEGEN_FUNCPTR *PFNISTEXTUREPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNINDEXUBPROC)(GLubyte );
    typedef void (CODEGEN_FUNCPTR *PFNINDEXUBVPROC)(const GLubyte *);

    // Extension: 1.2
    typedef void (CODEGEN_FUNCPTR *PFNBLENDCOLORPROC)(GLfloat , GLfloat , GLfloat , GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNBLENDEQUATIONPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNDRAWRANGEELEMENTSPROC)(GLenum , GLuint , GLuint , GLsizei , GLenum , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNTEXSUBIMAGE3DPROC)(GLenum , GLint , GLint , GLint , GLint , GLsizei , GLsizei , GLsizei , GLenum , GLenum , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNCOPYTEXSUBIMAGE3DPROC)(GLenum , GLint , GLint , GLint , GLint , GLint , GLint , GLsizei , GLsizei );

    // Extension: 1.3
    typedef void (CODEGEN_FUNCPTR *PFNACTIVETEXTUREPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNSAMPLECOVERAGEPROC)(GLfloat , GLboolean );
    typedef void (CODEGEN_FUNCPTR *PFNCOMPRESSEDTEXIMAGE3DPROC)(GLenum , GLint , GLenum , GLsizei , GLsizei , GLsizei , GLint , GLsizei , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNCOMPRESSEDTEXIMAGE2DPROC)(GLenum , GLint , GLenum , GLsizei , GLsizei , GLint , GLsizei , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNCOMPRESSEDTEXIMAGE1DPROC)(GLenum , GLint , GLenum , GLsizei , GLint , GLsizei , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNCOMPRESSEDTEXSUBIMAGE3DPROC)(GLenum , GLint , GLint , GLint , GLint , GLsizei , GLsizei , GLsizei , GLenum , GLsizei , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNCOMPRESSEDTEXSUBIMAGE2DPROC)(GLenum , GLint , GLint , GLint , GLsizei , GLsizei , GLenum , GLsizei , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNCOMPRESSEDTEXSUBIMAGE1DPROC)(GLenum , GLint , GLint , GLsizei , GLenum , GLsizei , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNGETCOMPRESSEDTEXIMAGEPROC)(GLenum , GLint , GLvoid *);

    // Extension: 1.4
    typedef void (CODEGEN_FUNCPTR *PFNBLENDFUNCSEPARATEPROC)(GLenum , GLenum , GLenum , GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNMULTIDRAWARRAYSPROC)(GLenum , const GLint *, const GLsizei *, GLsizei );
    typedef void (CODEGEN_FUNCPTR *PFNMULTIDRAWELEMENTSPROC)(GLenum , const GLsizei *, GLenum , const GLvoid* const *, GLsizei );
    typedef void (CODEGEN_FUNCPTR *PFNPOINTPARAMETERFPROC)(GLenum , GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNPOINTPARAMETERFVPROC)(GLenum , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNPOINTPARAMETERIPROC)(GLenum , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNPOINTPARAMETERIVPROC)(GLenum , const GLint *);

    // Extension: 1.5
    typedef void (CODEGEN_FUNCPTR *PFNGENQUERIESPROC)(GLsizei , GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNDELETEQUERIESPROC)(GLsizei , const GLuint *);
    typedef GLboolean (CODEGEN_FUNCPTR *PFNISQUERYPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNBEGINQUERYPROC)(GLenum , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNENDQUERYPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNGETQUERYIVPROC)(GLenum , GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETQUERYOBJECTIVPROC)(GLuint , GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETQUERYOBJECTUIVPROC)(GLuint , GLenum , GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNBINDBUFFERPROC)(GLenum , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNDELETEBUFFERSPROC)(GLsizei , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNGENBUFFERSPROC)(GLsizei , GLuint *);
    typedef GLboolean (CODEGEN_FUNCPTR *PFNISBUFFERPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNBUFFERDATAPROC)(GLenum , GLsizeiptr , const GLvoid *, GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNBUFFERSUBDATAPROC)(GLenum , GLintptr , GLsizeiptr , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNGETBUFFERSUBDATAPROC)(GLenum , GLintptr , GLsizeiptr , GLvoid *);
    typedef GLvoid* (CODEGEN_FUNCPTR *PFNMAPBUFFERPROC)(GLenum , GLenum );
    typedef GLboolean (CODEGEN_FUNCPTR *PFNUNMAPBUFFERPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNGETBUFFERPARAMETERIVPROC)(GLenum , GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETBUFFERPOINTERVPROC)(GLenum , GLenum , GLvoid* *);

    // Extension: 2.0
    typedef void (CODEGEN_FUNCPTR *PFNBLENDEQUATIONSEPARATEPROC)(GLenum , GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNDRAWBUFFERSPROC)(GLsizei , const GLenum *);
    typedef void (CODEGEN_FUNCPTR *PFNSTENCILOPSEPARATEPROC)(GLenum , GLenum , GLenum , GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNSTENCILFUNCSEPARATEPROC)(GLenum , GLenum , GLint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNSTENCILMASKSEPARATEPROC)(GLenum , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNATTACHSHADERPROC)(GLuint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNBINDATTRIBLOCATIONPROC)(GLuint , GLuint , const GLchar *);
    typedef void (CODEGEN_FUNCPTR *PFNCOMPILESHADERPROC)(GLuint );
    typedef GLuint (CODEGEN_FUNCPTR *PFNCREATEPROGRAMPROC)();
    typedef GLuint (CODEGEN_FUNCPTR *PFNCREATESHADERPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNDELETEPROGRAMPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNDELETESHADERPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNDETACHSHADERPROC)(GLuint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNDISABLEVERTEXATTRIBARRAYPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNENABLEVERTEXATTRIBARRAYPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNGETACTIVEATTRIBPROC)(GLuint , GLuint , GLsizei , GLsizei *, GLint *, GLenum *, GLchar *);
    typedef void (CODEGEN_FUNCPTR *PFNGETACTIVEUNIFORMPROC)(GLuint , GLuint , GLsizei , GLsizei *, GLint *, GLenum *, GLchar *);
    typedef void (CODEGEN_FUNCPTR *PFNGETATTACHEDSHADERSPROC)(GLuint , GLsizei , GLsizei *, GLuint *);
    typedef GLint (CODEGEN_FUNCPTR *PFNGETATTRIBLOCATIONPROC)(GLuint , const GLchar *);
    typedef void (CODEGEN_FUNCPTR *PFNGETPROGRAMIVPROC)(GLuint , GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETPROGRAMINFOLOGPROC)(GLuint , GLsizei , GLsizei *, GLchar *);
    typedef void (CODEGEN_FUNCPTR *PFNGETSHADERIVPROC)(GLuint , GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETSHADERINFOLOGPROC)(GLuint , GLsizei , GLsizei *, GLchar *);
    typedef void (CODEGEN_FUNCPTR *PFNGETSHADERSOURCEPROC)(GLuint , GLsizei , GLsizei *, GLchar *);
    typedef GLint (CODEGEN_FUNCPTR *PFNGETUNIFORMLOCATIONPROC)(GLuint , const GLchar *);
    typedef void (CODEGEN_FUNCPTR *PFNGETUNIFORMFVPROC)(GLuint , GLint , GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNGETUNIFORMIVPROC)(GLuint , GLint , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETVERTEXATTRIBDVPROC)(GLuint , GLenum , GLdouble *);
    typedef void (CODEGEN_FUNCPTR *PFNGETVERTEXATTRIBFVPROC)(GLuint , GLenum , GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNGETVERTEXATTRIBIVPROC)(GLuint , GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETVERTEXATTRIBPOINTERVPROC)(GLuint , GLenum , GLvoid* *);
    typedef GLboolean (CODEGEN_FUNCPTR *PFNISPROGRAMPROC)(GLuint );
    typedef GLboolean (CODEGEN_FUNCPTR *PFNISSHADERPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNLINKPROGRAMPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNSHADERSOURCEPROC)(GLuint , GLsizei , const GLchar* const *, const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNUSEPROGRAMPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM1FPROC)(GLint , GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM2FPROC)(GLint , GLfloat , GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM3FPROC)(GLint , GLfloat , GLfloat , GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM4FPROC)(GLint , GLfloat , GLfloat , GLfloat , GLfloat );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM1IPROC)(GLint , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM2IPROC)(GLint , GLint , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM3IPROC)(GLint , GLint , GLint , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM4IPROC)(GLint , GLint , GLint , GLint , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM1FVPROC)(GLint , GLsizei , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM2FVPROC)(GLint , GLsizei , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM3FVPROC)(GLint , GLsizei , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM4FVPROC)(GLint , GLsizei , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM1IVPROC)(GLint , GLsizei , const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM2IVPROC)(GLint , GLsizei , const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM3IVPROC)(GLint , GLsizei , const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM4IVPROC)(GLint , GLsizei , const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORMMATRIX2FVPROC)(GLint , GLsizei , GLboolean , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORMMATRIX3FVPROC)(GLint , GLsizei , GLboolean , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORMMATRIX4FVPROC)(GLint , GLsizei , GLboolean , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNVALIDATEPROGRAMPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBPOINTERPROC)(GLuint , GLint , GLenum , GLboolean , GLsizei , const GLvoid *);

    // Extension: 2.1
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORMMATRIX2X3FVPROC)(GLint , GLsizei , GLboolean , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORMMATRIX3X2FVPROC)(GLint , GLsizei , GLboolean , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORMMATRIX2X4FVPROC)(GLint , GLsizei , GLboolean , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORMMATRIX4X2FVPROC)(GLint , GLsizei , GLboolean , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORMMATRIX3X4FVPROC)(GLint , GLsizei , GLboolean , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORMMATRIX4X3FVPROC)(GLint , GLsizei , GLboolean , const GLfloat *);

    // Extension: ARB_vertex_array_object
    typedef void (CODEGEN_FUNCPTR *PFNBINDVERTEXARRAYPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNDELETEVERTEXARRAYSPROC)(GLsizei , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNGENVERTEXARRAYSPROC)(GLsizei , GLuint *);
    typedef GLboolean (CODEGEN_FUNCPTR *PFNISVERTEXARRAYPROC)(GLuint );

    // Extension: ARB_map_buffer_range
    typedef GLvoid* (CODEGEN_FUNCPTR *PFNMAPBUFFERRANGEPROC)(GLenum , GLintptr , GLsizeiptr , GLbitfield );
    typedef void (CODEGEN_FUNCPTR *PFNFLUSHMAPPEDBUFFERRANGEPROC)(GLenum , GLintptr , GLsizeiptr );

    // Extension: ARB_framebuffer_object
    typedef GLboolean (CODEGEN_FUNCPTR *PFNISRENDERBUFFERPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNBINDRENDERBUFFERPROC)(GLenum , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNDELETERENDERBUFFERSPROC)(GLsizei , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNGENRENDERBUFFERSPROC)(GLsizei , GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNRENDERBUFFERSTORAGEPROC)(GLenum , GLenum , GLsizei , GLsizei );
    typedef void (CODEGEN_FUNCPTR *PFNGETRENDERBUFFERPARAMETERIVPROC)(GLenum , GLenum , GLint *);
    typedef GLboolean (CODEGEN_FUNCPTR *PFNISFRAMEBUFFERPROC)(GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNBINDFRAMEBUFFERPROC)(GLenum , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNDELETEFRAMEBUFFERSPROC)(GLsizei , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNGENFRAMEBUFFERSPROC)(GLsizei , GLuint *);
    typedef GLenum (CODEGEN_FUNCPTR *PFNCHECKFRAMEBUFFERSTATUSPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNFRAMEBUFFERTEXTURE1DPROC)(GLenum , GLenum , GLenum , GLuint , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNFRAMEBUFFERTEXTURE2DPROC)(GLenum , GLenum , GLenum , GLuint , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNFRAMEBUFFERTEXTURE3DPROC)(GLenum , GLenum , GLenum , GLuint , GLint , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNFRAMEBUFFERRENDERBUFFERPROC)(GLenum , GLenum , GLenum , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNGETFRAMEBUFFERATTACHMENTPARAMETERIVPROC)(GLenum , GLenum , GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGENERATEMIPMAPPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNBLITFRAMEBUFFERPROC)(GLint , GLint , GLint , GLint , GLint , GLint , GLint , GLint , GLbitfield , GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNRENDERBUFFERSTORAGEMULTISAMPLEPROC)(GLenum , GLsizei , GLenum , GLsizei , GLsizei );
    typedef void (CODEGEN_FUNCPTR *PFNFRAMEBUFFERTEXTURELAYERPROC)(GLenum , GLenum , GLuint , GLint , GLint );

    // Extension: 3.0
    typedef void (CODEGEN_FUNCPTR *PFNCOLORMASKIPROC)(GLuint , GLboolean , GLboolean , GLboolean , GLboolean );
    typedef void (CODEGEN_FUNCPTR *PFNGETBOOLEANI_VPROC)(GLenum , GLuint , GLboolean *);
    typedef void (CODEGEN_FUNCPTR *PFNGETINTEGERI_VPROC)(GLenum , GLuint , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNENABLEIPROC)(GLenum , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNDISABLEIPROC)(GLenum , GLuint );
    typedef GLboolean (CODEGEN_FUNCPTR *PFNISENABLEDIPROC)(GLenum , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNBEGINTRANSFORMFEEDBACKPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNENDTRANSFORMFEEDBACKPROC)();
    typedef void (CODEGEN_FUNCPTR *PFNBINDBUFFERRANGEPROC)(GLenum , GLuint , GLuint , GLintptr , GLsizeiptr );
    typedef void (CODEGEN_FUNCPTR *PFNBINDBUFFERBASEPROC)(GLenum , GLuint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNTRANSFORMFEEDBACKVARYINGSPROC)(GLuint , GLsizei , const GLchar* const *, GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNGETTRANSFORMFEEDBACKVARYINGPROC)(GLuint , GLuint , GLsizei , GLsizei *, GLsizei *, GLenum *, GLchar *);
    typedef void (CODEGEN_FUNCPTR *PFNCLAMPCOLORPROC)(GLenum , GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNBEGINCONDITIONALRENDERPROC)(GLuint , GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNENDCONDITIONALRENDERPROC)();
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBIPOINTERPROC)(GLuint , GLint , GLenum , GLsizei , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNGETVERTEXATTRIBIIVPROC)(GLuint , GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETVERTEXATTRIBIUIVPROC)(GLuint , GLenum , GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI1IPROC)(GLuint , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI2IPROC)(GLuint , GLint , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI3IPROC)(GLuint , GLint , GLint , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI4IPROC)(GLuint , GLint , GLint , GLint , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI1UIPROC)(GLuint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI2UIPROC)(GLuint , GLuint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI3UIPROC)(GLuint , GLuint , GLuint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI4UIPROC)(GLuint , GLuint , GLuint , GLuint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI1IVPROC)(GLuint , const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI2IVPROC)(GLuint , const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI3IVPROC)(GLuint , const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI4IVPROC)(GLuint , const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI1UIVPROC)(GLuint , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI2UIVPROC)(GLuint , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI3UIVPROC)(GLuint , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI4UIVPROC)(GLuint , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI4BVPROC)(GLuint , const GLbyte *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI4SVPROC)(GLuint , const GLshort *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI4UBVPROC)(GLuint , const GLubyte *);
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXATTRIBI4USVPROC)(GLuint , const GLushort *);
    typedef void (CODEGEN_FUNCPTR *PFNGETUNIFORMUIVPROC)(GLuint , GLint , GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNBINDFRAGDATALOCATIONPROC)(GLuint , GLuint , const GLchar *);
    typedef GLint (CODEGEN_FUNCPTR *PFNGETFRAGDATALOCATIONPROC)(GLuint , const GLchar *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM1UIPROC)(GLint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM2UIPROC)(GLint , GLuint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM3UIPROC)(GLint , GLuint , GLuint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM4UIPROC)(GLint , GLuint , GLuint , GLuint , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM1UIVPROC)(GLint , GLsizei , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM2UIVPROC)(GLint , GLsizei , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM3UIVPROC)(GLint , GLsizei , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORM4UIVPROC)(GLint , GLsizei , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNTEXPARAMETERIIVPROC)(GLenum , GLenum , const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNTEXPARAMETERIUIVPROC)(GLenum , GLenum , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETTEXPARAMETERIIVPROC)(GLenum , GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETTEXPARAMETERIUIVPROC)(GLenum , GLenum , GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNCLEARBUFFERIVPROC)(GLenum , GLint , const GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNCLEARBUFFERUIVPROC)(GLenum , GLint , const GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNCLEARBUFFERFVPROC)(GLenum , GLint , const GLfloat *);
    typedef void (CODEGEN_FUNCPTR *PFNCLEARBUFFERFIPROC)(GLenum , GLint , GLfloat , GLint );
    typedef const GLubyte * (CODEGEN_FUNCPTR *PFNGETSTRINGIPROC)(GLenum , GLuint );

    // Extension: ARB_uniform_buffer_object
    typedef void (CODEGEN_FUNCPTR *PFNGETUNIFORMINDICESPROC)(GLuint , GLsizei , const GLchar* const *, GLuint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETACTIVEUNIFORMSIVPROC)(GLuint , GLsizei , const GLuint *, GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETACTIVEUNIFORMNAMEPROC)(GLuint , GLuint , GLsizei , GLsizei *, GLchar *);
    typedef GLuint (CODEGEN_FUNCPTR *PFNGETUNIFORMBLOCKINDEXPROC)(GLuint , const GLchar *);
    typedef void (CODEGEN_FUNCPTR *PFNGETACTIVEUNIFORMBLOCKIVPROC)(GLuint , GLuint , GLenum , GLint *);
    typedef void (CODEGEN_FUNCPTR *PFNGETACTIVEUNIFORMBLOCKNAMEPROC)(GLuint , GLuint , GLsizei , GLsizei *, GLchar *);
    typedef void (CODEGEN_FUNCPTR *PFNUNIFORMBLOCKBINDINGPROC)(GLuint , GLuint , GLuint );

    // Extension: ARB_copy_buffer
    typedef void (CODEGEN_FUNCPTR *PFNCOPYBUFFERSUBDATAPROC)(GLenum , GLenum , GLintptr , GLintptr , GLsizeiptr );

    // Extension: 3.1
    typedef void (CODEGEN_FUNCPTR *PFNDRAWARRAYSINSTANCEDPROC)(GLenum , GLint , GLsizei , GLsizei );
    typedef void (CODEGEN_FUNCPTR *PFNDRAWELEMENTSINSTANCEDPROC)(GLenum , GLsizei , GLenum , const GLvoid *, GLsizei );
    typedef void (CODEGEN_FUNCPTR *PFNTEXBUFFERPROC)(GLenum , GLenum , GLuint );
    typedef void (CODEGEN_FUNCPTR *PFNPRIMITIVERESTARTINDEXPROC)(GLuint );

    // Legacy
    typedef void (CODEGEN_FUNCPTR *PFNENABLECLIENTSTATEPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNDISABLECLIENTSTATEPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNVERTEXPOINTERPROC)(GLint , GLenum , GLsizei , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNNORMALPOINTERPROC)(GLenum , GLsizei , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNCOLORPOINTERPROC)(GLint , GLenum , GLsizei , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNTEXCOORDPOINTERPROC)(GLint , GLenum , GLsizei , const GLvoid *);
    typedef void (CODEGEN_FUNCPTR *PFNTEXENVIPROC)(GLenum , GLenum , GLint );
    typedef void (CODEGEN_FUNCPTR *PFNMATRIXMODEPROC)(GLenum );
    typedef void (CODEGEN_FUNCPTR *PFNLOADIDENTITYPROC)(void);
    typedef void (CODEGEN_FUNCPTR *PFNORTHOPROC)(GLdouble , GLdouble , GLdouble , GLdouble , GLdouble , GLdouble );
    typedef void (CODEGEN_FUNCPTR *PFNCOLOR3DPROC)(GLdouble , GLdouble , GLdouble );

    //////////////////////////////////////////////
    // Function pointers

    // Extension: 1.1
    PFNCULLFACEPROC CullFace;
    PFNFRONTFACEPROC FrontFace;
    PFNHINTPROC Hint;
    PFNLINEWIDTHPROC LineWidth;
    PFNPOINTSIZEPROC PointSize;
    PFNPOLYGONMODEPROC PolygonMode;
    PFNSCISSORPROC Scissor;
    PFNTEXPARAMETERFPROC TexParameterf;
    PFNTEXPARAMETERFVPROC TexParameterfv;
    PFNTEXPARAMETERIPROC TexParameteri;
    PFNTEXPARAMETERIVPROC TexParameteriv;
    PFNTEXIMAGE1DPROC TexImage1D;
    PFNTEXIMAGE2DPROC TexImage2D;
    PFNDRAWBUFFERPROC DrawBuffer;
    PFNCLEARPROC Clear;
    PFNCLEARCOLORPROC ClearColor;
    PFNCLEARSTENCILPROC ClearStencil;
    PFNCLEARDEPTHPROC ClearDepth;
    PFNSTENCILMASKPROC StencilMask;
    PFNCOLORMASKPROC ColorMask;
    PFNDEPTHMASKPROC DepthMask;
    PFNDISABLEPROC Disable;
    PFNENABLEPROC Enable;
    PFNFINISHPROC Finish;
    PFNFLUSHPROC Flush;
    PFNBLENDFUNCPROC BlendFunc;
    PFNLOGICOPPROC LogicOp;
    PFNSTENCILFUNCPROC StencilFunc;
    PFNSTENCILOPPROC StencilOp;
    PFNDEPTHFUNCPROC DepthFunc;
    PFNPIXELSTOREFPROC PixelStoref;
    PFNPIXELSTOREIPROC PixelStorei;
    PFNREADBUFFERPROC ReadBuffer;
    PFNREADPIXELSPROC ReadPixels;
    PFNGETBOOLEANVPROC GetBooleanv;
    PFNGETDOUBLEVPROC GetDoublev;
    PFNGETERRORPROC GetError;
    PFNGETFLOATVPROC GetFloatv;
    PFNGETINTEGERVPROC GetIntegerv;
    PFNGETSTRINGPROC GetString;
    PFNGETTEXIMAGEPROC GetTexImage;
    PFNGETTEXPARAMETERFVPROC GetTexParameterfv;
    PFNGETTEXPARAMETERIVPROC GetTexParameteriv;
    PFNGETTEXLEVELPARAMETERFVPROC GetTexLevelParameterfv;
    PFNGETTEXLEVELPARAMETERIVPROC GetTexLevelParameteriv;
    PFNISENABLEDPROC IsEnabled;
    PFNDEPTHRANGEPROC DepthRange;
    PFNVIEWPORTPROC Viewport;
    PFNDRAWARRAYSPROC DrawArrays;
    PFNDRAWELEMENTSPROC DrawElements;
    PFNGETPOINTERVPROC GetPointerv;
    PFNPOLYGONOFFSETPROC PolygonOffset;
    PFNCOPYTEXIMAGE1DPROC CopyTexImage1D;
    PFNCOPYTEXIMAGE2DPROC CopyTexImage2D;
    PFNCOPYTEXSUBIMAGE1DPROC CopyTexSubImage1D;
    PFNCOPYTEXSUBIMAGE2DPROC CopyTexSubImage2D;
    PFNTEXSUBIMAGE1DPROC TexSubImage1D;
    PFNTEXSUBIMAGE2DPROC TexSubImage2D;
    PFNBINDTEXTUREPROC BindTexture;
    PFNDELETETEXTURESPROC DeleteTextures;
    PFNGENTEXTURESPROC GenTextures;
    PFNISTEXTUREPROC IsTexture;
    PFNINDEXUBPROC Indexub;
    PFNINDEXUBVPROC Indexubv;

    // Extension: 1.2
    PFNBLENDCOLORPROC BlendColor;
    PFNBLENDEQUATIONPROC BlendEquation;
    PFNDRAWRANGEELEMENTSPROC DrawRangeElements;
    PFNTEXSUBIMAGE3DPROC TexSubImage3D;
    PFNCOPYTEXSUBIMAGE3DPROC CopyTexSubImage3D;

    // Extension: 1.3
    PFNACTIVETEXTUREPROC ActiveTexture;
    PFNSAMPLECOVERAGEPROC SampleCoverage;
    PFNCOMPRESSEDTEXIMAGE3DPROC CompressedTexImage3D;
    PFNCOMPRESSEDTEXIMAGE2DPROC CompressedTexImage2D;
    PFNCOMPRESSEDTEXIMAGE1DPROC CompressedTexImage1D;
    PFNCOMPRESSEDTEXSUBIMAGE3DPROC CompressedTexSubImage3D;
    PFNCOMPRESSEDTEXSUBIMAGE2DPROC CompressedTexSubImage2D;
    PFNCOMPRESSEDTEXSUBIMAGE1DPROC CompressedTexSubImage1D;
    PFNGETCOMPRESSEDTEXIMAGEPROC GetCompressedTexImage;

    // Extension: 1.4
    PFNBLENDFUNCSEPARATEPROC BlendFuncSeparate;
    PFNMULTIDRAWARRAYSPROC MultiDrawArrays;
    PFNMULTIDRAWELEMENTSPROC MultiDrawElements;
    PFNPOINTPARAMETERFPROC PointParameterf;
    PFNPOINTPARAMETERFVPROC PointParameterfv;
    PFNPOINTPARAMETERIPROC PointParameteri;
    PFNPOINTPARAMETERIVPROC PointParameteriv;

    // Extension: 1.5
    PFNGENQUERIESPROC GenQueries;
    PFNDELETEQUERIESPROC DeleteQueries;
    PFNISQUERYPROC IsQuery;
    PFNBEGINQUERYPROC BeginQuery;
    PFNENDQUERYPROC EndQuery;
    PFNGETQUERYIVPROC GetQueryiv;
    PFNGETQUERYOBJECTIVPROC GetQueryObjectiv;
    PFNGETQUERYOBJECTUIVPROC GetQueryObjectuiv;
    PFNBINDBUFFERPROC BindBuffer;
    PFNDELETEBUFFERSPROC DeleteBuffers;
    PFNGENBUFFERSPROC GenBuffers;
    PFNISBUFFERPROC IsBuffer;
    PFNBUFFERDATAPROC BufferData;
    PFNBUFFERSUBDATAPROC BufferSubData;
    PFNGETBUFFERSUBDATAPROC GetBufferSubData;
    PFNMAPBUFFERPROC MapBuffer;
    PFNUNMAPBUFFERPROC UnmapBuffer;
    PFNGETBUFFERPARAMETERIVPROC GetBufferParameteriv;
    PFNGETBUFFERPOINTERVPROC GetBufferPointerv;

    // Extension: 2.0
    PFNBLENDEQUATIONSEPARATEPROC BlendEquationSeparate;
    PFNDRAWBUFFERSPROC DrawBuffers;
    PFNSTENCILOPSEPARATEPROC StencilOpSeparate;
    PFNSTENCILFUNCSEPARATEPROC StencilFuncSeparate;
    PFNSTENCILMASKSEPARATEPROC StencilMaskSeparate;
    PFNATTACHSHADERPROC AttachShader;
    PFNBINDATTRIBLOCATIONPROC BindAttribLocation;
    PFNCOMPILESHADERPROC CompileShader;
    PFNCREATEPROGRAMPROC CreateProgram;
    PFNCREATESHADERPROC CreateShader;
    PFNDELETEPROGRAMPROC DeleteProgram;
    PFNDELETESHADERPROC DeleteShader;
    PFNDETACHSHADERPROC DetachShader;
    PFNDISABLEVERTEXATTRIBARRAYPROC DisableVertexAttribArray;
    PFNENABLEVERTEXATTRIBARRAYPROC EnableVertexAttribArray;
    PFNGETACTIVEATTRIBPROC GetActiveAttrib;
    PFNGETACTIVEUNIFORMPROC GetActiveUniform;
    PFNGETATTACHEDSHADERSPROC GetAttachedShaders;
    PFNGETATTRIBLOCATIONPROC GetAttribLocation;
    PFNGETPROGRAMIVPROC GetProgramiv;
    PFNGETPROGRAMINFOLOGPROC GetProgramInfoLog;
    PFNGETSHADERIVPROC GetShaderiv;
    PFNGETSHADERINFOLOGPROC GetShaderInfoLog;
    PFNGETSHADERSOURCEPROC GetShaderSource;
    PFNGETUNIFORMLOCATIONPROC GetUniformLocation;
    PFNGETUNIFORMFVPROC GetUniformfv;
    PFNGETUNIFORMIVPROC GetUniformiv;
    PFNGETVERTEXATTRIBDVPROC GetVertexAttribdv;
    PFNGETVERTEXATTRIBFVPROC GetVertexAttribfv;
    PFNGETVERTEXATTRIBIVPROC GetVertexAttribiv;
    PFNGETVERTEXATTRIBPOINTERVPROC GetVertexAttribPointerv;
    PFNISPROGRAMPROC IsProgram;
    PFNISSHADERPROC IsShader;
    PFNLINKPROGRAMPROC LinkProgram;
    PFNSHADERSOURCEPROC ShaderSource;
    PFNUSEPROGRAMPROC UseProgram;
    PFNUNIFORM1FPROC Uniform1f;
    PFNUNIFORM2FPROC Uniform2f;
    PFNUNIFORM3FPROC Uniform3f;
    PFNUNIFORM4FPROC Uniform4f;
    PFNUNIFORM1IPROC Uniform1i;
    PFNUNIFORM2IPROC Uniform2i;
    PFNUNIFORM3IPROC Uniform3i;
    PFNUNIFORM4IPROC Uniform4i;
    PFNUNIFORM1FVPROC Uniform1fv;
    PFNUNIFORM2FVPROC Uniform2fv;
    PFNUNIFORM3FVPROC Uniform3fv;
    PFNUNIFORM4FVPROC Uniform4fv;
    PFNUNIFORM1IVPROC Uniform1iv;
    PFNUNIFORM2IVPROC Uniform2iv;
    PFNUNIFORM3IVPROC Uniform3iv;
    PFNUNIFORM4IVPROC Uniform4iv;
    PFNUNIFORMMATRIX2FVPROC UniformMatrix2fv;
    PFNUNIFORMMATRIX3FVPROC UniformMatrix3fv;
    PFNUNIFORMMATRIX4FVPROC UniformMatrix4fv;
    PFNVALIDATEPROGRAMPROC ValidateProgram;
    PFNVERTEXATTRIBPOINTERPROC VertexAttribPointer;

    // Extension: 2.1
    PFNUNIFORMMATRIX2X3FVPROC UniformMatrix2x3fv;
    PFNUNIFORMMATRIX3X2FVPROC UniformMatrix3x2fv;
    PFNUNIFORMMATRIX2X4FVPROC UniformMatrix2x4fv;
    PFNUNIFORMMATRIX4X2FVPROC UniformMatrix4x2fv;
    PFNUNIFORMMATRIX3X4FVPROC UniformMatrix3x4fv;
    PFNUNIFORMMATRIX4X3FVPROC UniformMatrix4x3fv;

    // Extension: ARB_vertex_array_object
    PFNBINDVERTEXARRAYPROC BindVertexArray;
    PFNDELETEVERTEXARRAYSPROC DeleteVertexArrays;
    PFNGENVERTEXARRAYSPROC GenVertexArrays;
    PFNISVERTEXARRAYPROC IsVertexArray;

    // Extension: ARB_map_buffer_range
    PFNMAPBUFFERRANGEPROC MapBufferRange;
    PFNFLUSHMAPPEDBUFFERRANGEPROC FlushMappedBufferRange;

    // Extension: ARB_framebuffer_object
    PFNISRENDERBUFFERPROC IsRenderbuffer;
    PFNBINDRENDERBUFFERPROC BindRenderbuffer;
    PFNDELETERENDERBUFFERSPROC DeleteRenderbuffers;
    PFNGENRENDERBUFFERSPROC GenRenderbuffers;
    PFNRENDERBUFFERSTORAGEPROC RenderbufferStorage;
    PFNGETRENDERBUFFERPARAMETERIVPROC GetRenderbufferParameteriv;
    PFNISFRAMEBUFFERPROC IsFramebuffer;
    PFNBINDFRAMEBUFFERPROC BindFramebuffer;
    PFNDELETEFRAMEBUFFERSPROC DeleteFramebuffers;
    PFNGENFRAMEBUFFERSPROC GenFramebuffers;
    PFNCHECKFRAMEBUFFERSTATUSPROC CheckFramebufferStatus;
    PFNFRAMEBUFFERTEXTURE1DPROC FramebufferTexture1D;
    PFNFRAMEBUFFERTEXTURE2DPROC FramebufferTexture2D;
    PFNFRAMEBUFFERTEXTURE3DPROC FramebufferTexture3D;
    PFNFRAMEBUFFERRENDERBUFFERPROC FramebufferRenderbuffer;
    PFNGETFRAMEBUFFERATTACHMENTPARAMETERIVPROC GetFramebufferAttachmentParameteriv;
    PFNGENERATEMIPMAPPROC GenerateMipmap;
    PFNBLITFRAMEBUFFERPROC BlitFramebuffer;
    PFNRENDERBUFFERSTORAGEMULTISAMPLEPROC RenderbufferStorageMultisample;
    PFNFRAMEBUFFERTEXTURELAYERPROC FramebufferTextureLayer;

    // Extension: 3.0
    PFNCOLORMASKIPROC ColorMaski;
    PFNGETBOOLEANI_VPROC GetBooleani_v;
    PFNGETINTEGERI_VPROC GetIntegeri_v;
    PFNENABLEIPROC Enablei;
    PFNDISABLEIPROC Disablei;
    PFNISENABLEDIPROC IsEnabledi;
    PFNBEGINTRANSFORMFEEDBACKPROC BeginTransformFeedback;
    PFNENDTRANSFORMFEEDBACKPROC EndTransformFeedback;
    PFNBINDBUFFERRANGEPROC BindBufferRange;
    PFNBINDBUFFERBASEPROC BindBufferBase;
    PFNTRANSFORMFEEDBACKVARYINGSPROC TransformFeedbackVaryings;
    PFNGETTRANSFORMFEEDBACKVARYINGPROC GetTransformFeedbackVarying;
    PFNCLAMPCOLORPROC ClampColor;
    PFNBEGINCONDITIONALRENDERPROC BeginConditionalRender;
    PFNENDCONDITIONALRENDERPROC EndConditionalRender;
    PFNVERTEXATTRIBIPOINTERPROC VertexAttribIPointer;
    PFNGETVERTEXATTRIBIIVPROC GetVertexAttribIiv;
    PFNGETVERTEXATTRIBIUIVPROC GetVertexAttribIuiv;
    PFNVERTEXATTRIBI1IPROC VertexAttribI1i;
    PFNVERTEXATTRIBI2IPROC VertexAttribI2i;
    PFNVERTEXATTRIBI3IPROC VertexAttribI3i;
    PFNVERTEXATTRIBI4IPROC VertexAttribI4i;
    PFNVERTEXATTRIBI1UIPROC VertexAttribI1ui;
    PFNVERTEXATTRIBI2UIPROC VertexAttribI2ui;
    PFNVERTEXATTRIBI3UIPROC VertexAttribI3ui;
    PFNVERTEXATTRIBI4UIPROC VertexAttribI4ui;
    PFNVERTEXATTRIBI1IVPROC VertexAttribI1iv;
    PFNVERTEXATTRIBI2IVPROC VertexAttribI2iv;
    PFNVERTEXATTRIBI3IVPROC VertexAttribI3iv;
    PFNVERTEXATTRIBI4IVPROC VertexAttribI4iv;
    PFNVERTEXATTRIBI1UIVPROC VertexAttribI1uiv;
    PFNVERTEXATTRIBI2UIVPROC VertexAttribI2uiv;
    PFNVERTEXATTRIBI3UIVPROC VertexAttribI3uiv;
    PFNVERTEXATTRIBI4UIVPROC VertexAttribI4uiv;
    PFNVERTEXATTRIBI4BVPROC VertexAttribI4bv;
    PFNVERTEXATTRIBI4SVPROC VertexAttribI4sv;
    PFNVERTEXATTRIBI4UBVPROC VertexAttribI4ubv;
    PFNVERTEXATTRIBI4USVPROC VertexAttribI4usv;
    PFNGETUNIFORMUIVPROC GetUniformuiv;
    PFNBINDFRAGDATALOCATIONPROC BindFragDataLocation;
    PFNGETFRAGDATALOCATIONPROC GetFragDataLocation;
    PFNUNIFORM1UIPROC Uniform1ui;
    PFNUNIFORM2UIPROC Uniform2ui;
    PFNUNIFORM3UIPROC Uniform3ui;
    PFNUNIFORM4UIPROC Uniform4ui;
    PFNUNIFORM1UIVPROC Uniform1uiv;
    PFNUNIFORM2UIVPROC Uniform2uiv;
    PFNUNIFORM3UIVPROC Uniform3uiv;
    PFNUNIFORM4UIVPROC Uniform4uiv;
    PFNTEXPARAMETERIIVPROC TexParameterIiv;
    PFNTEXPARAMETERIUIVPROC TexParameterIuiv;
    PFNGETTEXPARAMETERIIVPROC GetTexParameterIiv;
    PFNGETTEXPARAMETERIUIVPROC GetTexParameterIuiv;
    PFNCLEARBUFFERIVPROC ClearBufferiv;
    PFNCLEARBUFFERUIVPROC ClearBufferuiv;
    PFNCLEARBUFFERFVPROC ClearBufferfv;
    PFNCLEARBUFFERFIPROC ClearBufferfi;
    PFNGETSTRINGIPROC GetStringi;

    // Extension: ARB_uniform_buffer_object
    PFNGETUNIFORMINDICESPROC GetUniformIndices;
    PFNGETACTIVEUNIFORMSIVPROC GetActiveUniformsiv;
    PFNGETACTIVEUNIFORMNAMEPROC GetActiveUniformName;
    PFNGETUNIFORMBLOCKINDEXPROC GetUniformBlockIndex;
    PFNGETACTIVEUNIFORMBLOCKIVPROC GetActiveUniformBlockiv;
    PFNGETACTIVEUNIFORMBLOCKNAMEPROC GetActiveUniformBlockName;
    PFNUNIFORMBLOCKBINDINGPROC UniformBlockBinding;

    // Extension: ARB_copy_buffer
    PFNCOPYBUFFERSUBDATAPROC CopyBufferSubData;

    // Extension: 3.1
    PFNDRAWARRAYSINSTANCEDPROC DrawArraysInstanced;
    PFNDRAWELEMENTSINSTANCEDPROC DrawElementsInstanced;
    PFNTEXBUFFERPROC TexBuffer;
    PFNPRIMITIVERESTARTINDEXPROC PrimitiveRestartIndex;

    // Legacy
    PFNENABLECLIENTSTATEPROC EnableClientState;
    PFNDISABLECLIENTSTATEPROC DisableClientState;
    PFNVERTEXPOINTERPROC VertexPointer;
    PFNNORMALPOINTERPROC NormalPointer;
    PFNCOLORPOINTERPROC ColorPointer;
    PFNTEXCOORDPOINTERPROC TexCoordPointer;

    PFNTEXENVIPROC TexEnvi;

    PFNMATRIXMODEPROC MatrixMode;
    PFNLOADIDENTITYPROC LoadIdentity;
    PFNORTHOPROC Ortho;

    PFNCOLOR3DPROC Color3d;

    //////////////////////////////////////////////
    // Switch functions

    // Extension: 1.1

    static void CODEGEN_FUNCPTR Switch_CullFace(GLenum mode)
    {
        CullFace = (PFNCULLFACEPROC)IntGetProcAddress("glCullFace");
        CullFace(mode);
    }

    static void CODEGEN_FUNCPTR Switch_FrontFace(GLenum mode)
    {
        FrontFace = (PFNFRONTFACEPROC)IntGetProcAddress("glFrontFace");
        FrontFace(mode);
    }

    static void CODEGEN_FUNCPTR Switch_Hint(GLenum target, GLenum mode)
    {
        Hint = (PFNHINTPROC)IntGetProcAddress("glHint");
        Hint(target, mode);
    }

    static void CODEGEN_FUNCPTR Switch_LineWidth(GLfloat width)
    {
        LineWidth = (PFNLINEWIDTHPROC)IntGetProcAddress("glLineWidth");
        LineWidth(width);
    }

    static void CODEGEN_FUNCPTR Switch_PointSize(GLfloat size)
    {
        PointSize = (PFNPOINTSIZEPROC)IntGetProcAddress("glPointSize");
        PointSize(size);
    }

    static void CODEGEN_FUNCPTR Switch_PolygonMode(GLenum face, GLenum mode)
    {
        PolygonMode = (PFNPOLYGONMODEPROC)IntGetProcAddress("glPolygonMode");
        PolygonMode(face, mode);
    }

    static void CODEGEN_FUNCPTR Switch_Scissor(GLint x, GLint y, GLsizei width, GLsizei height)
    {
        Scissor = (PFNSCISSORPROC)IntGetProcAddress("glScissor");
        Scissor(x, y, width, height);
    }

    static void CODEGEN_FUNCPTR Switch_TexParameterf(GLenum target, GLenum pname, GLfloat param)
    {
        TexParameterf = (PFNTEXPARAMETERFPROC)IntGetProcAddress("glTexParameterf");
        TexParameterf(target, pname, param);
    }

    static void CODEGEN_FUNCPTR Switch_TexParameterfv(GLenum target, GLenum pname, const GLfloat *params)
    {
        TexParameterfv = (PFNTEXPARAMETERFVPROC)IntGetProcAddress("glTexParameterfv");
        TexParameterfv(target, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_TexParameteri(GLenum target, GLenum pname, GLint param)
    {
        TexParameteri = (PFNTEXPARAMETERIPROC)IntGetProcAddress("glTexParameteri");
        TexParameteri(target, pname, param);
    }

    static void CODEGEN_FUNCPTR Switch_TexParameteriv(GLenum target, GLenum pname, const GLint *params)
    {
        TexParameteriv = (PFNTEXPARAMETERIVPROC)IntGetProcAddress("glTexParameteriv");
        TexParameteriv(target, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_TexImage1D(GLenum target, GLint level, GLint internalformat, GLsizei width, GLint border, GLenum format, GLenum type, const GLvoid *pixels)
    {
        TexImage1D = (PFNTEXIMAGE1DPROC)IntGetProcAddress("glTexImage1D");
        TexImage1D(target, level, internalformat, width, border, format, type, pixels);
    }

    static void CODEGEN_FUNCPTR Switch_TexImage2D(GLenum target, GLint level, GLint internalformat, GLsizei width, GLsizei height, GLint border, GLenum format, GLenum type, const GLvoid *pixels)
    {
        TexImage2D = (PFNTEXIMAGE2DPROC)IntGetProcAddress("glTexImage2D");
        TexImage2D(target, level, internalformat, width, height, border, format, type, pixels);
    }

    static void CODEGEN_FUNCPTR Switch_DrawBuffer(GLenum mode)
    {
        DrawBuffer = (PFNDRAWBUFFERPROC)IntGetProcAddress("glDrawBuffer");
        DrawBuffer(mode);
    }

    static void CODEGEN_FUNCPTR Switch_Clear(GLbitfield mask)
    {
        Clear = (PFNCLEARPROC)IntGetProcAddress("glClear");
        Clear(mask);
    }

    static void CODEGEN_FUNCPTR Switch_ClearColor(GLfloat red, GLfloat green, GLfloat blue, GLfloat alpha)
    {
        ClearColor = (PFNCLEARCOLORPROC)IntGetProcAddress("glClearColor");
        ClearColor(red, green, blue, alpha);
    }

    static void CODEGEN_FUNCPTR Switch_ClearStencil(GLint s)
    {
        ClearStencil = (PFNCLEARSTENCILPROC)IntGetProcAddress("glClearStencil");
        ClearStencil(s);
    }

    static void CODEGEN_FUNCPTR Switch_ClearDepth(GLdouble depth)
    {
        ClearDepth = (PFNCLEARDEPTHPROC)IntGetProcAddress("glClearDepth");
        ClearDepth(depth);
    }

    static void CODEGEN_FUNCPTR Switch_StencilMask(GLuint mask)
    {
        StencilMask = (PFNSTENCILMASKPROC)IntGetProcAddress("glStencilMask");
        StencilMask(mask);
    }

    static void CODEGEN_FUNCPTR Switch_ColorMask(GLboolean red, GLboolean green, GLboolean blue, GLboolean alpha)
    {
        ColorMask = (PFNCOLORMASKPROC)IntGetProcAddress("glColorMask");
        ColorMask(red, green, blue, alpha);
    }

    static void CODEGEN_FUNCPTR Switch_DepthMask(GLboolean flag)
    {
        DepthMask = (PFNDEPTHMASKPROC)IntGetProcAddress("glDepthMask");
        DepthMask(flag);
    }

    static void CODEGEN_FUNCPTR Switch_Disable(GLenum cap)
    {
        Disable = (PFNDISABLEPROC)IntGetProcAddress("glDisable");
        Disable(cap);
    }

    static void CODEGEN_FUNCPTR Switch_Enable(GLenum cap)
    {
        Enable = (PFNENABLEPROC)IntGetProcAddress("glEnable");
        Enable(cap);
    }

    static void CODEGEN_FUNCPTR Switch_Finish()
    {
        Finish = (PFNFINISHPROC)IntGetProcAddress("glFinish");
        Finish();
    }

    static void CODEGEN_FUNCPTR Switch_Flush()
    {
        Flush = (PFNFLUSHPROC)IntGetProcAddress("glFlush");
        Flush();
    }

    static void CODEGEN_FUNCPTR Switch_BlendFunc(GLenum sfactor, GLenum dfactor)
    {
        BlendFunc = (PFNBLENDFUNCPROC)IntGetProcAddress("glBlendFunc");
        BlendFunc(sfactor, dfactor);
    }

    static void CODEGEN_FUNCPTR Switch_LogicOp(GLenum opcode)
    {
        LogicOp = (PFNLOGICOPPROC)IntGetProcAddress("glLogicOp");
        LogicOp(opcode);
    }

    static void CODEGEN_FUNCPTR Switch_StencilFunc(GLenum func, GLint ref, GLuint mask)
    {
        StencilFunc = (PFNSTENCILFUNCPROC)IntGetProcAddress("glStencilFunc");
        StencilFunc(func, ref, mask);
    }

    static void CODEGEN_FUNCPTR Switch_StencilOp(GLenum fail, GLenum zfail, GLenum zpass)
    {
        StencilOp = (PFNSTENCILOPPROC)IntGetProcAddress("glStencilOp");
        StencilOp(fail, zfail, zpass);
    }

    static void CODEGEN_FUNCPTR Switch_DepthFunc(GLenum func)
    {
        DepthFunc = (PFNDEPTHFUNCPROC)IntGetProcAddress("glDepthFunc");
        DepthFunc(func);
    }

    static void CODEGEN_FUNCPTR Switch_PixelStoref(GLenum pname, GLfloat param)
    {
        PixelStoref = (PFNPIXELSTOREFPROC)IntGetProcAddress("glPixelStoref");
        PixelStoref(pname, param);
    }

    static void CODEGEN_FUNCPTR Switch_PixelStorei(GLenum pname, GLint param)
    {
        PixelStorei = (PFNPIXELSTOREIPROC)IntGetProcAddress("glPixelStorei");
        PixelStorei(pname, param);
    }

    static void CODEGEN_FUNCPTR Switch_ReadBuffer(GLenum mode)
    {
        ReadBuffer = (PFNREADBUFFERPROC)IntGetProcAddress("glReadBuffer");
        ReadBuffer(mode);
    }

    static void CODEGEN_FUNCPTR Switch_ReadPixels(GLint x, GLint y, GLsizei width, GLsizei height, GLenum format, GLenum type, GLvoid *pixels)
    {
        ReadPixels = (PFNREADPIXELSPROC)IntGetProcAddress("glReadPixels");
        ReadPixels(x, y, width, height, format, type, pixels);
    }

    static void CODEGEN_FUNCPTR Switch_GetBooleanv(GLenum pname, GLboolean *params)
    {
        GetBooleanv = (PFNGETBOOLEANVPROC)IntGetProcAddress("glGetBooleanv");
        GetBooleanv(pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetDoublev(GLenum pname, GLdouble *params)
    {
        GetDoublev = (PFNGETDOUBLEVPROC)IntGetProcAddress("glGetDoublev");
        GetDoublev(pname, params);
    }

    static GLenum CODEGEN_FUNCPTR Switch_GetError()
    {
        GetError = (PFNGETERRORPROC)IntGetProcAddress("glGetError");
        return GetError();
    }

    static void CODEGEN_FUNCPTR Switch_GetFloatv(GLenum pname, GLfloat *params)
    {
        GetFloatv = (PFNGETFLOATVPROC)IntGetProcAddress("glGetFloatv");
        GetFloatv(pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetIntegerv(GLenum pname, GLint *params)
    {
        GetIntegerv = (PFNGETINTEGERVPROC)IntGetProcAddress("glGetIntegerv");
        GetIntegerv(pname, params);
    }

    static const GLubyte * CODEGEN_FUNCPTR Switch_GetString(GLenum name)
    {
        GetString = (PFNGETSTRINGPROC)IntGetProcAddress("glGetString");
        return GetString(name);
    }

    static void CODEGEN_FUNCPTR Switch_GetTexImage(GLenum target, GLint level, GLenum format, GLenum type, GLvoid *pixels)
    {
        GetTexImage = (PFNGETTEXIMAGEPROC)IntGetProcAddress("glGetTexImage");
        GetTexImage(target, level, format, type, pixels);
    }

    static void CODEGEN_FUNCPTR Switch_GetTexParameterfv(GLenum target, GLenum pname, GLfloat *params)
    {
        GetTexParameterfv = (PFNGETTEXPARAMETERFVPROC)IntGetProcAddress("glGetTexParameterfv");
        GetTexParameterfv(target, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetTexParameteriv(GLenum target, GLenum pname, GLint *params)
    {
        GetTexParameteriv = (PFNGETTEXPARAMETERIVPROC)IntGetProcAddress("glGetTexParameteriv");
        GetTexParameteriv(target, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetTexLevelParameterfv(GLenum target, GLint level, GLenum pname, GLfloat *params)
    {
        GetTexLevelParameterfv = (PFNGETTEXLEVELPARAMETERFVPROC)IntGetProcAddress("glGetTexLevelParameterfv");
        GetTexLevelParameterfv(target, level, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetTexLevelParameteriv(GLenum target, GLint level, GLenum pname, GLint *params)
    {
        GetTexLevelParameteriv = (PFNGETTEXLEVELPARAMETERIVPROC)IntGetProcAddress("glGetTexLevelParameteriv");
        GetTexLevelParameteriv(target, level, pname, params);
    }

    static GLboolean CODEGEN_FUNCPTR Switch_IsEnabled(GLenum cap)
    {
        IsEnabled = (PFNISENABLEDPROC)IntGetProcAddress("glIsEnabled");
        return IsEnabled(cap);
    }

    static void CODEGEN_FUNCPTR Switch_DepthRange(GLdouble ren_near, GLdouble ren_far)
    {
        DepthRange = (PFNDEPTHRANGEPROC)IntGetProcAddress("glDepthRange");
        DepthRange(ren_near, ren_far);
    }

    static void CODEGEN_FUNCPTR Switch_Viewport(GLint x, GLint y, GLsizei width, GLsizei height)
    {
        Viewport = (PFNVIEWPORTPROC)IntGetProcAddress("glViewport");
        Viewport(x, y, width, height);
    }

    static void CODEGEN_FUNCPTR Switch_DrawArrays(GLenum mode, GLint first, GLsizei count)
    {
        DrawArrays = (PFNDRAWARRAYSPROC)IntGetProcAddress("glDrawArrays");
        DrawArrays(mode, first, count);
    }

    static void CODEGEN_FUNCPTR Switch_DrawElements(GLenum mode, GLsizei count, GLenum type, const GLvoid *indices)
    {
        DrawElements = (PFNDRAWELEMENTSPROC)IntGetProcAddress("glDrawElements");
        DrawElements(mode, count, type, indices);
    }

    static void CODEGEN_FUNCPTR Switch_GetPointerv(GLenum pname, GLvoid* *params)
    {
        GetPointerv = (PFNGETPOINTERVPROC)IntGetProcAddress("glGetPointerv");
        GetPointerv(pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_PolygonOffset(GLfloat factor, GLfloat units)
    {
        PolygonOffset = (PFNPOLYGONOFFSETPROC)IntGetProcAddress("glPolygonOffset");
        PolygonOffset(factor, units);
    }

    static void CODEGEN_FUNCPTR Switch_CopyTexImage1D(GLenum target, GLint level, GLenum internalformat, GLint x, GLint y, GLsizei width, GLint border)
    {
        CopyTexImage1D = (PFNCOPYTEXIMAGE1DPROC)IntGetProcAddress("glCopyTexImage1D");
        CopyTexImage1D(target, level, internalformat, x, y, width, border);
    }

    static void CODEGEN_FUNCPTR Switch_CopyTexImage2D(GLenum target, GLint level, GLenum internalformat, GLint x, GLint y, GLsizei width, GLsizei height, GLint border)
    {
        CopyTexImage2D = (PFNCOPYTEXIMAGE2DPROC)IntGetProcAddress("glCopyTexImage2D");
        CopyTexImage2D(target, level, internalformat, x, y, width, height, border);
    }

    static void CODEGEN_FUNCPTR Switch_CopyTexSubImage1D(GLenum target, GLint level, GLint xoffset, GLint x, GLint y, GLsizei width)
    {
        CopyTexSubImage1D = (PFNCOPYTEXSUBIMAGE1DPROC)IntGetProcAddress("glCopyTexSubImage1D");
        CopyTexSubImage1D(target, level, xoffset, x, y, width);
    }

    static void CODEGEN_FUNCPTR Switch_CopyTexSubImage2D(GLenum target, GLint level, GLint xoffset, GLint yoffset, GLint x, GLint y, GLsizei width, GLsizei height)
    {
        CopyTexSubImage2D = (PFNCOPYTEXSUBIMAGE2DPROC)IntGetProcAddress("glCopyTexSubImage2D");
        CopyTexSubImage2D(target, level, xoffset, yoffset, x, y, width, height);
    }

    static void CODEGEN_FUNCPTR Switch_TexSubImage1D(GLenum target, GLint level, GLint xoffset, GLsizei width, GLenum format, GLenum type, const GLvoid *pixels)
    {
        TexSubImage1D = (PFNTEXSUBIMAGE1DPROC)IntGetProcAddress("glTexSubImage1D");
        TexSubImage1D(target, level, xoffset, width, format, type, pixels);
    }

    static void CODEGEN_FUNCPTR Switch_TexSubImage2D(GLenum target, GLint level, GLint xoffset, GLint yoffset, GLsizei width, GLsizei height, GLenum format, GLenum type, const GLvoid *pixels)
    {
        TexSubImage2D = (PFNTEXSUBIMAGE2DPROC)IntGetProcAddress("glTexSubImage2D");
        TexSubImage2D(target, level, xoffset, yoffset, width, height, format, type, pixels);
    }

    static void CODEGEN_FUNCPTR Switch_BindTexture(GLenum target, GLuint texture)
    {
        BindTexture = (PFNBINDTEXTUREPROC)IntGetProcAddress("glBindTexture");
        BindTexture(target, texture);
    }

    static void CODEGEN_FUNCPTR Switch_DeleteTextures(GLsizei n, const GLuint *textures)
    {
        DeleteTextures = (PFNDELETETEXTURESPROC)IntGetProcAddress("glDeleteTextures");
        DeleteTextures(n, textures);
    }

    static void CODEGEN_FUNCPTR Switch_GenTextures(GLsizei n, GLuint *textures)
    {
        GenTextures = (PFNGENTEXTURESPROC)IntGetProcAddress("glGenTextures");
        GenTextures(n, textures);
    }

    static GLboolean CODEGEN_FUNCPTR Switch_IsTexture(GLuint texture)
    {
        IsTexture = (PFNISTEXTUREPROC)IntGetProcAddress("glIsTexture");
        return IsTexture(texture);
    }

    static void CODEGEN_FUNCPTR Switch_Indexub(GLubyte c)
    {
        Indexub = (PFNINDEXUBPROC)IntGetProcAddress("glIndexub");
        Indexub(c);
    }

    static void CODEGEN_FUNCPTR Switch_Indexubv(const GLubyte *c)
    {
        Indexubv = (PFNINDEXUBVPROC)IntGetProcAddress("glIndexubv");
        Indexubv(c);
    }

    // Extension: 1.2

    static void CODEGEN_FUNCPTR Switch_BlendColor(GLfloat red, GLfloat green, GLfloat blue, GLfloat alpha)
    {
        BlendColor = (PFNBLENDCOLORPROC)IntGetProcAddress("glBlendColor");
        BlendColor(red, green, blue, alpha);
    }

    static void CODEGEN_FUNCPTR Switch_BlendEquation(GLenum mode)
    {
        BlendEquation = (PFNBLENDEQUATIONPROC)IntGetProcAddress("glBlendEquation");
        BlendEquation(mode);
    }

    static void CODEGEN_FUNCPTR Switch_DrawRangeElements(GLenum mode, GLuint start, GLuint end, GLsizei count, GLenum type, const GLvoid *indices)
    {
        DrawRangeElements = (PFNDRAWRANGEELEMENTSPROC)IntGetProcAddress("glDrawRangeElements");
        DrawRangeElements(mode, start, end, count, type, indices);
    }

    static void CODEGEN_FUNCPTR Switch_TexSubImage3D(GLenum target, GLint level, GLint xoffset, GLint yoffset, GLint zoffset, GLsizei width, GLsizei height, GLsizei depth, GLenum format, GLenum type, const GLvoid *pixels)
    {
        TexSubImage3D = (PFNTEXSUBIMAGE3DPROC)IntGetProcAddress("glTexSubImage3D");
        TexSubImage3D(target, level, xoffset, yoffset, zoffset, width, height, depth, format, type, pixels);
    }

    static void CODEGEN_FUNCPTR Switch_CopyTexSubImage3D(GLenum target, GLint level, GLint xoffset, GLint yoffset, GLint zoffset, GLint x, GLint y, GLsizei width, GLsizei height)
    {
        CopyTexSubImage3D = (PFNCOPYTEXSUBIMAGE3DPROC)IntGetProcAddress("glCopyTexSubImage3D");
        CopyTexSubImage3D(target, level, xoffset, yoffset, zoffset, x, y, width, height);
    }

    // Extension: 1.3

    static void CODEGEN_FUNCPTR Switch_ActiveTexture(GLenum texture)
    {
        ActiveTexture = (PFNACTIVETEXTUREPROC)IntGetProcAddress("glActiveTexture");
        ActiveTexture(texture);
    }

    static void CODEGEN_FUNCPTR Switch_SampleCoverage(GLfloat value, GLboolean invert)
    {
        SampleCoverage = (PFNSAMPLECOVERAGEPROC)IntGetProcAddress("glSampleCoverage");
        SampleCoverage(value, invert);
    }

    static void CODEGEN_FUNCPTR Switch_CompressedTexImage3D(GLenum target, GLint level, GLenum internalformat, GLsizei width, GLsizei height, GLsizei depth, GLint border, GLsizei imageSize, const GLvoid *data)
    {
        CompressedTexImage3D = (PFNCOMPRESSEDTEXIMAGE3DPROC)IntGetProcAddress("glCompressedTexImage3D");
        CompressedTexImage3D(target, level, internalformat, width, height, depth, border, imageSize, data);
    }

    static void CODEGEN_FUNCPTR Switch_CompressedTexImage2D(GLenum target, GLint level, GLenum internalformat, GLsizei width, GLsizei height, GLint border, GLsizei imageSize, const GLvoid *data)
    {
        CompressedTexImage2D = (PFNCOMPRESSEDTEXIMAGE2DPROC)IntGetProcAddress("glCompressedTexImage2D");
        CompressedTexImage2D(target, level, internalformat, width, height, border, imageSize, data);
    }

    static void CODEGEN_FUNCPTR Switch_CompressedTexImage1D(GLenum target, GLint level, GLenum internalformat, GLsizei width, GLint border, GLsizei imageSize, const GLvoid *data)
    {
        CompressedTexImage1D = (PFNCOMPRESSEDTEXIMAGE1DPROC)IntGetProcAddress("glCompressedTexImage1D");
        CompressedTexImage1D(target, level, internalformat, width, border, imageSize, data);
    }

    static void CODEGEN_FUNCPTR Switch_CompressedTexSubImage3D(GLenum target, GLint level, GLint xoffset, GLint yoffset, GLint zoffset, GLsizei width, GLsizei height, GLsizei depth, GLenum format, GLsizei imageSize, const GLvoid *data)
    {
        CompressedTexSubImage3D = (PFNCOMPRESSEDTEXSUBIMAGE3DPROC)IntGetProcAddress("glCompressedTexSubImage3D");
        CompressedTexSubImage3D(target, level, xoffset, yoffset, zoffset, width, height, depth, format, imageSize, data);
    }

    static void CODEGEN_FUNCPTR Switch_CompressedTexSubImage2D(GLenum target, GLint level, GLint xoffset, GLint yoffset, GLsizei width, GLsizei height, GLenum format, GLsizei imageSize, const GLvoid *data)
    {
        CompressedTexSubImage2D = (PFNCOMPRESSEDTEXSUBIMAGE2DPROC)IntGetProcAddress("glCompressedTexSubImage2D");
        CompressedTexSubImage2D(target, level, xoffset, yoffset, width, height, format, imageSize, data);
    }

    static void CODEGEN_FUNCPTR Switch_CompressedTexSubImage1D(GLenum target, GLint level, GLint xoffset, GLsizei width, GLenum format, GLsizei imageSize, const GLvoid *data)
    {
        CompressedTexSubImage1D = (PFNCOMPRESSEDTEXSUBIMAGE1DPROC)IntGetProcAddress("glCompressedTexSubImage1D");
        CompressedTexSubImage1D(target, level, xoffset, width, format, imageSize, data);
    }

    static void CODEGEN_FUNCPTR Switch_GetCompressedTexImage(GLenum target, GLint level, GLvoid *img)
    {
        GetCompressedTexImage = (PFNGETCOMPRESSEDTEXIMAGEPROC)IntGetProcAddress("glGetCompressedTexImage");
        GetCompressedTexImage(target, level, img);
    }

    // Extension: 1.4

    static void CODEGEN_FUNCPTR Switch_BlendFuncSeparate(GLenum sfactorRGB, GLenum dfactorRGB, GLenum sfactorAlpha, GLenum dfactorAlpha)
    {
        BlendFuncSeparate = (PFNBLENDFUNCSEPARATEPROC)IntGetProcAddress("glBlendFuncSeparate");
        BlendFuncSeparate(sfactorRGB, dfactorRGB, sfactorAlpha, dfactorAlpha);
    }

    static void CODEGEN_FUNCPTR Switch_MultiDrawArrays(GLenum mode, const GLint *first, const GLsizei *count, GLsizei drawcount)
    {
        MultiDrawArrays = (PFNMULTIDRAWARRAYSPROC)IntGetProcAddress("glMultiDrawArrays");
        MultiDrawArrays(mode, first, count, drawcount);
    }

    static void CODEGEN_FUNCPTR Switch_MultiDrawElements(GLenum mode, const GLsizei *count, GLenum type, const GLvoid* const *indices, GLsizei drawcount)
    {
        MultiDrawElements = (PFNMULTIDRAWELEMENTSPROC)IntGetProcAddress("glMultiDrawElements");
        MultiDrawElements(mode, count, type, indices, drawcount);
    }

    static void CODEGEN_FUNCPTR Switch_PointParameterf(GLenum pname, GLfloat param)
    {
        PointParameterf = (PFNPOINTPARAMETERFPROC)IntGetProcAddress("glPointParameterf");
        PointParameterf(pname, param);
    }

    static void CODEGEN_FUNCPTR Switch_PointParameterfv(GLenum pname, const GLfloat *params)
    {
        PointParameterfv = (PFNPOINTPARAMETERFVPROC)IntGetProcAddress("glPointParameterfv");
        PointParameterfv(pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_PointParameteri(GLenum pname, GLint param)
    {
        PointParameteri = (PFNPOINTPARAMETERIPROC)IntGetProcAddress("glPointParameteri");
        PointParameteri(pname, param);
    }

    static void CODEGEN_FUNCPTR Switch_PointParameteriv(GLenum pname, const GLint *params)
    {
        PointParameteriv = (PFNPOINTPARAMETERIVPROC)IntGetProcAddress("glPointParameteriv");
        PointParameteriv(pname, params);
    }

    // Extension: 1.5

    static void CODEGEN_FUNCPTR Switch_GenQueries(GLsizei n, GLuint *ids)
    {
        GenQueries = (PFNGENQUERIESPROC)IntGetProcAddress("glGenQueries");
        GenQueries(n, ids);
    }

    static void CODEGEN_FUNCPTR Switch_DeleteQueries(GLsizei n, const GLuint *ids)
    {
        DeleteQueries = (PFNDELETEQUERIESPROC)IntGetProcAddress("glDeleteQueries");
        DeleteQueries(n, ids);
    }

    static GLboolean CODEGEN_FUNCPTR Switch_IsQuery(GLuint id)
    {
        IsQuery = (PFNISQUERYPROC)IntGetProcAddress("glIsQuery");
        return IsQuery(id);
    }

    static void CODEGEN_FUNCPTR Switch_BeginQuery(GLenum target, GLuint id)
    {
        BeginQuery = (PFNBEGINQUERYPROC)IntGetProcAddress("glBeginQuery");
        BeginQuery(target, id);
    }

    static void CODEGEN_FUNCPTR Switch_EndQuery(GLenum target)
    {
        EndQuery = (PFNENDQUERYPROC)IntGetProcAddress("glEndQuery");
        EndQuery(target);
    }

    static void CODEGEN_FUNCPTR Switch_GetQueryiv(GLenum target, GLenum pname, GLint *params)
    {
        GetQueryiv = (PFNGETQUERYIVPROC)IntGetProcAddress("glGetQueryiv");
        GetQueryiv(target, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetQueryObjectiv(GLuint id, GLenum pname, GLint *params)
    {
        GetQueryObjectiv = (PFNGETQUERYOBJECTIVPROC)IntGetProcAddress("glGetQueryObjectiv");
        GetQueryObjectiv(id, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetQueryObjectuiv(GLuint id, GLenum pname, GLuint *params)
    {
        GetQueryObjectuiv = (PFNGETQUERYOBJECTUIVPROC)IntGetProcAddress("glGetQueryObjectuiv");
        GetQueryObjectuiv(id, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_BindBuffer(GLenum target, GLuint buffer)
    {
        BindBuffer = (PFNBINDBUFFERPROC)IntGetProcAddress("glBindBuffer");
        BindBuffer(target, buffer);
    }

    static void CODEGEN_FUNCPTR Switch_DeleteBuffers(GLsizei n, const GLuint *buffers)
    {
        DeleteBuffers = (PFNDELETEBUFFERSPROC)IntGetProcAddress("glDeleteBuffers");
        DeleteBuffers(n, buffers);
    }

    static void CODEGEN_FUNCPTR Switch_GenBuffers(GLsizei n, GLuint *buffers)
    {
        GenBuffers = (PFNGENBUFFERSPROC)IntGetProcAddress("glGenBuffers");
        GenBuffers(n, buffers);
    }

    static GLboolean CODEGEN_FUNCPTR Switch_IsBuffer(GLuint buffer)
    {
        IsBuffer = (PFNISBUFFERPROC)IntGetProcAddress("glIsBuffer");
        return IsBuffer(buffer);
    }

    static void CODEGEN_FUNCPTR Switch_BufferData(GLenum target, GLsizeiptr size, const GLvoid *data, GLenum usage)
    {
        BufferData = (PFNBUFFERDATAPROC)IntGetProcAddress("glBufferData");
        BufferData(target, size, data, usage);
    }

    static void CODEGEN_FUNCPTR Switch_BufferSubData(GLenum target, GLintptr offset, GLsizeiptr size, const GLvoid *data)
    {
        BufferSubData = (PFNBUFFERSUBDATAPROC)IntGetProcAddress("glBufferSubData");
        BufferSubData(target, offset, size, data);
    }

    static void CODEGEN_FUNCPTR Switch_GetBufferSubData(GLenum target, GLintptr offset, GLsizeiptr size, GLvoid *data)
    {
        GetBufferSubData = (PFNGETBUFFERSUBDATAPROC)IntGetProcAddress("glGetBufferSubData");
        GetBufferSubData(target, offset, size, data);
    }

    static GLvoid* CODEGEN_FUNCPTR Switch_MapBuffer(GLenum target, GLenum access)
    {
        MapBuffer = (PFNMAPBUFFERPROC)IntGetProcAddress("glMapBuffer");
        return MapBuffer(target, access);
    }

    static GLboolean CODEGEN_FUNCPTR Switch_UnmapBuffer(GLenum target)
    {
        UnmapBuffer = (PFNUNMAPBUFFERPROC)IntGetProcAddress("glUnmapBuffer");
        return UnmapBuffer(target);
    }

    static void CODEGEN_FUNCPTR Switch_GetBufferParameteriv(GLenum target, GLenum pname, GLint *params)
    {
        GetBufferParameteriv = (PFNGETBUFFERPARAMETERIVPROC)IntGetProcAddress("glGetBufferParameteriv");
        GetBufferParameteriv(target, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetBufferPointerv(GLenum target, GLenum pname, GLvoid* *params)
    {
        GetBufferPointerv = (PFNGETBUFFERPOINTERVPROC)IntGetProcAddress("glGetBufferPointerv");
        GetBufferPointerv(target, pname, params);
    }

    // Extension: 2.0

    static void CODEGEN_FUNCPTR Switch_BlendEquationSeparate(GLenum modeRGB, GLenum modeAlpha)
    {
        BlendEquationSeparate = (PFNBLENDEQUATIONSEPARATEPROC)IntGetProcAddress("glBlendEquationSeparate");
        BlendEquationSeparate(modeRGB, modeAlpha);
    }

    static void CODEGEN_FUNCPTR Switch_DrawBuffers(GLsizei n, const GLenum *bufs)
    {
        DrawBuffers = (PFNDRAWBUFFERSPROC)IntGetProcAddress("glDrawBuffers");
        DrawBuffers(n, bufs);
    }

    static void CODEGEN_FUNCPTR Switch_StencilOpSeparate(GLenum face, GLenum sfail, GLenum dpfail, GLenum dppass)
    {
        StencilOpSeparate = (PFNSTENCILOPSEPARATEPROC)IntGetProcAddress("glStencilOpSeparate");
        StencilOpSeparate(face, sfail, dpfail, dppass);
    }

    static void CODEGEN_FUNCPTR Switch_StencilFuncSeparate(GLenum face, GLenum func, GLint ref, GLuint mask)
    {
        StencilFuncSeparate = (PFNSTENCILFUNCSEPARATEPROC)IntGetProcAddress("glStencilFuncSeparate");
        StencilFuncSeparate(face, func, ref, mask);
    }

    static void CODEGEN_FUNCPTR Switch_StencilMaskSeparate(GLenum face, GLuint mask)
    {
        StencilMaskSeparate = (PFNSTENCILMASKSEPARATEPROC)IntGetProcAddress("glStencilMaskSeparate");
        StencilMaskSeparate(face, mask);
    }

    static void CODEGEN_FUNCPTR Switch_AttachShader(GLuint program, GLuint shader)
    {
        AttachShader = (PFNATTACHSHADERPROC)IntGetProcAddress("glAttachShader");
        AttachShader(program, shader);
    }

    static void CODEGEN_FUNCPTR Switch_BindAttribLocation(GLuint program, GLuint index, const GLchar *name)
    {
        BindAttribLocation = (PFNBINDATTRIBLOCATIONPROC)IntGetProcAddress("glBindAttribLocation");
        BindAttribLocation(program, index, name);
    }

    static void CODEGEN_FUNCPTR Switch_CompileShader(GLuint shader)
    {
        CompileShader = (PFNCOMPILESHADERPROC)IntGetProcAddress("glCompileShader");
        CompileShader(shader);
    }

    static GLuint CODEGEN_FUNCPTR Switch_CreateProgram()
    {
        CreateProgram = (PFNCREATEPROGRAMPROC)IntGetProcAddress("glCreateProgram");
        return CreateProgram();
    }

    static GLuint CODEGEN_FUNCPTR Switch_CreateShader(GLenum type)
    {
        CreateShader = (PFNCREATESHADERPROC)IntGetProcAddress("glCreateShader");
        return CreateShader(type);
    }

    static void CODEGEN_FUNCPTR Switch_DeleteProgram(GLuint program)
    {
        DeleteProgram = (PFNDELETEPROGRAMPROC)IntGetProcAddress("glDeleteProgram");
        DeleteProgram(program);
    }

    static void CODEGEN_FUNCPTR Switch_DeleteShader(GLuint shader)
    {
        DeleteShader = (PFNDELETESHADERPROC)IntGetProcAddress("glDeleteShader");
        DeleteShader(shader);
    }

    static void CODEGEN_FUNCPTR Switch_DetachShader(GLuint program, GLuint shader)
    {
        DetachShader = (PFNDETACHSHADERPROC)IntGetProcAddress("glDetachShader");
        DetachShader(program, shader);
    }

    static void CODEGEN_FUNCPTR Switch_DisableVertexAttribArray(GLuint index)
    {
        DisableVertexAttribArray = (PFNDISABLEVERTEXATTRIBARRAYPROC)IntGetProcAddress("glDisableVertexAttribArray");
        DisableVertexAttribArray(index);
    }

    static void CODEGEN_FUNCPTR Switch_EnableVertexAttribArray(GLuint index)
    {
        EnableVertexAttribArray = (PFNENABLEVERTEXATTRIBARRAYPROC)IntGetProcAddress("glEnableVertexAttribArray");
        EnableVertexAttribArray(index);
    }

    static void CODEGEN_FUNCPTR Switch_GetActiveAttrib(GLuint program, GLuint index, GLsizei bufSize, GLsizei *length, GLint *size, GLenum *type, GLchar *name)
    {
        GetActiveAttrib = (PFNGETACTIVEATTRIBPROC)IntGetProcAddress("glGetActiveAttrib");
        GetActiveAttrib(program, index, bufSize, length, size, type, name);
    }

    static void CODEGEN_FUNCPTR Switch_GetActiveUniform(GLuint program, GLuint index, GLsizei bufSize, GLsizei *length, GLint *size, GLenum *type, GLchar *name)
    {
        GetActiveUniform = (PFNGETACTIVEUNIFORMPROC)IntGetProcAddress("glGetActiveUniform");
        GetActiveUniform(program, index, bufSize, length, size, type, name);
    }

    static void CODEGEN_FUNCPTR Switch_GetAttachedShaders(GLuint program, GLsizei maxCount, GLsizei *count, GLuint *obj)
    {
        GetAttachedShaders = (PFNGETATTACHEDSHADERSPROC)IntGetProcAddress("glGetAttachedShaders");
        GetAttachedShaders(program, maxCount, count, obj);
    }

    static GLint CODEGEN_FUNCPTR Switch_GetAttribLocation(GLuint program, const GLchar *name)
    {
        GetAttribLocation = (PFNGETATTRIBLOCATIONPROC)IntGetProcAddress("glGetAttribLocation");
        return GetAttribLocation(program, name);
    }

    static void CODEGEN_FUNCPTR Switch_GetProgramiv(GLuint program, GLenum pname, GLint *params)
    {
        GetProgramiv = (PFNGETPROGRAMIVPROC)IntGetProcAddress("glGetProgramiv");
        GetProgramiv(program, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetProgramInfoLog(GLuint program, GLsizei bufSize, GLsizei *length, GLchar *infoLog)
    {
        GetProgramInfoLog = (PFNGETPROGRAMINFOLOGPROC)IntGetProcAddress("glGetProgramInfoLog");
        GetProgramInfoLog(program, bufSize, length, infoLog);
    }

    static void CODEGEN_FUNCPTR Switch_GetShaderiv(GLuint shader, GLenum pname, GLint *params)
    {
        GetShaderiv = (PFNGETSHADERIVPROC)IntGetProcAddress("glGetShaderiv");
        GetShaderiv(shader, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetShaderInfoLog(GLuint shader, GLsizei bufSize, GLsizei *length, GLchar *infoLog)
    {
        GetShaderInfoLog = (PFNGETSHADERINFOLOGPROC)IntGetProcAddress("glGetShaderInfoLog");
        GetShaderInfoLog(shader, bufSize, length, infoLog);
    }

    static void CODEGEN_FUNCPTR Switch_GetShaderSource(GLuint shader, GLsizei bufSize, GLsizei *length, GLchar *source)
    {
        GetShaderSource = (PFNGETSHADERSOURCEPROC)IntGetProcAddress("glGetShaderSource");
        GetShaderSource(shader, bufSize, length, source);
    }

    static GLint CODEGEN_FUNCPTR Switch_GetUniformLocation(GLuint program, const GLchar *name)
    {
        GetUniformLocation = (PFNGETUNIFORMLOCATIONPROC)IntGetProcAddress("glGetUniformLocation");
        return GetUniformLocation(program, name);
    }

    static void CODEGEN_FUNCPTR Switch_GetUniformfv(GLuint program, GLint location, GLfloat *params)
    {
        GetUniformfv = (PFNGETUNIFORMFVPROC)IntGetProcAddress("glGetUniformfv");
        GetUniformfv(program, location, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetUniformiv(GLuint program, GLint location, GLint *params)
    {
        GetUniformiv = (PFNGETUNIFORMIVPROC)IntGetProcAddress("glGetUniformiv");
        GetUniformiv(program, location, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetVertexAttribdv(GLuint index, GLenum pname, GLdouble *params)
    {
        GetVertexAttribdv = (PFNGETVERTEXATTRIBDVPROC)IntGetProcAddress("glGetVertexAttribdv");
        GetVertexAttribdv(index, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetVertexAttribfv(GLuint index, GLenum pname, GLfloat *params)
    {
        GetVertexAttribfv = (PFNGETVERTEXATTRIBFVPROC)IntGetProcAddress("glGetVertexAttribfv");
        GetVertexAttribfv(index, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetVertexAttribiv(GLuint index, GLenum pname, GLint *params)
    {
        GetVertexAttribiv = (PFNGETVERTEXATTRIBIVPROC)IntGetProcAddress("glGetVertexAttribiv");
        GetVertexAttribiv(index, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetVertexAttribPointerv(GLuint index, GLenum pname, GLvoid* *pointer)
    {
        GetVertexAttribPointerv = (PFNGETVERTEXATTRIBPOINTERVPROC)IntGetProcAddress("glGetVertexAttribPointerv");
        GetVertexAttribPointerv(index, pname, pointer);
    }

    static GLboolean CODEGEN_FUNCPTR Switch_IsProgram(GLuint program)
    {
        IsProgram = (PFNISPROGRAMPROC)IntGetProcAddress("glIsProgram");
        return IsProgram(program);
    }

    static GLboolean CODEGEN_FUNCPTR Switch_IsShader(GLuint shader)
    {
        IsShader = (PFNISSHADERPROC)IntGetProcAddress("glIsShader");
        return IsShader(shader);
    }

    static void CODEGEN_FUNCPTR Switch_LinkProgram(GLuint program)
    {
        LinkProgram = (PFNLINKPROGRAMPROC)IntGetProcAddress("glLinkProgram");
        LinkProgram(program);
    }

    static void CODEGEN_FUNCPTR Switch_ShaderSource(GLuint shader, GLsizei count, const GLchar* const *string, const GLint *length)
    {
        ShaderSource = (PFNSHADERSOURCEPROC)IntGetProcAddress("glShaderSource");
        ShaderSource(shader, count, string, length);
    }

    static void CODEGEN_FUNCPTR Switch_UseProgram(GLuint program)
    {
        UseProgram = (PFNUSEPROGRAMPROC)IntGetProcAddress("glUseProgram");
        UseProgram(program);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform1f(GLint location, GLfloat v0)
    {
        Uniform1f = (PFNUNIFORM1FPROC)IntGetProcAddress("glUniform1f");
        Uniform1f(location, v0);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform2f(GLint location, GLfloat v0, GLfloat v1)
    {
        Uniform2f = (PFNUNIFORM2FPROC)IntGetProcAddress("glUniform2f");
        Uniform2f(location, v0, v1);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform3f(GLint location, GLfloat v0, GLfloat v1, GLfloat v2)
    {
        Uniform3f = (PFNUNIFORM3FPROC)IntGetProcAddress("glUniform3f");
        Uniform3f(location, v0, v1, v2);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform4f(GLint location, GLfloat v0, GLfloat v1, GLfloat v2, GLfloat v3)
    {
        Uniform4f = (PFNUNIFORM4FPROC)IntGetProcAddress("glUniform4f");
        Uniform4f(location, v0, v1, v2, v3);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform1i(GLint location, GLint v0)
    {
        Uniform1i = (PFNUNIFORM1IPROC)IntGetProcAddress("glUniform1i");
        Uniform1i(location, v0);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform2i(GLint location, GLint v0, GLint v1)
    {
        Uniform2i = (PFNUNIFORM2IPROC)IntGetProcAddress("glUniform2i");
        Uniform2i(location, v0, v1);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform3i(GLint location, GLint v0, GLint v1, GLint v2)
    {
        Uniform3i = (PFNUNIFORM3IPROC)IntGetProcAddress("glUniform3i");
        Uniform3i(location, v0, v1, v2);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform4i(GLint location, GLint v0, GLint v1, GLint v2, GLint v3)
    {
        Uniform4i = (PFNUNIFORM4IPROC)IntGetProcAddress("glUniform4i");
        Uniform4i(location, v0, v1, v2, v3);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform1fv(GLint location, GLsizei count, const GLfloat *value)
    {
        Uniform1fv = (PFNUNIFORM1FVPROC)IntGetProcAddress("glUniform1fv");
        Uniform1fv(location, count, value);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform2fv(GLint location, GLsizei count, const GLfloat *value)
    {
        Uniform2fv = (PFNUNIFORM2FVPROC)IntGetProcAddress("glUniform2fv");
        Uniform2fv(location, count, value);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform3fv(GLint location, GLsizei count, const GLfloat *value)
    {
        Uniform3fv = (PFNUNIFORM3FVPROC)IntGetProcAddress("glUniform3fv");
        Uniform3fv(location, count, value);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform4fv(GLint location, GLsizei count, const GLfloat *value)
    {
        Uniform4fv = (PFNUNIFORM4FVPROC)IntGetProcAddress("glUniform4fv");
        Uniform4fv(location, count, value);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform1iv(GLint location, GLsizei count, const GLint *value)
    {
        Uniform1iv = (PFNUNIFORM1IVPROC)IntGetProcAddress("glUniform1iv");
        Uniform1iv(location, count, value);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform2iv(GLint location, GLsizei count, const GLint *value)
    {
        Uniform2iv = (PFNUNIFORM2IVPROC)IntGetProcAddress("glUniform2iv");
        Uniform2iv(location, count, value);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform3iv(GLint location, GLsizei count, const GLint *value)
    {
        Uniform3iv = (PFNUNIFORM3IVPROC)IntGetProcAddress("glUniform3iv");
        Uniform3iv(location, count, value);
    }

    static void CODEGEN_FUNCPTR Switch_Uniform4iv(GLint location, GLsizei count, const GLint *value)
    {
        Uniform4iv = (PFNUNIFORM4IVPROC)IntGetProcAddress("glUniform4iv");
        Uniform4iv(location, count, value);
    }

    static void CODEGEN_FUNCPTR Switch_UniformMatrix2fv(GLint location, GLsizei count, GLboolean transpose, const GLfloat *value)
    {
        UniformMatrix2fv = (PFNUNIFORMMATRIX2FVPROC)IntGetProcAddress("glUniformMatrix2fv");
        UniformMatrix2fv(location, count, transpose, value);
    }

    static void CODEGEN_FUNCPTR Switch_UniformMatrix3fv(GLint location, GLsizei count, GLboolean transpose, const GLfloat *value)
    {
        UniformMatrix3fv = (PFNUNIFORMMATRIX3FVPROC)IntGetProcAddress("glUniformMatrix3fv");
        UniformMatrix3fv(location, count, transpose, value);
    }

    static void CODEGEN_FUNCPTR Switch_UniformMatrix4fv(GLint location, GLsizei count, GLboolean transpose, const GLfloat *value)
    {
        UniformMatrix4fv = (PFNUNIFORMMATRIX4FVPROC)IntGetProcAddress("glUniformMatrix4fv");
        UniformMatrix4fv(location, count, transpose, value);
    }

    static void CODEGEN_FUNCPTR Switch_ValidateProgram(GLuint program)
    {
        ValidateProgram = (PFNVALIDATEPROGRAMPROC)IntGetProcAddress("glValidateProgram");
        ValidateProgram(program);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribPointer(GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const GLvoid *pointer)
    {
        VertexAttribPointer = (PFNVERTEXATTRIBPOINTERPROC)IntGetProcAddress("glVertexAttribPointer");
        VertexAttribPointer(index, size, type, normalized, stride, pointer);
    }

    // Extension: 2.1

    static void CODEGEN_FUNCPTR Switch_UniformMatrix2x3fv(GLint location, GLsizei count, GLboolean transpose, const GLfloat *value)
    {
        UniformMatrix2x3fv = (PFNUNIFORMMATRIX2X3FVPROC)IntGetProcAddress("glUniformMatrix2x3fv");
        UniformMatrix2x3fv(location, count, transpose, value);
    }

    static void CODEGEN_FUNCPTR Switch_UniformMatrix3x2fv(GLint location, GLsizei count, GLboolean transpose, const GLfloat *value)
    {
        UniformMatrix3x2fv = (PFNUNIFORMMATRIX3X2FVPROC)IntGetProcAddress("glUniformMatrix3x2fv");
        UniformMatrix3x2fv(location, count, transpose, value);
    }

    static void CODEGEN_FUNCPTR Switch_UniformMatrix2x4fv(GLint location, GLsizei count, GLboolean transpose, const GLfloat *value)
    {
        UniformMatrix2x4fv = (PFNUNIFORMMATRIX2X4FVPROC)IntGetProcAddress("glUniformMatrix2x4fv");
        UniformMatrix2x4fv(location, count, transpose, value);
    }

    static void CODEGEN_FUNCPTR Switch_UniformMatrix4x2fv(GLint location, GLsizei count, GLboolean transpose, const GLfloat *value)
    {
        UniformMatrix4x2fv = (PFNUNIFORMMATRIX4X2FVPROC)IntGetProcAddress("glUniformMatrix4x2fv");
        UniformMatrix4x2fv(location, count, transpose, value);
    }

    static void CODEGEN_FUNCPTR Switch_UniformMatrix3x4fv(GLint location, GLsizei count, GLboolean transpose, const GLfloat *value)
    {
        UniformMatrix3x4fv = (PFNUNIFORMMATRIX3X4FVPROC)IntGetProcAddress("glUniformMatrix3x4fv");
        UniformMatrix3x4fv(location, count, transpose, value);
    }

    static void CODEGEN_FUNCPTR Switch_UniformMatrix4x3fv(GLint location, GLsizei count, GLboolean transpose, const GLfloat *value)
    {
        UniformMatrix4x3fv = (PFNUNIFORMMATRIX4X3FVPROC)IntGetProcAddress("glUniformMatrix4x3fv");
        UniformMatrix4x3fv(location, count, transpose, value);
    }

    // Extension: ARB_vertex_array_object

    static void CODEGEN_FUNCPTR Switch_BindVertexArray(GLuint ren_array)
    {
        BindVertexArray = (PFNBINDVERTEXARRAYPROC)IntGetProcAddress("glBindVertexArray");
        BindVertexArray(ren_array);
    }

    static void CODEGEN_FUNCPTR Switch_DeleteVertexArrays(GLsizei n, const GLuint *arrays)
    {
        DeleteVertexArrays = (PFNDELETEVERTEXARRAYSPROC)IntGetProcAddress("glDeleteVertexArrays");
        DeleteVertexArrays(n, arrays);
    }

    static void CODEGEN_FUNCPTR Switch_GenVertexArrays(GLsizei n, GLuint *arrays)
    {
        GenVertexArrays = (PFNGENVERTEXARRAYSPROC)IntGetProcAddress("glGenVertexArrays");
        GenVertexArrays(n, arrays);
    }

    static GLboolean CODEGEN_FUNCPTR Switch_IsVertexArray(GLuint ren_array)
    {
        IsVertexArray = (PFNISVERTEXARRAYPROC)IntGetProcAddress("glIsVertexArray");
        return IsVertexArray(ren_array);
    }

    // Extension: ARB_map_buffer_range

    static GLvoid* CODEGEN_FUNCPTR Switch_MapBufferRange(GLenum target, GLintptr offset, GLsizeiptr length, GLbitfield access)
    {
        MapBufferRange = (PFNMAPBUFFERRANGEPROC)IntGetProcAddress("glMapBufferRange");
        return MapBufferRange(target, offset, length, access);
    }

    static void CODEGEN_FUNCPTR Switch_FlushMappedBufferRange(GLenum target, GLintptr offset, GLsizeiptr length)
    {
        FlushMappedBufferRange = (PFNFLUSHMAPPEDBUFFERRANGEPROC)IntGetProcAddress("glFlushMappedBufferRange");
        FlushMappedBufferRange(target, offset, length);
    }

    // Extension: ARB_framebuffer_object

    static GLboolean CODEGEN_FUNCPTR Switch_IsRenderbuffer(GLuint renderbuffer)
    {
        IsRenderbuffer = (PFNISRENDERBUFFERPROC)IntGetProcAddress("glIsRenderbuffer");
        return IsRenderbuffer(renderbuffer);
    }

    static void CODEGEN_FUNCPTR Switch_BindRenderbuffer(GLenum target, GLuint renderbuffer)
    {
        BindRenderbuffer = (PFNBINDRENDERBUFFERPROC)IntGetProcAddress("glBindRenderbuffer");
        BindRenderbuffer(target, renderbuffer);
    }

    static void CODEGEN_FUNCPTR Switch_DeleteRenderbuffers(GLsizei n, const GLuint *renderbuffers)
    {
        DeleteRenderbuffers = (PFNDELETERENDERBUFFERSPROC)IntGetProcAddress("glDeleteRenderbuffers");
        DeleteRenderbuffers(n, renderbuffers);
    }

    static void CODEGEN_FUNCPTR Switch_GenRenderbuffers(GLsizei n, GLuint *renderbuffers)
    {
        GenRenderbuffers = (PFNGENRENDERBUFFERSPROC)IntGetProcAddress("glGenRenderbuffers");
        GenRenderbuffers(n, renderbuffers);
    }

    static void CODEGEN_FUNCPTR Switch_RenderbufferStorage(GLenum target, GLenum internalformat, GLsizei width, GLsizei height)
    {
        RenderbufferStorage = (PFNRENDERBUFFERSTORAGEPROC)IntGetProcAddress("glRenderbufferStorage");
        RenderbufferStorage(target, internalformat, width, height);
    }

    static void CODEGEN_FUNCPTR Switch_GetRenderbufferParameteriv(GLenum target, GLenum pname, GLint *params)
    {
        GetRenderbufferParameteriv = (PFNGETRENDERBUFFERPARAMETERIVPROC)IntGetProcAddress("glGetRenderbufferParameteriv");
        GetRenderbufferParameteriv(target, pname, params);
    }

    static GLboolean CODEGEN_FUNCPTR Switch_IsFramebuffer(GLuint framebuffer)
    {
        IsFramebuffer = (PFNISFRAMEBUFFERPROC)IntGetProcAddress("glIsFramebuffer");
        return IsFramebuffer(framebuffer);
    }

    static void CODEGEN_FUNCPTR Switch_BindFramebuffer(GLenum target, GLuint framebuffer)
    {
        BindFramebuffer = (PFNBINDFRAMEBUFFERPROC)IntGetProcAddress("glBindFramebuffer");
        BindFramebuffer(target, framebuffer);
    }

    static void CODEGEN_FUNCPTR Switch_DeleteFramebuffers(GLsizei n, const GLuint *framebuffers)
    {
        DeleteFramebuffers = (PFNDELETEFRAMEBUFFERSPROC)IntGetProcAddress("glDeleteFramebuffers");
        DeleteFramebuffers(n, framebuffers);
    }

    static void CODEGEN_FUNCPTR Switch_GenFramebuffers(GLsizei n, GLuint *framebuffers)
    {
        GenFramebuffers = (PFNGENFRAMEBUFFERSPROC)IntGetProcAddress("glGenFramebuffers");
        GenFramebuffers(n, framebuffers);
    }

    static GLenum CODEGEN_FUNCPTR Switch_CheckFramebufferStatus(GLenum target)
    {
        CheckFramebufferStatus = (PFNCHECKFRAMEBUFFERSTATUSPROC)IntGetProcAddress("glCheckFramebufferStatus");
        return CheckFramebufferStatus(target);
    }

    static void CODEGEN_FUNCPTR Switch_FramebufferTexture1D(GLenum target, GLenum attachment, GLenum textarget, GLuint texture, GLint level)
    {
        FramebufferTexture1D = (PFNFRAMEBUFFERTEXTURE1DPROC)IntGetProcAddress("glFramebufferTexture1D");
        FramebufferTexture1D(target, attachment, textarget, texture, level);
    }

    static void CODEGEN_FUNCPTR Switch_FramebufferTexture2D(GLenum target, GLenum attachment, GLenum textarget, GLuint texture, GLint level)
    {
        FramebufferTexture2D = (PFNFRAMEBUFFERTEXTURE2DPROC)IntGetProcAddress("glFramebufferTexture2D");
        FramebufferTexture2D(target, attachment, textarget, texture, level);
    }

    static void CODEGEN_FUNCPTR Switch_FramebufferTexture3D(GLenum target, GLenum attachment, GLenum textarget, GLuint texture, GLint level, GLint zoffset)
    {
        FramebufferTexture3D = (PFNFRAMEBUFFERTEXTURE3DPROC)IntGetProcAddress("glFramebufferTexture3D");
        FramebufferTexture3D(target, attachment, textarget, texture, level, zoffset);
    }

    static void CODEGEN_FUNCPTR Switch_FramebufferRenderbuffer(GLenum target, GLenum attachment, GLenum renderbuffertarget, GLuint renderbuffer)
    {
        FramebufferRenderbuffer = (PFNFRAMEBUFFERRENDERBUFFERPROC)IntGetProcAddress("glFramebufferRenderbuffer");
        FramebufferRenderbuffer(target, attachment, renderbuffertarget, renderbuffer);
    }

    static void CODEGEN_FUNCPTR Switch_GetFramebufferAttachmentParameteriv(GLenum target, GLenum attachment, GLenum pname, GLint *params)
    {
        GetFramebufferAttachmentParameteriv = (PFNGETFRAMEBUFFERATTACHMENTPARAMETERIVPROC)IntGetProcAddress("glGetFramebufferAttachmentParameteriv");
        GetFramebufferAttachmentParameteriv(target, attachment, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GenerateMipmap(GLenum target)
    {
        GenerateMipmap = (PFNGENERATEMIPMAPPROC)IntGetProcAddress("glGenerateMipmap");
        GenerateMipmap(target);
    }

    static void CODEGEN_FUNCPTR Switch_BlitFramebuffer(GLint srcX0, GLint srcY0, GLint srcX1, GLint srcY1, GLint dstX0, GLint dstY0, GLint dstX1, GLint dstY1, GLbitfield mask, GLenum filter)
    {
        BlitFramebuffer = (PFNBLITFRAMEBUFFERPROC)IntGetProcAddress("glBlitFramebuffer");
        BlitFramebuffer(srcX0, srcY0, srcX1, srcY1, dstX0, dstY0, dstX1, dstY1, mask, filter);
    }

    static void CODEGEN_FUNCPTR Switch_RenderbufferStorageMultisample(GLenum target, GLsizei samples, GLenum internalformat, GLsizei width, GLsizei height)
    {
        RenderbufferStorageMultisample = (PFNRENDERBUFFERSTORAGEMULTISAMPLEPROC)IntGetProcAddress("glRenderbufferStorageMultisample");
        RenderbufferStorageMultisample(target, samples, internalformat, width, height);
    }

    static void CODEGEN_FUNCPTR Switch_FramebufferTextureLayer(GLenum target, GLenum attachment, GLuint texture, GLint level, GLint layer)
    {
        FramebufferTextureLayer = (PFNFRAMEBUFFERTEXTURELAYERPROC)IntGetProcAddress("glFramebufferTextureLayer");
        FramebufferTextureLayer(target, attachment, texture, level, layer);
    }

    // Extension: 3.0

    static void CODEGEN_FUNCPTR Switch_ColorMaski(GLuint index, GLboolean r, GLboolean g, GLboolean b, GLboolean a)
    {
        ColorMaski = (PFNCOLORMASKIPROC)IntGetProcAddress("glColorMaski");
        ColorMaski(index, r, g, b, a);
    }

    static void CODEGEN_FUNCPTR Switch_GetBooleani_v(GLenum target, GLuint index, GLboolean *data)
    {
        GetBooleani_v = (PFNGETBOOLEANI_VPROC)IntGetProcAddress("glGetBooleani_v");
        GetBooleani_v(target, index, data);
    }

    static void CODEGEN_FUNCPTR Switch_GetIntegeri_v(GLenum target, GLuint index, GLint *data)
    {
        GetIntegeri_v = (PFNGETINTEGERI_VPROC)IntGetProcAddress("glGetIntegeri_v");
        GetIntegeri_v(target, index, data);
    }

    static void CODEGEN_FUNCPTR Switch_Enablei(GLenum target, GLuint index)
    {
        Enablei = (PFNENABLEIPROC)IntGetProcAddress("glEnablei");
        Enablei(target, index);
    }

    static void CODEGEN_FUNCPTR Switch_Disablei(GLenum target, GLuint index)
    {
        Disablei = (PFNDISABLEIPROC)IntGetProcAddress("glDisablei");
        Disablei(target, index);
    }

    static GLboolean CODEGEN_FUNCPTR Switch_IsEnabledi(GLenum target, GLuint index)
    {
        IsEnabledi = (PFNISENABLEDIPROC)IntGetProcAddress("glIsEnabledi");
        return IsEnabledi(target, index);
    }

    static void CODEGEN_FUNCPTR Switch_BeginTransformFeedback(GLenum primitiveMode)
    {
        BeginTransformFeedback = (PFNBEGINTRANSFORMFEEDBACKPROC)IntGetProcAddress("glBeginTransformFeedback");
        BeginTransformFeedback(primitiveMode);
    }

    static void CODEGEN_FUNCPTR Switch_EndTransformFeedback()
    {
        EndTransformFeedback = (PFNENDTRANSFORMFEEDBACKPROC)IntGetProcAddress("glEndTransformFeedback");
        EndTransformFeedback();
    }

    static void CODEGEN_FUNCPTR Switch_BindBufferRange(GLenum target, GLuint index, GLuint buffer, GLintptr offset, GLsizeiptr size)
    {
        BindBufferRange = (PFNBINDBUFFERRANGEPROC)IntGetProcAddress("glBindBufferRange");
        BindBufferRange(target, index, buffer, offset, size);
    }

    static void CODEGEN_FUNCPTR Switch_BindBufferBase(GLenum target, GLuint index, GLuint buffer)
    {
        BindBufferBase = (PFNBINDBUFFERBASEPROC)IntGetProcAddress("glBindBufferBase");
        BindBufferBase(target, index, buffer);
    }

    static void CODEGEN_FUNCPTR Switch_TransformFeedbackVaryings(GLuint program, GLsizei count, const GLchar* const *varyings, GLenum bufferMode)
    {
        TransformFeedbackVaryings = (PFNTRANSFORMFEEDBACKVARYINGSPROC)IntGetProcAddress("glTransformFeedbackVaryings");
        TransformFeedbackVaryings(program, count, varyings, bufferMode);
    }

    static void CODEGEN_FUNCPTR Switch_GetTransformFeedbackVarying(GLuint program, GLuint index, GLsizei bufSize, GLsizei *length, GLsizei *size, GLenum *type, GLchar *name)
    {
        GetTransformFeedbackVarying = (PFNGETTRANSFORMFEEDBACKVARYINGPROC)IntGetProcAddress("glGetTransformFeedbackVarying");
        GetTransformFeedbackVarying(program, index, bufSize, length, size, type, name);
    }

    static void CODEGEN_FUNCPTR Switch_ClampColor(GLenum target, GLenum clamp)
    {
        ClampColor = (PFNCLAMPCOLORPROC)IntGetProcAddress("glClampColor");
        ClampColor(target, clamp);
    }

    static void CODEGEN_FUNCPTR Switch_BeginConditionalRender(GLuint id, GLenum mode)
    {
        BeginConditionalRender = (PFNBEGINCONDITIONALRENDERPROC)IntGetProcAddress("glBeginConditionalRender");
        BeginConditionalRender(id, mode);
    }

    static void CODEGEN_FUNCPTR Switch_EndConditionalRender()
    {
        EndConditionalRender = (PFNENDCONDITIONALRENDERPROC)IntGetProcAddress("glEndConditionalRender");
        EndConditionalRender();
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribIPointer(GLuint index, GLint size, GLenum type, GLsizei stride, const GLvoid *pointer)
    {
        VertexAttribIPointer = (PFNVERTEXATTRIBIPOINTERPROC)IntGetProcAddress("glVertexAttribIPointer");
        VertexAttribIPointer(index, size, type, stride, pointer);
    }

    static void CODEGEN_FUNCPTR Switch_GetVertexAttribIiv(GLuint index, GLenum pname, GLint *params)
    {
        GetVertexAttribIiv = (PFNGETVERTEXATTRIBIIVPROC)IntGetProcAddress("glGetVertexAttribIiv");
        GetVertexAttribIiv(index, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_GetVertexAttribIuiv(GLuint index, GLenum pname, GLuint *params)
    {
        GetVertexAttribIuiv = (PFNGETVERTEXATTRIBIUIVPROC)IntGetProcAddress("glGetVertexAttribIuiv");
        GetVertexAttribIuiv(index, pname, params);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI1i(GLuint index, GLint x)
    {
        VertexAttribI1i = (PFNVERTEXATTRIBI1IPROC)IntGetProcAddress("glVertexAttribI1i");
        VertexAttribI1i(index, x);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI2i(GLuint index, GLint x, GLint y)
    {
        VertexAttribI2i = (PFNVERTEXATTRIBI2IPROC)IntGetProcAddress("glVertexAttribI2i");
        VertexAttribI2i(index, x, y);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI3i(GLuint index, GLint x, GLint y, GLint z)
    {
        VertexAttribI3i = (PFNVERTEXATTRIBI3IPROC)IntGetProcAddress("glVertexAttribI3i");
        VertexAttribI3i(index, x, y, z);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI4i(GLuint index, GLint x, GLint y, GLint z, GLint w)
    {
        VertexAttribI4i = (PFNVERTEXATTRIBI4IPROC)IntGetProcAddress("glVertexAttribI4i");
        VertexAttribI4i(index, x, y, z, w);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI1ui(GLuint index, GLuint x)
    {
        VertexAttribI1ui = (PFNVERTEXATTRIBI1UIPROC)IntGetProcAddress("glVertexAttribI1ui");
        VertexAttribI1ui(index, x);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI2ui(GLuint index, GLuint x, GLuint y)
    {
        VertexAttribI2ui = (PFNVERTEXATTRIBI2UIPROC)IntGetProcAddress("glVertexAttribI2ui");
        VertexAttribI2ui(index, x, y);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI3ui(GLuint index, GLuint x, GLuint y, GLuint z)
    {
        VertexAttribI3ui = (PFNVERTEXATTRIBI3UIPROC)IntGetProcAddress("glVertexAttribI3ui");
        VertexAttribI3ui(index, x, y, z);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI4ui(GLuint index, GLuint x, GLuint y, GLuint z, GLuint w)
    {
        VertexAttribI4ui = (PFNVERTEXATTRIBI4UIPROC)IntGetProcAddress("glVertexAttribI4ui");
        VertexAttribI4ui(index, x, y, z, w);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI1iv(GLuint index, const GLint *v)
    {
        VertexAttribI1iv = (PFNVERTEXATTRIBI1IVPROC)IntGetProcAddress("glVertexAttribI1iv");
        VertexAttribI1iv(index, v);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI2iv(GLuint index, const GLint *v)
    {
        VertexAttribI2iv = (PFNVERTEXATTRIBI2IVPROC)IntGetProcAddress("glVertexAttribI2iv");
        VertexAttribI2iv(index, v);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI3iv(GLuint index, const GLint *v)
    {
        VertexAttribI3iv = (PFNVERTEXATTRIBI3IVPROC)IntGetProcAddress("glVertexAttribI3iv");
        VertexAttribI3iv(index, v);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI4iv(GLuint index, const GLint *v)
    {
        VertexAttribI4iv = (PFNVERTEXATTRIBI4IVPROC)IntGetProcAddress("glVertexAttribI4iv");
        VertexAttribI4iv(index, v);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI1uiv(GLuint index, const GLuint *v)
    {
        VertexAttribI1uiv = (PFNVERTEXATTRIBI1UIVPROC)IntGetProcAddress("glVertexAttribI1uiv");
        VertexAttribI1uiv(index, v);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI2uiv(GLuint index, const GLuint *v)
    {
        VertexAttribI2uiv = (PFNVERTEXATTRIBI2UIVPROC)IntGetProcAddress("glVertexAttribI2uiv");
        VertexAttribI2uiv(index, v);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI3uiv(GLuint index, const GLuint *v)
    {
        VertexAttribI3uiv = (PFNVERTEXATTRIBI3UIVPROC)IntGetProcAddress("glVertexAttribI3uiv");
        VertexAttribI3uiv(index, v);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI4uiv(GLuint index, const GLuint *v)
    {
        VertexAttribI4uiv = (PFNVERTEXATTRIBI4UIVPROC)IntGetProcAddress("glVertexAttribI4uiv");
        VertexAttribI4uiv(index, v);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI4bv(GLuint index, const GLbyte *v)
    {
        VertexAttribI4bv = (PFNVERTEXATTRIBI4BVPROC)IntGetProcAddress("glVertexAttribI4bv");
        VertexAttribI4bv(index, v);
    }

    static void CODEGEN_FUNCPTR Switch_VertexAttribI4sv(GLuint in
... [Content truncated - file is very large]
```

## High-Level Overview

This is a C++ implementation file containing the core logic and algorithms for OpenCV functionality.

**Key Characteristics:**
- Implements algorithms and data processing routines
- May contain performance-critical code
- Uses C++ features like templates, classes, and STL
- Integrates with OpenCV's module system


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **InitializeVariables**: A class/struct defined in this file

### Functions and Methods

- **HAVE_OPENGL()**: A function/method defined in this file
- **GLboolean()**: A function/method defined in this file
- **void()**: A function/method defined in this file
- **__APPLE__()**: A function/method defined in this file
- **GLenum()**: A function/method defined in this file
- **const()**: A function/method defined in this file
- **_MSC_VER()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `dlfcn.h`
- `gl_core_3_1.hpp`
- `GL/glx.h`
- `stdio.h`
- `precomp.hpp`

**Python Imports:**
- `this`


### Architectural Role

This file operates within the OpenCV module system, interfacing with other components through well-defined APIs and data structures.

## Performance and Complexity

### Computational Complexity

The algorithms and data structures in this file have various complexity characteristics depending on the operations performed.

### Memory Considerations

Memory usage patterns depend on the specific functionality implemented, including stack allocations, heap allocations, and resource management strategies.

### Performance Optimization

OpenCV employs various optimization techniques including:
- SIMD vectorization where applicable
- Multi-threading support
- Hardware acceleration (CUDA, OpenCL, etc.)
- Efficient memory access patterns

## Security and Safety Considerations

### Potential Vulnerabilities

Code that processes external data should be carefully reviewed for:
- Buffer overflow vulnerabilities
- Integer overflow/underflow
- Input validation issues
- Resource exhaustion attacks

### Safety Measures

OpenCV includes various safety mechanisms:
- Bounds checking in debug builds
- Exception handling
- Resource management (RAII in C++)
- Input sanitization

## Testing and Usage

### How to Use This File

This file is typically used as part of the larger OpenCV library and is not intended to be used in isolation.

### Testing Approach

Testing should cover:
- Unit tests for individual functions
- Integration tests for component interactions
- Performance benchmarks
- Edge case validation

## Related Files

This file is related to other files in the same module and may interact with files in other modules.

