# Documentation for `modules/core/src/ocl.cpp`

## File Metadata

- **Full Path**: `modules/core/src/ocl.cpp`
- **File Name**: `ocl.cpp`
- **File Size**: 252,504 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/src/ocl.cpp](../../../modules/core/src/ocl.cpp)

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
// Copyright (C) 2013, OpenCV Foundation, all rights reserved.
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
// In no event shall the OpenCV Foundation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

#include "precomp.hpp"

#ifndef HAVE_OPENCL
#include "ocl_disabled.impl.hpp"
#else // HAVE_OPENCL

#include <list>
#include <map>
#include <deque>
#include <set>
#include <string>
#include <sstream>
#include <fstream>
#if !(defined _MSC_VER) || (defined _MSC_VER && _MSC_VER > 1700)
#include <inttypes.h>
#endif

#include <opencv2/core/utils/configuration.private.hpp>

#include <opencv2/core/utils/logger.defines.hpp>
#undef CV_LOG_STRIP_LEVEL
#define CV_LOG_STRIP_LEVEL CV_LOG_LEVEL_DEBUG + 1
#include <opencv2/core/utils/logger.hpp>

#include "opencv2/core/ocl_genbase.hpp"
#include "opencl_kernels_core.hpp"

#include "opencv2/core/utils/lock.private.hpp"
#include "opencv2/core/utils/filesystem.hpp"
#include "opencv2/core/utils/filesystem.private.hpp"

#define CV__ALLOCATOR_STATS_LOG(...) CV_LOG_VERBOSE(NULL, 0, "OpenCL allocator: " << __VA_ARGS__)
#include "opencv2/core/utils/allocator_stats.impl.hpp"
#undef CV__ALLOCATOR_STATS_LOG

#define CV_OPENCL_ALWAYS_SHOW_BUILD_LOG          0
#define CV_OPENCL_SHOW_BUILD_OPTIONS             0
#define CV_OPENCL_SHOW_BUILD_KERNELS             0

#define CV_OPENCL_SHOW_RUN_KERNELS               0
#define CV_OPENCL_SYNC_RUN_KERNELS               0
#define CV_OPENCL_TRACE_CHECK                    0

#define CV_OPENCL_VALIDATE_BINARY_PROGRAMS       1

#define CV_OPENCL_SHOW_SVM_ERROR_LOG             1
#define CV_OPENCL_SHOW_SVM_LOG                   0

#include "opencv2/core/bufferpool.hpp"
#ifndef LOG_BUFFER_POOL
# if 0
#   define LOG_BUFFER_POOL printf
# else
#   define LOG_BUFFER_POOL(...)
# endif
#endif

#if CV_OPENCL_SHOW_SVM_LOG
// TODO add timestamp logging
#define CV_OPENCL_SVM_TRACE_P printf("line %d (ocl.cpp): ", __LINE__); printf
#else
#define CV_OPENCL_SVM_TRACE_P(...)
#endif

#if CV_OPENCL_SHOW_SVM_ERROR_LOG
// TODO add timestamp logging
#define CV_OPENCL_SVM_TRACE_ERROR_P printf("Error on line %d (ocl.cpp): ", __LINE__); printf
#else
#define CV_OPENCL_SVM_TRACE_ERROR_P(...)
#endif

#include "opencv2/core/opencl/runtime/opencl_clblas.hpp"
#include "opencv2/core/opencl/runtime/opencl_clfft.hpp"

#include "opencv2/core/opencl/runtime/opencl_core.hpp"

#ifdef HAVE_OPENCL_SVM
#include "opencv2/core/opencl/runtime/opencl_svm_20.hpp"
#include "opencv2/core/opencl/runtime/opencl_svm_hsa_extension.hpp"
#include "opencv2/core/opencl/opencl_svm.hpp"
#endif

#include "umatrix.hpp"

namespace cv { namespace ocl {

#define IMPLEMENT_REFCOUNTABLE() \
    void addref() { CV_XADD(&refcount, 1); } \
    void release() { if( CV_XADD(&refcount, -1) == 1 && !cv::__termination) delete this; } \
    int refcount

static cv::utils::AllocatorStatistics opencl_allocator_stats;

CV_EXPORTS cv::utils::AllocatorStatisticsInterface& getOpenCLAllocatorStatistics();
cv::utils::AllocatorStatisticsInterface& getOpenCLAllocatorStatistics()
{
    return opencl_allocator_stats;
}

#ifndef _DEBUG
static bool isRaiseError()
{
    static bool initialized = false;
    static bool value = false;
    if (!initialized)
    {
        value = cv::utils::getConfigurationParameterBool("OPENCV_OPENCL_RAISE_ERROR", false);
        initialized = true;
    }
    return value;
}
#endif

static void onOpenCLKernelBuildError()
{
    // NB: no need to cache this value
    bool value = cv::utils::getConfigurationParameterBool("OPENCV_OPENCL_ABORT_ON_BUILD_ERROR", false);
    if (value)
    {
        fprintf(stderr, "Abort on OpenCL kernel build failure!\n");
        abort();
    }
}

#if CV_OPENCL_TRACE_CHECK
static inline
void traceOpenCLCheck(cl_int status, const char* message)
{
    std::cout << "OpenCV(OpenCL:" << status << "): " << message << std::endl << std::flush;
}
#define CV_OCL_TRACE_CHECK_RESULT(status, message) traceOpenCLCheck(status, message)
#else
#define CV_OCL_TRACE_CHECK_RESULT(status, message) /* nothing */
#endif

#define CV_OCL_API_ERROR_MSG(check_result, msg) \
    cv::format("OpenCL error %s (%d) during call: %s", getOpenCLErrorString(check_result), check_result, msg)

#define CV_OCL_CHECK_RESULT(check_result, msg) \
    do { \
        CV_OCL_TRACE_CHECK_RESULT(check_result, msg); \
        if (check_result != CL_SUCCESS) \
        { \
            static_assert(std::is_convertible<decltype(msg), const char*>::value, "msg of CV_OCL_CHECK_RESULT must be const char*"); \
            cv::String error_msg = CV_OCL_API_ERROR_MSG(check_result, msg); \
            CV_Error(Error::OpenCLApiCallError, error_msg); \
        } \
    } while (0)

#define CV_OCL_CHECK_(expr, check_result) do { expr; CV_OCL_CHECK_RESULT(check_result, #expr); } while (0)

#define CV_OCL_CHECK(expr) do { cl_int __cl_result = (expr); CV_OCL_CHECK_RESULT(__cl_result, #expr); } while (0)

#ifdef _DEBUG
#define CV_OCL_DBG_CHECK_RESULT(check_result, msg) CV_OCL_CHECK_RESULT(check_result, msg)
#define CV_OCL_DBG_CHECK(expr) CV_OCL_CHECK(expr)
#define CV_OCL_DBG_CHECK_(expr, check_result) CV_OCL_CHECK_(expr, check_result)
#else
#define CV_OCL_DBG_CHECK_RESULT(check_result, msg) \
    do { \
        CV_OCL_TRACE_CHECK_RESULT(check_result, msg); \
        if (check_result != CL_SUCCESS && isRaiseError()) \
        { \
            static_assert(std::is_convertible<decltype(msg), const char*>::value, "msg of CV_OCL_DBG_CHECK_RESULT must be const char*"); \
            cv::String error_msg = CV_OCL_API_ERROR_MSG(check_result, msg); \
            CV_Error(Error::OpenCLApiCallError, error_msg); \
        } \
    } while (0)
#define CV_OCL_DBG_CHECK_(expr, check_result) do { expr; CV_OCL_DBG_CHECK_RESULT(check_result, #expr); } while (0)
#define CV_OCL_DBG_CHECK(expr) do { cl_int __cl_result = (expr); CV_OCL_DBG_CHECK_RESULT(__cl_result, #expr); } while (0)
#endif


static const bool CV_OPENCL_CACHE_ENABLE = utils::getConfigurationParameterBool("OPENCV_OPENCL_CACHE_ENABLE", true);
static const bool CV_OPENCL_CACHE_WRITE = utils::getConfigurationParameterBool("OPENCV_OPENCL_CACHE_WRITE", true);
static const bool CV_OPENCL_CACHE_LOCK_ENABLE = utils::getConfigurationParameterBool("OPENCV_OPENCL_CACHE_LOCK_ENABLE", true);
static const bool CV_OPENCL_CACHE_CLEANUP = utils::getConfigurationParameterBool("OPENCV_OPENCL_CACHE_CLEANUP", true);

#if CV_OPENCL_VALIDATE_BINARY_PROGRAMS
static const bool CV_OPENCL_VALIDATE_BINARY_PROGRAMS_VALUE = utils::getConfigurationParameterBool("OPENCV_OPENCL_VALIDATE_BINARY_PROGRAMS", false);
#endif

// Option to disable calls clEnqueueReadBufferRect / clEnqueueWriteBufferRect / clEnqueueCopyBufferRect
static const bool CV_OPENCL_DISABLE_BUFFER_RECT_OPERATIONS = utils::getConfigurationParameterBool("OPENCV_OPENCL_DISABLE_BUFFER_RECT_OPERATIONS",
#ifdef __APPLE__
        true
#else
        false
#endif
);

static String getBuildExtraOptions()
{
    static String param_buildExtraOptions;
    static bool initialized = false;
    if (!initialized)
    {
        param_buildExtraOptions = utils::getConfigurationParameterString("OPENCV_OPENCL_BUILD_EXTRA_OPTIONS", "");
        initialized = true;
        if (!param_buildExtraOptions.empty())
            CV_LOG_WARNING(NULL, "OpenCL: using extra build options: '" << param_buildExtraOptions << "'");
    }
    return param_buildExtraOptions;
}

static const bool CV_OPENCL_ENABLE_MEM_USE_HOST_PTR = utils::getConfigurationParameterBool("OPENCV_OPENCL_ENABLE_MEM_USE_HOST_PTR", true);
static const size_t CV_OPENCL_ALIGNMENT_MEM_USE_HOST_PTR = utils::getConfigurationParameterSizeT("OPENCV_OPENCL_ALIGNMENT_MEM_USE_HOST_PTR", 4);


struct UMat2D
{
    UMat2D(const UMat& m)
    {
        offset = (int)m.offset;
        step = (int)m.step;
        rows = m.rows;
        cols = m.cols;
    }
    int offset;
    int step;
    int rows;
    int cols;
};

struct UMat3D
{
    UMat3D(const UMat& m)
    {
        offset = (int)m.offset;
        step = (int)m.step.p[1];
        slicestep = (int)m.step.p[0];
        slices = (int)m.size.p[0];
        rows = m.size.p[1];
        cols = m.size.p[2];
    }
    int offset;
    int slicestep;
    int step;
    int slices;
    int rows;
    int cols;
};

// Computes 64-bit "cyclic redundancy check" sum, as specified in ECMA-182
static uint64 crc64( const uchar* data, size_t size, uint64 crc0=0 )
{
    static uint64 table[256];
    static bool initialized = false;

    if( !initialized )
    {
        for( int i = 0; i < 256; i++ )
        {
            uint64 c = i;
            for( int j = 0; j < 8; j++ )
                c = ((c & 1) ? CV_BIG_UINT(0xc96c5795d7870f42) : 0) ^ (c >> 1);
            table[i] = c;
        }
        initialized = true;
    }

    uint64 crc = ~crc0;
    for( size_t idx = 0; idx < size; idx++ )
        crc = table[(uchar)crc ^ data[idx]] ^ (crc >> 8);

    return ~crc;
}

#if OPENCV_HAVE_FILESYSTEM_SUPPORT
struct OpenCLBinaryCacheConfigurator
{
    cv::String cache_path_;
    cv::String cache_lock_filename_;
    cv::Ptr<utils::fs::FileLock> cache_lock_;

    typedef std::map<std::string, std::string> ContextCacheType;
    ContextCacheType prepared_contexts_;
    Mutex mutex_prepared_contexts_;

    OpenCLBinaryCacheConfigurator()
    {
        CV_LOG_DEBUG(NULL, "Initializing OpenCL cache configuration...");
        if (!CV_OPENCL_CACHE_ENABLE)
        {
            CV_LOG_INFO(NULL, "OpenCL cache is disabled");
            return;
        }
        cache_path_ = utils::fs::getCacheDirectory("opencl_cache", "OPENCV_OPENCL_CACHE_DIR");
        if (cache_path_.empty())
        {
            CV_LOG_INFO(NULL, "Specify OPENCV_OPENCL_CACHE_DIR configuration parameter to enable OpenCL cache");
        }
        do
        {
            try
            {
                if (cache_path_.empty())
                    break;
                if (cache_path_ == "disabled")
                    break;
                if (!utils::fs::createDirectories(cache_path_))
                {
                    CV_LOG_DEBUG(NULL, "Can't use OpenCL cache directory: " << cache_path_);
                    clear();
                    break;
                }

                if (CV_OPENCL_CACHE_LOCK_ENABLE)
                {
                    cache_lock_filename_ = cache_path_ + ".lock";
                    if (!utils::fs::exists(cache_lock_filename_))
                    {
                        CV_LOG_DEBUG(NULL, "Creating lock file... (" << cache_lock_filename_ << ")");
                        std::ofstream lock_filename(cache_lock_filename_.c_str(), std::ios::out);
                        if (!lock_filename.is_open())
                        {
                            CV_LOG_WARNING(NULL, "Can't create lock file for OpenCL program cache: " << cache_lock_filename_);
                            break;
                        }
                    }

                    try
                    {
                        cache_lock_ = makePtr<utils::fs::FileLock>(cache_lock_filename_.c_str());
                        CV_LOG_VERBOSE(NULL, 0, "Checking cache lock... (" << cache_lock_filename_ << ")");
                        {
                            utils::shared_lock_guard<utils::fs::FileLock> lock(*cache_lock_);
                        }
                        CV_LOG_VERBOSE(NULL, 0, "Checking cache lock... Done!");
                    }
                    catch (const cv::Exception& e)
                    {
                        CV_LOG_WARNING(NULL, "Can't create OpenCL program cache lock: " << cache_lock_filename_ << std::endl << e.what());
                    }
                    catch (...)
                    {
                        CV_LOG_WARNING(NULL, "Can't create OpenCL program cache lock: " << cache_lock_filename_);
                    }
                }
                else
                {
                    if (CV_OPENCL_CACHE_WRITE)
                    {
                        CV_LOG_WARNING(NULL, "OpenCL cache lock is disabled while cache write is allowed "
                                "(not safe for multiprocess environment)");
                    }
                    else
                    {
                        CV_LOG_INFO(NULL, "OpenCL cache lock is disabled");
                    }
                }
            }
            catch (const cv::Exception& e)
            {
                CV_LOG_WARNING(NULL, "Can't prepare OpenCL program cache: " << cache_path_ << std::endl << e.what());
                clear();
            }
        } while (0);
        if (!cache_path_.empty())
        {
            if (cache_lock_.empty() && CV_OPENCL_CACHE_LOCK_ENABLE)
            {
                CV_LOG_WARNING(NULL, "Initialized OpenCL cache directory, but interprocess synchronization lock is not available. "
                        "Consider to disable OpenCL cache: OPENCV_OPENCL_CACHE_DIR=disabled");
            }
            else
            {
                CV_LOG_INFO(NULL, "Successfully initialized OpenCL cache directory: " << cache_path_);
            }
        }
    }

    void clear()
    {
        cache_path_.clear();
        cache_lock_filename_.clear();
        cache_lock_.release();
    }

    std::string prepareCacheDirectoryForContext(const std::string& ctx_prefix,
            const std::string& cleanup_prefix)
    {
        if (cache_path_.empty())
            return std::string();

        AutoLock lock(mutex_prepared_contexts_);

        ContextCacheType::iterator found_it = prepared_contexts_.find(ctx_prefix);
        if (found_it != prepared_contexts_.end())
            return found_it->second;

        CV_LOG_INFO(NULL, "Preparing OpenCL cache configuration for context: " << ctx_prefix);

        std::string target_directory = cache_path_ + ctx_prefix + "/";
        bool result = utils::fs::isDirectory(target_directory);
        if (!result)
        {
            try
            {
                CV_LOG_VERBOSE(NULL, 0, "Creating directory: " << target_directory);
                if (utils::fs::createDirectories(target_directory))
                {
                    result = true;
                }
                else
                {
                    CV_LOG_WARNING(NULL, "Can't create directory: " << target_directory);
                }
            }
            catch (const cv::Exception& e)
            {
                CV_LOG_ERROR(NULL, "Can't create OpenCL program cache directory for context: " << target_directory << std::endl << e.what());
            }
        }
        target_directory = result ? target_directory : std::string();
        prepared_contexts_.insert(std::pair<std::string, std::string>(ctx_prefix, target_directory));

        if (result && CV_OPENCL_CACHE_CLEANUP && CV_OPENCL_CACHE_WRITE && !cleanup_prefix.empty())
        {
            try
            {
                std::vector<String> entries;
                utils::fs::glob_relative(cache_path_, cleanup_prefix + "*", entries, false, true);
                std::vector<String> remove_entries;
                for (size_t i = 0; i < entries.size(); i++)
                {
                    const String& name = entries[i];
                    if (0 == name.find(cleanup_prefix))
                    {
                        if (0 == name.find(ctx_prefix))
                            continue; // skip current
                        remove_entries.push_back(name);
                    }
                }
                if (!remove_entries.empty())
                {
                    CV_LOG_WARNING(NULL, (remove_entries.size() == 1
                            ? "Detected OpenCL cache directory for other version of OpenCL device."
                            : "Detected OpenCL cache directories for other versions of OpenCL device.")
                            << " We assume that these directories are obsolete after OpenCL runtime/drivers upgrade.");
                    CV_LOG_WARNING(NULL, "Trying to remove these directories...");
                    for (size_t i = 0; i < remove_entries.size(); i++)
                    {
                        CV_LOG_WARNING(NULL, "- " << remove_entries[i]);
                    }
                    CV_LOG_WARNING(NULL, "Note: You can disable this behavior via this option: OPENCV_OPENCL_CACHE_CLEANUP=0");

                    for (size_t i = 0; i < remove_entries.size(); i++)
                    {
                        const String& name = remove_entries[i];
                        cv::String path = utils::fs::join(cache_path_, name);
                        try
                        {
                            utils::fs::remove_all(path);
                            CV_LOG_WARNING(NULL, "Removed: " << path);
                        }
                        catch (const cv::Exception& e)
                        {
                            CV_LOG_ERROR(NULL, "Exception during removal of obsolete OpenCL cache directory: " << path << std::endl << e.what());
                        }
                    }
                }
            }
            catch (...)
            {
                CV_LOG_WARNING(NULL, "Can't check for obsolete OpenCL cache directories");
            }
        }

        CV_LOG_VERBOSE(NULL, 1, "  Result: " << (target_directory.empty() ? std::string("Failed") : target_directory));
        return target_directory;
    }

    static OpenCLBinaryCacheConfigurator& getSingletonInstance()
    {
        CV_SINGLETON_LAZY_INIT_REF(OpenCLBinaryCacheConfigurator, new OpenCLBinaryCacheConfigurator());
    }
};
class BinaryProgramFile
{
    enum { MAX_ENTRIES = 64 };

    typedef unsigned int uint32_t;

    struct CV_DECL_ALIGNED(4) FileHeader
    {
        uint32_t sourceSignatureSize;
        //char sourceSignature[];
    };

    struct CV_DECL_ALIGNED(4) FileTable
    {
        uint32_t numberOfEntries;
        //uint32_t firstEntryOffset[];
    };

    struct CV_DECL_ALIGNED(4) FileEntry
    {
        uint32_t nextEntryFileOffset; // 0 for the last entry in chain
        uint32_t keySize;
        uint32_t dataSize;
        //char key[];
        //char data[];
    };

    const std::string fileName_;
    const char* const sourceSignature_;
    const size_t sourceSignatureSize_;

    std::fstream f;

    uint32_t entryOffsets[MAX_ENTRIES];

    uint32_t getHash(const std::string& options)
    {
        uint64 hash = crc64((const uchar*)options.c_str(), options.size(), 0);
        return hash & (MAX_ENTRIES - 1);
    }

    inline size_t getFileSize()
    {
        size_t pos = (size_t)f.tellg();
        f.seekg(0, std::fstream::end);
        size_t fileSize = (size_t)f.tellg();
        f.seekg(pos, std::fstream::beg);
        return fileSize;
    }
    inline uint32_t readUInt32()
    {
        uint32_t res = 0;
        f.read((char*)&res, sizeof(uint32_t));
        CV_Assert(!f.fail());
        return res;
    }
    inline void writeUInt32(const uint32_t value)
    {
        uint32_t v = value;
        f.write((char*)&v, sizeof(uint32_t));
        CV_Assert(!f.fail());
    }

    inline void seekReadAbsolute(size_t pos)
    {
        f.seekg(pos, std::fstream::beg);
        CV_Assert(!f.fail());
    }
    inline void seekReadRelative(size_t pos)
    {
        f.seekg(pos, std::fstream::cur);
        CV_Assert(!f.fail());
    }

    inline void seekWriteAbsolute(size_t pos)
    {
        f.seekp(pos, std::fstream::beg);
        CV_Assert(!f.fail());
    }

    void clearFile()
    {
        f.close();
        if (0 != remove(fileName_.c_str()))
            CV_LOG_ERROR(NULL, "Can't remove: " << fileName_);
        return;
    }

public:
    BinaryProgramFile(const std::string& fileName, const char* sourceSignature)
        : fileName_(fileName), sourceSignature_(sourceSignature), sourceSignatureSize_(sourceSignature_ ? strlen(sourceSignature_) : 0)
    {
        CV_StaticAssert(sizeof(uint32_t) == 4, "");
        CV_Assert(sourceSignature_ != NULL);
        CV_Assert(sourceSignatureSize_ > 0);
        memset(entryOffsets, 0, sizeof(entryOffsets));

        f.rdbuf()->pubsetbuf(0, 0); // disable buffering
        f.open(fileName_.c_str(), std::ios::in|std::ios::out|std::ios::binary);
        if(f.is_open() && getFileSize() > 0)
        {
            bool isValid = false;
            try
            {
                uint32_t fileSourceSignatureSize = readUInt32();
                if (fileSourceSignatureSize == sourceSignatureSize_)
                {
                    cv::AutoBuffer<char> fileSourceSignature(fileSourceSignatureSize + 1);
                    f.read(fileSourceSignature.data(), fileSourceSignatureSize);
                    if (f.eof())
                    {
                        CV_LOG_ERROR(NULL, "Unexpected EOF");
                    }
                    else if (memcmp(sourceSignature, fileSourceSignature.data(), fileSourceSignatureSize) == 0)
                    {
                        isValid = true;
                    }
                }
                if (!isValid)
                {
                    CV_LOG_ERROR(NULL, "Source code signature/hash mismatch (program source code has been changed/updated)");
                }
            }
            catch (const cv::Exception& e)
            {
                CV_LOG_ERROR(NULL, "Can't open binary program file: " << fileName << " : " << e.what());
            }
            catch (...)
            {
                CV_LOG_ERROR(NULL, "Can't open binary program file: " << fileName << " : Unknown error");
            }
            if (!isValid)
            {
                clearFile();
            }
            else
            {
                seekReadAbsolute(0);
            }
        }
    }

    bool read(const std::string& key, std::vector<char>& buf)
    {
        if (!f.is_open())
            return false;

        size_t fileSize = getFileSize();
        if (fileSize == 0)
        {
            CV_LOG_ERROR(NULL, "Invalid file (empty): " << fileName_);
            clearFile();
            return false;
        }
        seekReadAbsolute(0);

        // bypass FileHeader
        uint32_t fileSourceSignatureSize = readUInt32();
        CV_Assert(fileSourceSignatureSize > 0);
        seekReadRelative(fileSourceSignatureSize);

        uint32_t numberOfEntries = readUInt32();
        CV_Assert(numberOfEntries > 0);
        if (numberOfEntries != MAX_ENTRIES)
        {
            CV_LOG_ERROR(NULL, "Invalid file: " << fileName_);
            clearFile();
            return false;
        }
        f.read((char*)&entryOffsets[0], sizeof(entryOffsets));
        CV_Assert(!f.fail());

        uint32_t entryNum = getHash(key);

        uint32_t entryOffset = entryOffsets[entryNum];
        FileEntry entry;
        while (entryOffset > 0)
        {
            seekReadAbsolute(entryOffset);
            //CV_StaticAssert(sizeof(entry) == sizeof(uint32_t) * 3, "");
            f.read((char*)&entry, sizeof(entry));
            CV_Assert(!f.fail());
            cv::AutoBuffer<char> fileKey(entry.keySize + 1);
            if (key.size() == entry.keySize)
            {
                if (entry.keySize > 0)
                {
                    f.read(fileKey.data(), entry.keySize);
                    CV_Assert(!f.fail());
                }
                if (memcmp(fileKey.data(), key.c_str(), entry.keySize) == 0)
                {
                    buf.resize(entry.dataSize);
                    f.read(&buf[0], entry.dataSize);
                    CV_Assert(!f.fail());
                    seekReadAbsolute(0);
                    CV_LOG_VERBOSE(NULL, 0, "Read...");
                    return true;
                }
            }
            if (entry.nextEntryFileOffset == 0)
                break;
            entryOffset = entry.nextEntryFileOffset;
        }
        return false;
    }

    bool write(const std::string& key, std::vector<char>& buf)
    {
        if (!f.is_open())
        {
            f.open(fileName_.c_str(), std::ios::in|std::ios::out|std::ios::binary);
            if (!f.is_open())
            {
                f.open(fileName_.c_str(), std::ios::out|std::ios::binary);
                if (!f.is_open())
                {
                    CV_LOG_ERROR(NULL, "Can't create file: " << fileName_);
                    return false;
                }
            }
        }

        size_t fileSize = getFileSize();
        if (fileSize == 0)
        {
            // Write header
            seekWriteAbsolute(0);
            writeUInt32((uint32_t)sourceSignatureSize_);
            f.write(sourceSignature_, sourceSignatureSize_);
            CV_Assert(!f.fail());

            writeUInt32(MAX_ENTRIES);
            memset(entryOffsets, 0, sizeof(entryOffsets));
            f.write((char*)entryOffsets, sizeof(entryOffsets));
            CV_Assert(!f.fail());
            f.flush();
            CV_Assert(!f.fail());
            f.close();
            f.open(fileName_.c_str(), std::ios::in|std::ios::out|std::ios::binary);
            CV_Assert(f.is_open());
            fileSize = getFileSize();
        }
        seekReadAbsolute(0);

        // bypass FileHeader
        uint32_t fileSourceSignatureSize = readUInt32();
        CV_Assert(fileSourceSignatureSize == sourceSignatureSize_);
        seekReadRelative(fileSourceSignatureSize);

        uint32_t numberOfEntries = readUInt32();
        CV_Assert(numberOfEntries > 0);
        if (numberOfEntries != MAX_ENTRIES)
        {
            CV_LOG_ERROR(NULL, "Invalid file: " << fileName_);
            clearFile();
            return false;
        }
        size_t tableEntriesOffset = (size_t)f.tellg();
        f.read((char*)&entryOffsets[0], sizeof(entryOffsets));
        CV_Assert(!f.fail());

        uint32_t entryNum = getHash(key);

        uint32_t entryOffset = entryOffsets[entryNum];
        FileEntry entry;
        while (entryOffset > 0)
        {
            seekReadAbsolute(entryOffset);
            //CV_StaticAssert(sizeof(entry) == sizeof(uint32_t) * 3, "");
            f.read((char*)&entry, sizeof(entry));
            CV_Assert(!f.fail());
            cv::AutoBuffer<char> fileKey(entry.keySize + 1);
            if (key.size() == entry.keySize)
            {
                if (entry.keySize > 0)
                {
                    f.read(fileKey.data(), entry.keySize);
                    CV_Assert(!f.fail());
                }
                if (0 == memcmp(fileKey.data(), key.c_str(), entry.keySize))
                {
                    // duplicate
                    CV_LOG_VERBOSE(NULL, 0, "Duplicate key ignored: " << fileName_);
                    return false;
                }
            }
            if (entry.nextEntryFileOffset == 0)
                break;
            entryOffset = entry.nextEntryFileOffset;
        }
        seekReadAbsolute(0);
        if (entryOffset > 0)
        {
            seekWriteAbsolute(entryOffset);
            entry.nextEntryFileOffset = (uint32_t)fileSize;
            f.write((char*)&entry, sizeof(entry));
            CV_Assert(!f.fail());
        }
        else
        {
            entryOffsets[entryNum] = (uint32_t)fileSize;
            seekWriteAbsolute(tableEntriesOffset);
            f.write((char*)entryOffsets, sizeof(entryOffsets));
            CV_Assert(!f.fail());
        }
        seekWriteAbsolute(fileSize);
        entry.nextEntryFileOffset = 0;
        entry.dataSize = (uint32_t)buf.size();
        entry.keySize = (uint32_t)key.size();
        f.write((char*)&entry, sizeof(entry));
        CV_Assert(!f.fail());
        f.write(key.c_str(), entry.keySize);
        CV_Assert(!f.fail());
        f.write(&buf[0], entry.dataSize);
        CV_Assert(!f.fail());
        f.flush();
        CV_Assert(!f.fail());
        CV_LOG_VERBOSE(NULL, 0, "Write... (" << buf.size() << " bytes)");
        return true;
    }
};
#endif // OPENCV_HAVE_FILESYSTEM_SUPPORT



struct OpenCLExecutionContext::Impl
{
    ocl::Context context_;
    int device_;  // device index in context
    ocl::Queue queue_;
    int useOpenCL_;

protected:
    Impl() = delete;

    void _init_device(cl_device_id deviceID)
    {
        CV_Assert(deviceID);
        int ndevices = (int)context_.ndevices();
        CV_Assert(ndevices > 0);
        bool found = false;
        for (int i = 0; i < ndevices; i++)
        {
            ocl::Device d = context_.device(i);
            cl_device_id dhandle = (cl_device_id)d.ptr();
            if (dhandle == deviceID)
            {
                device_ = i;
                found = true;
                break;
            }
        }
        CV_Assert(found && "OpenCL device can't work with passed OpenCL context");
    }

    void _init_device(const ocl::Device& device)
    {
        CV_Assert(device.ptr());
        int ndevices = (int)context_.ndevices();
        CV_Assert(ndevices > 0);
        bool found = false;
        for (int i = 0; i < ndevices; i++)
        {
            ocl::Device d = context_.device(i);
            if (d.getImpl() == device.getImpl())
            {
                device_ = i;
                found = true;
                break;
            }
        }
        CV_Assert(found && "OpenCL device can't work with passed OpenCL context");
    }

public:
    Impl(cl_platform_id platformID, cl_context context, cl_device_id deviceID)
        : device_(0), useOpenCL_(-1)
    {
        CV_UNUSED(platformID);
        CV_Assert(context);
        CV_Assert(deviceID);

        context_ = Context::fromHandle(context);
        _init_device(deviceID);
        queue_ = Queue(context_, context_.device(device_));
    }

    Impl(const ocl::Context& context, const ocl::Device& device, const ocl::Queue& queue)
        : device_(0), useOpenCL_(-1)
    {
        CV_Assert(context.ptr());
        CV_Assert(device.ptr());

        context_ = context;
        _init_device(device);
        queue_ = queue;
    }

    Impl(const ocl::Context& context, const ocl::Device& device)
        : device_(0), useOpenCL_(-1)
    {
        CV_Assert(context.ptr());
        CV_Assert(device.ptr());

        context_ = context;
        _init_device(device);
        queue_ = Queue(context_, context_.device(device_));
    }

    Impl(const ocl::Context& context, const int device, const ocl::Queue& queue)
        : context_(context)
        , device_(device)
        , queue_(queue)
        , useOpenCL_(-1)
    {
        // nothing
    }
    Impl(const Impl& other)
        : context_(other.context_)
        , device_(other.device_)
        , queue_(other.queue_)
        , useOpenCL_(-1)
    {
        // nothing
    }

    inline bool useOpenCL() const { return const_cast<Impl*>(this)->useOpenCL(); }
    bool useOpenCL()
    {
        if (useOpenCL_ < 0)
        {
            try
            {
                useOpenCL_ = 0;
                if (!context_.empty() && context_.ndevices() > 0)
                {
                    const Device& d = context_.device(device_);
                    useOpenCL_ = d.available();
                }
            }
            catch (const cv::Exception&)
            {
                // nothing
            }
            if (!useOpenCL_)
                CV_LOG_INFO(NULL, "OpenCL: can't use OpenCL execution context");
        }
        return useOpenCL_ > 0;
    }

    void setUseOpenCL(bool flag)
    {
        if (!flag)
            useOpenCL_ = 0;
        else
            useOpenCL_ = -1;
    }

    static const std::shared_ptr<Impl>& getInitializedExecutionContext()
    {
        CV_TRACE_FUNCTION();

        CV_LOG_INFO(NULL, "OpenCL: initializing thread execution context");

        static bool initialized = false;
        static std::shared_ptr<Impl> g_primaryExecutionContext;

        if (!initialized)
        {
            cv::AutoLock lock(getInitializationMutex());
            if (!initialized)
            {
                CV_LOG_INFO(NULL, "OpenCL: creating new execution context...");
                try
                {
                    Context c = ocl::Context::create(std::string());
                    if (c.ndevices())
                    {
                        int deviceId = 0;
                        auto& d = c.device(deviceId);
                        if (d.available())
                        {
                            auto q = ocl::Queue(c, d);
                            if (!q.ptr())
                            {
                                CV_LOG_ERROR(NULL, "OpenCL: Can't create default OpenCL queue");
                            }
                            else
                            {
                                g_primaryExecutionContext = std::make_shared<Impl>(c, deviceId, q);
                                CV_LOG_INFO(NULL, "OpenCL: device=" << d.name());
                            }
                        }
                        else
                        {
                            CV_LOG_ERROR(NULL, "OpenCL: OpenCL device is not available (CL_DEVICE_AVAILABLE returns false)");
                        }
                    }
                    else
                    {
                        CV_LOG_INFO(NULL, "OpenCL: context is not available/disabled");
                    }
                }
                catch (const std::exception& e)
                {
                    CV_LOG_INFO(NULL, "OpenCL: Can't initialize OpenCL context/device/queue: " << e.what());
                }
                catch (...)
                {
                    CV_LOG_WARNING(NULL, "OpenCL: Can't initialize OpenCL context/device/queue: unknown C++ exception");
                }
                initialized = true;
            }
        }
        return g_primaryExecutionContext;
    }
};

Context& OpenCLExecutionContext::getContext() const
{
    CV_Assert(p);
    return p->context_;
}
Device& OpenCLExecutionContext::getDevice() const
{
    CV_Assert(p);
    return p->context_.device(p->device_);
}
Queue& OpenCLExecutionContext::getQueue() const
{
    CV_Assert(p);
    return p->queue_;
}

bool OpenCLExecutionContext::useOpenCL() const
{
    if (p)
        return p->useOpenCL();
    return false;
}
void OpenCLExecutionContext::setUseOpenCL(bool flag)
{
    CV_Assert(p);
    p->setUseOpenCL(flag);
}

/* static */
OpenCLExecutionContext& OpenCLExecutionContext::getCurrent()
{
    CV_TRACE_FUNCTION();
    CoreTLSData& data = getCoreTlsData();
    OpenCLExecutionContext& c = data.oclExecutionContext;
    if (!data.oclExecutionContextInitialized)
    {
        data.oclExecutionContextInitialized = true;
        if (c.empty() && haveOpenCL())
            c.p = Impl::getInitializedExecutionContext();
    }
    return c;
}

/* static */
OpenCLExecutionContext& OpenCLExecutionContext::getCurrentRef()
{
    CV_TRACE_FUNCTION();
    CoreTLSData& data = getCoreTlsData();
    OpenCLExecutionContext& c = data.oclExecutionContext;
    return c;
}

void OpenCLExecutionContext::bind() const
{
    CV_TRACE_FUNCTION();
    CV_Assert(p);
    CoreTLSData& data = getCoreTlsData();
    data.oclExecutionContext = *this;
    data.oclExecutionContextInitialized = true;
    data.useOpenCL = p->useOpenCL_;  // propagate "-1", avoid call useOpenCL()
}


OpenCLExecutionContext OpenCLExecutionContext::cloneWithNewQueue() const
{
    CV_TRACE_FUNCTION();
    CV_Assert(p);
    const Queue q(getContext(), getDevice());
    return cloneWithNewQueue(q);
}

OpenCLExecutionContext OpenCLExecutionContext::cloneWithNewQueue(const ocl::Queue& q) const
{
    CV_TRACE_FUNCTION();
    CV_Assert(p);
    CV_Assert(q.ptr() != NULL);
    OpenCLExecutionContext c;
    c.p = std::make_shared<Impl>(p->context_, p->device_, q);
    return c;
}

/* static */
OpenCLExecutionContext OpenCLExecutionContext::create(const Context& context, const Device& device, const ocl::Queue& queue)
{
    CV_TRACE_FUNCTION();
    if (!haveOpenCL())
        CV_Error(cv::Error::OpenCLApiCallError, "OpenCL runtime is not available!");

    CV_Assert(!context.empty());
    CV_Assert(context.ptr());
    CV_Assert(!device.empty());
    CV_Assert(device.ptr());
    OpenCLExecutionContext ctx;
    ctx.p = std::make_shared<OpenCLExecutionContext::Impl>(context, device, queue);
    return ctx;

}

/* static */
OpenCLExecutionContext OpenCLExecutionContext::create(const Context& context, const Device& device)
{
    CV_TRACE_FUNCTION();
    if (!haveOpenCL())
        CV_Error(cv::Error::OpenCLApiCallError, "OpenCL runtime is not available!");

    CV_Assert(!context.empty());
    CV_Assert(context.ptr());
    CV_Assert(!device.empty());
    CV_Assert(device.ptr());
    OpenCLExecutionContext ctx;
    ctx.p = std::make_shared<OpenCLExecutionContext::Impl>(context, device);
    return ctx;

}

void OpenCLExecutionContext::release()
{
    CV_TRACE_FUNCTION();
    p.reset();
}



// true if we have initialized OpenCL subsystem with available platforms
static bool g_isOpenCLInitialized = false;
static bool g_isOpenCLAvailable = false;

bool haveOpenCL()
{
    CV_TRACE_FUNCTION();

    if (!g_isOpenCLInitialized)
    {
        CV_TRACE_REGION("Init_OpenCL_Runtime");
        std::string envPath = utils::getConfigurationParameterString("OPENCV_OPENCL_RUNTIME");
        if (!envPath.empty())
        {
            if (envPath == "disabled")
            {
                g_isOpenCLAvailable = false;
                g_isOpenCLInitialized = true;
                return false;
            }
        }

        cv::AutoLock lock(getInitializationMutex());
        CV_LOG_INFO(NULL, "Initialize OpenCL runtime...");
        try
        {
            cl_uint n = 0;
            g_isOpenCLAvailable = ::clGetPlatformIDs(0, NULL, &n) == CL_SUCCESS;
            g_isOpenCLAvailable &= n > 0;
            CV_LOG_INFO(NULL, "OpenCL: found " << n << " platforms");
        }
        catch (...)
        {
            g_isOpenCLAvailable = false;
        }
        g_isOpenCLInitialized = true;
    }
    return g_isOpenCLAvailable;
}

bool useOpenCL()
{
    CoreTLSData& data = getCoreTlsData();
    if (data.useOpenCL < 0)
    {
        try
        {
            data.useOpenCL = 0;
            if (haveOpenCL())
            {
                auto c = OpenCLExecutionContext::getCurrent();
                data.useOpenCL = c.useOpenCL();
            }
        }
        catch (...)
        {
            CV_LOG_INFO(NULL, "OpenCL: can't initialize thread OpenCL execution context");
        }
    }
    return data.useOpenCL > 0;
}

bool isOpenCLActivated()
{
    if (!g_isOpenCLAvailable)
        return false; // prevent unnecessary OpenCL activation via useOpenCL()->haveOpenCL() calls
    return useOpenCL();
}

void setUseOpenCL(bool flag)
{
    CV_TRACE_FUNCTION();

    CoreTLSData& data = getCoreTlsData();
    auto& c = OpenCLExecutionContext::getCurrentRef();
    if (!c.empty())
    {
        c.setUseOpenCL(flag);
        data.useOpenCL = c.useOpenCL();
    }
    else
    {
        if (!flag)
            data.useOpenCL = 0;
        else
            data.useOpenCL = -1; // enabled by default (if context is not initialized)
    }
}



#ifdef HAVE_CLAMDBLAS

class AmdBlasHelper
{
public:
    static AmdBlasHelper & getInstance()
    {
        CV_SINGLETON_LAZY_INIT_REF(AmdBlasHelper, new AmdBlasHelper())
    }

    bool isAvailable() const
    {
        return g_isAmdBlasAvailable;
    }

    ~AmdBlasHelper()
    {
        // Do not tear down clBLAS.
        // The user application may still use clBLAS even after OpenCV is unloaded.
        /*try
        {
            clblasTeardown();
        }
        catch (...) { }*/
    }

protected:
    AmdBlasHelper()
    {
        if (!g_isAmdBlasInitialized)
        {
            AutoLock lock(getInitializationMutex());

            if (!g_isAmdBlasInitialized)
            {
                if (haveOpenCL())
                {
                    try
                    {
                        g_isAmdBlasAvailable = clblasSetup() == clblasSuccess;
                    }
                    catch (...)
                    {
                        g_isAmdBlasAvailable = false;
                    }
                }
                else
                    g_isAmdBlasAvailable = false;

                g_isAmdBlasInitialized = true;
            }
        }
    }

private:
    static bool g_isAmdBlasInitialized;
    static bool g_isAmdBlasAvailable;
};

bool AmdBlasHelper::g_isAmdBlasAvailable = false;
bool AmdBlasHelper::g_isAmdBlasInitialized = false;

bool haveAmdBlas()
{
    return AmdBlasHelper::getInstance().isAvailable();
}

#else

bool haveAmdBlas()
{
    return false;
}

#endif

#ifdef HAVE_CLAMDFFT

class AmdFftHelper
{
public:
    static AmdFftHelper & getInstance()
    {
        CV_SINGLETON_LAZY_INIT_REF(AmdFftHelper, new AmdFftHelper())
    }

    bool isAvailable() const
    {
        return g_isAmdFftAvailable;
    }

    ~AmdFftHelper()
    {
        // Do not tear down clFFT.
        // The user application may still use clFFT even after OpenCV is unloaded.
        /*try
        {
            clfftTeardown();
        }
        catch (...) { }*/
    }

protected:
    AmdFftHelper()
    {
        if (!g_isAmdFftInitialized)
        {
            AutoLock lock(getInitializationMutex());

            if (!g_isAmdFftInitialized)
            {
                if (haveOpenCL())
                {
                    try
                    {
                        cl_uint major, minor, patch;
                        CV_Assert(clfftInitSetupData(&setupData) == CLFFT_SUCCESS);

                        // it throws exception in case AmdFft binaries are not found
                        CV_Assert(clfftGetVersion(&major, &minor, &patch) == CLFFT_SUCCESS);
                        g_isAmdFftAvailable = true;
                    }
                    catch (const Exception &)
                    {
                        g_isAmdFftAvailable = false;
                    }
                }
                else
                    g_isAmdFftAvailable = false;

                g_isAmdFftInitialized = true;
            }
        }
    }

private:
    static clfftSetupData setupData;
    static bool g_isAmdFftInitialized;
    static bool g_isAmdFftAvailable;
};

clfftSetupData AmdFftHelper::setupData;
bool AmdFftHelper::g_isAmdFftAvailable = false;
bool AmdFftHelper::g_isAmdFftInitialized = false;

bool haveAmdFft()
{
    return AmdFftHelper::getInstance().isAvailable();
}

#else

bool haveAmdFft()
{
    return false;
}

#endif

bool haveSVM()
{
#ifdef HAVE_OPENCL_SVM
    return true;
#else
    return false;
#endif
}

void finish()
{
    Queue::getDefault().finish();
}

/////////////////////////////////////////// Platform /////////////////////////////////////////////

struct Platform::Impl
{
    Impl()
    {
        refcount = 1;
        handle = 0;
        initialized = false;
    }

    ~Impl() {}

    void init()
    {
        if( !initialized )
        {
            //cl_uint num_entries
            cl_uint n = 0;
            if( clGetPlatformIDs(1, &handle, &n) != CL_SUCCESS || n == 0 )
                handle = 0;
            if( handle != 0 )
            {
                char buf[1000];
                size_t len = 0;
                CV_OCL_DBG_CHECK(clGetPlatformInfo(handle, CL_PLATFORM_VENDOR, sizeof(buf), buf, &len));
                buf[len] = '\0';
                vendor = String(buf);
            }

            initialized = true;
        }
    }

    IMPLEMENT_REFCOUNTABLE();

    cl_platform_id handle;
    String vendor;
    bool initialized;
};

Platform::Platform() CV_NOEXCEPT
{
    p = 0;
}

Platform::~Platform()
{
    if(p)
        p->release();
}

Platform::Platform(const Platform& pl)
{
    p = (Impl*)pl.p;
    if(p)
        p->addref();
}

Platform& Platform::operator = (const Platform& pl)
{
    Impl* newp = (Impl*)pl.p;
    if(newp)
        newp->addref();
    if(p)
        p->release();
    p = newp;
    return *this;
}

Platform::Platform(Platform&& pl) CV_NOEXCEPT
{
    p = pl.p;
    pl.p = nullptr;
}

Platform& Platform::operator = (Platform&& pl) CV_NOEXCEPT
{
    if (this != &pl) {
        if(p)
            p->release();
        p = pl.p;
        pl.p = nullptr;
    }
    return *this;
}

void* Platform::ptr() const
{
    return p ? p->handle : 0;
}

Platform& Platform::getDefault()
{
    CV_LOG_ONCE_WARNING(NULL, "OpenCL: Platform::getDefault() is deprecated and will be removed. Use cv::ocl::getPlatfomsInfo() for enumeration of available platforms");
    static Platform p;
    if( !p.p )
    {
        p.p = new Impl;
        p.p->init();
    }
    return p;
}

/////////////////////////////////////// Device ////////////////////////////////////////////

// Version has format:
//   OpenCL<space><major_version.minor_version><space><vendor-specific information>
// by specification
//   http://www.khronos.org/registry/cl/sdk/1.1/docs/man/xhtml/clGetDeviceInfo.html
//   http://www.khronos.org/registry/cl/sdk/1.2/docs/man/xhtml/clGetDeviceInfo.html
//   https://www.khronos.org/registry/OpenCL/sdk/1.1/docs/man/xhtml/clGetPlatformInfo.html
//   https://www.khronos.org/registry/OpenCL/sdk/1.2/docs/man/xhtml/clGetPlatformInfo.html
static void parseOpenCLVersion(const String &version, int &major, int &minor)
{
    major = minor = 0;
    if (10 >= version.length())
        return;
    const char *pstr = version.c_str();
    if (0 != strncmp(pstr, "OpenCL ", 7))
        return;
    size_t ppos = version.find('.', 7);
    if (String::npos == ppos)
        return;
    String temp = version.substr(7, ppos - 7);
    major = atoi(temp.c_str());
    temp = version.substr(ppos + 1);
    minor = atoi(temp.c_str());
}

struct Device::Impl
{
    Impl(void* d)
        : refcount(1)
        , handle(0)
    {
        try
        {
            cl_device_id device = (cl_device_id)d;
            _init(device);
            CV_OCL_CHECK(clRetainDevice(device));  // increment reference counter on success only
        }
        catch (...)
        {
            throw;
        }
    }

    void _init(cl_device_id d)
    {
        handle = (cl_device_id)d;

        name_ = getStrProp(CL_DEVICE_NAME);
        version_ = getStrProp(CL_DEVICE_VERSION);
        extensions_ = getStrProp(CL_DEVICE_EXTENSIONS);
        doubleFPConfig_ = getProp<cl_device_fp_config, int>(CL_DEVICE_DOUBLE_FP_CONFIG);
        halfFPConfig_ = getProp<cl_device_fp_config, int>(CL_DEVICE_HALF_FP_CONFIG);
        hostUnifiedMemory_ = getBoolProp(CL_DEVICE_HOST_UNIFIED_MEMORY);
        maxComputeUnits_ = getProp<cl_uint, int>(CL_DEVICE_MAX_COMPUTE_UNITS);
        maxWorkGroupSize_ = getProp<size_t, size_t>(CL_DEVICE_MAX_WORK_GROUP_SIZE);
        type_ = getProp<cl_device_type, int>(CL_DEVICE_TYPE);
        driverVersion_ = getStrProp(CL_DRIVER_VERSION);
        addressBits_ = getProp<cl_uint, int>(CL_DEVICE_ADDRESS_BITS);

        String deviceVersion_ = getStrProp(CL_DEVICE_VERSION);
        parseOpenCLVersion(deviceVersion_, deviceVersionMajor_, deviceVersionMinor_);

        size_t pos = 0;
        while (pos < extensions_.size())
        {
            size_t pos2 = extensions_.find(' ', pos);
            if (pos2 == String::npos)
                pos2 = extensions_.size();
            if (pos2 > pos)
            {
                std::string extensionName = extensions_.substr(pos, pos2 - pos);
                extensions_set_.insert(extensionName);
            }
            pos = pos2 + 1;
        }

        khr_fp64_support_ = isExtensionSupported("cl_khr_fp64");
        khr_fp16_support_ = isExtensionSupported("cl_khr_fp16");

        intelSubgroupsSupport_ = isExtensionSupported("cl_intel_subgroups");

        vendorName_ = getStrProp(CL_DEVICE_VENDOR);
        if (vendorName_ == "Advanced Micro Devices, Inc." ||
            vendorName_ == "AMD")
            vendorID_ = VENDOR_AMD;
        else if (vendorName_ == "Intel(R) Corporation" || vendorName_ == "Intel" || vendorName_ == "Intel Inc." || strstr(name_.c_str(), "Iris") != 0)
            vendorID_ = VENDOR_INTEL;
        else if (vendorName_ == "NVIDIA Corporation")
            vendorID_ = VENDOR_NVIDIA;
        else
            vendorID_ = UNKNOWN_VENDOR;

        const size_t CV_OPENCL_DEVICE_MAX_WORK_GROUP_SIZE = utils::getConfigurationParameterSizeT("OPENCV_OPENCL_DEVICE_MAX_WORK_GROUP_SIZE", 0);
        if (CV_OPENCL_DEVICE_MAX_WORK_GROUP_SIZE > 0)
        {
            const size_t new_maxWorkGroupSize = std::min(maxWorkGroupSize_, CV_OPENCL_DEVICE_MAX_WORK_GROUP_SIZE);
            if (new_maxWorkGroupSize != maxWorkGroupSize_)
                CV_LOG_WARNING(NULL, "OpenCL: using workgroup size: " << new_maxWorkGroupSize << " (was " << maxWorkGroupSize_ << ")");
            maxWorkGroupSize_ = new_maxWorkGroupSize;
        }
#if 0
        if (isExtensionSupported("cl_khr_spir"))
        {
#ifndef CL_DEVICE_SPIR_VERSIONS
#define CL_DEVICE_SPIR_VERSIONS                     0x40E0
#endif
            cv::String spir_versions = getStrProp(CL_DEVICE_SPIR_VERSIONS);
            std::cout << spir_versions << std::endl;
        }
#endif
    }

    ~Impl()
    {
#ifdef _WIN32
        if (!cv::__termination)
#endif
        {
            if (handle)
            {
                CV_OCL_CHECK(clReleaseDevice(handle));
                handle = 0;
            }
        }
    }

    template<typename _TpCL, typename _TpOut>
    _TpOut getProp(cl_device_info prop) const
    {
        _TpCL temp=_TpCL();
        size_t sz = 0;

        return clGetDeviceInfo(handle, prop, sizeof(temp), &temp, &sz) == CL_SUCCESS &&
            sz == sizeof(temp) ? _TpOut(temp) : _TpOut();
    }

    bool getBoolProp(cl_device_info prop) const
    {
        cl_bool temp = CL_FALSE;
        size_t sz = 0;

        return clGetDeviceInfo(handle, prop, sizeof(temp), &temp, &sz) == CL_SUCCESS &&
            sz == sizeof(temp) ? temp != 0 : false;
    }

    String getStrProp(cl_device_info prop) const
    {
        char buf[4096];
        size_t sz=0;
        return clGetDeviceInfo(handle, prop, sizeof(buf)-16, buf, &sz) == CL_SUCCESS &&
            sz < sizeof(buf) ? String(buf) : String();
    }

    bool isExtensionSupported(const std::string& extensionName) const
    {
        return extensions_set_.count(extensionName) > 0;
    }


    IMPLEMENT_REFCOUNTABLE();

    cl_device_id handle;

    String name_;
    String version_;
    std::string extensions_;
    int doubleFPConfig_;
    bool khr_fp64_support_;
    int halfFPConfig_;
    bool khr_fp16_support_;
    bool hostUnifiedMemory_;
    int maxComputeUnits_;
    size_t maxWorkGroupSize_;
    int type_;
    int addressBits_;
    int deviceVersionMajor_;
    int deviceVersionMinor_;
    String driverVersion_;
    String vendorName_;
    int vendorID_;
    bool intelSubgroupsSupport_;

    std::set<std::string> extensions_set_;
};


Device::Device() CV_NOEXCEPT
{
    p = 0;
}

Device::Device(void* d)
{
    p = 0;
    set(d);
}

Device::Device(const Device& d)
{
    p = d.p;
    if(p)
        p->addref();
}

Device& Device::operator = (const Device& d)
{
    Impl* newp = (Impl*)d.p;
    if(newp)
        newp->addref();
    if(p)
        p->release();
    p = newp;
    return *this;
}

Device::Device(Device&& d) CV_NOEXCEPT
{
    p = d.p;
    d.p = nullptr;
}

Device& Device::operator = (Device&& d) CV_NOEXCEPT
{
    if (this != &d) {
        if(p)
            p->release();
        p = d.p;
        d.p = nullptr;
    }
    return *this;
}

Device::~Device()
{
    if(p)
        p->release();
}

void Device::set(void* d)
{
    if(p)
        p->release();
    p = new Impl(d);
    if (p->handle)
    {
        CV_OCL_CHECK(clReleaseDevice((cl_device_id)d));
    }
}

Device Device::fromHandle(void* d)
{
    Device device(d);
    return device;
}

void* Device::ptr() const
{
    return p ? p->handle : 0;
}

String Device::name() const
{ return p ? p->name_ : String(); }

String Device::extensions() const
{ return p ? String(p->extensions_) : String(); }

bool Device::isExtensionSupported(const String& extensionName) const
{ return p ? p->isExtensionSupported(extensionName) : false; }

String Device::version() const
{ return p ? p->version_ : String(); }

String Device::vendorName() const
{ return p ? p->vendorName_ : String(); }

int Device::vendorID() const
{ return p ? p->vendorID_ : 0; }

String Device::OpenCL_C_Version() const
{ return p ? p->getStrProp(CL_DEVICE_OPENCL_C_VERSION) : String(); }

String Device::OpenCLVersion() const
{ return p ? p->getStrProp(CL_DEVICE_VERSION) : String(); }

int Device::deviceVersionMajor() const
{ return p ? p->deviceVersionMajor_ : 0; }

int Device::deviceVersionMinor() const
{ return p ? p->deviceVersionMinor_ : 0; }

String Device::driverVersion() const
{ return p ? p->driverVersion_ : String(); }

int Device::type() const
{ return p ? p->type_ : 0; }

int Device::addressBits() const
{ return p ? p->addressBits_ : 0; }

bool Device::available() const
{ return p ? p->getBoolProp(CL_DEVICE_AVAILABLE) : false; }

bool Device::compilerAvailable() const
{ return p ? p->getBoolProp(CL_DEVICE_COMPILER_AVAILABLE) : false; }

bool Device::linkerAvailable() const
#ifdef CL_VERSION_1_2
{ return p ? p->getBoolProp(CL_DEVICE_LINKER_AVAILABLE) : false; }
#else
{ CV_REQUIRE_OPENCL_1_2_ERROR; }
#endif

int Device::doubleFPConfig() const
{ return p ? p->doubleFPConfig_ : 0; }

int Device::singleFPConfig() const
{ return p ? p->getProp<cl_device_fp_config, int>(CL_DEVICE_SINGLE_FP_CONFIG) : 0; }

int Device::halfFPConfig() const
{ return p ? p->halfFPConfig_ : 0; }

bool Device::hasFP64() const
{ return p ? p->khr_fp64_support_ : false; }
bool Device::hasFP16() const
{ return p ? p->khr_fp16_support_ : false; }

bool Device::endianLittle() const
{ return p ? p->getBoolProp(CL_DEVICE_ENDIAN_LITTLE) : false; }

bool Device::errorCorrectionSupport() const
{ return p ? p->getBoolProp(CL_DEVICE_ERROR_CORRECTION_SUPPORT) : false; }

int Device::executionCapabilities() const
{ return p ? p->getProp<cl_device_exec_capabilities, int>(CL_DEVICE_EXECUTION_CAPABILITIES) : 0; }

size_t Device::globalMemCacheSize() const
{ return p ? p->getProp<cl_ulong, size_t>(CL_DEVICE_GLOBAL_MEM_CACHE_SIZE) : 0; }

int Device::globalMemCacheType() const
{ return p ? p->getProp<cl_device_mem_cache_type, int>(CL_DEVICE_GLOBAL_MEM_CACHE_TYPE) : 0; }

int Device::globalMemCacheLineSize() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_GLOBAL_MEM_CACHELINE_SIZE) : 0; }

size_t Device::globalMemSize() const
{ return p ? p->getProp<cl_ulong, size_t>(CL_DEVICE_GLOBAL_MEM_SIZE) : 0; }

size_t Device::localMemSize() const
{ return p ? p->getProp<cl_ulong, size_t>(CL_DEVICE_LOCAL_MEM_SIZE) : 0; }

int Device::localMemType() const
{ return p ? p->getProp<cl_device_local_mem_type, int>(CL_DEVICE_LOCAL_MEM_TYPE) : 0; }

bool Device::hostUnifiedMemory() const
{ return p ? p->hostUnifiedMemory_ : false; }

bool Device::imageSupport() const
{ return p ? p->getBoolProp(CL_DEVICE_IMAGE_SUPPORT) : false; }

bool Device::imageFromBufferSupport() const
{
    return p ? p->isExtensionSupported("cl_khr_image2d_from_buffer") : false;
}

uint Device::imagePitchAlignment() const
{
#ifdef CL_DEVICE_IMAGE_PITCH_ALIGNMENT
    return p ? p->getProp<cl_uint, uint>(CL_DEVICE_IMAGE_PITCH_ALIGNMENT) : 0;
#else
    return 0;
#endif
}

uint Device::imageBaseAddressAlignment() const
{
#ifdef CL_DEVICE_IMAGE_BASE_ADDRESS_ALIGNMENT
    return p ? p->getProp<cl_uint, uint>(CL_DEVICE_IMAGE_BASE_ADDRESS_ALIGNMENT) : 0;
#else
    return 0;
#endif
}

size_t Device::image2DMaxWidth() const
{ return p ? p->getProp<size_t, size_t>(CL_DEVICE_IMAGE2D_MAX_WIDTH) : 0; }

size_t Device::image2DMaxHeight() const
{ return p ? p->getProp<size_t, size_t>(CL_DEVICE_IMAGE2D_MAX_HEIGHT) : 0; }

size_t Device::image3DMaxWidth() const
{ return p ? p->getProp<size_t, size_t>(CL_DEVICE_IMAGE3D_MAX_WIDTH) : 0; }

size_t Device::image3DMaxHeight() const
{ return p ? p->getProp<size_t, size_t>(CL_DEVICE_IMAGE3D_MAX_HEIGHT) : 0; }

size_t Device::image3DMaxDepth() const
{ return p ? p->getProp<size_t, size_t>(CL_DEVICE_IMAGE3D_MAX_DEPTH) : 0; }

size_t Device::imageMaxBufferSize() const
#ifdef CL_VERSION_1_2
{ return p ? p->getProp<size_t, size_t>(CL_DEVICE_IMAGE_MAX_BUFFER_SIZE) : 0; }
#else
{ CV_REQUIRE_OPENCL_1_2_ERROR; }
#endif

size_t Device::imageMaxArraySize() const
#ifdef CL_VERSION_1_2
{ return p ? p->getProp<size_t, size_t>(CL_DEVICE_IMAGE_MAX_ARRAY_SIZE) : 0; }
#else
{ CV_REQUIRE_OPENCL_1_2_ERROR; }
#endif

bool Device::intelSubgroupsSupport() const
{ return p ? p->intelSubgroupsSupport_ : false; }

int Device::maxClockFrequency() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_MAX_CLOCK_FREQUENCY) : 0; }

int Device::maxComputeUnits() const
{ return p ? p->maxComputeUnits_ : 0; }

int Device::maxConstantArgs() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_MAX_CONSTANT_ARGS) : 0; }

size_t Device::maxConstantBufferSize() const
{ return p ? p->getProp<cl_ulong, size_t>(CL_DEVICE_MAX_CONSTANT_BUFFER_SIZE) : 0; }

size_t Device::maxMemAllocSize() const
{ return p ? p->getProp<cl_ulong, size_t>(CL_DEVICE_MAX_MEM_ALLOC_SIZE) : 0; }

size_t Device::maxParameterSize() const
{ return p ? p->getProp<cl_ulong, size_t>(CL_DEVICE_MAX_PARAMETER_SIZE) : 0; }

int Device::maxReadImageArgs() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_MAX_READ_IMAGE_ARGS) : 0; }

int Device::maxWriteImageArgs() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_MAX_WRITE_IMAGE_ARGS) : 0; }

int Device::maxSamplers() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_MAX_SAMPLERS) : 0; }

size_t Device::maxWorkGroupSize() const
{ return p ? p->maxWorkGroupSize_ : 0; }

int Device::maxWorkItemDims() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_MAX_WORK_ITEM_DIMENSIONS) : 0; }

void Device::maxWorkItemSizes(size_t* sizes) const
{
    if(p)
    {
        const int MAX_DIMS = 32;
        size_t retsz = 0;
        CV_OCL_DBG_CHECK(clGetDeviceInfo(p->handle, CL_DEVICE_MAX_WORK_ITEM_SIZES,
                MAX_DIMS*sizeof(sizes[0]), &sizes[0], &retsz));
    }
}

int Device::memBaseAddrAlign() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_MEM_BASE_ADDR_ALIGN) : 0; }

int Device::nativeVectorWidthChar() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_NATIVE_VECTOR_WIDTH_CHAR) : 0; }

int Device::nativeVectorWidthShort() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_NATIVE_VECTOR_WIDTH_SHORT) : 0; }

int Device::nativeVectorWidthInt() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_NATIVE_VECTOR_WIDTH_INT) : 0; }

int Device::nativeVectorWidthLong() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_NATIVE_VECTOR_WIDTH_LONG) : 0; }

int Device::nativeVectorWidthFloat() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_NATIVE_VECTOR_WIDTH_FLOAT) : 0; }

int Device::nativeVectorWidthDouble() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_NATIVE_VECTOR_WIDTH_DOUBLE) : 0; }

int Device::nativeVectorWidthHalf() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_NATIVE_VECTOR_WIDTH_HALF) : 0; }

int Device::preferredVectorWidthChar() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_PREFERRED_VECTOR_WIDTH_CHAR) : 0; }

int Device::preferredVectorWidthShort() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_PREFERRED_VECTOR_WIDTH_SHORT) : 0; }

int Device::preferredVectorWidthInt() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_PREFERRED_VECTOR_WIDTH_INT) : 0; }

int Device::preferredVectorWidthLong() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_PREFERRED_VECTOR_WIDTH_LONG) : 0; }

int Device::preferredVectorWidthFloat() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_PREFERRED_VECTOR_WIDTH_FLOAT) : 0; }

int Device::preferredVectorWidthDouble() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_PREFERRED_VECTOR_WIDTH_DOUBLE) : 0; }

int Device::preferredVectorWidthHalf() const
{ return p ? p->getProp<cl_uint, int>(CL_DEVICE_PREFERRED_VECTOR_WIDTH_HALF) : 0; }

size_t Device::printfBufferSize() const
#ifdef CL_VERSION_1_2
{ return p ? p->getProp<size_t, size_t>(CL_DEVICE_PRINTF_BUFFER_SIZE) : 0; }
#else
{ CV_REQUIRE_OPENCL_1_2_ERROR; }
#endif


size_t Device::profilingTimerResolution() const
{ return p ? p->getProp<size_t, size_t>(CL_DEVICE_PROFILING_TIMER_RESOLUTION) : 0; }

const Device& Device::getDefault()
{
    auto& c = OpenCLExecutionContext::getCurrent();
    if (!c.empty())
    {
        return c.getDevice();
    }

    static Device dummy;
    return dummy;
}

////////////////////////////////////// Context ///////////////////////////////////////////////////

template <typename Functor, typename ObjectType>
inline cl_int getStringInfo(Functor f, ObjectType obj, cl_uint name, std::string& param)
{
    ::size_t required;
    cl_int err = f(obj, name, 0, NULL, &required);
    if (err != CL_SUCCESS)
        return err;

    param.clear();
    if (required > 0)
    {
        AutoBuffer<char> buf(required + 1);
        char* ptr = buf.data(); // cleanup is not needed
        err = f(obj, name, required, ptr, NULL);
        if (err != CL_SUCCESS)
            return err;
        param = ptr;
    }

    return CL_SUCCESS;
}

static void split(const std::string &s, char delim, std::vector<std::string> &elems)
{
    elems.clear();
    if (s.size() == 0)
        return;
    std::istringstream ss(s);
    std::string item;
    while (!ss.eof())
    {
        std::getline(ss, item, delim);
        elems.push_back(item);
    }
}

// Layout: <Platform>:<CPU|GPU|ACCELERATOR|nothing=GPU/CPU>:<deviceName>
// Sample: AMD:GPU:
// Sample: AMD:GPU:Tahiti
// Sample: :GPU|CPU: = '' = ':' = '::'
static bool parseOpenCLDeviceConfiguration(const std::string& configurationStr,
        std::string& platform, std::vector<std::string>& deviceTypes, std::string& deviceNameOrID)
{
    std::vector<std::string> parts;
    split(configurationStr, ':', parts);
    if (parts.size() > 3)
    {
        CV_LOG_ERROR(NULL, "OpenCL: Invalid configuration string for OpenCL device: " << configurationStr);
        return false;
    }
    if (parts.size() > 2)
        deviceNameOrID = parts[2];
    if (parts.size() > 1)
    {
        split(parts[1], '|', deviceTypes);
    }
    if (parts.size() > 0)
    {
        platform = parts[0];
    }
    return true;
}

static cl_device_id selectOpenCLDevice(const std::string & configuration_ = std::string())
{
    std::string platform, deviceName;
    std::vector<std::string> deviceTypes;

    std::string configuration(configuration_);
    if (configuration.empty())
        configuration = utils::getConfigurationParameterString("OPENCV_OPENCL_DEVICE");

    if (!configuration.empty() &&
            (configuration == "disabled" ||
             !parseOpenCLDeviceConfiguration(configuration, platform, deviceTypes, deviceName)
            ))
        return NULL;

    bool isID = false;
    int deviceID = -1;
    if (deviceName.length() == 1)
    // We limit ID range to 0..9, because we want to write:
    // - '2500' to mean i5-2500
    // - '8350' to mean AMD FX-8350
    // - '650' to mean GeForce 650
    // To extend ID range change condition to '> 0'
    {
        isID = true;
        for (size_t i = 0; i < deviceName.length(); i++)
        {
            if (!isdigit(deviceName[i]))
            {
                isID = false;
                break;
            }
        }
        if (isID)
        {
            deviceID = atoi(deviceName.c_str());
            if (deviceID < 0)
                return NULL;
        }
    }

    std::vector<cl_platform_id> platforms;
    {
        cl_uint numPlatforms = 0;
        CV_OCL_DBG_CHECK(clGetPlatformIDs(0, NULL, &numPlatforms));

        if (numPlatforms == 0)
            return NULL;
        platforms.resize((size_t)numPlatforms);
        CV_OCL_DBG_CHECK(clGetPlatformIDs(numPlatforms, &platforms[0], &numPlatforms));
        platforms.resize(numPlatforms);
    }

    if (platform.length() > 0)
    {
        for (std::vector<cl_platform_id>::iterator currentPlatform = platforms.begin(); currentPlatform != platforms.end();)
        {
            std::string name;
            CV_OCL_DBG_CHECK(getStringInfo(clGetPlatformInfo, *currentPlatform, CL_PLATFORM_NAME, name));
            if (name.find(platform) != std::string::npos)
            {
                ++currentPlatform;
            }
            else
            {
                currentPlatform = platforms.erase(currentPlatform);
            }
        }
        if (platforms.size() == 0)
        {
            CV_LOG_ERROR(NULL, "OpenCL: Can't find OpenCL platform by name: " << platform);
            goto not_found;
        }
    }
    if (deviceTypes.size() == 0)
    {
        if (!isID)
        {
            deviceTypes.push_back("GPU");
            if (!configuration.empty())
                deviceTypes.push_back("CPU");
        }
        else
            deviceTypes.push_back("ALL");
    }
    for (size_t t = 0; t < deviceTypes.size(); t++)
    {
        int deviceType = 0;
        std::string tempStrDeviceType = deviceTypes[t];
        std::transform(tempStrDeviceType.begin(), tempStrDeviceType.end(), tempStrDeviceType.begin(), details::char_tolower);

        if (tempStrDeviceType == "gpu" || tempStrDeviceType == "dgpu" || tempStrDeviceType == "igpu")
            deviceType = Device::TYPE_GPU;
        else if (tempStrDeviceType == "cpu")
            deviceType = Device::TYPE_CPU;
        else if (tempStrDeviceType == "accelerator")
            deviceType = Device::TYPE_ACCELERATOR;
        else if (tempStrDeviceType == "all")
            deviceType = Device::TYPE_ALL;
        else
        {
            CV_LOG_ERROR(NULL, "OpenCL: Unsupported device type for OpenCL device (GPU, CPU, ACCELERATOR): " << deviceTypes[t]);
            goto not_found;
        }

        std::vector<cl_device_id> devices;
        for (std::vector<cl_platform_id>::iterator currentPlatform = platforms.begin(); currentPlatform != platforms.end(); ++currentPlatform)
        {
            cl_uint count = 0;
            cl_int status = clGetDeviceIDs(*currentPlatform, deviceType, 0, NULL, &count);
            if (!(status == CL_SUCCESS || status == CL_DEVICE_NOT_FOUND))
            {
                CV_OCL_DBG_CHECK_RESULT(status, "clGetDeviceIDs get count");
            }
            if (count == 0)
                continue;
            size_t base = devices.size();
            devices.resize(base + count);
            status = clGetDeviceIDs(*currentPlatform, deviceType, count, &devices[base], &count);
            if (!(status == CL_SUCCESS || status == CL_DEVICE_NOT_FOUND))
            {
                CV_OCL_DBG_CHECK_RESULT(status, "clGetDeviceIDs get IDs");
            }
        }

        for (size_t i = (isID ? deviceID : 0);
             (isID ? (i == (size_t)deviceID) : true) && (i < devices.size());
             i++)
        {
            std::string name;
            CV_OCL_DBG_CHECK(getStringInfo(clGetDeviceInfo, devices[i], CL_DEVICE_NAME, name));
            cl_bool useGPU = true;
            if(tempStrDeviceType == "dgpu" || tempStrDeviceType == "igpu")
            {
                cl_bool isIGPU = CL_FALSE;
                CV_OCL_DBG_CHECK(clGetDeviceInfo(devices[i], CL_DEVICE_HOST_UNIFIED_MEMORY, sizeof(isIGPU), &isIGPU, NULL));
                useGPU = tempStrDeviceType == "dgpu" ? !isIGPU : isIGPU;
            }
            if ( (isID || name.find(deviceName) != std::string::npos) && useGPU)
            {
                // TODO check for OpenCL 1.1
                return devices[i];
            }
        }
    }

not_found:
    if (configuration.empty())
        return NULL; // suppress messages on stderr

    std::ostringstream msg;
    msg << "ERROR: Requested OpenCL device not found, check configuration: '" << configuration << "'" << std::endl
        << "    Platform: " << (platform.length() == 0 ? "any" : platform) << std::endl
        << "    Device types:";
    for (size_t t = 0; t < deviceTypes.size(); t++)
        msg << ' ' << deviceTypes[t];

    msg << std::endl << "    Device name: " << (deviceName.length() == 0 ? "any" : deviceName);

    CV_LOG_ERROR(NULL, msg.str());
    return NULL;
}

#ifdef HAVE_OPENCL_SVM
namespace svm {

enum AllocatorFlags { // don't use first 16 bits
        OPENCL_SVM_COARSE_GRAIN_BUFFER = 1 << 16, // clSVMAlloc + SVM map/unmap
        OPENCL_SVM_FINE_GRAIN_BUFFER = 2 << 16, // clSVMAlloc
        OPENCL_SVM_FINE_GRAIN_SYSTEM = 3 << 16, // direct access
        OPENCL_SVM_BUFFER_MASK = 3 << 16,
        OPENCL_SVM_BUFFER_MAP = 4 << 16
};

static bool checkForceSVMUmatUsage()
{
    static bool initialized = false;
    static bool force = false;
    if (!initialized)
    {
        force = utils::getConfigurationParameterBool("OPENCV_OPENCL_SVM_FORCE_UMAT_USAGE", false);
        initialized = true;
    }
    return force;
}
static bool checkDisableSVMUMatUsage()
{
    static bool initialized = false;
    static bool force = false;
    if (!initialized)
    {
        force = utils::getConfigurationParameterBool("OPENCV_OPENCL_SVM_DISABLE_UMAT_USAGE", false);
        initialized = true;
    }
    return force;
}
static bool checkDisableSVM()
{
    static bool initialized = false;
    static bool force = false;
    if (!initialized)
    {
        force = utils::getConfigurationParameterBool("OPENCV_OPENCL_SVM_DISABLE", false);
        initialized = true;
    }
    return force;
}
// see SVMCapabilities
static unsigned int getSVMCapabilitiesMask()
{
    static bool initialized = false;
    static unsigned int mask = 0;
    if (!initialized)
    {
        const std::string envValue = utils::getConfigurationParameterString("OPENCV_OPENCL_SVM_CAPABILITIES_MASK");
        if (envValue.empty())
        {
            return ~0U; // all bits 1
        }
        mask = atoi(envValue.c_str());
        initialized = true;
    }
    return mask;
}
} // namespace
#endif

static size_t getProgramCountLimit()
{
    static bool initialized = false;
    static size_t count = 0;
    if (!initialized)
    {
        count = utils::getConfigurationParameterSizeT("OPENCV_OPENCL_PROGRAM_CACHE", 0);
        initialized = true;
    }
    return count;
}

static int g_contextId = 0;

class OpenCLBufferPoolImpl;
class OpenCLSVMBufferPoolImpl;

struct Context::Impl
{
    static Context::Impl* get(Context& context) { return context.p; }

    typedef std::deque<Context::Impl*> container_t;
    static container_t& getGlobalContainer()
    {
        // never delete this container (Impl lifetime is greater due to TLS storage)
        static container_t* g_contexts = new container_t();
        return *g_contexts;
    }

protected:
    Impl(const std::string& configuration_)
        : refcount(1)
        , contextId(CV_XADD(&g_contextId, 1))
        , configuration(configuration_)
        , handle(0)
#ifdef HAVE_OPENCL_SVM
        , svmInitialized(false)
#endif
    {
        if (!haveOpenCL())
            CV_Error(cv::Error::OpenCLApiCallError, "OpenCL runtime is not available!");

        cv::AutoLock lock(cv::getInitializationMutex());
        auto& container = getGlobalContainer();
        container.resize(std::max(container.size(), (size_t)contextId + 1));
        container[contextId] = this;
    }

    ~Impl()
    {
#ifdef _WIN32
        if (!cv::__termination)
#endif
        {
            if (handle)
            {
                CV_OCL_DBG_CHECK(clReleaseContext(handle));
                handle = NULL;
            }
            devices.clear();
        }

        userContextStorage.clear();

        {
            cv::AutoLock lock(cv::getInitializationMutex());
            auto& container = getGlobalContainer();
            CV_CheckLT((size_t)contextId, container.size(), "");
            container[contextId] = NULL;
        }
    }

    void init_device_list()
    {
        CV_Assert(handle);

        cl_uint ndevices = 0;
        CV_OCL_CHECK(clGetContextInfo(handle, CL_CONTEXT_NUM_DEVICES, sizeof(ndevices), &ndevices, NULL));
        CV_Assert(ndevices > 0);

        cv::AutoBuffer<cl_device_id> cl_devices(ndevices);
        size_t devices_ret_size = 0;
        CV_OCL_CHECK(clGetContextInfo(handle, CL_CONTEXT_DEVICES, cl_devices.size() * sizeof(cl_device_id), &cl_devices[0], &devices_ret_size));
        CV_CheckEQ(devices_ret_size, cl_devices.size() * sizeof(cl_device_id), "");

        devices.clear();
        for (unsigned i = 0; i < ndevices; i++)
        {
            devices.emplace_back(Device::fromHandle(cl_devices[i]));
        }
    }

    void __init_buffer_pools();  // w/o synchronization
    void _init_buffer_pools() const
    {
        if (!bufferPool_)
        {
            cv::AutoLock lock(cv::getInitializationMutex());
            if (!bufferPool_)
            {
                const_cast<Impl*>(this)->__init_buffer_pools();
            }
        }
    }
public:
    static Impl* findContext(const std::string& configuration)
    {
        CV_TRACE_FUNCTION();
        cv::AutoLock lock(cv::getInitializationMutex());
        auto& container = getGlobalContainer();
        if (configuration.empty() && !container.empty())
            return container[0];
        for (auto it = container.begin(); it != container.end(); ++it)
        {
            Impl* i = *it;
            if (i && i->configuration == configuration)
            {
                return i;
            }
        }
        return NULL;
    }

    static Impl* findOrCreateContext(const std::string& configuration_)
    {
        CV_TRACE_FUNCTION();
        std::string configuration = configuration_;
        if (configuration_.empty())
        {
            const std::string c = utils::getConfigurationParameterString("OPENCV_OPENCL_DEVICE");
            if (!c.empty())
                configuration = c;
        }
        Impl* impl = findContext(configuration);
        if (impl)
        {
            CV_LOG_INFO(NULL, "OpenCL: reuse context@" << impl->contextId << " for configuration: " << configuration)
            impl->addref();
            return impl;
        }

        cl_device_id d = selectOpenCLDevice(configuration);
        if (d == NULL)
            return NULL;

        impl = new Impl(configuration);
        try
        {
            impl->createFromDevice(d);
            if (impl->handle)
                return impl;
            delete impl;
            return NULL;
        }
        catch (...)
        {
            delete impl;
            throw;
        }
    }

    static Impl* findOrCreateContext(cl_context h)
    {
        CV_TRACE_FUNCTION();

        CV_Assert(h);

        std::string configuration = cv::format("@ctx-%p", (void*)h);
        Impl* impl = findContext(configuration);
        if (impl)
        {
            CV_LOG_INFO(NULL, "OpenCL: reuse context@" << impl->contextId << " for configuration: " << configuration)
            impl->addref();
            return impl;
        }

        impl = new Impl(configuration);
        try
        {
            CV_OCL_CHECK(clRetainContext(h));
            impl->handle = h;
            impl->init_device_list();
            return impl;
        }
        catch (...)
        {
            delete impl;
            throw;
        }
    }

    static Impl* findOrCreateContext(const ocl::Device& device)
    {
        CV_TRACE_FUNCTION();

        CV_Assert(!device.empty());
        cl_device_id d = (cl_device_id)device.ptr();
        CV_Assert(d);

        std::string configuration = cv::format("@dev-%p", (void*)d);
        Impl* impl = findContext(configuration);
        if (impl)
        {
            CV_LOG_INFO(NULL, "OpenCL: reuse context@" << impl->contextId << " for configuration: " << configuration)
            impl->addref();
            return impl;
        }

        impl = new Impl(configuration);
        try
        {
            impl->createFromDevice(d);
            CV_Assert(impl->handle);
            return impl;
        }
        catch (...)
        {
            delete impl;
            throw;
        }
    }

    void setDefault()
    {
        CV_TRACE_FUNCTION();
        cl_device_id d = selectOpenCLDevice();

        if (d == NULL)
            return;

        createFromDevice(d);
    }

    void createFromDevice(cl_device_id d)
    {
        CV_TRACE_FUNCTION();
        CV_Assert(handle == NULL);

        cl_platform_id pl = NULL;
        CV_OCL_DBG_CHECK(clGetDeviceInfo(d, CL_DEVICE_PLATFORM, sizeof(cl_platform_id), &pl, NULL));

        cl_context_properties prop[] =
        {
            CL_CONTEXT_PLATFORM, (cl_context_properties)pl,
            0
        };

        // !!! in the current implementation force the number of devices to 1 !!!
        cl_uint nd = 1;
        cl_int status;

        handle = clCreateContext(prop, nd, &d, 0, 0, &status);
        CV_OCL_DBG_CHECK_RESULT(status, "clCreateContext");

        bool ok = handle != 0 && status == CL_SUCCESS;
        if( ok )
        {
            devices.resize(nd);
            devices[0].set(d);
        }
        else
            handle = NULL;
    }

    Program getProg(const ProgramSource& src, const String& buildflags, String& errmsg);

    void unloadProg(Program& prog)
    {
        cv::AutoLock lock(program_cache_mutex);
        for (CacheList::iterator i = cacheList.begin(); i != cacheList.end(); ++i)
        {
              phash_t::iterator it = phash.find(*i);
              if (it != phash.end())
              {
                  if (it->second.ptr() == prog.ptr())
                  {
                      phash.erase(*i);
                      cacheList.erase(i);
                      return;
                  }
              }
        }
    }

    std::string& getPrefixString()
    {
        if (prefix.empty())
        {
            cv::AutoLock lock(program_cache_mutex);
            if (prefix.empty())
            {
                CV_Assert(!devices.empty());
                const Device& d = devices[0];
                int bits = d.addressBits();
                if (bits > 0 && bits != 64)
                    prefix = cv::format("%d-bit--", bits);
                prefix += d.vendorName() + "--" + d.name() + "--" + d.driverVersion();
                // sanitize chars
                for (size_t i = 0; i < prefix.size(); i++)
                {
                    char c = prefix[i];
                    if (!((c >= '0' && c <= '9') || (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z') || c == '_' || c == '-'))
                    {
                        prefix[i] = '_';
                    }
                }
            }
        }
        return prefix;
    }

    std::string& getPrefixBase()
    {
        if (prefix_base.empty())
        {
            cv::AutoLock lock(program_cache_mutex);
            if (prefix_base.empty())
            {
                const Device& d = devices[0];
                int bits = d.addressBits();
                if (bits > 0 && bits != 64)
                    prefix_base = cv::format("%d-bit--", bits);
                prefix_base += d.vendorName() + "--" + d.name() + "--";
                // sanitize chars
                for (size_t i = 0; i < prefix_base.size(); i++)
                {
                    char c = prefix_base[i];
                    if (!((c >= '0' && c <= '9') || (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z') || c == '_' || c == '-'))
                    {
                        prefix_base[i] = '_';
                    }
                }
            }
        }
        return prefix_base;
    }

    IMPLEMENT_REFCOUNTABLE();

    const int contextId;  // global unique ID
    const std::string configuration;

    cl_context handle;
    std::vector<Device> devices;

    std::string prefix;
    std::string prefix_base;

    cv::Mutex program_cache_mutex;
    typedef std::map<std::string, Program> phash_t;
    phash_t phash;
    typedef std::list<cv::String> CacheList;
    CacheList cacheList;

    std::shared_ptr<OpenCLBufferPoolImpl> bufferPool_;
    std::shared_ptr<OpenCLBufferPoolImpl> bufferPoolHostPtr_;
    OpenCLBufferPoolImpl& getBufferPool() const
    {
        _init_buffer_pools();
        CV_DbgAssert(bufferPool_);
        return *bufferPool_.get();
    }
    OpenCLBufferPoolImpl& getBufferPoolHostPtr() const
    {
        _init_buffer_pools();
        CV_DbgAssert(bufferPoolHostPtr_);
        return *bufferPoolHostPtr_.get();
    }

    std::map<std::type_index, std::shared_ptr<UserContext>> userContextStorage;
    cv::Mutex userContextMutex;
    void setUserContext(std::type_index typeId, const std::shared_ptr<UserContext>& userContext) {
        cv::AutoLock lock(userContextMutex);
        userContextStorage[typeId] = userContext;
    }
    std::shared_ptr<UserContext> getUserContext(std::type_index typeId) {
        cv::AutoLock lock(userContextMutex);
        auto it = userContextStorage.find(typeId);
        if (it != userContextStorage.end())
            return it->second;
        else
            return nullptr;
    }

#ifdef HAVE_OPENCL_SVM
    bool svmInitialized;
    bool svmAvailable;
    bool svmEnabled;
    svm::SVMCapabilities svmCapabilities;
    svm::SVMFunctions svmFunctions;

    void svmInit()
    {
        CV_Assert(handle != NULL);
        const Device& device = devices[0];
        cl_device_svm_capabilities deviceCaps = 0;
        CV_Assert(((void)0, CL_DEVICE_SVM_CAPABILITIES == CL_DEVICE_SVM_CAPABILITIES_AMD)); // Check assumption
        cl_int status = clGetDeviceInfo((cl_device_id)device.ptr(), CL_DEVICE_SVM_CAPABILITIES, sizeof(deviceCaps), &deviceCaps, NULL);
        if (status != CL_SUCCESS)
        {
            CV_OPENCL_SVM_TRACE_ERROR_P("CL_DEVICE_SVM_CAPABILITIES via clGetDeviceInfo failed: %d\n", status);
            goto noSVM;
        }
        CV_OPENCL_SVM_TRACE_P("CL_DEVICE_SVM_CAPABILITIES returned: 0x%x\n", (int)deviceCaps);
        CV_Assert(((void)0, CL_DEVICE_SVM_COARSE_GRAIN_BUFFER == CL_DEVICE_SVM_COARSE_GRAIN_BUFFER_AMD)); // Check assumption
        svmCapabilities.value_ =
                ((deviceCaps & CL_DEVICE_SVM_COARSE_GRAIN_BUFFER) ? svm::SVMCapabilities::SVM_COARSE_GRAIN_BUFFER : 0) |
                ((deviceCaps & CL_DEVICE_SVM_FINE_GRAIN_BUFFER) ? svm::SVMCapabilities::SVM_FINE_GRAIN_BUFFER : 0) |
                ((deviceCaps & CL_DEVICE_SVM_FINE_GRAIN_SYSTEM) ? svm::SVMCapabilities::SVM_FINE_GRAIN_SYSTEM : 0) |
                ((deviceCaps & CL_DEVICE_SVM_ATOMICS) ? svm::SVMCapabilities::SVM_ATOMICS : 0);
        svmCapabilities.value_ &= svm::getSVMCapabilitiesMask();
        if (svmCapabilities.value_ == 0)
        {
            CV_OPENCL_SVM_TRACE_ERROR_P("svmCapabilities is empty\n");
            goto noSVM;
        }
        try
        {
            // Try OpenCL 2.0
            CV_OPENCL_SVM_TRACE_P("Try SVM from OpenCL 2.0 ...\n");
            void* ptr = clSVMAlloc(handle, CL_MEM_READ_WRITE, 100, 0);
            if (!ptr)
            {
                CV_OPENCL_SVM_TRACE_ERROR_P("clSVMAlloc returned NULL...\n");
                CV_Error(Error::StsBadArg, "clSVMAlloc returned NULL");
            }
            try
            {
                bool error = false;
                cl_command_queue q = (cl_command_queue)Queue::getDefault().ptr();
                if (CL_SUCCESS != clEnqueueSVMMap(q, CL_TRUE, CL_MAP_WRITE, ptr, 100, 0, NULL, NULL))
                {
                    CV_OPENCL_SVM_TRACE_ERROR_P("clEnqueueSVMMap failed...\n");
                    CV_Error(Error::StsBadArg, "clEnqueueSVMMap FAILED");
                }
                clFinish(q);
                try
                {
                    ((int*)ptr)[0] = 100;
                }
                catch (...)
                {
                    CV_OPENCL_SVM_TRACE_ERROR_P("SVM buffer access test FAILED\n");
                    error = true;
                }
                if (CL_SUCCESS != clEnqueueSVMUnmap(q, ptr, 0, NULL, NULL))
                {
                    CV_OPENCL_SVM_TRACE_ERROR_P("clEnqueueSVMUnmap failed...\n");
                    CV_Error(Error::StsBadArg, "clEnqueueSVMUnmap FAILED");
                }
                clFinish(q);
                if (error)
                {
                    CV_Error(Error::StsBadArg, "OpenCL SVM buffer access test was FAILED");
                }
            }
            catch (...)
            {
                CV_OPENCL_SVM_TRACE_ERROR_P("OpenCL SVM buffer access test was FAILED\n");
                clSVMFree(handle, ptr);
                throw;
            }
            clSVMFree(handle, ptr);
            svmFunctions.fn_clSVMAlloc = clSVMAlloc;
            svmFunctions.fn_clSVMFree = clSVMFree;
            svmFunctions.fn_clSetKernelArgSVMPointer = clSetKernelArgSVMPointer;
            //svmFunctions.fn_clSetKernelExecInfo = clSetKernelExecInfo;
            //svmFunctions.fn_clEnqueueSVMFree = clEnqueueSVMFree;
            svmFunctions.fn_clEnqueueSVMMemcpy = clEnqueueSVMMemcpy;
            svmFunctions.fn_clEnqueueSVMMemFill = clEnqueueSVMMemFill;
            svmFunctions.fn_clEnqueueSVMMap = clEnqueueSVMMap;
            svmFunctions.fn_clEnqueueSVMUnmap = clEnqueueSVMUnmap;
        }
        catch (...)
        {
            CV_OPENCL_SVM_TRACE_P("clSVMAlloc failed, trying HSA extension...\n");
            try
            {
                // Try HSA extension
                String extensions = device.extensions();
                if (extensions.find("cl_amd_svm") == String::npos)
                {
                    CV_OPENCL_SVM_TRACE_P("Device extension doesn't have cl_amd_svm: %s\n", extensions.c_str());
                    goto noSVM;
                }
                cl_platform_id p = NULL;
                CV_OCL_CHECK(status = clGetDeviceInfo((cl_device_id)device.ptr(), CL_DEVICE_PLATFORM, sizeof(cl_platform_id), &p, NULL));
                svmFunctions.fn_clSVMAlloc = (clSVMAllocAMD_fn)clGetExtensionFunctionAddressForPlatform(p, "clSVMAllocAMD");
                svmFunctions.fn_clSVMFree = (clSVMFreeAMD_fn)clGetExtensionFunctionAddressForPlatform(p, "clSVMFreeAMD");
                svmFunctions.fn_clSetKernelArgSVMPointer = (clSetKernelArgSVMPointerAMD_fn)clGetExtensionFunctionAddressForPlatform(p, "clSetKernelArgSVMPointerAMD");
                //svmFunctions.fn_clSetKernelExecInfo = (clSetKernelExecInfoAMD_fn)clGetExtensionFunctionAddressForPlatform(p, "clSetKernelExecInfoAMD");
                //svmFunctions.fn_clEnqueueSVMFree = (clEnqueueSVMFreeAMD_fn)clGetExtensionFunctionAddressForPlatform(p, "clEnqueueSVMFreeAMD");
                svmFunctions.fn_clEnqueueSVMMemcpy = (clEnqueueSVMMemcpyAMD_fn)clGetExtensionFunctionAddressForPlatform(p, "clEnqueueSVMMemcpyAMD");
                svmFunctions.fn_clEnqueueSVMMemFill = (clEnqueueSVMMemFillAMD_fn)clGetExtensionFunctionAddressForPlatform(p, "clEnqueueSVMMemFillAMD");
                svmFunctions.fn_clEnqueueSVMMap = (clEnqueueSVMMapAMD_fn)clGetExtensionFunctionAddressForPlatform(p, "clEnqueueSVMMapAMD");
                svmFunctions.fn_clEnqueueSVMUnmap = (clEnqueueSVMUnmapAMD_fn)clGetExtensionFunctionAddressForPlatform(p, "clEnqueueSVMUnmapAMD");
                CV_Assert(svmFunctions.isValid());
            }
            catch (...)
            {
                CV_OPENCL_SVM_TRACE_P("Something is totally wrong\n");
                goto noSVM;
            }
        }

        svmAvailable = true;
        svmEnabled = !svm::checkDisableSVM();
        svmInitialized = true;
        CV_OPENCL_SVM_TRACE_P("OpenCV OpenCL SVM support initialized\n");
        return;
    noSVM:
        CV_OPENCL_SVM_TRACE_P("OpenCL SVM is not detected\n");
        svmAvailable = false;
        svmEnabled = false;
        svmCapabilities.value_ = 0;
        svmInitialized = true;
        svmFunctions.fn_clSVMAlloc = NULL;
        return;
    }

    std::shared_ptr<OpenCLSVMBufferPoolImpl> bufferPoolSVM_;

    OpenCLSVMBufferPoolImpl& getBufferPoolSVM() const
    {
        _init_buffer_pools();
        CV_DbgAssert(bufferPoolSVM_);
        return *bufferPoolSVM_.get();
    }
#endif

    friend class Program;
};


Context::Context() CV_NOEXCEPT
{
    p = 0;
}

Context::~Context()
{
    release();
}

// deprecated
Context::Context(int dtype)
{
    p = 0;
    create(dtype);
}

void Context::release()
{
    if (p)
    {
        p->release();
        p = NULL;
    }
}

bool Context::create()
{
    release();
    if (!haveOpenCL())
        return false;
    p = Impl::findOrCreateContext(std::string());
    if (p && p->handle)
        return true;
    release();
    return false;
}

// deprecated
bool Context::create(int dtype)
{
    if( !haveOpenCL() )
        return false;
    release();
    if (dtype == CL_DEVICE_TYPE_DEFAULT || (unsigned)dtype == (unsigned)CL_DEVICE_TYPE_ALL)
    {
        p = Impl::findOrCreateContext("");
    }
    else if (dtype == CL_DEVICE_TYPE_GPU)
    {
        p = Impl::findOrCreateContext(":GPU:");
    }
    else if (dtype == CL_DEVICE_TYPE_CPU)
    {
        p = Impl::findOrCreateContext(":CPU:");
    }
    else
    {
        CV_LOG_ERROR(NULL, "OpenCL: Can't recognize OpenCV device type=" << dtype);
    }
    if (p && !p->handle)
    {
        release();
    }
    return p != 0;
}

Context::Context(const Context& c)
{
    p = (Impl*)c.p;
    if(p)
        p->addref();
}

Context& Context::operator = (const Context& c)
{
    Impl* newp = (Impl*)c.p;
    if(newp)
        newp->addref();
    if(p)
        p->release();
    p = newp;
    return *this;
}

Context::Context(Context&& c) CV_NOEXCEPT
{
    p = c.p;
    c.p = nullptr;
}

Context& Context::operator = (Context&& c) CV_NOEXCEPT
{
    if (this != &c) {
        if(p)
            p->release();
        p = c.p;
        c.p = nullptr;
    }
    return *this;
}

void* Context::ptr() const
{
    return p == NULL ? NULL : p->handle;
}

size_t Context::ndevices() const
{
    return p ? p->devices.size() : 0;
}

Device& Context::device(size_t idx) const
{
    static Device dummy;
    return !p || idx >= p->devices.size() ? dummy : p->devices[idx];
}

Context& Context::getDefault(bool initialize)
{
    auto& c = OpenCLExecutionContext::getCurrent();
    if (!c.empty())
    {
        auto& ctx = c.getContext();
        return ctx;
    }

    CV_UNUSED(initialize);
    static Context dummy;
    return dummy;
}

Program Context::getProg(const ProgramSource& prog,
                         const String& buildopts, String& errmsg)
{
    return p ? p->getProg(prog, buildopts, errmsg) : Program();
}

void Context::unloadProg(Program& prog)
{
    if (p)
        p->unloadProg(prog);
}

/* static */
Context Context::fromHandle(void* context)
{
    Context ctx;
    ctx.p = Impl::findOrCreateContext((cl_context)context);
    return ctx;
}

/* static */
Context Context::fromDevice(const ocl::Device& device)
{
    Context ctx;
    ctx.p = Impl::findOrCreateContext(device);
    return ctx;
}

/* static */
Context Context::create(const std::string& configuration)
{
    Context ctx;
    ctx.p = Impl::findOrCreateContext(configuration);
    return ctx;
}

void* Context::getOpenCLContextProperty(int propertyId) const
{
    if (p == NULL)
        return nullptr;
    ::size_t size = 0;
    CV_OCL_CHECK(clGetContextInfo(p->handle, CL_CONTEXT_PROPERTIES, 0, NULL, &size));
    std::vector<cl_context_properties> prop(size / sizeof(cl_context_properties), (cl_context_properties)0);
    CV_OCL_CHECK(clGetContextInfo(p->handle, CL_CONTEXT_PROPERTIES, size, prop.data(), NULL));
    for (size_t i = 0; i < prop.size(); i += 2)
    {
        if (prop[i] == (cl_context_properties)propertyId)
        {
            CV_LOG_DEBUG(NULL, "OpenCL: found context property=" << propertyId << ") => " << (void*)prop[i + 1]);
            return (void*)prop[i + 1];
        }
    }
    return nullptr;
}

#ifdef HAVE_OPENCL_SVM
bool Context::useSVM() const
{
    Context::Impl* i = p;
    CV_Assert(i);
    if (!i->svmInitialized)
        i->svmInit();
    return i->svmEnabled;
}
void Context::setUseSVM(bool enabled)
{
    Context::Impl* i = p;
    CV_Assert(i);
    if (!i->svmInitialized)
        i->svmInit();
    if (enabled && !i->svmAvailable)
    {
        CV_Error(Error::StsError, "OpenCL Shared Virtual Memory (SVM) is not supported by OpenCL device");
    }
    i->svmEnabled = enabled;
}
#else
bool Context::useSVM() const { return false; }
void Context::setUseSVM(bool enabled) { CV_Assert(!enabled); }
#endif

#ifdef HAVE_OPENCL_SVM
namespace svm {

const SVMCapabilities getSVMCapabilitites(const ocl::Context& context)
{
    Context::Impl* i = context.p;
    CV_Assert(i);
    if (!i->svmInitialized)
        i->svmInit();
    return i->svmCapabilities;
}

CV_EXPORTS const SVMFunctions* getSVMFunctions(const ocl::Context& context)
{
    Context::Impl* i = context.p;
    CV_Assert(i);
    CV_Assert(i->svmInitialized); // getSVMCapabilitites() must be called first
    CV_Assert(i->svmFunctions.fn_clSVMAlloc != NULL);
    return &i->svmFunctions;
}

CV_EXPORTS bool useSVM(UMatUsageFlags usageFlags)
{
    if (checkForceSVMUmatUsage())
        return true;
    if (checkDisableSVMUMatUsage())
        return false;
    if ((usageFlags & USAGE_ALLOCATE_SHARED_MEMORY) != 0)
        return true;
    return false; // don't use SVM by default
}

} // namespace cv::ocl::svm
#endif // HAVE_OPENCL_SVM

Context::UserContext::~UserContext()
{
}

void Context::setUserContext(std::type_index typeId, const std::shared_ptr<Context::UserContext>& userContext)
{
    CV_Assert(p);
    p->setUserContext(typeId, userContext);
}

std::shared_ptr<Context::UserContext> Context::getUserContext(std::type_index typeId)
{
    CV_Assert(p);
    return p->getUserContext(typeId);
}

static void get_platform_name(cl_platform_id id, String& name)
{
    // get platform name string length
    size_t sz = 0;
    CV_OCL_CHECK(clGetPlatformInfo(id, CL_PLATFORM_NAME, 0, 0, &sz));

    // get platform name string
    AutoBuffer<char> buf(sz + 1);
    CV_OCL_CHECK(clGetPlatformInfo(id, CL_PLATFORM_NAME, sz, buf.data(), 0));

    // just in case, ensure trailing zero for ASCIIZ string
    buf[sz] = 0;

    name = buf.data();
}

/*
// Attaches OpenCL context to OpenCV
*/
void attachContext(const String& platformName, void* platformID, void* context, void* deviceID)
{
    auto ctx = OpenCLExecutionContext::create(platformName, platformID, context, deviceID);
    ctx.bind();
}

/* static */
OpenCLExecutionContext OpenCLExecutionContext::create(
        const std::string& platformName, void* platformID, void* context, void* deviceID
)
{
    if (!haveOpenCL())
        CV_Error(cv::Error::OpenCLApiCallError, "OpenCL runtime is not available!");

    cl_uint cnt = 0;
    CV_OCL_CHECK(clGetPlatformIDs(0, 0, &cnt));

    if (cnt == 0)
        CV_Error(cv::Error::OpenCLApiCallError, "No OpenCL platform available!");

    std::vector<cl_platform_id> platforms(cnt);

    CV_OCL_CHECK(clGetPlatformIDs(cnt, &platforms[0], 0));

    bool platformAvailable = false;

    // check if external platformName contained in list of available platforms in OpenCV
    for (unsigned int i = 0; i < cnt; i++)
    {
        String availablePlatformName;
        get_platform_name(platforms[i], availablePlatformName);
        // external platform is found in the list of available platforms
        if (platformName == availablePlatformName)
        {
            platformAvailable = true;
            break;
        }
    }

    if (!platformAvailable)
        CV_Error(cv::Error::OpenCLApiCallError, "No matched platforms available!");

    // check if platformID corresponds to platformName
    String actualPlatformName;
    get_platform_name((cl_platform_id)platformID, actualPlatformName);
    if (platformName != actualPlatformName)
        CV_Error(cv::Error::OpenCLApiCallError, "No matched platforms available!");

    OpenCLExecutionContext ctx;
    ctx.p = std::make_shared<OpenCLExecutionContext::Impl>((cl_platform_id)platformID, (cl_context)context, (cl_device_id)deviceID);
    CV_OCL_CHECK(clReleaseContext((cl_context)context));
    CV_OCL_CHECK(clReleaseDevice((cl_device_id)deviceID));
    return ctx;
}

void initializeContextFromHandle(Context& ctx, void* _platform, void* _context, void* _device)
{
    // internal call, less checks
    cl_platform_id platformID = (cl_platform_id)_platform;
    cl_context context = (cl_context)_context;
    cl_device_id deviceID = (cl_device_id)_device;

    std::string platformName = PlatformInfo(&platformID).name();

    auto clExecCtx = OpenCLExecutionContext::create(platformName, platformID, context, deviceID);
    CV_Assert(!clExecCtx.empty());
    ctx = clExecCtx.getContext();
}

/////////////////////////////////////////// Queue /////////////////////////////////////////////

struct Queue::Impl
{
    inline void __init()
    {
        refcount = 1;
        handle = 0;
        isProfilingQueue_ = false;
    }

    Impl(cl_command_queue q)
    {
        __init();
        handle = q;

        cl_command_queue_properties props = 0;
        CV_OCL_CHECK(clGetCommandQueueInfo(handle, CL_QUEUE_PROPERTIES, sizeof(cl_command_queue_properties), &props, NULL));
        isProfilingQueue_ = !!(props & CL_QUEUE_PROFILING_ENABLE);
    }

    Impl(cl_command_queue q, bool isProfilingQueue)
    {
        __init();
        handle = q;
        isProfilingQueue_ = isProfilingQueue;
    }

    Impl(const Context& c, const Device& d, bool withProfiling = false)
    {
        __init();

        const Context* pc = &c;
        cl_context ch = (cl_context)pc->ptr();
        if( !ch )
        {
            pc = &Context::getDefault();
            ch = (cl_context)pc->ptr();
        }
        cl_device_id dh = (cl_device_id)d.ptr();
        if( !dh )
            dh = (cl_device_id)pc->device(0).ptr();
        cl_int retval = 0;
        cl_command_queue_properties props = withProfiling ? CL_QUEUE_PROFILING_ENABLE : 0;
        CV_OCL_DBG_CHECK_(handle = clCreateCommandQueue(ch, dh, props, &retval), retval);
        isProfilingQueue_ = withProfiling;
    }

    ~Impl()
    {
#ifdef _WIN32
        if (!cv::__termination)
#endif
        {
            if(handle)
            {
                CV_OCL_DBG_CHECK(clFinish(handle));
                CV_OCL_DBG_CHECK(clReleaseCommandQueue(handle));
                handle = NULL;
            }
        }
    }

    const cv::ocl::Queue& getProfilingQueue(const cv::ocl::Queue& self)
    {
        
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

- **Platform**: A class/struct defined in this file
- **OpenCLExecutionContext**: A class/struct defined in this file
- **CLBufferEntry**: A class/struct defined in this file
- **Image2D**: A class/struct defined in this file
- **Kernel**: A class/struct defined in this file
- **UMat2D**: A class/struct defined in this file
- **CV_DECL_ALIGNED**: A class/struct defined in this file
- **AlignedDataPtr**: A class/struct defined in this file
- **AmdFftHelper**: A class/struct defined in this file
- **UMat3D**: A class/struct defined in this file
- **void**: A class/struct defined in this file
- **AmdBlasHelper**: A class/struct defined in this file
- **Timer**: A class/struct defined in this file
- **AlignedDataPtr2D**: A class/struct defined in this file
- **OpenCLBufferPoolImpl**: A class/struct defined in this file
- **OpenCLBufferPool**: A class/struct defined in this file
- **Context**: A class/struct defined in this file
- **temporary**: A class/struct defined in this file
- **Program**: A class/struct defined in this file
- **ProgramSource**: A class/struct defined in this file
- **OpenCLBufferPoolBaseImpl**: A class/struct defined in this file
- **OpenCLSVMBufferPoolImpl**: A class/struct defined in this file
- **PlatformInfo**: A class/struct defined in this file
- **internal**: A class/struct defined in this file
- **OpenCLAllocator**: A class/struct defined in this file
- **OpenCLBinaryCacheConfigurator**: A class/struct defined in this file
- **CLSVMBufferEntry**: A class/struct defined in this file
- **BinaryProgramFile**: A class/struct defined in this file
- **Queue**: A class/struct defined in this file
- **Device**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_REMOVE_DEPRECATED_API()**: A function/method defined in this file
- **HAVE_CLAMDBLAS()**: A function/method defined in this file
- **CL_DEVICE_SPIR_VERSIONS()**: A function/method defined in this file
- **CV_OCL_CODE_()**: A function/method defined in this file
- **_WIN32()**: A function/method defined in this file
- **PROCESS_SRC()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file
- **HAVE_OPENCL_SVM()**: A function/method defined in this file
- **CV_LOG_STRIP_LEVEL()**: A function/method defined in this file
- **unsigned()**: A function/method defined in this file
- **struct()**: A function/method defined in this file
- **CV_OPENCL_DATA_PTR_ALIGNMENT()**: A function/method defined in this file
- **_DEBUG()**: A function/method defined in this file
- **LOG_BUFFER_POOL()**: A function/method defined in this file
- **HAVE_CLAMDFFT()**: A function/method defined in this file
- **CL_VERSION_1_2()**: A function/method defined in this file
- **CV__ALLOCATOR_STATS_LOG()**: A function/method defined in this file
- **std()**: A function/method defined in this file
- **CL_DEVICE_IMAGE_PITCH_ALIGNMENT()**: A function/method defined in this file
- **CV_OCL_CODE()**: A function/method defined in this file
- **CV_OPENCL_RUN_ASSERT()**: A function/method defined in this file
- **__APPLE__()**: A function/method defined in this file
- **CL_DEVICE_IMAGE_BASE_ADDRESS_ALIGNMENT()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/opencl/runtime/opencl_clfft.hpp`
- `opencv2/core/utils/configuration.private.hpp`
- `opencv2/core/opencl/runtime/opencl_core.hpp`
- `opencv2/core/utils/filesystem.private.hpp`
- `opencv2/core/opencl/runtime/opencl_clblas.hpp`
- `opencv2/core/utils/lock.private.hpp`
- `opencv2/core/utils/filesystem.hpp`
- `ocl_disabled.impl.hpp`
- `precomp.hpp`
- `list`
- `opencv2/core/utils/logger.defines.hpp`
- `inttypes.h`
- `deque`
- `opencv2/core/opencl/runtime/opencl_svm_hsa_extension.hpp`
- `opencv2/core/utils/logger.hpp`
- `opencv2/core/bufferpool.hpp`
- `opencv2/core/opencl/opencl_svm.hpp`
- `opencv2/core/ocl_genbase.hpp`
- `opencl_kernels_core.hpp`
- `opencv2/core/utils/allocator_stats.impl.hpp`
- `umatrix.hpp`
- `opencv2/core/opencl/runtime/opencl_svm_20.hpp`
- `fstream`
- `set`
- `string`
- `sstream`
- `map`

**Python Imports:**
- `Mat`
- `any`
- `the`
- `user`
- `binary...`
- `MatAllocator`
- `OpenCL`
- `this`
- `cache`
- `sources`


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

