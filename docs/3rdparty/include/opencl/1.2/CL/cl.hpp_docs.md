# Documentation for `3rdparty/include/opencl/1.2/CL/cl.hpp`

## File Metadata

- **Full Path**: `3rdparty/include/opencl/1.2/CL/cl.hpp`
- **File Name**: `cl.hpp`
- **File Size**: 277,159 bytes
- **File Type**: .hpp
- **Link to Source**: [3rdparty/include/opencl/1.2/CL/cl.hpp](../../../../../3rdparty/include/opencl/1.2/CL/cl.hpp)

## Purpose and Role

This file is located in the `3rdparty/include/opencl/1.2/CL` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*******************************************************************************
 * Copyright (c) 2008-2013 The Khronos Group Inc.
 *
 * Permission is hereby granted, free of charge, to any person obtaining a
 * copy of this software and/or associated documentation files (the
 * "Materials"), to deal in the Materials without restriction, including
 * without limitation the rights to use, copy, modify, merge, publish,
 * distribute, sublicense, and/or sell copies of the Materials, and to
 * permit persons to whom the Materials are furnished to do so, subject to
 * the following conditions:
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Materials.
 *
 * THE MATERIALS ARE PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
 * EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
 * MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
 * IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
 * CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
 * TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
 * MATERIALS OR THE USE OR OTHER DEALINGS IN THE MATERIALS.
 ******************************************************************************/

/*! \file
 *
 *   \brief C++ bindings for OpenCL 1.0 (rev 48), OpenCL 1.1 (rev 33) and 
 *       OpenCL 1.2 (rev 15)    
 *   \author Benedict R. Gaster, Laurent Morichetti and Lee Howes
 *   
 *   Additions and fixes from:
 *       Brian Cole, March 3rd 2010 and April 2012 
 *       Matt Gruenke, April 2012.
 *       Bruce Merry, February 2013.
 *   
 *   \version 1.2.5
 *   \date June 2013
 *
 *   Optional extension support
 *
 *         cl
 *         cl_ext_device_fission
 *				#define USE_CL_DEVICE_FISSION
 */

/*! \mainpage
 * \section intro Introduction
 * For many large applications C++ is the language of choice and so it seems
 * reasonable to define C++ bindings for OpenCL.
 *
 *
 * The interface is contained with a single C++ header file \em cl.hpp and all
 * definitions are contained within the namespace \em cl. There is no additional
 * requirement to include \em cl.h and to use either the C++ or original C
 * bindings it is enough to simply include \em cl.hpp.
 *
 * The bindings themselves are lightweight and correspond closely to the
 * underlying C API. Using the C++ bindings introduces no additional execution
 * overhead.
 *
 * For detail documentation on the bindings see:
 *
 * The OpenCL C++ Wrapper API 1.2 (revision 09)
 *  http://www.khronos.org/registry/cl/specs/opencl-cplusplus-1.2.pdf
 *
 * \section example Example
 *
 * The following example shows a general use case for the C++
 * bindings, including support for the optional exception feature and
 * also the supplied vector and string classes, see following sections for
 * decriptions of these features.
 *
 * \code
 * #define __CL_ENABLE_EXCEPTIONS
 * 
 * #if defined(__APPLE__) || defined(__MACOSX)
 * #include <OpenCL/cl.hpp>
 * #else
 * #include <CL/cl.hpp>
 * #endif
 * #include <cstdio>
 * #include <cstdlib>
 * #include <iostream>
 * 
 *  const char * helloStr  = "__kernel void "
 *                           "hello(void) "
 *                           "{ "
 *                           "  "
 *                           "} ";
 * 
 *  int
 *  main(void)
 *  {
 *     cl_int err = CL_SUCCESS;
 *     try {
 *
 *       std::vector<cl::Platform> platforms;
 *       cl::Platform::get(&platforms);
 *       if (platforms.size() == 0) {
 *           std::cout << "Platform size 0\n";
 *           return -1;
 *       }
 *
 *       cl_context_properties properties[] = 
 *          { CL_CONTEXT_PLATFORM, (cl_context_properties)(platforms[0])(), 0};
 *       cl::Context context(CL_DEVICE_TYPE_CPU, properties); 
 * 
 *       std::vector<cl::Device> devices = context.getInfo<CL_CONTEXT_DEVICES>();
 * 
 *       cl::Program::Sources source(1,
 *           std::make_pair(helloStr,strlen(helloStr)));
 *       cl::Program program_ = cl::Program(context, source);
 *       program_.build(devices);
 * 
 *       cl::Kernel kernel(program_, "hello", &err);
 * 
 *       cl::Event event;
 *       cl::CommandQueue queue(context, devices[0], 0, &err);
 *       queue.enqueueNDRangeKernel(
 *           kernel, 
 *           cl::NullRange, 
 *           cl::NDRange(4,4),
 *           cl::NullRange,
 *           NULL,
 *           &event); 
 * 
 *       event.wait();
 *     }
 *     catch (cl::Error err) {
 *        std::cerr 
 *           << "ERROR: "
 *           << err.what()
 *           << "("
 *           << err.err()
 *           << ")"
 *           << std::endl;
 *     }
 * 
 *    return EXIT_SUCCESS;
 *  }
 * 
 * \endcode
 *
 */
#ifndef CL_HPP_
#define CL_HPP_

#ifdef _WIN32

#include <windows.h>
#include <malloc.h>
#include <iterator>
#include <intrin.h>

#if defined(__CL_ENABLE_EXCEPTIONS)
#include <exception>
#endif // #if defined(__CL_ENABLE_EXCEPTIONS)

#pragma push_macro("max")
#undef max
#if defined(USE_DX_INTEROP)
#include <CL/cl_d3d10.h>
#include <CL/cl_dx9_media_sharing.h>
#endif
#endif // _WIN32

// 
#if defined(USE_CL_DEVICE_FISSION)
#include <CL/cl_ext.h>
#endif

#if defined(__APPLE__) || defined(__MACOSX)
#include <OpenGL/OpenGL.h>
#include <OpenCL/opencl.h>
#include <libkern/OSAtomic.h>
#else
#include <GL/gl.h>
#include <CL/opencl.h>
#endif // !__APPLE__

// To avoid accidentally taking ownership of core OpenCL types
// such as cl_kernel constructors are made explicit
// under OpenCL 1.2
#if defined(CL_VERSION_1_2) && !defined(CL_USE_DEPRECATED_OPENCL_1_1_APIS)
#define __CL_EXPLICIT_CONSTRUCTORS explicit
#else // #if defined(CL_USE_DEPRECATED_OPENCL_1_1_APIS)
#define __CL_EXPLICIT_CONSTRUCTORS 
#endif // #if defined(CL_USE_DEPRECATED_OPENCL_1_1_APIS)

// Define deprecated prefixes and suffixes to ensure compilation
// in case they are not pre-defined
#if !defined(CL_EXT_PREFIX__VERSION_1_1_DEPRECATED)
#define CL_EXT_PREFIX__VERSION_1_1_DEPRECATED  
#endif // #if !defined(CL_EXT_PREFIX__VERSION_1_1_DEPRECATED)
#if !defined(CL_EXT_SUFFIX__VERSION_1_1_DEPRECATED)
#define CL_EXT_SUFFIX__VERSION_1_1_DEPRECATED
#endif // #if !defined(CL_EXT_PREFIX__VERSION_1_1_DEPRECATED)

#if !defined(CL_CALLBACK)
#define CL_CALLBACK
#endif //CL_CALLBACK

#include <utility>
#include <limits>

#if !defined(__NO_STD_VECTOR)
#include <vector>
#endif

#if !defined(__NO_STD_STRING)
#include <string>
#endif 

#if defined(__linux__) || defined(__APPLE__) || defined(__MACOSX)
#include <alloca.h>

#include <emmintrin.h>
#include <xmmintrin.h>
#endif // linux

#include <cstring>


/*! \namespace cl
 *
 * \brief The OpenCL C++ bindings are defined within this namespace.
 *
 */
namespace cl {

class Memory;

/**
 * Deprecated APIs for 1.2
 */
#if defined(CL_USE_DEPRECATED_OPENCL_1_1_APIS) || (defined(CL_VERSION_1_1) && !defined(CL_VERSION_1_2)) 
#define __INIT_CL_EXT_FCN_PTR(name) \
    if(!pfn_##name) { \
        pfn_##name = (PFN_##name) \
            clGetExtensionFunctionAddress(#name); \
        if(!pfn_##name) { \
        } \
    }
#endif // #if defined(CL_VERSION_1_1)

#if defined(CL_VERSION_1_2)
#define __INIT_CL_EXT_FCN_PTR_PLATFORM(platform, name) \
    if(!pfn_##name) { \
        pfn_##name = (PFN_##name) \
            clGetExtensionFunctionAddressForPlatform(platform, #name); \
        if(!pfn_##name) { \
        } \
    }
#endif // #if defined(CL_VERSION_1_1)

class Program;
class Device;
class Context;
class CommandQueue;
class Memory;
class Buffer;

#if defined(__CL_ENABLE_EXCEPTIONS)
/*! \brief Exception class 
 * 
 *  This may be thrown by API functions when __CL_ENABLE_EXCEPTIONS is defined.
 */
class Error : public std::exception
{
private:
    cl_int err_;
    const char * errStr_;
public:
    /*! \brief Create a new CL error exception for a given error code
     *  and corresponding message.
     * 
     *  \param err error code value.
     *
     *  \param errStr a descriptive string that must remain in scope until
     *                handling of the exception has concluded.  If set, it
     *                will be returned by what().
     */
    Error(cl_int err, const char * errStr = NULL) : err_(err), errStr_(errStr)
    {}

    ~Error() throw() {}

    /*! \brief Get error string associated with exception
     *
     * \return A memory pointer to the error message string.
     */
    virtual const char * what() const throw ()
    {
        if (errStr_ == NULL) {
            return "empty";
        }
        else {
            return errStr_;
        }
    }

    /*! \brief Get error code associated with exception
     *
     *  \return The error code.
     */
    cl_int err(void) const { return err_; }
};

#define __ERR_STR(x) #x
#else
#define __ERR_STR(x) NULL
#endif // __CL_ENABLE_EXCEPTIONS


namespace detail
{
#if defined(__CL_ENABLE_EXCEPTIONS)
static inline cl_int errHandler (
    cl_int err,
    const char * errStr = NULL)
{
    if (err != CL_SUCCESS) {
        throw Error(err, errStr);
    }
    return err;
}
#else
static inline cl_int errHandler (cl_int err, const char * errStr = NULL)
{
    (void) errStr; // suppress unused variable warning
    return err;
}
#endif // __CL_ENABLE_EXCEPTIONS
}



//! \cond DOXYGEN_DETAIL
#if !defined(__CL_USER_OVERRIDE_ERROR_STRINGS)
#define __GET_DEVICE_INFO_ERR               __ERR_STR(clGetDeviceInfo)
#define __GET_PLATFORM_INFO_ERR             __ERR_STR(clGetPlatformInfo)
#define __GET_DEVICE_IDS_ERR                __ERR_STR(clGetDeviceIDs)
#define __GET_PLATFORM_IDS_ERR              __ERR_STR(clGetPlatformIDs)
#define __GET_CONTEXT_INFO_ERR              __ERR_STR(clGetContextInfo)
#define __GET_EVENT_INFO_ERR                __ERR_STR(clGetEventInfo)
#define __GET_EVENT_PROFILE_INFO_ERR        __ERR_STR(clGetEventProfileInfo)
#define __GET_MEM_OBJECT_INFO_ERR           __ERR_STR(clGetMemObjectInfo)
#define __GET_IMAGE_INFO_ERR                __ERR_STR(clGetImageInfo)
#define __GET_SAMPLER_INFO_ERR              __ERR_STR(clGetSamplerInfo)
#define __GET_KERNEL_INFO_ERR               __ERR_STR(clGetKernelInfo)
#if defined(CL_VERSION_1_2)
#define __GET_KERNEL_ARG_INFO_ERR               __ERR_STR(clGetKernelArgInfo)
#endif // #if defined(CL_VERSION_1_2)
#define __GET_KERNEL_WORK_GROUP_INFO_ERR    __ERR_STR(clGetKernelWorkGroupInfo)
#define __GET_PROGRAM_INFO_ERR              __ERR_STR(clGetProgramInfo)
#define __GET_PROGRAM_BUILD_INFO_ERR        __ERR_STR(clGetProgramBuildInfo)
#define __GET_COMMAND_QUEUE_INFO_ERR        __ERR_STR(clGetCommandQueueInfo)

#define __CREATE_CONTEXT_ERR                __ERR_STR(clCreateContext)
#define __CREATE_CONTEXT_FROM_TYPE_ERR      __ERR_STR(clCreateContextFromType)
#define __GET_SUPPORTED_IMAGE_FORMATS_ERR   __ERR_STR(clGetSupportedImageFormats)

#define __CREATE_BUFFER_ERR                 __ERR_STR(clCreateBuffer)
#define __COPY_ERR                          __ERR_STR(cl::copy)
#define __CREATE_SUBBUFFER_ERR              __ERR_STR(clCreateSubBuffer)
#define __CREATE_GL_BUFFER_ERR              __ERR_STR(clCreateFromGLBuffer)
#define __CREATE_GL_RENDER_BUFFER_ERR       __ERR_STR(clCreateFromGLBuffer)
#define __GET_GL_OBJECT_INFO_ERR            __ERR_STR(clGetGLObjectInfo)
#if defined(CL_VERSION_1_2)
#define __CREATE_IMAGE_ERR                  __ERR_STR(clCreateImage)
#define __CREATE_GL_TEXTURE_ERR             __ERR_STR(clCreateFromGLTexture)
#define __IMAGE_DIMENSION_ERR               __ERR_STR(Incorrect image dimensions)
#endif // #if defined(CL_VERSION_1_2)
#define __CREATE_SAMPLER_ERR                __ERR_STR(clCreateSampler)
#define __SET_MEM_OBJECT_DESTRUCTOR_CALLBACK_ERR __ERR_STR(clSetMemObjectDestructorCallback)

#define __CREATE_USER_EVENT_ERR             __ERR_STR(clCreateUserEvent)
#define __SET_USER_EVENT_STATUS_ERR         __ERR_STR(clSetUserEventStatus)
#define __SET_EVENT_CALLBACK_ERR            __ERR_STR(clSetEventCallback)
#define __WAIT_FOR_EVENTS_ERR               __ERR_STR(clWaitForEvents)

#define __CREATE_KERNEL_ERR                 __ERR_STR(clCreateKernel)
#define __SET_KERNEL_ARGS_ERR               __ERR_STR(clSetKernelArg)
#define __CREATE_PROGRAM_WITH_SOURCE_ERR    __ERR_STR(clCreateProgramWithSource)
#define __CREATE_PROGRAM_WITH_BINARY_ERR    __ERR_STR(clCreateProgramWithBinary)
#if defined(CL_VERSION_1_2)
#define __CREATE_PROGRAM_WITH_BUILT_IN_KERNELS_ERR    __ERR_STR(clCreateProgramWithBuiltInKernels)
#endif // #if defined(CL_VERSION_1_2)
#define __BUILD_PROGRAM_ERR                 __ERR_STR(clBuildProgram)
#if defined(CL_VERSION_1_2)
#define __COMPILE_PROGRAM_ERR                  __ERR_STR(clCompileProgram)

#endif // #if defined(CL_VERSION_1_2)
#define __CREATE_KERNELS_IN_PROGRAM_ERR     __ERR_STR(clCreateKernelsInProgram)

#define __CREATE_COMMAND_QUEUE_ERR          __ERR_STR(clCreateCommandQueue)
#define __SET_COMMAND_QUEUE_PROPERTY_ERR    __ERR_STR(clSetCommandQueueProperty)
#define __ENQUEUE_READ_BUFFER_ERR           __ERR_STR(clEnqueueReadBuffer)
#define __ENQUEUE_READ_BUFFER_RECT_ERR      __ERR_STR(clEnqueueReadBufferRect)
#define __ENQUEUE_WRITE_BUFFER_ERR          __ERR_STR(clEnqueueWriteBuffer)
#define __ENQUEUE_WRITE_BUFFER_RECT_ERR     __ERR_STR(clEnqueueWriteBufferRect)
#define __ENQEUE_COPY_BUFFER_ERR            __ERR_STR(clEnqueueCopyBuffer)
#define __ENQEUE_COPY_BUFFER_RECT_ERR       __ERR_STR(clEnqueueCopyBufferRect)
#define __ENQUEUE_FILL_BUFFER_ERR           __ERR_STR(clEnqueueFillBuffer)
#define __ENQUEUE_READ_IMAGE_ERR            __ERR_STR(clEnqueueReadImage)
#define __ENQUEUE_WRITE_IMAGE_ERR           __ERR_STR(clEnqueueWriteImage)
#define __ENQUEUE_COPY_IMAGE_ERR            __ERR_STR(clEnqueueCopyImage)
#define __ENQUEUE_FILL_IMAGE_ERR           __ERR_STR(clEnqueueFillImage)
#define __ENQUEUE_COPY_IMAGE_TO_BUFFER_ERR  __ERR_STR(clEnqueueCopyImageToBuffer)
#define __ENQUEUE_COPY_BUFFER_TO_IMAGE_ERR  __ERR_STR(clEnqueueCopyBufferToImage)
#define __ENQUEUE_MAP_BUFFER_ERR            __ERR_STR(clEnqueueMapBuffer)
#define __ENQUEUE_MAP_IMAGE_ERR             __ERR_STR(clEnqueueMapImage)
#define __ENQUEUE_UNMAP_MEM_OBJECT_ERR      __ERR_STR(clEnqueueUnMapMemObject)
#define __ENQUEUE_NDRANGE_KERNEL_ERR        __ERR_STR(clEnqueueNDRangeKernel)
#define __ENQUEUE_TASK_ERR                  __ERR_STR(clEnqueueTask)
#define __ENQUEUE_NATIVE_KERNEL             __ERR_STR(clEnqueueNativeKernel)
#if defined(CL_VERSION_1_2)
#define __ENQUEUE_MIGRATE_MEM_OBJECTS_ERR   __ERR_STR(clEnqueueMigrateMemObjects)
#endif // #if defined(CL_VERSION_1_2)

#define __ENQUEUE_ACQUIRE_GL_ERR            __ERR_STR(clEnqueueAcquireGLObjects)
#define __ENQUEUE_RELEASE_GL_ERR            __ERR_STR(clEnqueueReleaseGLObjects)


#define __RETAIN_ERR                        __ERR_STR(Retain Object)
#define __RELEASE_ERR                       __ERR_STR(Release Object)
#define __FLUSH_ERR                         __ERR_STR(clFlush)
#define __FINISH_ERR                        __ERR_STR(clFinish)
#define __VECTOR_CAPACITY_ERR               __ERR_STR(Vector capacity error)

/**
 * CL 1.2 version that uses device fission.
 */
#if defined(CL_VERSION_1_2)
#define __CREATE_SUB_DEVICES                __ERR_STR(clCreateSubDevices)
#else
#define __CREATE_SUB_DEVICES                __ERR_STR(clCreateSubDevicesEXT)
#endif // #if defined(CL_VERSION_1_2)

/**
 * Deprecated APIs for 1.2
 */
#if defined(CL_USE_DEPRECATED_OPENCL_1_1_APIS) || (defined(CL_VERSION_1_1) && !defined(CL_VERSION_1_2)) 
#define __ENQUEUE_MARKER_ERR                __ERR_STR(clEnqueueMarker)
#define __ENQUEUE_WAIT_FOR_EVENTS_ERR       __ERR_STR(clEnqueueWaitForEvents)
#define __ENQUEUE_BARRIER_ERR               __ERR_STR(clEnqueueBarrier)
#define __UNLOAD_COMPILER_ERR               __ERR_STR(clUnloadCompiler)
#define __CREATE_GL_TEXTURE_2D_ERR          __ERR_STR(clCreateFromGLTexture2D)
#define __CREATE_GL_TEXTURE_3D_ERR          __ERR_STR(clCreateFromGLTexture3D)
#define __CREATE_IMAGE2D_ERR                __ERR_STR(clCreateImage2D)
#define __CREATE_IMAGE3D_ERR                __ERR_STR(clCreateImage3D)
#endif // #if defined(CL_VERSION_1_1)

#endif // __CL_USER_OVERRIDE_ERROR_STRINGS
//! \endcond

/**
 * CL 1.2 marker and barrier commands
 */
#if defined(CL_VERSION_1_2)
#define __ENQUEUE_MARKER_WAIT_LIST_ERR                __ERR_STR(clEnqueueMarkerWithWaitList)
#define __ENQUEUE_BARRIER_WAIT_LIST_ERR               __ERR_STR(clEnqueueBarrierWithWaitList)
#endif // #if defined(CL_VERSION_1_2)

#if !defined(__USE_DEV_STRING) && !defined(__NO_STD_STRING)
typedef std::string STRING_CLASS;
#elif !defined(__USE_DEV_STRING) 

/*! \class string
 * \brief Simple string class, that provides a limited subset of std::string
 * functionality but avoids many of the issues that come with that class.
 
 *  \note Deprecated. Please use std::string as default or
 *  re-define the string class to match the std::string
 *  interface by defining STRING_CLASS
 */
class CL_EXT_PREFIX__VERSION_1_1_DEPRECATED string CL_EXT_SUFFIX__VERSION_1_1_DEPRECATED
{
private:
    ::size_t size_;
    char * str_;
public:
    //! \brief Constructs an empty string, allocating no memory.
    string(void) : size_(0), str_(NULL)
    {
    }

    /*! \brief Constructs a string populated from an arbitrary value of
     *  specified size.
     * 
     *  An extra '\0' is added, in case none was contained in str.
     *
     *  \param str the initial value of the string instance.  Note that '\0'     
     *             characters receive no special treatment.  If NULL,
     *             the string is left empty, with a size of 0.
     *
     *  \param size the number of characters to copy from str.
     */
    string(const char * str, ::size_t size) :
        size_(size),
        str_(NULL)
    {
        if( size > 0 ) {
            str_ = new char[size_+1];
            if (str_ != NULL) {
                memcpy(str_, str, size_  * sizeof(char));
                str_[size_] = '\0';
            }
            else {
                size_ = 0;
            }
        }
    }

    /*! \brief Constructs a string populated from a null-terminated value.
     *
     *  \param str the null-terminated initial value of the string instance.
     *             If NULL, the string is left empty, with a size of 0.
     */
    string(const char * str) :
        size_(0),
        str_(NULL)
    {
        if( str ) {
            size_= ::strlen(str);
        }
        if( size_ > 0 ) {
            str_ = new char[size_ + 1];
            if (str_ != NULL) {
                memcpy(str_, str, (size_ + 1) * sizeof(char));
            }
        }
    }

    void resize( ::size_t n )
    {
        if( size_ == n ) {
            return;
        }
        if (n == 0) {
            if( str_ ) {
                delete [] str_;
            }
            str_ = NULL;
            size_ = 0;
        } 
        else {
            char *newString = new char[n + 1];
            int copySize = n;
            if( size_ < n ) {
                copySize = size_;
            }
            size_ = n;
            
            if(str_) {
                memcpy(newString, str_, (copySize + 1) * sizeof(char));
            }
            if( copySize < size_ ) {
                memset(newString + copySize, 0, size_ - copySize);
            }
            newString[size_] = '\0';

            delete [] str_;
            str_ = newString;
        }
    }

    const char& operator[] ( ::size_t pos ) const
    {
        return str_[pos];
    }

    char& operator[] ( ::size_t pos )
    {
        return str_[pos];
    }

    /*! \brief Copies the value of another string to this one.
     *
     *  \param rhs the string to copy.
     *
     *  \returns a reference to the modified instance.
     */
    string& operator=(const string& rhs)
    {
        if (this == &rhs) {
            return *this;
        }

        if( str_ != NULL ) {
            delete [] str_;
            str_ = NULL;
            size_ = 0;
        }

        if (rhs.size_ == 0 || rhs.str_ == NULL) {
            str_ = NULL;
            size_ = 0;
        } 
        else {
            str_ = new char[rhs.size_ + 1];
            size_ = rhs.size_;
            
            if (str_ != NULL) {
                memcpy(str_, rhs.str_, (size_ + 1) * sizeof(char));
            }
            else {
                size_ = 0;
            }
        }

        return *this;
    }

    /*! \brief Constructs a string by copying the value of another instance.
     *
     *  \param rhs the string to copy.
     */
    string(const string& rhs) :
        size_(0),
        str_(NULL)
    {
        *this = rhs;
    }

    //! \brief Destructor - frees memory used to hold the current value.
    ~string()
    {
        delete[] str_;
        str_ = NULL;
    }
    
    //! \brief Queries the length of the string, excluding any added '\0's.
    ::size_t size(void) const   { return size_; }

    //! \brief Queries the length of the string, excluding any added '\0's.
    ::size_t length(void) const { return size(); }

    /*! \brief Returns a pointer to the private copy held by this instance,
     *  or "" if empty/unset.
     */
    const char * c_str(void) const { return (str_) ? str_ : "";}
};
typedef cl::string STRING_CLASS;
#endif // #elif !defined(__USE_DEV_STRING) 

#if !defined(__USE_DEV_VECTOR) && !defined(__NO_STD_VECTOR)
#define VECTOR_CLASS std::vector
#elif !defined(__USE_DEV_VECTOR) 
#define VECTOR_CLASS cl::vector 

#if !defined(__MAX_DEFAULT_VECTOR_SIZE)
#define __MAX_DEFAULT_VECTOR_SIZE 10
#endif

/*! \class vector
 * \brief Fixed sized vector implementation that mirroring 
 *
 *  \note Deprecated. Please use std::vector as default or
 *  re-define the vector class to match the std::vector
 *  interface by defining VECTOR_CLASS

 *  \note Not recommended for use with custom objects as
 *  current implementation will construct N elements
 *
 * std::vector functionality.
 *  \brief Fixed sized vector compatible with std::vector.
 *
 *  \note
 *  This differs from std::vector<> not just in memory allocation,
 *  but also in terms of when members are constructed, destroyed,
 *  and assigned instead of being copy constructed.
 *
 *  \param T type of element contained in the vector.
 *
 *  \param N maximum size of the vector.
 */
template <typename T, unsigned int N = __MAX_DEFAULT_VECTOR_SIZE>
class CL_EXT_PREFIX__VERSION_1_1_DEPRECATED vector CL_EXT_SUFFIX__VERSION_1_1_DEPRECATED
{
private:
    T data_[N];
    unsigned int size_;

public:
    //! \brief Constructs an empty vector with no memory allocated.
    vector() :  
        size_(static_cast<unsigned int>(0))
    {}

    //! \brief Deallocates the vector's memory and destroys all of its elements.
    ~vector() 
    {
        clear();
    }

    //! \brief Returns the number of elements currently contained.
    unsigned int size(void) const
    {
        return size_;
    }
    
    /*! \brief Empties the vector of all elements.
     *  \note
     *  This does not deallocate memory but will invoke destructors
     *  on contained elements.
     */
    void clear()
    {
        while(!empty()) {
            pop_back();
        }
    }

    /*! \brief Appends an element after the last valid element.
     * Calling this on a vector that has reached capacity will throw an 
     * exception if exceptions are enabled.
     */
    void push_back (const T& x)
    { 
        if (size() < N) {    
            new (&data_[size_]) T(x);
            size_++;
        } else {
            detail::errHandler(CL_MEM_OBJECT_ALLOCATION_FAILURE, __VECTOR_CAPACITY_ERR);
        }
    }

    /*! \brief Removes the last valid element from the vector.
     * Calling this on an empty vector will throw an exception
     * if exceptions are enabled.
     */
    void pop_back(void)
    {
        if (size_ != 0) {
            --size_;
            data_[size_].~T();
        } else {
            detail::errHandler(CL_MEM_OBJECT_ALLOCATION_FAILURE, __VECTOR_CAPACITY_ERR);
        }
    }
  
    /*! \brief Constructs with a value copied from another.
     *
     *  \param vec the vector to copy.
     */
    vector(const vector<T, N>& vec) : 
        size_(vec.size_)
    {
        if (size_ != 0) {	
            assign(vec.begin(), vec.end());
        }
    } 

    /*! \brief Constructs with a specified number of initial elements.
     *
     *  \param size number of initial elements.
     *
     *  \param val value of initial elements.
     */
    vector(unsigned int size, const T& val = T()) :
        size_(0)
    {
        for (unsigned int i = 0; i < size; i++) {
            push_back(val);
        }
    }

    /*! \brief Overwrites the current content with that copied from another
     *         instance.
     *
     *  \param rhs vector to copy.
     *
     *  \returns a reference to this.
     */
    vector<T, N>& operator=(const vector<T, N>& rhs)
    {
        if (this == &rhs) {
            return *this;
        }

        if (rhs.size_ != 0) {	
            assign(rhs.begin(), rhs.end());
        } else {
            clear();
        }
    
        return *this;
    }

    /*! \brief Tests equality against another instance.
     *
     *  \param vec the vector against which to compare.
     */
    bool operator==(vector<T,N> &vec)
    {
        if (size() != vec.size()) {
            return false;
        }

        for( unsigned int i = 0; i < size(); ++i ) {
            if( operator[](i) != vec[i] ) {
                return false;
            }
        }
        return true;
    }
  
    //! \brief Conversion operator to T*.
    operator T* ()             { return data_; }

    //! \brief Conversion operator to const T*.
    operator const T* () const { return data_; }
   
    //! \brief Tests whether this instance has any elements.
    bool empty (void) const
    {
        return size_==0;
    }
  
    //! \brief Returns the maximum number of elements this instance can hold.
    unsigned int max_size (void) const
    {
        return N;
    }

    //! \brief Returns the maximum number of elements this instance can hold.
    unsigned int capacity () const
    {
        return N;
    }

    /*! \brief Returns a reference to a given element.
     *
     *  \param index which element to access.     *
     *  \note
     *  The caller is responsible for ensuring index is >= 0 and < size().
     */
    T& operator[](int index)
    {
        return data_[index];
    }
  
    /*! \brief Returns a const reference to a given element.
     *
     *  \param index which element to access.
     *
     *  \note
     *  The caller is responsible for ensuring index is >= 0 and < size().
     */
    const T& operator[](int index) const
    {
        return data_[index];
    }
  
    /*! \brief Assigns elements of the vector based on a source iterator range.
     *
     *  \param start Beginning iterator of source range
     *  \param end Enditerator of source range
     *
     *  \note
     *  Will throw an exception if exceptions are enabled and size exceeded.
     */
    template<class I>
    void assign(I start, I end)
    {
        clear();   
        while(start != end) {
            push_back(*start);
            start++;
        }
    }

    /*! \class iterator
     * \brief Const iterator class for vectors
     */
    class iterator
    {
    private:
        const vector<T,N> *vec_;
        int index_;

        /**
         * Internal iterator constructor to capture reference
         * to the vector it iterates over rather than taking 
         * the vector by copy.
         */
        iterator (const vector<T,N> &vec, int index) :
            vec_(&vec)
        {            
            if( !vec.empty() ) {
                index_ = index;
            } else {
                index_ = -1;
            }
        }

    public:
        iterator(void) : 
            index_(-1),
            vec_(NULL)
        {
        }

        iterator(const iterator& rhs) :
            vec_(rhs.vec_),
            index_(rhs.index_)
        {
        }

        ~iterator(void) {}

        static iterator begin(const cl::vector<T,N> &vec)
        {
            iterator i(vec, 0);

            return i;
        }

        static iterator end(const cl::vector<T,N> &vec)
        {
            iterator i(vec, vec.size());

            return i;
        }
    
        bool operator==(iterator i)
        {
            return ((vec_ == i.vec_) && 
                    (index_ == i.index_));
        }

        bool operator!=(iterator i)
        {
            return (!(*this==i));
        }

        iterator& operator++()
        {
            ++index_;
            return *this;
        }

        iterator operator++(int)
        {
            iterator retVal(*this);
            ++index_;
            return retVal;
        }

        iterator& operator--()
        {
            --index_;
            return *this;
        }

        iterator operator--(int)
        {
            iterator retVal(*this);
            --index_;
            return retVal;
        }

        const T& operator *() const
        {
            return (*vec_)[index_];
        }
    };

    iterator begin(void)
    {
        return iterator::begin(*this);
    }

    iterator begin(void) const
    {
        return iterator::begin(*this);
    }

    iterator end(void)
    {
        return iterator::end(*this);
    }

    iterator end(void) const
    {
        return iterator::end(*this);
    }

    T& front(void)
    {
        return data_[0];
    }

    T& back(void)
    {
        return data_[size_];
    }

    const T& front(void) const
    {
        return data_[0];
    }

    const T& back(void) const
    {
        return data_[size_-1];
    }
};  
#endif // #if !defined(__USE_DEV_VECTOR) && !defined(__NO_STD_VECTOR)





namespace detail {
#define __DEFAULT_NOT_INITIALIZED 1 
#define __DEFAULT_BEING_INITIALIZED 2
#define __DEFAULT_INITIALIZED 4

    /*
     * Compare and exchange primitives are needed for handling of defaults
    */
    inline int compare_exchange(volatile int * dest, int exchange, int comparand)
    {
#ifdef _WIN32
        return (int)(InterlockedCompareExchange(
           (volatile long*)dest, 
           (long)exchange, 
           (long)comparand));
#elif defined(__APPLE__) || defined(__MACOSX)
		return OSAtomicOr32Orig((uint32_t)exchange, (volatile uint32_t*)dest);
#else // !_WIN32 || defined(__APPLE__) || defined(__MACOSX)
        return (__sync_val_compare_and_swap(
            dest, 
            comparand, 
            exchange));
#endif // !_WIN32
    }

    inline void fence() { _mm_mfence(); }
}; // namespace detail

    
/*! \brief class used to interface between C++ and
 *  OpenCL C calls that require arrays of size_t values, whose
 *  size is known statically.
 */
template <int N>
class size_t
{ 
private:
    ::size_t data_[N];

public:
    //! \brief Initialize size_t to all 0s
    size_t()
    {
        for( int i = 0; i < N; ++i ) {
            data_[i] = 0;
        }
    }

    ::size_t& operator[](int index)
    {
        return data_[index];
    }

    const ::size_t& operator[](int index) const
    {
        return data_[index];
    }

    //! \brief Conversion operator to T*.
    operator ::size_t* ()             { return data_; }

    //! \brief Conversion operator to const T*.
    operator const ::size_t* () const { return data_; }
};

namespace detail {

// Generic getInfoHelper. The final parameter is used to guide overload
// resolution: the actual parameter passed is an int, which makes this
// a worse conversion sequence than a specialization that declares the
// parameter as an int.
template<typename Functor, typename T>
inline cl_int getInfoHelper(Functor f, cl_uint name, T* param, long)
{
    return f(name, sizeof(T), param, NULL);
}

// Specialized getInfoHelper for VECTOR_CLASS params
template <typename Func, typename T>
inline cl_int getInfoHelper(Func f, cl_uint name, VECTOR_CLASS<T>* param, long)
{
    ::size_t required;
    cl_int err = f(name, 0, NULL, &required);
    if (err != CL_SUCCESS) {
        return err;
    }

    T* value = (T*) alloca(required);
    err = f(name, required, value, NULL);
    if (err != CL_SUCCESS) {
        return err;
    }

    param->assign(&value[0], &value[required/sizeof(T)]);
    return CL_SUCCESS;
}

/* Specialization for reference-counted types. This depends on the
 * existence of Wrapper<T>::cl_type, and none of the other types having the
 * cl_type member. Note that simplify specifying the parameter as Wrapper<T>
 * does not work, because when using a derived type (e.g. Context) the generic
 * template will provide a better match.
 */
template <typename Func, typename T>
inline cl_int getInfoHelper(Func f, cl_uint name, VECTOR_CLASS<T>* param, int, typename T::cl_type = 0)
{
    ::size_t required;
    cl_int err = f(name, 0, NULL, &required);
    if (err != CL_SUCCESS) {
        return err;
    }

    typename T::cl_type * value = (typename T::cl_type *) alloca(required);
    err = f(name, required, value, NULL);
    if (err != CL_SUCCESS) {
        return err;
    }

    ::size_t elements = required / sizeof(typename T::cl_type);
    param->assign(&value[0], &value[elements]);
    for (::size_t i = 0; i < elements; i++)
    {
        if (value[i] != NULL)
        {
            err = (*param)[i].retain();
            if (err != CL_SUCCESS) {
                return err;
            }
        }
    }
    return CL_SUCCESS;
}

// Specialized for getInfo<CL_PROGRAM_BINARIES>
template <typename Func>
inline cl_int getInfoHelper(Func f, cl_uint name, VECTOR_CLASS<char *>* param, int)
{
    cl_int err = f(name, param->size() * sizeof(char *), &(*param)[0], NULL);

    if (err != CL_SUCCESS) {
        return err;
    }

    return CL_SUCCESS;
}

// Specialized GetInfoHelper for STRING_CLASS params
template <typename Func>
inline cl_int getInfoHelper(Func f, cl_uint name, STRING_CLASS* param, long)
{
    ::size_t required;
    cl_int err = f(name, 0, NULL, &required);
    if (err != CL_SUCCESS) {
        return err;
    }

    char* value = (char*) alloca(required);
    err = f(name, required, value, NULL);
    if (err != CL_SUCCESS) {
        return err;
    }

    *param = value;
    return CL_SUCCESS;
}

// Specialized GetInfoHelper for cl::size_t params
template <typename Func, ::size_t N>
inline cl_int getInfoHelper(Func f, cl_uint name, size_t<N>* param, long)
{
    ::size_t required;
    cl_int err = f(name, 0, NULL, &required);
    if (err != CL_SUCCESS) {
        return err;
    }

    ::size_t* value = (::size_t*) alloca(required);
    err = f(name, required, value, NULL);
    if (err != CL_SUCCESS) {
        return err;
    }

    for(int i = 0; i < N; ++i) {
        (*param)[i] = value[i];
    }

    return CL_SUCCESS;
}

template<typename T> struct ReferenceHandler;

/* Specialization for reference-counted types. This depends on the
 * existence of Wrapper<T>::cl_type, and none of the other types having the
 * cl_type member. Note that simplify specifying the parameter as Wrapper<T>
 * does not work, because when using a derived type (e.g. Context) the generic
 * template will provide a better match.
 */
template<typename Func, typename T>
inline cl_int getInfoHelper(Func f, cl_uint name, T* param, int, typename T::cl_type = 0)
{
    typename T::cl_type value;
    cl_int err = f(name, sizeof(value), &value, NULL);
    if (err != CL_SUCCESS) {
        return err;
    }
    *param = value;
    if (value != NULL)
    {
        err = param->retain();
        if (err != CL_SUCCESS) {
            return err;
        }
    }
    return CL_SUCCESS;
}

#define __PARAM_NAME_INFO_1_0(F) \
    F(cl_platform_info, CL_PLATFORM_PROFILE, STRING_CLASS) \
    F(cl_platform_info, CL_PLATFORM_VERSION, STRING_CLASS) \
    F(cl_platform_info, CL_PLATFORM_NAME, STRING_CLASS) \
    F(cl_platform_info, CL_PLATFORM_VENDOR, STRING_CLASS) \
    F(cl_platform_info, CL_PLATFORM_EXTENSIONS, STRING_CLASS) \
    \
    F(cl_device_info, CL_DEVICE_TYPE, cl_device_type) \
    F(cl_device_info, CL_DEVICE_VENDOR_ID, cl_uint) \
    F(cl_device_info, CL_DEVICE_MAX_COMPUTE_UNITS, cl_uint) \
    F(cl_device_info, CL_DEVICE_MAX_WORK_ITEM_DIMENSIONS, cl_uint) \
    F(cl_device_info, CL_DEVICE_MAX_WORK_GROUP_SIZE, ::size_t) \
    F(cl_device_info, CL_DEVICE_MAX_WORK_ITEM_SIZES, VECTOR_CLASS< ::size_t>) \
    F(cl_device_info, CL_DEVICE_PREFERRED_VECTOR_WIDTH_CHAR, cl_uint) \
    F(cl_device_info, CL_DEVICE_PREFERRED_VECTOR_WIDTH_SHORT, cl_uint) \
    F(cl_device_info, CL_DEVICE_PREFERRED_VECTOR_WIDTH_INT, cl_uint) \
    F(cl_device_info, CL_DEVICE_PREFERRED_VECTOR_WIDTH_LONG, cl_uint) \
    F(cl_device_info, CL_DEVICE_PREFERRED_VECTOR_WIDTH_FLOAT, cl_uint) \
    F(cl_device_info, CL_DEVICE_PREFERRED_VECTOR_WIDTH_DOUBLE, cl_uint) \
    F(cl_device_info, CL_DEVICE_MAX_CLOCK_FREQUENCY, cl_uint) \
    F(cl_device_info, CL_DEVICE_ADDRESS_BITS, cl_uint) \
    F(cl_device_info, CL_DEVICE_MAX_READ_IMAGE_ARGS, cl_uint) \
    F(cl_device_info, CL_DEVICE_MAX_WRITE_IMAGE_ARGS, cl_uint) \
    F(cl_device_info, CL_DEVICE_MAX_MEM_ALLOC_SIZE, cl_ulong) \
    F(cl_device_info, CL_DEVICE_IMAGE2D_MAX_WIDTH, ::size_t) \
    F(cl_device_info, CL_DEVICE_IMAGE2D_MAX_HEIGHT, ::size_t) \
    F(cl_device_info, CL_DEVICE_IMAGE3D_MAX_WIDTH, ::size_t) \
    F(cl_device_info, CL_DEVICE_IMAGE3D_MAX_HEIGHT, ::size_t) \
    F(cl_device_info, CL_DEVICE_IMAGE3D_MAX_DEPTH, ::size_t) \
    F(cl_device_info, CL_DEVICE_IMAGE_SUPPORT, cl_bool) \
    F(cl_device_info, CL_DEVICE_MAX_PARAMETER_SIZE, ::size_t) \
    F(cl_device_info, CL_DEVICE_MAX_SAMPLERS, cl_uint) \
    F(cl_device_info, CL_DEVICE_MEM_BASE_ADDR_ALIGN, cl_uint) \
    F(cl_device_info, CL_DEVICE_MIN_DATA_TYPE_ALIGN_SIZE, cl_uint) \
    F(cl_device_info, CL_DEVICE_SINGLE_FP_CONFIG, cl_device_fp_config) \
    F(cl_device_info, CL_DEVICE_GLOBAL_MEM_CACHE_TYPE, cl_device_mem_cache_type) \
    F(cl_device_info, CL_DEVICE_GLOBAL_MEM_CACHELINE_SIZE, cl_uint)\
    F(cl_device_info, CL_DEVICE_GLOBAL_MEM_CACHE_SIZE, cl_ulong) \
    F(cl_device_info, CL_DEVICE_GLOBAL_MEM_SIZE, cl_ulong) \
    F(cl_device_info, CL_DEVICE_MAX_CONSTANT_BUFFER_SIZE, cl_ulong) \
    F(cl_device_info, CL_DEVICE_MAX_CONSTANT_ARGS, cl_uint) \
    F(cl_device_info, CL_DEVICE_LOCAL_MEM_TYPE, cl_device_local_mem_type) \
    F(cl_device_info, CL_DEVICE_LOCAL_MEM_SIZE, cl_ulong) \
    F(cl_device_info, CL_DEVICE_ERROR_CORRECTION_SUPPORT, cl_bool) \
    F(cl_device_info, CL_DEVICE_PROFILING_TIMER_RESOLUTION, ::size_t) \
    F(cl_device_info, CL_DEVICE_ENDIAN_LITTLE, cl_bool) \
    F(cl_device_info, CL_DEVICE_AVAILABLE, cl_bool) \
    F(cl_device_info, CL_DEVICE_COMPILER_AVAILABLE, cl_bool) \
    F(cl_device_info, CL_DEVICE_EXECUTION_CAPABILITIES, cl_device_exec_capabilities) \
    F(cl_device_info, CL_DEVICE_QUEUE_PROPERTIES, cl_command_queue_properties) \
    F(cl_device_info, CL_DEVICE_PLATFORM, cl_platform_id) \
    F(cl_device_info, CL_DEVICE_NAME, STRING_CLASS) \
    F(cl_device_info, CL_DEVICE_VENDOR, STRING_CLASS) \
    F(cl_device_info, CL_DRIVER_VERSION, STRING_CLASS) \
    F(cl_device_info, CL_DEVICE_PROFILE, STRING_CLASS) \
    F(cl_device_info, CL_DEVICE_VERSION, STRING_CLASS) \
    F(cl_device_info, CL_DEVICE_EXTENSIONS, STRING_CLASS) \
    \
    F(cl_context_info, CL_CONTEXT_REFERENCE_COUNT, cl_uint) \
    F(cl_context_info, CL_CONTEXT_DEVICES, VECTOR_CLASS<Device>) \
    F(cl_context_info, CL_CONTEXT_PROPERTIES, VECTOR_CLASS<cl_context_properties>) \
    \
    F(cl_event_info, CL_EVENT_COMMAND_QUEUE, cl::CommandQueue) \
    F(cl_event_info, CL_EVENT_COMMAND_TYPE, cl_command_type) \
    F(cl_event_info, CL_EVENT_REFERENCE_COUNT, cl_uint) \
    F(cl_event_info, CL_EVENT_COMMAND_EXECUTION_STATUS, cl_uint) \
    \
    F(cl_profiling_info, CL_PROFILING_COMMAND_QUEUED, cl_ulong) \
    F(cl_profiling_info, CL_PROFILING_COMMAND_SUBMIT, cl_ulong) \
    F(cl_profiling_info, CL_PROFILING_COMMAND_START, cl_ulong) \
    F(cl_profiling_info, CL_PROFILING_COMMAND_END, cl_ulong) \
    \
    F(cl_mem_info, CL_MEM_TYPE, cl_mem_object_type) \
    F(cl_mem_info, CL_MEM_FLAGS, cl_mem_flags) \
    F(cl_mem_info, CL_MEM_SIZE, ::size_t) \
    F(cl_mem_info, CL_MEM_HOST_PTR, void*) \
    F(cl_mem_info, CL_MEM_MAP_COUNT, cl_uint) \
    F(cl_mem_info, CL_MEM_REFERENCE_COUNT, cl_uint) \
    F(cl_mem_info, CL_MEM_CONTEXT, cl::Context) \
    \
    F(cl_image_info, CL_IMAGE_FORMAT, cl_image_format) \
    F(cl_image_info, CL_IMAGE_ELEMENT_SIZE, ::size_t) \
    F(cl_image_info, CL_IMAGE_ROW_PITCH, ::size_t) \
    F(cl_image_info, CL_IMAGE_SLICE_PITCH, ::size_t) \
    F(cl_image_info, CL_IMAGE_WIDTH, ::size_t) \
    F(cl_image_info, CL_IMAGE_HEIGHT, ::size_t) \
    F(cl_image_info, CL_IMAGE_DEPTH, ::size_t) \
    \
    F(cl_sampler_info, CL_SAMPLER_REFERENCE_COUNT, cl_uint) \
    F(cl_sampler_info, CL_SAMPLER_CONTEXT, cl::Context) \
    F(cl_sampler_info, CL_SAMPLER_NORMALIZED_COORDS, cl_addressing_mode) \
    F(cl_sampler_info, CL_SAMPLER_ADDRESSING_MODE, cl_filter_mode) \
    F(cl_sampler_info, CL_SAMPLER_FILTER_MODE, cl_bool) \
    \
    F(cl_program_info, CL_PROGRAM_REFERENCE_COUNT, cl_uint) \
    F(cl_program_info, CL_PROGRAM_CONTEXT, cl::Context) \
    F(cl_program_info, CL_PROGRAM_NUM_DEVICES, cl_uint) \
    F(cl_program_info, CL_PROGRAM_DEVICES, VECTOR_CLASS<Device>) \
    F(cl_program_info, CL_PROGRAM_SOURCE, STRING_CLASS) \
    F(cl_program_info, CL_PROGRAM_BINARY_SIZES, VECTOR_CLASS< ::size_t>) \
    F(cl_program_info, CL_PROGRAM_BINARIES, VECTOR_CLASS<char *>) \
    \
    F(cl_program_build_info, CL_PROGRAM_BUILD_STATUS, cl_build_status) \
    F(cl_program_build_info, CL_PROGRAM_BUILD_OPTIONS, STRING_CLASS) \
    F(cl_program_build_info, CL_PROGRAM_BUILD_LOG, STRING_CLASS) \
    \
    F(cl_kernel_info, CL_KERNEL_FUNCTION_NAME, STRING_CLASS) \
    F(cl_kernel_info, CL_KERNEL_NUM_ARGS, cl_uint) \
    F(cl_kernel_info, CL_KERNEL_REFERENCE_COUNT, cl_uint) \
    F(cl_kernel_info, CL_KERNEL_CONTEXT, cl::Context) \
    F(cl_kernel_info, CL_KERNEL_PROGRAM, cl::Program) \
    \
    F(cl_kernel_work_group_info, CL_KERNEL_WORK_GROUP_SIZE, ::size_t) \
    F(cl_kernel_work_group_info, CL_KERNEL_COMPILE_WORK_GROUP_SIZE, cl::size_t<3>) \
    F(cl_kernel_work_group_info, CL_KERNEL_LOCAL_MEM_SIZE, cl_ulong) \
    \
    F(cl_command_queue_info, CL_QUEUE_CONTEXT, cl::Context) \
    F(cl_command_queue_info, CL_QUEUE_DEVICE, cl::Device) \
    F(cl_command_queue_info, CL_QUEUE_REFERENCE_COUNT, cl_uint) \
    F(cl_command_queue_info, CL_QUEUE_PROPERTIES, cl_command_queue_properties)

#if defined(CL_VERSION_1_1)
#define __PARAM_NAME_INFO_1_1(F) \
    F(cl_context_info, CL_CONTEXT_NUM_DEVICES, cl_uint)\
    F(cl_device_info, CL_DEVICE_PREFERRED_VECTOR_WIDTH_HALF, cl_uint) \
    F(cl_device_info, CL_DEVICE_NATIVE_VECTOR_WIDTH_CHAR, cl_uint) \
    F(cl_device_info, CL_DEVICE_NATIVE_VECTOR_WIDTH_SHORT, cl_uint) \
    F(cl_device_info, CL_DEVICE_NATIVE_VECTOR_WIDTH_INT, cl_uint) \
    F(cl_device_info, CL_DEVICE_NATIVE_VECTOR_WIDTH_LONG, cl_uint) \
    F(cl_device_info, CL_DEVICE_NATIVE_VECTOR_WIDTH_FLOAT, cl_uint) \
    F(cl_device_info, CL_DEVICE_NATIVE_VECTOR_WIDTH_DOUBLE, cl_uint) \
    F(cl_device_info, CL_DEVICE_NATIVE_VECTOR_WIDTH_HALF, cl_uint) \
    F(cl_device_info, CL_DEVICE_DOUBLE_FP_CONFIG, cl_device_fp_config) \
    F(cl_device_info, CL_DEVICE_HALF_FP_CONFIG, cl_device_fp_config) \
    F(cl_device_info, CL_DEVICE_HOST_UNIFIED_MEMORY, cl_bool) \
    F(cl_device_info, CL_DEVICE_OPENCL_C_VERSION, STRING_CLASS) \
    \
    F(cl_mem_info, CL_MEM_ASSOCIATED_MEMOBJECT, cl::Memory) \
    F(cl_mem_info, CL_MEM_OFFSET, ::size_t) \
    \
    F(cl_kernel_work_group_info, CL_KERNEL_PREFERRED_WORK_GROUP_SIZE_MULTIPLE, ::size_t) \
    F(cl_kernel_work_group_info, CL_KERNEL_PRIVATE_MEM_SIZE, cl_ulong) \
    \
    F(cl_event_info, CL_EVENT_CONTEXT, cl::Context)
#endif // CL_VERSION_1_1

    
#if defined(CL_VERSION_1_2)
#define __PARAM_NAME_INFO_1_2(F) \
    F(cl_image_info, CL_IMAGE_BUFFER, cl::Buffer) \
    \
    F(cl_program_info, CL_PROGRAM_NUM_KERNELS, ::size_t) \
    F(cl_program_info, CL_PROGRAM_KERNEL_NAMES, STRING_CLASS) \
    \
    F(cl_program_build_info, CL_PROGRAM_BINARY_TYPE, cl_program_binary_type) \
    \
    F(cl_kernel_info, CL_KERNEL_ATTRIBUTES, STRING_CLASS) \
    \
    F(cl_kernel_arg_info, CL_KERNEL_ARG_ADDRESS_QUALIFIER, cl_kernel_arg_address_qualifier) \
    F(cl_kernel_arg_info, CL_KERNEL_ARG_ACCESS_QUALIFIER, cl_kernel_arg_access_qualifier) \
    F(cl_kernel_arg_info, CL_KERNEL_ARG_TYPE_NAME, STRING_CLASS) \
    F(cl_kernel_arg_info, CL_KERNEL_ARG_NAME, STRING_CLASS) \
    \
    F(cl_device_info, CL_DEVICE_PARENT_DEVICE, cl_device_id) \
    F(cl_device_info, CL_DEVICE_PARTITION_PROPERTIES, VECTOR_CLASS<cl_device_partition_property>) \
    F(cl_device_info, CL_DEVICE_PARTITION_TYPE, VECTOR_CLASS<cl_device_partition_property>)  \
    F(cl_device_info, CL_DEVICE_REFERENCE_COUNT, cl_uint) \
    F(cl_device_info, CL_DEVICE_PREFERRED_INTEROP_USER_SYNC, ::size_t) \
    F(cl_device_info, CL_DEVICE_PARTITION_AFFINITY_DOMAIN, cl_device_affinity_domain) \
    F(cl_device_info, CL_DEVICE_BUILT_IN_KERNELS, STRING_CLASS)
#endif // #if defined(CL_VERSION_1_2)

#if defined(USE_CL_DEVICE_FISSION)
#define __PARAM_NAME_DEVICE_FISSION(F) \
    F(cl_device_info, CL_DEVICE_PARENT_DEVICE_EXT, cl_device_id) \
    F(cl_device_info, CL_DEVICE_PARTITION_TYPES_EXT, VECTOR_CLASS<cl_device_partition_property_ext>) \
    F(cl_device_info, CL_DEVICE_AFFINITY_DOMAINS_EXT, VECTOR_CLASS<cl_device_partition_property_ext>) \
    F(cl_device_info, CL_DEVICE_REFERENCE_COUNT_EXT , cl_uint) \
    F(cl_device_info, CL_DEVICE_PARTITION_STYLE_EXT, VECTOR_CLASS<cl_device_partition_property_ext>)
#endif // USE_CL_DEVICE_FISSION

template <typename enum_type, cl_int Name>
struct param_traits {};

#define __CL_DECLARE_PARAM_TRAITS(token, param_name, T) \
struct token;                                        \
template<>                                           \
struct param_traits<detail:: token,param_name>       \
{                                                    \
    enum { value = param_name };                     \
    typedef T param_type;                            \
};

__PARAM_NAME_INFO_1_0(__CL_DECLARE_PARAM_TRAITS)
#if defined(CL_VERSION_1_1)
__PARAM_NAME_INFO_1_1(__CL_DECLARE_PARAM_TRAITS)
#endif // CL_VERSION_1_1
#if defined(CL_VERSION_1_2)
__PARAM_NAME_INFO_1_2(__CL_DECLARE_PARAM_TRAITS)
#endif // CL_VERSION_1_1

#if defined(USE_CL_DEVICE_FISSION)
__PARAM_NAME_DEVICE_FISSION(__CL_DECLARE_PARAM_TRAITS);
#endif // USE_CL_DEVICE_FISSION

#ifdef CL_PLATFORM_ICD_SUFFIX_KHR
__CL_DECLARE_PARAM_TRAITS(cl_platform_info, CL_PLATFORM_ICD_SUFFIX_KHR, STRING_CLASS)
#endif

#ifdef CL_DEVICE_PROFILING_TIMER_OFFSET_AMD
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_PROFILING_TIMER_OFFSET_AMD, cl_ulong)
#endif

#ifdef CL_DEVICE_GLOBAL_FREE_MEMORY_AMD
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_GLOBAL_FREE_MEMORY_AMD, VECTOR_CLASS< ::size_t>)
#endif
#ifdef CL_DEVICE_SIMD_PER_COMPUTE_UNIT_AMD
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_SIMD_PER_COMPUTE_UNIT_AMD, cl_uint)
#endif
#ifdef CL_DEVICE_SIMD_WIDTH_AMD
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_SIMD_WIDTH_AMD, cl_uint)
#endif
#ifdef CL_DEVICE_SIMD_INSTRUCTION_WIDTH_AMD
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_SIMD_INSTRUCTION_WIDTH_AMD, cl_uint)
#endif
#ifdef CL_DEVICE_WAVEFRONT_WIDTH_AMD
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_WAVEFRONT_WIDTH_AMD, cl_uint)
#endif
#ifdef CL_DEVICE_GLOBAL_MEM_CHANNELS_AMD
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_GLOBAL_MEM_CHANNELS_AMD, cl_uint)
#endif
#ifdef CL_DEVICE_GLOBAL_MEM_CHANNEL_BANKS_AMD
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_GLOBAL_MEM_CHANNEL_BANKS_AMD, cl_uint)
#endif
#ifdef CL_DEVICE_GLOBAL_MEM_CHANNEL_BANK_WIDTH_AMD
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_GLOBAL_MEM_CHANNEL_BANK_WIDTH_AMD, cl_uint)
#endif
#ifdef CL_DEVICE_LOCAL_MEM_SIZE_PER_COMPUTE_UNIT_AMD
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_LOCAL_MEM_SIZE_PER_COMPUTE_UNIT_AMD, cl_uint)
#endif
#ifdef CL_DEVICE_LOCAL_MEM_BANKS_AMD
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_LOCAL_MEM_BANKS_AMD, cl_uint)
#endif

#ifdef CL_DEVICE_COMPUTE_CAPABILITY_MAJOR_NV
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_COMPUTE_CAPABILITY_MAJOR_NV, cl_uint)
#endif
#ifdef CL_DEVICE_COMPUTE_CAPABILITY_MINOR_NV
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_COMPUTE_CAPABILITY_MINOR_NV, cl_uint)
#endif
#ifdef CL_DEVICE_REGISTERS_PER_BLOCK_NV
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_REGISTERS_PER_BLOCK_NV, cl_uint)
#endif
#ifdef CL_DEVICE_WARP_SIZE_NV
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_WARP_SIZE_NV, cl_uint)
#endif
#ifdef CL_DEVICE_GPU_OVERLAP_NV
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_GPU_OVERLAP_NV, cl_bool)
#endif
#ifdef CL_DEVICE_KERNEL_EXEC_TIMEOUT_NV
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_KERNEL_EXEC_TIMEOUT_NV, cl_bool)
#endif
#ifdef CL_DEVICE_INTEGRATED_MEMORY_NV
__CL_DECLARE_PARAM_TRAITS(cl_device_info, CL_DEVICE_INTEGRATED_MEMORY_NV, cl_bool)
#endif

// Convenience functions

template <typename Func, typename T>
inline cl_int
getInfo(Func f, cl_uint name, T* param)
{
    return getInfoHelper(f, name, param, 0);
}

template <typename Func, typename Arg0>
struct GetInfoFunctor0
{
    Func f_; const Arg0& arg0_;
    cl_int operator ()(
        cl_uint param, ::size_t size, void* value, ::size_t* size_ret)
    { return f_(arg0_, param, size, value, size_ret); }
};

template <typename Func, typename Arg0, typename Arg1>
struct GetInfoFunctor1
{
    Func f_; const Arg0& arg0_; const Arg1& arg1_;
    cl_int operator ()(
        cl_uint param, ::size_t size, void* value, ::size_t* size_ret)
    { return f_(arg0_, arg1_, param, size, value, size_ret); }
};

template <typename Func, typename Arg0, typename T>
inline cl_int
getInfo(Func f, const Arg0& arg0, cl_uint name, T* param)
{
    GetInfoFunctor0<Func, Arg0> f0 = { f, arg0 };
    return getInfoHelper(f0, name, param, 0);
}

template <typename Func, typename Arg0, typename Arg1, typename T>
inline cl_int
getInfo(Func f, const Arg0& arg0, const Arg1& arg1, cl_uint name, T* param)
{
    GetInfoFunctor1<Func, Arg0, Arg1> f0 = { f, arg0, arg1 };
    return getInfoHelper(f0, name, param, 0);
}

template<typename T>
struct ReferenceHandler
{ };

#if defined(CL_VERSION_1_2)
/**
 * OpenCL 1.2 devices do have retain/release.
 */
template <>
struct ReferenceHandler<cl_device_id>
{
    /**
     * Retain the device.
     * \param device A valid device created using createSubDevices
     * \return 
     *   CL_SUCCESS if the function executed successfully.
     *   CL_INVALID_DEVICE if device was not a valid subdevice
     *   CL_OUT_OF_RESOURCES
     *   CL_OUT_OF_HOST_MEMORY
     */
    static cl_int retain(cl_device_id device)
    { return ::clRetainDevice(device); }
    /**
     * Retain the device.
     * \param device A valid device created using createSubDevices
     * \return 
     *   CL_SUCCESS if the function executed successfully.
     *   CL_INVALID_DEVICE if device was not a valid subdevice
     *   CL_OUT_OF_RESOURCES
     *   CL_OUT_OF_HOST_MEMORY
     */
    static cl_int release(cl_device_id device)
    { return ::clReleaseDevice(device); }
};
#else // #if defined(CL_VERSION_1_2)
/**
 * OpenCL 1.1 devices do not have retain/release.
 */
template <>
struct ReferenceHandler<cl_device_id>
{
    // cl_device_id does not have retain().
    static cl_int retain(cl_device_id)
    { return CL_SUCCESS; }
    // cl_device_id does not have release().
    static cl_int release(cl_device_id)
    { return CL_SUCCESS; }
};
#endif // #if defined(CL_VERSION_1_2)

template <>
struct ReferenceHandler<cl_platform_id>
{
    // cl_platform_id does not have retain().
    static cl_int retain(cl_platform_id)
    { return CL_SUCCESS; }
    // cl_platform_id does not have release().
    static cl_int release(cl_platform_id)
    { return CL_SUCCESS; }
};

template <>
struct ReferenceHandler<cl_context>
{
    static cl_int retain(cl_context context)
    { return ::clRetainContext(context); }
    static cl_int release(cl_context context)
    { return ::clReleaseContext(context); }
};

template <>
struct ReferenceHandler<cl_command_queue>
{
    static cl_int retain(cl_command_queue queue)
    { return ::clRetainCommandQueue(queue); }
    static cl_int release(cl_command_queue queue)
    { return ::clReleaseCommandQueue(queue); }
};

template <>
struct ReferenceHandler<cl_mem>
{
    static cl_int retain(cl_mem memory)
    { return ::clRetainMemObject(memory); }
    static cl_int release(cl_mem memory)
    { return ::clReleaseMemObject(memory); }
};

template <>
struct ReferenceHandler<cl_sampler>
{
    static cl_int retain(cl_sampler sampler)
    { return ::clRetainSampler(sampler); }
    static cl_int release(cl_sampler sampler)
    { return ::clReleaseSampler(sampler); }
};

template <>
struct ReferenceHandler<cl_program>
{
    static cl_int retain(cl_program program)
    { return ::clRetainProgram(program); }
    static cl_int release(cl_program program)
    { return ::clReleaseProgram(program); }
};

template <>
struct ReferenceHandler<cl_kernel>
{
    static cl_int retain(cl_kernel kernel)
    { return ::clRetainKernel(kernel); }
    static cl_int release(cl_kernel kernel)
    { return ::clReleaseKernel(kernel); }
};

template <>
struct ReferenceHandler<cl_event>
{
    static cl_int retain(cl_event event)
    { return ::clRetainEvent(event); }
    static cl_int release(cl_event event)
    { return ::clReleaseEvent(event); }
};


// Extracts version number with major in the upper 16 bits, minor in the lower 16
static cl_uint getVersion(const char *versionInfo)
{
    int highVersion = 0;
    int lowVersion = 0;
    int index = 7;
    while(versionInfo[index] != '.' ) {
        highVersion *= 10;
        highVersion += versionInfo[index]-'0';
        ++index;
    }
    ++index;
    while(versionInfo[index] != ' ' ) {
        lowVersion *= 10;
        lowVersion += versionInfo[index]-'0';
        ++index;
    }
    return (highVersion << 16) | lowVersion;
}

static cl_uint getPlatformVersion(cl_platform_id platform)
{
    ::size_t size = 0;
    clGetPlatformInfo(platform, CL_PLATFORM_VERSION, 0, NULL, &size);
    char *versionInfo = (char *) alloca(size);
    clGetPlatformInfo(platform, CL_PLATFORM_VERSION, size, &versionInfo[0], &size);
    return getVersion(versionInfo);
}

static cl_uint getDevicePlatformVersion(cl_device_id device)
{
    cl_platform_id platform;
    clGetDeviceInfo(device, CL_DEVICE_PLATFORM, sizeof(platform), &platform, NULL);
    return getPlatformVersion(platform);
}

#if defined(CL_VERSION_1_2) && defined(CL_USE_DEPRECATED_OPENCL_1_1_APIS)
static cl_uint getContextPlatformVersion(cl_context context)
{
    // The platform cannot be queried directly, so we first have to grab a
    // device and obtain its context
    ::size_t size = 0;
    clGetContextInfo(context, CL_CONTEXT_DEVICES, 0, NULL, &size);
    if (size == 0)
        return 0;
    cl_device_id *devices = (cl_device_id *) alloca(size);
    clGetContextInfo(context, CL_CONTEXT_DEVICES, size, devices, NULL);
    return getDevicePlatformVersion(devices[0]);
}
#endif // #if defined(CL_VERSION_1_2) && defined(CL_USE_DEPRECATED_OPENCL_1_1_APIS)

template <typename T>
class Wrapper
{
public:
    typedef T cl_type;

protected:
    cl_type object_;

public:
    Wrapper() : object_(NULL) { }

    Wrapper(const cl_type &obj) : object_(obj) { }

    ~Wrapper()
    {
        if (object_ != NULL) { release(); }
    }

    Wrapper(const Wrapper<cl_type>& rhs)
    {
        object_ = rhs.object_;
        if (object_ != NULL) { detail::errHandler(retain(), __RETAIN_ERR); }
    }

    Wrapper<cl_type>& operator = (const Wrapper<cl_type>& rhs)
    {
        if (object_ != NULL) { detail::errHandler(release(), __RELEASE_ERR); }
        object_ = rhs.object_;
        if (object_ != NULL) { detail::errHandler(retain(), __RETAIN_ERR); }
        return *this;
    }

    Wrapper<cl_type>& operator = (const cl_type &rhs)
    {
        if (object_ != NULL) { detail::errHandler(release(), __RELEASE_ERR); }
        object_ = rhs;
        return *this;
    }

    cl_type operator ()() const { return object_; }

    cl_type& operator ()() { return object_; }

protected:
    template<typename Func, typename U>
    friend inline cl_int getInfoHelper(Func, cl_uint, U*, int, typename U::cl_type);

    cl_int retain() const
    {
        return ReferenceHandler<cl_type>::retain(object_);
    }

    cl_int release() const
    {
        return ReferenceHandler<cl_type>::release(object_);
    }
};

template <>
class Wrapper<cl_device_id>
{
public:
    typedef cl_device_id cl_type;

protected:
    cl_type object_;
    bool referenceCountable_;

    static bool isReferenceCountable(cl_device_id device)
    {
        bool retVal = false;
        if (device != NULL) {
            int version = getDevicePlatformVersion(device);
            if(version > ((1 << 16) + 1)) {
                retVal = true;
            }
        }
        return retVal;
    }

public:
    Wrapper() : object_(NULL), referenceCountable_(false) 
    { 
    }
    
    Wrapper(const cl_type &obj) : object_(obj), referenceCountable_(false) 
    {
        referenceCountable_ = isReferenceCountable(obj); 
    }

    ~Wrapper()
    {
        if (object_ != NULL) { release(); }
    }
    
    Wrapper(const Wrapper<cl_type>& rhs)
    {
        object_ = rhs.object_;
        referenceCountable_ = isReferenceCountable(object_); 
        if (object_ != NULL) { detail::errHandler(retain(), __RETAIN_ERR); }
    }

    Wrapper<cl_type>& operator = (const Wrapper<cl_type>& rhs)
    {
        if (object_ != NULL) { detail::errHandler(release(), __RELEASE_ERR); }
        object_ = rhs.object_;
        referenceCountable_ = rhs.referenceCountable_;
        if (object_ != NULL) { detail::errHandler(retain(), __RETAIN_ERR); }
        return *this;
    }

    Wrapper<cl_type>& operator = (const cl_type &rhs)
    {
        if (object_ != NULL) { detail::errHandler(release(), __RELEASE_ERR); }
        object_ = rhs;
        referenceCountable_ = isReferenceCountable(object_); 
        return *this;
    }

    cl_type operator ()() const { return object_; }

    cl_type& operator ()() { return object_; }

protected:
    template<typename Func, typename U>
    friend inline cl_int getInfoHelper(Func, cl_uint, U*, int, typename U::cl_type);

    template<typename Func, typename U>
    friend inline cl_int getInfoHelper(Func, cl_uint, VECTOR_CLASS<U>*, int, typename U::cl_type);

    cl_int retain() const
    {
        if( referenceCountable_ ) {
            return ReferenceHandler<cl_type>::retain(object_);
        }
        else {
            return CL_SUCCESS;
        }
    }

    cl_int release() const
    {
        if( referenceCountable_ ) {
            return ReferenceHandler<cl_type>::release(object_);
        }
        else {
            return CL_SUCCESS;
        }
    }
};

} // namespace detail
//! \endcond

/*! \stuct ImageFormat
 *  \brief Adds constructors and member functions for cl_image_format.
 *
 *  \see cl_image_format
 */
struct ImageFormat : public cl_image_format
{
    //! \brief Default constructor - performs no initialization.
    ImageFormat(){}

    //! \brief Initializing constructor.
    ImageFormat(cl_channel_order order, cl_channel_type type)
    {
        image_channel_order = order;
        image_channel_data_type = type;
    }

    //! \brief Assignment operator.
    ImageFormat& operator = (const ImageFormat& rhs)
    {
        if (this != &rhs) {
            this->image_channel_data_type = rhs.image_channel_data_type;
            this->image_channel_order     = rhs.image_channel_order;
        }
        return *this;
    }
};

/*! \brief Class interface for cl_device_id.
 *
 *  \note Copies of these objects are inexpensive, since they don't 'own'
 *        any underlying resources or data structures.
 *
 *  \see cl_device_id
 */
class Device : public detail::Wrapper<cl_device_id>
{
public:
    //! \brief Default constructor - initializes to NULL.
    Device() : detail::Wrapper<cl_type>() { }

    /*! \brief Copy constructor.
     * 
     *  This simply copies the device ID value, which is an inexpensive operation.
     */
    Device(const Device& device) : detail::Wrapper<cl_type>(device) { }

    /*! \brief Constructor from cl_device_id.
     * 
     *  This simply copies the device ID value, which is an inexpensive operation.
     */
    Device(const cl_device_id &device) : detail::Wrapper<cl_type>(device) { }

    /*! \brief Returns the first device on the default context.
     *
     *  \see Context::getDefault()
     */
    static Device getDefault(cl_int * err = NULL);

    /*! \brief Assignment operator from Device.
     * 
     *  This simply copies the device ID value, which is an inexpensive operation.
     */
    Device& operator = (const Device& rhs)
    {
        if (this != &rhs) {
            detail::Wrapper<cl_type>::operator=(rhs);
        }
        return *this;
    }

    /*! \brief Assignment operator from cl_device_id.
     * 
     *  This simply copies the device ID value, which is an inexpensive operation.
     */
    Device& operator = (const cl_device_id& rhs)
    {
        detail::Wrapper<cl_type>::operator=(rhs);
        return *this;
    }

    //! \brief Wrapper for clGetDeviceInfo().
    template <typename T>
    cl_int getInfo(cl_device_info name, T* param) const
    {
        return detail::errHandler(
            detail::getInfo(&::clGetDeviceInfo, object_, name, param),
            __GET_DEVICE_INFO_ERR);
    }

    //! \brief Wrapper for clGetDeviceInfo() that returns by value.
    template <cl_int name> typename
    detail::param_traits<detail::cl_device_info, name>::param_type
    getInfo(cl_int* err = NULL) const
    {
        typename detail::param_traits<
            detail::cl_device_info, name>::param_type param;
        cl_int result = getInfo(name, &param);
        if (err != NULL) {
            *err = result;
        }
        return param;
    }

    /**
     * CL 1.2 version
     */
#if defined(CL_VERSION_1_2)
    //! \brief Wrapper for clCreateSubDevicesEXT().
    cl_int createSubDevices(
        const cl_device_partition_property * properties,
        VECTOR_CLASS<Device>* devices)
    {
        cl_uint n = 0;
        cl_int err = clCreateSubDevices(object_, properties, 0, NULL, &n);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __CREATE_SUB_DEVICES);
        }

        cl_device_id* ids = (cl_device_id*) alloca(n * sizeof(cl_device_id));
        err = clCreateSubDevices(object_, properties, n, ids, NULL);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __CREATE_SUB_DEVICES);
        }

        devices->assign(&ids[0], &ids[n]);
        return CL_SUCCESS;
    }
#endif // #if defined(CL_VERSION_1_2)

/**
 * CL 1.1 version that uses device fission.
 */
#if defined(CL_VERSION_1_1)
#if defined(USE_CL_DEVICE_FISSION)
    cl_int createSubDevices(
        const cl_device_partition_property_ext * properties,
        VECTOR_CLASS<Device>* devices)
    {
        typedef CL_API_ENTRY cl_int 
            ( CL_API_CALL * PFN_clCreateSubDevicesEXT)(
                cl_device_id /*in_device*/,
                const cl_device_partition_property_ext * /* properties */,
                cl_uint /*num_entries*/,
                cl_device_id * /*out_devices*/,
                cl_uint * /*num_devices*/ ) CL_EXT_SUFFIX__VERSION_1_1;

        static PFN_clCreateSubDevicesEXT pfn_clCreateSubDevicesEXT = NULL;
        __INIT_CL_EXT_FCN_PTR(clCreateSubDevicesEXT);

        cl_uint n = 0;
        cl_int err = pfn_clCreateSubDevicesEXT(object_, properties, 0, NULL, &n);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __CREATE_SUB_DEVICES);
        }

        cl_device_id* ids = (cl_device_id*) alloca(n * sizeof(cl_device_id));
        err = pfn_clCreateSubDevicesEXT(object_, properties, n, ids, NULL);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __CREATE_SUB_DEVICES);
        }

        devices->assign(&ids[0], &ids[n]);
        return CL_SUCCESS;
    }
#endif // #if defined(USE_CL_DEVICE_FISSION)
#endif // #if defined(CL_VERSION_1_1)
};

/*! \brief Class interface for cl_platform_id.
 *
 *  \note Copies of these objects are inexpensive, since they don't 'own'
 *        any underlying resources or data structures.
 *
 *  \see cl_platform_id
 */
class Platform : public detail::Wrapper<cl_platform_id>
{
public:
    //! \brief Default constructor - initializes to NULL.
    Platform() : detail::Wrapper<cl_type>()  { }

    /*! \brief Copy constructor.
     * 
     *  This simply copies the platform ID value, which is an inexpensive operation.
     */
    Platform(const Platform& platform) : detail::Wrapper<cl_type>(platform) { }

    /*! \brief Constructor from cl_platform_id.
     * 
     *  This simply copies the platform ID value, which is an inexpensive operation.
     */
    Platform(const cl_platform_id &platform) : detail::Wrapper<cl_type>(platform) { }

    /*! \brief Assignment operator from Platform.
     * 
     *  This simply copies the platform ID value, which is an inexpensive operation.
     */
    Platform& operator = (const Platform& rhs)
    {
        if (this != &rhs) {
            detail::Wrapper<cl_type>::operator=(rhs);
        }
        return *this;
    }

    /*! \brief Assignment operator from cl_platform_id.
     * 
     *  This simply copies the platform ID value, which is an inexpensive operation.
     */
    Platform& operator = (const cl_platform_id& rhs)
    {
        detail::Wrapper<cl_type>::operator=(rhs);
        return *this;
    }

    //! \brief Wrapper for clGetPlatformInfo().
    cl_int getInfo(cl_platform_info name, STRING_CLASS* param) const
    {
        return detail::errHandler(
            detail::getInfo(&::clGetPlatformInfo, object_, name, param),
            __GET_PLATFORM_INFO_ERR);
    }

    //! \brief Wrapper for clGetPlatformInfo() that returns by value.
    template <cl_int name> typename
    detail::param_traits<detail::cl_platform_info, name>::param_type
    getInfo(cl_int* err = NULL) const
    {
        typename detail::param_traits<
            detail::cl_platform_info, name>::param_type param;
        cl_int result = getInfo(name, &param);
        if (err != NULL) {
            *err = result;
        }
        return param;
    }

    /*! \brief Gets a list of devices for this platform.
     * 
     *  Wraps clGetDeviceIDs().
     */
    cl_int getDevices(
        cl_device_type type,
        VECTOR_CLASS<Device>* devices) const
    {
        cl_uint n = 0;
        if( devices == NULL ) {
            return detail::errHandler(CL_INVALID_ARG_VALUE, __GET_DEVICE_IDS_ERR);
        }
        cl_int err = ::clGetDeviceIDs(object_, type, 0, NULL, &n);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __GET_DEVICE_IDS_ERR);
        }

        cl_device_id* ids = (cl_device_id*) alloca(n * sizeof(cl_device_id));
        err = ::clGetDeviceIDs(object_, type, n, ids, NULL);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __GET_DEVICE_IDS_ERR);
        }

        devices->assign(&ids[0], &ids[n]);
        return CL_SUCCESS;
    }

#if defined(USE_DX_INTEROP)
   /*! \brief Get the list of available D3D10 devices.
     *
     *  \param d3d_device_source.
     *
     *  \param d3d_object.
     *
     *  \param d3d_device_set.
     *
     *  \param devices returns a vector of OpenCL D3D10 devices found. The cl::Device
     *  values returned in devices can be used to identify a specific OpenCL
     *  device. If \a devices argument is NULL, this argument is ignored.
     *
     *  \return One of the following values:
     *    - CL_SUCCESS if the function is executed successfully.
     *
     *  The application can query specific capabilities of the OpenCL device(s)
     *  returned by cl::getDevices. This can be used by the application to
     *  determine which device(s) to use.
     *
     * \note In the case that exceptions are enabled and a return value
     * other than CL_SUCCESS is generated, then cl::Error exception is
     * generated.
     */
    cl_int getDevices(
        cl_d3d10_device_source_khr d3d_device_source,
        void *                     d3d_object,
        cl_d3d10_device_set_khr    d3d_device_set,
        VECTOR_CLASS<Device>* devices) const
    {
        typedef CL_API_ENTRY cl_int (CL_API_CALL *PFN_clGetDeviceIDsFromD3D10KHR)(
            cl_platform_id platform, 
            cl_d3d10_device_source_khr d3d_device_source, 
            void * d3d_object,
            cl_d3d10_device_set_khr d3d_device_set,
            cl_uint num_entries,
            cl_device_id * devices,
            cl_uint* num_devices);

        if( devices == NULL ) {
            return detail::errHandler(CL_INVALID_ARG_VALUE, __GET_DEVICE_IDS_ERR);
        }

        static PFN_clGetDeviceIDsFromD3D10KHR pfn_clGetDeviceIDsFromD3D10KHR = NULL;
        __INIT_CL_EXT_FCN_PTR_PLATFORM(object_, clGetDeviceIDsFromD3D10KHR);

        cl_uint n = 0;
        cl_int err = pfn_clGetDeviceIDsFromD3D10KHR(
            object_, 
            d3d_device_source, 
            d3d_object,
            d3d_device_set, 
            0, 
            NULL, 
            &n);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __GET_DEVICE_IDS_ERR);
        }

        cl_device_id* ids = (cl_device_id*) alloca(n * sizeof(cl_device_id));
        err = pfn_clGetDeviceIDsFromD3D10KHR(
            object_, 
            d3d_device_source, 
            d3d_object,
            d3d_device_set,
            n, 
            ids, 
            NULL);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __GET_DEVICE_IDS_ERR);
        }

        devices->assign(&ids[0], &ids[n]);
        return CL_SUCCESS;
    }
#endif

    /*! \brief Gets a list of available platforms.
     * 
     *  Wraps clGetPlatformIDs().
     */
    static cl_int get(
        VECTOR_CLASS<Platform>* platforms)
    {
        cl_uint n = 0;

        if( platforms == NULL ) {
            return detail::errHandler(CL_INVALID_ARG_VALUE, __GET_PLATFORM_IDS_ERR);
        }

        cl_int err = ::clGetPlatformIDs(0, NULL, &n);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __GET_PLATFORM_IDS_ERR);
        }

        cl_platform_id* ids = (cl_platform_id*) alloca(
            n * sizeof(cl_platform_id));
        err = ::clGetPlatformIDs(n, ids, NULL);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __GET_PLATFORM_IDS_ERR);
        }

        platforms->assign(&ids[0], &ids[n]);
        return CL_SUCCESS;
    }

    /*! \brief Gets the first available platform.
     * 
     *  Wraps clGetPlatformIDs(), returning the first result.
     */
    static cl_int get(
        Platform * platform)
    {
        cl_uint n = 0;

        if( platform == NULL ) {
            return detail::errHandler(CL_INVALID_ARG_VALUE, __GET_PLATFORM_IDS_ERR);
        }

        cl_int err = ::clGetPlatformIDs(0, NULL, &n);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __GET_PLATFORM_IDS_ERR);
        }

        cl_platform_id* ids = (cl_platform_id*) alloca(
            n * sizeof(cl_platform_id));
        err = ::clGetPlatformIDs(n, ids, NULL);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __GET_PLATFORM_IDS_ERR);
        }

        *platform = ids[0];
        return CL_SUCCESS;
    }

    /*! \brief Gets the first available platform, returning it by value.
     * 
     *  Wraps clGetPlatformIDs(), returning the first result.
     */
    static Platform get(
        cl_int * errResult = NULL)
    {
        Platform platform;
        cl_uint n = 0;
        cl_int err = ::clGetPlatformIDs(0, NULL, &n);
        if (err != CL_SUCCESS) {
            detail::errHandler(err, __GET_PLATFORM_IDS_ERR);
            if (errResult != NULL) {
                *errResult = err;
            }
        }

        cl_platform_id* ids = (cl_platform_id*) alloca(
            n * sizeof(cl_platform_id));
        err = ::clGetPlatformIDs(n, ids, NULL);

        if (err != CL_SUCCESS) {
            detail::errHandler(err, __GET_PLATFORM_IDS_ERR);
        }

        if (errResult != NULL) {
            *errResult = err;
        }
        
        return ids[0];
    }

    static Platform getDefault( 
        cl_int *errResult = NULL )
    {
        return get(errResult);
    }

    
#if defined(CL_VERSION_1_2)
    //! \brief Wrapper for clUnloadCompiler().
    cl_int
    unloadCompiler()
    {
        return ::clUnloadPlatformCompiler(object_);
    }
#endif // #if defined(CL_VERSION_1_2)
}; // class Platform

/**
 * Deprecated APIs for 1.2
 */
#if defined(CL_USE_DEPRECATED_OPENCL_1_1_APIS) || (defined(CL_VERSION_1_1) && !defined(CL_VERSION_1_2))
/**
 * Unload the OpenCL compiler.
 * \note Deprecated for OpenCL 1.2. Use Platform::unloadCompiler instead.
 */
inline CL_EXT_PREFIX__VERSION_1_1_DEPRECATED cl_int
UnloadCompiler() CL_EXT_SUFFIX__VERSION_1_1_DEPRECATED;
inline cl_int
UnloadCompiler()
{
    return ::clUnloadCompiler();
}
#endif // #if defined(CL_VERSION_1_1)

/*! \brief Class interface for cl_context.
 *
 *  \note Copies of these objects are shallow, meaning that the copy will refer
 *        to the same underlying cl_context as the original.  For details, see
 *        clRetainContext() and clReleaseContext().
 *
 *  \see cl_context
 */
class Context 
    : public detail::Wrapper<cl_context>
{
private:
    static volatile int default_initialized_;
    static Context default_;
    static volatile cl_int default_error_;
public:
    /*! \brief Destructor.
     *
     *  This calls clReleaseContext() on the value held by this instance.
     */
    ~Context() { }

    /*! \brief Constructs a context including a list of specified devices.
     *
     *  Wraps clCreateContext().
     */
    Context(
        const VECTOR_CLASS<Device>& devices,
        cl_context_properties* properties = NULL,
        void (CL_CALLBACK * notifyFptr)(
            const char *,
            const void *,
            ::size_t,
            void *) = NULL,
        void* data = NULL,
        cl_int* err = NULL)
    {
        cl_int error;

        ::size_t numDevices = devices.size();
        cl_device_id* deviceIDs = (cl_device_id*) alloca(numDevices * sizeof(cl_device_id));
        for( ::size_t deviceIndex = 0; deviceIndex < numDevices; ++deviceIndex ) {
            deviceIDs[deviceIndex] = (devices[deviceIndex])();
        }

        object_ = ::clCreateContext(
            properties, (cl_uint) numDevices,
            deviceIDs,
            notifyFptr, data, &error);

        detail::errHandler(error, __CREATE_CONTEXT_ERR);
        if (err != NULL) {
            *err = error;
        }
    }

    Context(
        const Device& device,
        cl_context_properties* properties = NULL,
        void (CL_CALLBACK * notifyFptr)(
            const char *,
            const void *,
            ::size_t,
            void *) = NULL,
        void* data = NULL,
        cl_int* err = NULL)
    {
        cl_int error;

        cl_device_id deviceID = device();

        object_ = ::clCreateContext(
            properties, 1,
            &deviceID,
            notifyFptr, data, &error);

        detail::errHandler(error, __CREATE_CONTEXT_ERR);
        if (err != NULL) {
            *err = error;
        }
    }

    /*! \brief Constructs a context including all devices of a specified type.
     *
     *  Wraps clCreateContextFromType().
     */
    Context(
        cl_device_type type,
        cl_context_properties* properties = NULL,
        void (CL_CALLBACK * notifyFptr)(
            const char *,
            const void *,
            ::size_t,
            void *) = NULL,
        void* data = NULL,
        cl_int* err = NULL)
    {
        cl_int error;

#if !defined(__APPLE__) || !defined(__MACOS)
        cl_context_properties prop[4] = {CL_CONTEXT_PLATFORM, 0, 0, 0 };	
        if (properties == NULL) {
            prop[1] = (cl_context_properties)Platform::get(&error)();
            if (error != CL_SUCCESS) {
                detail::errHandler(error, __CREATE_CONTEXT_FROM_TYPE_ERR);
                if (err != NULL) {
                    *err = error;
                    return;
                }
            }

            properties = &prop[0];
        }
#endif
        object_ = ::clCreateContextFromType(
            properties, type, notifyFptr, data, &error);

        detail::errHandler(error, __CREATE_CONTEXT_FROM_TYPE_ERR);
        if (err != NULL) {
            *err = error;
        }
    }

    /*! \brief Returns a singleton context including all devices of CL_DEVICE_TYPE_DEFAULT.
     *
     *  \note All calls to this function return the same cl_context as the first.
     */
    static Context getDefault(cl_int * err = NULL) 
    {
        int state = detail::compare_exchange(
            &default_initialized_, 
            __DEFAULT_BEING_INITIALIZED, __DEFAULT_NOT_INITIALIZED);
        
        if (state & __DEFAULT_INITIALIZED) {
            if (err != NULL) {
                *err = default_error_;
            }
            return default_;
        }

        if (state & __DEFAULT_BEING_INITIALIZED) {
              // Assume writes will propagate eventually...
              while(default_initialized_ != __DEFAULT_INITIALIZED) {
                  detail::fence();
              }

            if (err != NULL) {
                *err = default_error_;
            }
            return default_;
        }

        cl_int error;
        default_ = Context(
            CL_DEVICE_TYPE_DEFAULT,
            NULL,
            NULL,
            NULL,
            &error);

        detail::fence();

        default_error_ = error;
        // Assume writes will propagate eventually...
        default_initialized_ = __DEFAULT_INITIALIZED;

        detail::fence();

        if (err != NULL) {
            *err = default_error_;
        }
        return default_;

    }

    //! \brief Default constructor - initializes to NULL.
    Context() : detail::Wrapper<cl_type>() { }

    /*! \brief Copy constructor.
     * 
     *  This calls clRetainContext() on the parameter's cl_context.
     */
    Context(const Context& context) : detail::Wrapper<cl_type>(context) { }

    /*! \brief Constructor from cl_context - takes ownership.
     * 
     *  This effectively transfers ownership of a refcount on the cl_context
     *  into the new Context object.
     */
    __CL_EXPLICIT_CONSTRUCTORS Context(const cl_context& context) : detail::Wrapper<cl_type>(context) { }

    /*! \brief Assignment operator from Context.
     * 
     *  This calls clRetainContext() on the parameter and clReleaseContext() on
     *  the previous value held by this instance.
     */
    Context& operator = (const Context& rhs)
    {
        if (this != &rhs) {
            detail::Wrapper<cl_type>::operator=(rhs);
        }
        return *this;
    }

    /*! \brief Assignment operator from cl_context - takes ownership.
     * 
     *  This effectively transfers ownership of a refcount on the rhs and calls
     *  clReleaseContext() on the value previously held by this instance.
     */
    Context& operator = (const cl_context& rhs)
    {
        detail::Wrapper<cl_type>::operator=(rhs);
        return *this;
    }

    //! \brief Wrapper for clGetContextInfo().
    template <typename T>
    cl_int getInfo(cl_context_info name, T* param) const
    {
        return detail::errHandler(
            detail::getInfo(&::clGetContextInfo, object_, name, param),
            __GET_CONTEXT_INFO_ERR);
    }

    //! \brief Wrapper for clGetContextInfo() that returns by value.
    template <cl_int name> typename
    detail::param_traits<detail::cl_context_info, name>::param_type
    getInfo(cl_int* err = NULL) const
    {
        typename detail::param_traits<
            detail::cl_context_info, name>::param_type param;
        cl_int result = getInfo(name, &param);
        if (err != NULL) {
            *err = result;
        }
        return param;
    }

    /*! \brief Gets a list of supported image formats.
     *  
     *  Wraps clGetSupportedImageFormats().
     */
    cl_int getSupportedImageFormats(
        cl_mem_flags flags,
        cl_mem_object_type type,
        VECTOR_CLASS<ImageFormat>* formats) const
    {
        cl_uint numEntries;
        cl_int err = ::clGetSupportedImageFormats(
           object_, 
           flags,
           type, 
           0, 
           NULL, 
           &numEntries);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __GET_SUPPORTED_IMAGE_FORMATS_ERR);
        }

        ImageFormat* value = (ImageFormat*)
            alloca(numEntries * sizeof(ImageFormat));
        err = ::clGetSupportedImageFormats(
            object_, 
            flags, 
            type, 
            numEntries,
            (cl_image_format*) value, 
            NULL);
        if (err != CL_SUCCESS) {
            return detail::errHandler(err, __GET_SUPPORTED_IMAGE_FORMATS_ERR);
        }

        formats->assign(&value[0], &value[numEntries]);
        return CL_SUCCESS;
    }
};

inline Device Device::getDefault(cl_int * err)
{
    cl_int error;
    Device device;

    Context context = Context::getDefault(&error);
    detail::errHandler(error, __CREATE_COMMAND_QUEUE_ERR);

    if (error != CL_SUCCESS) {
        if (err != NULL) {
            *err = error;
        }
    }
    else {
        device = context.getInfo<CL_CONTEXT_DEVICES>()[0];
        if (err != NULL) {
            *err = CL_SUCCESS;
        }
    }

    return device;
}


#ifdef _WIN32
__declspec(selectany) volatile int Context::default_initialized_ = __DEFAULT_NOT_INITIALIZED;
__declspec(selectany) Context Context::default_;
__declspec(selectany) volatile cl_int Context::default_error_ = CL_SUCCESS;
#else
__attribute__((weak)) volatile int Context::default_initialized_ = __DEFAULT_NOT_INITIALIZED;
__attribute__((weak)) Context Context::default_;
__attribute__((weak)) volatile cl_int Context::default_error_ = CL_SUCCESS;
#endif

/*! \brief Class interface for cl_event.
 *
 *  \note Copies of these objects are shallow, meaning that the copy will refer
 *        to the same underlying cl_event as the original.  For details, see
 *        clRetainEvent() and clReleaseEvent().
 *
 *  \see cl_event
 */
class Event : public detail::Wrapper<cl_event>
{
public:
    /*! \brief Destructor.
     *
     *  This calls clReleaseEvent() on the value held by this instance.
     */
    ~Event() { }
 
    //! \brief Default constructor - initializes to NULL.
    Event() : detail::Wrapper<cl_type>() { }

    /*! \brief Copy constructor.
     * 
     *  This calls clRetainEvent() on the parameter's cl_event.
     */
    Event(const Event& event) : detail::Wrapper<cl_type>(event) { }

    /*! \brief Constructor from cl_event - takes ownership.
     * 
     *  This effectively transfers ownership of a refcount on the cl_event
     *  into the new Event object.
     */
    Event(const cl_event& event) : detail::Wrapper<cl_type>(event) { }

    /*! \brief Assignment operator from cl_event - takes ownership.
     *
     *  This effectively transfers ownership of a refcount on the rhs and calls
     *  clReleaseEvent() on the value previously held by this instance.
     */
    Event& operator = (const Event& rhs)
    {
        if (this != &rhs) {
            detail::Wrapper<cl_type>::operator=(rhs);
        }
        return *this;
    }

    /*! \brief Assignment operator from cl_event.
     * 
     *  This calls clRetainEvent() on the parameter and clReleaseEvent() on
     *  the previous value held by this instance.
     */
    Event& operator = (const cl_event& rhs)
    {
        detail::Wrapper<cl_type>::operator=(rhs);
        return *this;
    }

    //! \brief Wrapper for clGetEventInfo().
    template <typename T>
    cl_int getInfo(cl_event_info name, T* param) const
    {
        return detail::errHandler(
            detail::getInfo(&::clGetEventInfo, object_, name, param),
            __GET_EVENT_INFO_ERR);
    }

    //! \brief Wrapper for clGetEventInfo() that returns by value.
    template <cl_int name> typename
    detail::param_traits<detail::cl_event_info, name>::param_type
    getInfo(cl_int* err = NULL) const
    {
        typename detail::param_traits<
            detail::cl_event_info, name>::param_type param;
        cl_int result = getInfo(name, &param);
        if (err != NULL) {
            *err = result;
        }
        return param;
    }

    //! \brief Wrapper for clGetEventProfilingInfo().
    template <typename T>
    cl_int getProfilingInfo(cl_profiling_info name, T* param) const
    {
        return detail::errHandler(detail::getInfo(
            &::clGetEventProfilingInfo, object_, name, param),
            __GET_EVENT_PROFILE_INFO_ERR);
    }

    //! \brief Wrapper for clGetEventProfilingInfo() that returns by value.
    template <cl_int name> typename
    detail::param_traits<detail::cl_profiling_info, name>::param_type
    getProfilingInfo(cl_int* err = NULL) const
    {
        typename detail::param_traits<
            detail::cl_profiling_info, name>::param_type param;
        cl_int result = getProfilingInfo(name, &param);
        if (err != NULL) {
            *err = result;
        }
        return param;
    }

    /*! \brief Blocks the calling thread until this event completes.
     * 
     *  Wraps clWaitForEvents().
     */
    cl_int wait() const
    {
        return detail::errHandler(
            ::clWaitForEvents(1, &object_),
            __WAIT_FOR_EVENTS_ERR);
    }

#if defined(CL_VERSION_1_1)
    /*! \brief Registers a user callback function for a specific command execution status.
     *
     *  Wraps clSetEventCallback().
     */
    cl_int setCallback(
        cl_int type,
        void (CL_CALLBACK * pfn_notify)(cl_event, cl_int, void *),		
        void * user_data = NULL)
    {
        return detail::errHandler(
            ::clSetEventCallback(
                object_,
                type,
                pfn_notify,
                user_data), 
            __SET_EVENT_CALLBACK_ERR);
    }
#endif

    /*! \brief Blocks the calling thread until every event specified is complete.
     * 
     *  Wraps clWaitForEvents().
     */
    static cl_int
    waitForEvents(const VECTOR_CLASS<Event>& events)
    {
        return detail::errHandler(
            ::clWaitForEvents(
                (cl_uint) events.size(), (cl_event*)&events.front()),
            __WAIT_FOR_EVENTS_ERR);
    }
};

#if defined(CL_VERSION_1_1)
/*! \brief Class interface for user events (a subset of cl_event's).
 * 
 *  See Event for details about copy semantics, etc.
 */
class UserEvent : public Event
{
public:
    /*! \brief Constructs a user event on a given context.
     *
     *  Wraps clCreateUserEvent().
     */
    UserEvent(
        const Context& context,
        cl_int * err = NULL)
    {
        cl_int error;
        object_ = ::clCreateUserEvent(
            context(),
            &error);

        detail::errHandler(error, __CREATE_USER_EVENT_ERR);
        if (err != NULL) {
            *err = error;
        }
    }

    //! \brief Default constructor - initializes to NULL.
    UserEvent() : Event() { }

    //! \brief Copy constructor - performs shallow copy.
    UserEvent(const UserEvent& event) : Event(event) { }

    //! \brief Assignment Operator - performs shallow copy.
    UserEvent& operator = (const UserEvent& rhs)
    {
        if (this != &rhs) {
            Event::operator=(rhs);
        }
        return *this;
    }

    /*! \brief Sets the execution status of a user event object.
     *
     *  Wraps clSetUserEventStatus().
     */
    cl_int setStatus(cl_int status)
    {
        return detail::errHandler(
            ::clSetUserEventStatus(object_,status), 
            __SET_USER_EVENT_STATUS_ERR);
    }
};
#endif

/*! \brief Blocks the calling thread until every event specified is complete.
 * 
 *  Wraps clWaitForEvents().
 */
inline static cl_int
WaitForEvents(const VECTOR_CLASS<Event>& events)
{
    return detail::errHandler(
        ::clWaitForEvents(
            (cl_uint) events.size(), (cl_event*)&events.front()),
        __WAIT_FOR_EVENTS_ERR);
}

/*! \brief Class interface for cl_mem.
 *
 *  \note Copies of these objects are shallow, meaning that the copy will refer
 *        to the same underlying cl_mem as the original.  For details, see
 *        clRetainMemObject() and clReleaseMemObject().
 *
 *  \see cl_mem
 */
class Memory : public detail::Wrapper<cl_mem>
{
public:
 
    /*! \brief Destructor.
     *
     *  This calls clReleaseMemObject() on the value held by this instance.
     */
    ~Memory() {}

    //! \brief Default constructor - initializes to NULL.
    Memory() : detail::Wrapper<cl_type>() { }

    /*! \brief Copy constructor - performs shallow copy.
     * 
     *  This calls clRetainMemObject() on the parameter's cl_mem.
     */
    Memory(const Memory& memory) : detail::Wrapper<cl_type>(memory) { }

    /*! \brief Constructor from cl_mem - takes ownership.
     * 
     *  This effectively transfers ownership of a refcount on the cl_mem
     *  into the new Memory object.
     */
    __CL_EXPLICIT_CONSTRUCTORS Memory(const cl_mem& memory) : detail::Wrapper<cl_type>(memory) { }

    /*! \brief Assignment operator from Memory.
     * 
     *  This calls clRetainMemObject() on the parameter and clReleaseMemObject()
     *  on the previous value held by this instance.
     */
    Memory& operator = (const Memory& rhs)
    {
        if (this != &rhs) {
            detail::Wrapper<cl_type>::operator=(rhs);
        }
        return *this;
    }

    /*! \brief Assignment operator from cl_mem - takes ownership.
     *
     *  This effectively transfers ownership of a refcount on the rhs and calls
     *  clReleaseMemObject() on the value previously held by this instance.
     */
    Memory& operator = (const cl_mem& rhs)
    {
        detail::Wrapper<cl_type>::operator=(rhs);
        return *this;
    }

    //! \brief Wrapper for clGetMemObjectInfo().
    template <typename T>
    cl_int getInfo(cl_mem_info name, T* param) const
    {
        return detail::errHandler(
            detail::getInfo(&::clGetMemObjectInfo, object_, name, param),
            __GET_MEM_OBJECT_INFO_ERR);
    }

    //! \brief Wrapper for clGetMemObjectInfo() that returns by value.
    template <cl_int name> typename
    detail::param_traits<detail::cl_mem_info, name>::param_type
    getInfo(cl_int* err = NULL) const
    {
        typename detail::param_traits<
            detail::cl_mem_info, name>::param_type param;
        cl_int result = getInfo(name, &param);
        if (err != NULL) {
            *err = result;
        }
        return param;
    }

#if defined(CL_VERSION_1_1)
    /*! \brief Registers a callback function to be called when the memory object
     *         is no longer needed.
     *
     *  Wraps clSetMemObjectDestructorCallback().
     *
     *  Repeated calls to this function, for a given cl_mem value, will append
     *  to the list of functions called (in reverse order) when memory object's
     *  resources are freed and the memory object is deleted.
     *
     *  \note
     *  The registered callbacks are associated with the underlying cl_mem
     *  value - not the Memory class instance.
     */
    cl_int setDestructorCallback(
        void (CL_CALLBACK * pfn_notify)(cl_mem, void *),		
        void * user_data = NULL)
    {
        return detail::errHandler(
            ::clSetMemObjectDestructorCallback(
                object_,
                pfn_notify,
                user_data), 
            __SET_MEM_OBJECT_DESTRUCTOR_CALLBACK_ERR);
    }
#endif

};

// Pre-declare copy functions
class Buffer;
template< typename IteratorType >
cl_int copy( IteratorType startIterator, IteratorType endIterator, cl::Buffer &buffer );
template< typename IteratorType >
cl_int copy( const cl::Buffer &buffer, IteratorType startIterator, IteratorType endIterator );

/*! \brief Class interface for Buffer Memory Objects.
 * 
 *  See Memory for details about copy semantics, etc.
 *
 *  \see Memory
 */
class Buffer : public Memory
{
public:

    /*! \brief Constructs a Buffer in a specified context.
     *
     *  Wraps clCreateBuffer().
     *
     *  \param host_ptr Storage to be used if the CL_MEM_USE_HOST_PTR flag was
     *                  specified.  Note alignment & exclusivity requirements.
     */
    Buffer(
        const Context& context,
        cl_mem_flags flags,
        ::size_t size,
        void* host_ptr = NULL,
        cl_int* err = NULL)
    {
        cl_int error;
        object_ = ::clCreateBuffer(context(), flags, size, host_ptr, &error);

        detail::errHandler(error, __CREATE_BUFFER_ERR);
        if (err != NULL) {
            *err = error;
        }
    }

    /*! \brief Constructs a Buffer in the default context.
     *
     *  Wraps clCreateBuffer().
     *
     *  \param host_ptr Storage to be used if the CL_MEM_USE_HOST_PTR flag was
     *                  specified.  Note alignment & exclusivity requirements.
     *
     *  \see Context::getDefault()
     */
    Buffer(
         cl_mem_flags flags,
        ::size_t size,
        void* host_ptr = NULL,
        cl_int* err = NULL)
    {
        cl_int error;

        Context context = Context::getDefault(err);

        object_ = ::clCreateBuffer(context(), flags, size, host_ptr, &error);

        detail::errHandler(error, __CREATE_BUFFER_ERR);
        if (err != NULL) {
            *err = error;
        }
    }

    /*!
     * \brief Construct a Buffer from a host container via iterators.
     * If useHostPtr is specified iterators must be random access.
     */
    template< typename IteratorType >
    Buffer(
        IteratorType startIterator,
        IteratorType endIterator,
        bool readOnly,
        bool useHostPtr = false,
        cl_int* err = NULL)
    {
        typedef typename std::iterator_traits<IteratorType>::value_type DataType;
        cl_int error;

        cl_mem_flags flags = 0;
        if( readOnly ) {
            flags |= CL_MEM_READ_ONLY;
        }
        else {
            flags |= CL_MEM_READ_WRITE;
        }
        if( useHostPtr ) {
            flags |= CL_MEM_USE_HOST_PTR;
        }
        
        ::size_t size = sizeof(DataType)*(endIterator - startIterator);

        Context context = Context::getDefault(err);

        if( useHostPtr ) {
            object_ = ::clCreateBuffer(context(), flags, size, static_cast<DataType*>(&*startIterator), &error);
        } else {
            object_ = ::clCreateBuffer(context(), flags, size, 0, &error);
        }

        detail::errHandler(error, __CREATE_BUFFER_ERR);
        if (err != NULL) {
            *err = error;
        }

        if( !useHostPtr ) {
            error = cl::copy(startIterator, endIterator, *this);
            detail::errHandler(error, __CREATE_BUFFER_ERR);
            if (err != NULL) {
                *err = error;
            }
        }
    }

    //! \brief Default constructor - initializes to NULL.
    Buffer() : Memory() { }

    /*! \brief Copy constructor - performs shallow copy.
     *
     *  See Memory for further details.
     */
    Buffer(const Buffer& buffer) : Memory(buffer) { }

    /*! \brief Constructor from cl_mem - takes ownership.
     *
     *  See Memory for further details.
     */
    __CL_EXPLICIT_CONSTRUCTORS Buffer(const cl_mem& buffer) : Memory(buffer) { }

    /*! \brief Assignment from Buffer - performs shallow copy.
     *
     *  See Memory for further details.
     */
    Buffer& operator = (const Buffer& rhs)
    {
        if (this != &rhs) {
            Memory::operator=(rhs);
        }
        return *this;
    }

    /*! \brief Assignment from cl_mem - performs shallow copy.
     *
     *  See Memory for further details.
     */
    Buffer& operator = (const cl_mem& rhs)
    {
        Memory::operator=(rhs);
        return *this;
    }

#if defined(CL_VERSION_1_1)
    /*! \brief Creates a new buffer object from this.
     *
     *  Wraps clCreateSubBuffer().
     */
    Buffer createSubBuffer(
        cl_mem_flags flags,
        cl_buffer_create_type buffer_create_type,
        const void * buffer_create_info,
        cl_int * err = NULL)
    {
        Buffer result;
        cl_int error;
        result.object_ = ::clCreateSubBuffer(
            object_, 
            flags, 
            buffer_create_type, 
            buffer_create_info, 
            &error);

        detail::errHandler(error, __CREATE_SUBBUFFER_ERR);
        if (err != NULL) {
            *err = error;
        }

        return result;
    }		
#endif
};

#if defined (USE_DX_INTEROP)
/*! \brief Class interface for creating OpenCL buffers from ID3D10Buffer's.
 *
 *  This is provided to facilitate interoperability with Direct3D.
 * 
 *  See Memory for details about copy semantics, etc.
 *
 *  \see Memory
 */
class BufferD3D10 : public Buffer
{
public:
    typedef CL_API_ENTRY cl_mem (CL_API_CALL *PFN_clCreateFromD3D10BufferKHR)(
    cl_context context, cl_mem_flags flags, ID3D10Buffer*  buffer,
    cl_int* errcode_ret);

    /*! \brief Constructs a BufferD3D10, in a specified context, from a
     *         given ID3D10Buffer.
     *
     *  Wraps clCreateFromD3D10BufferKHR().
     */
    BufferD3D10(
        const Context& context,
        cl_mem_flags flags,
        ID3D10Buffer* bufobj,
        cl_int * err = NULL)
    {
        static PFN_clCreateFromD3D10BufferKHR pfn_clCreateFromD3D10BufferKHR = NULL;

#if defined(CL_VERSION_1_2)
        vector<cl_context_properties> props = context.getInfo<CL_CONTEXT_PROPERTIES>();
        cl_platform platform = -1;
        for( int i = 0; i < props.size(); ++i ) {
            if( props[i] == CL_CONTEXT_PLATFORM ) {
                platform = props[i+1];
            }
        }
        __INIT_CL_EXT_FCN_PTR_PLATFORM(platform, clCreateFromD3D10BufferKHR);
#endif
#if defined(CL_VERSION_1_1)
        __INIT_CL_EXT_FCN_PTR(clCreateFromD3D10BufferKHR);
#endif

        cl_int error;
        object_ = pfn_clCreateFromD3D10BufferKHR(
            context(),
            flags,
            bufobj,
            &error);

        detail::errHandler(error, __CREATE_GL_BUFFER_ERR);
        if (err != NULL) {
            *err = error;
        }
    }

    //! \brief Default constructor - initializes to NULL.
    BufferD3D10() : Buffer() { }

    /*! \brief Copy constructor - performs shallow copy.
     *
     *  See Memory for further details.
     */
    BufferD3D10(const BufferD3D10& buffer) : Buffer(buffer) { }

    /*! \brief Constructor from cl_mem - takes ownership.
     *
     *  See Memory for further details.
     */
    __CL_EXPLICIT_CONSTRUCTORS BufferD3D10(const cl_mem& buffer) : Buffer(buffer) { }

    /*! \brief Assignment from BufferD3D10 - performs shallow copy.
     *
     *  See Memory for further details.
     */
    BufferD3D10& operator = (const BufferD3D10& rhs)
    {
        if (this != &rhs) {
            Buffer::operator=(rhs);
        }
        return *this;
    }

    /*! \brief Assignment from cl_mem - performs shallow copy.
     *
     *  See Memory for further details.
     */
    BufferD3D10& operator = (const cl_mem& rhs)
    {
        Buffer::operator=(rhs);
        return *this;
    }
};
#endif

/*! \brief Class interface for GL Buffer Memory Objects.
 *
 *  This is provided to facilitate interoperability with OpenGL.
 * 
 *  See Memory for details about copy semantics, etc.
 * 
 *  \see Memory
 */
class BufferGL : public Buffer
{
public:
    /*! \brief Constructs a BufferGL in a specified context, from a given
     *         GL buffer.
     *
     *  Wraps clCreateFromGLBuffer().
     */
    BufferGL(
        const Context& context,
        cl_mem
... [Content truncated - file is very large]
```

## High-Level Overview

This is a C++ header file that declares interfaces, classes, and function prototypes.

**Key Characteristics:**
- Defines public APIs and interfaces
- Contains class declarations and templates
- May include inline function implementations
- Provides documentation through comments
- Uses header guards or #pragma once


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **Platform**: A class/struct defined in this file
- **param_traits**: A class/struct defined in this file
- **CL_EXT_PREFIX__VERSION_1_1_DEPRECATED**: A class/struct defined in this file
- **GetInfoFunctor1**: A class/struct defined in this file
- **ReferenceHandler**: A class/struct defined in this file
- **token**: A class/struct defined in this file
- **ImageFormat**: A class/struct defined in this file
- **is**: A class/struct defined in this file
- **Memory**: A class/struct defined in this file
- **I**: A class/struct defined in this file
- **between**: A class/struct defined in this file
- **GetInfoFunctor0**: A class/struct defined in this file
- **Context**: A class/struct defined in this file
- **CommandQueue**: A class/struct defined in this file
- **Program**: A class/struct defined in this file
- **for**: A class/struct defined in this file
- **size_t**: A class/struct defined in this file
- **by**: A class/struct defined in this file
- **Error**: A class/struct defined in this file
- **to**: A class/struct defined in this file
- **N**: A class/struct defined in this file
- **Wrapper**: A class/struct defined in this file
- **vector**: A class/struct defined in this file
- **used**: A class/struct defined in this file
- **Buffer**: A class/struct defined in this file
- **iterator**: A class/struct defined in this file
- **string**: A class/struct defined in this file
- **Device**: A class/struct defined in this file

### Functions and Methods

- **CL_DEVICE_SIMD_PER_COMPUTE_UNIT_AMD()**: A function/method defined in this file
- **CL_DEVICE_KERNEL_EXEC_TIMEOUT_NV()**: A function/method defined in this file
- **max()**: A function/method defined in this file
- **CL_DEVICE_COMPUTE_CAPABILITY_MAJOR_NV()**: A function/method defined in this file
- **Event()**: A function/method defined in this file
- **_WIN32()**: A function/method defined in this file
- **CL_PLATFORM_ICD_SUFFIX_KHR()**: A function/method defined in this file
- **CL_DEVICE_GPU_OVERLAP_NV()**: A function/method defined in this file
- **CL_API_ENTRY()**: A function/method defined in this file
- **CL_DEVICE_PROFILING_TIMER_OFFSET_AMD()**: A function/method defined in this file
- **VECTOR_CLASS()**: A function/method defined in this file
- **CL_DEVICE_WAVEFRONT_WIDTH_AMD()**: A function/method defined in this file
- **CL_DEVICE_SIMD_INSTRUCTION_WIDTH_AMD()**: A function/method defined in this file
- **CL_DEVICE_COMPUTE_CAPABILITY_MINOR_NV()**: A function/method defined in this file
- **cl_device_id()**: A function/method defined in this file
- **is()**: A function/method defined in this file
- **executed()**: A function/method defined in this file
- **CL_DEVICE_LOCAL_MEM_SIZE_PER_COMPUTE_UNIT_AMD()**: A function/method defined in this file
- **CL_DEVICE_GLOBAL_MEM_CHANNEL_BANK_WIDTH_AMD()**: A function/method defined in this file
- **T()**: A function/method defined in this file
- **CL_DEVICE_INTEGRATED_MEMORY_NV()**: A function/method defined in this file
- **CL_HPP_()**: A function/method defined in this file
- **CL_DEVICE_GLOBAL_MEM_CHANNEL_BANKS_AMD()**: A function/method defined in this file
- **CL_DEVICE_LOCAL_MEM_BANKS_AMD()**: A function/method defined in this file
- **CL_DEVICE_GLOBAL_FREE_MEMORY_AMD()**: A function/method defined in this file
- **CL_DEVICE_REGISTERS_PER_BLOCK_NV()**: A function/method defined in this file
- **for()**: A function/method defined in this file
- **cl()**: A function/method defined in this file
- **return()**: A function/method defined in this file
- **expansion()**: A function/method defined in this file
- **typename()**: A function/method defined in this file
- **std()**: A function/method defined in this file
- **CL_DEVICE_SIMD_WIDTH_AMD()**: A function/method defined in this file
- **CL_DEVICE_GLOBAL_MEM_CHANNELS_AMD()**: A function/method defined in this file
- **to()**: A function/method defined in this file
- **CL_DEVICE_WARP_SIZE_NV()**: A function/method defined in this file
- **detail()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `CL/cl_d3d10.h`
- `intrin.h`
- `iostream`
- `alloca.h`
- `malloc.h`
- `libkern/OSAtomic.h`
- `cstdlib`
- `utility`
- `OpenGL/OpenGL.h`
- `CL/opencl.h`
- `xmmintrin.h`
- `OpenCL/opencl.h`
- `windows.h`
- `exception`
- `GL/gl.h`
- `CL/cl_dx9_media_sharing.h`
- `CL/cl_ext.h`
- `cstdio`
- `CL/cl.hpp`
- `cstring`
- `emmintrin.h`
- `vector`
- `iterator`
- `string`
- `OpenCL/cl.hpp`
- `limits`

**Python Imports:**
- `another`
- `cl_event`
- `Image2D`
- `Platform.`
- `cl_platform_id.`
- `Memory.`
- `ID3D10Buffer`
- `BufferD3D10`
- `this.`
- `cl_context`
- `a`
- `the`
- `cl_device_id.`
- `cl_event.`
- `an`
- `BufferGL`
- `cl_mem`
- `Image`
- `std`
- `another.`
- `Device.`
- `Buffer`
- `Image1D`
- `str.`
- `Context.`


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

