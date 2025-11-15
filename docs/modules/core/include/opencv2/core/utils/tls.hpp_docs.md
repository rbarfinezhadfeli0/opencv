# Documentation for `modules/core/include/opencv2/core/utils/tls.hpp`

## File Metadata

- **Full Path**: `modules/core/include/opencv2/core/utils/tls.hpp`
- **File Name**: `tls.hpp`
- **File Size**: 6,682 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/core/include/opencv2/core/utils/tls.hpp](../../../../../../modules/core/include/opencv2/core/utils/tls.hpp)

## Purpose and Role

This file is located in the `modules/core/include/opencv2/core/utils` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_UTILS_TLS_HPP
#define OPENCV_UTILS_TLS_HPP

#ifndef OPENCV_CORE_UTILITY_H
#error "tls.hpp must be included after opencv2/core/utility.hpp or opencv2/core.hpp"
#endif

namespace cv {

//! @addtogroup core_utils
//! @{

namespace details { class TlsStorage; }

/** TLS container base implementation
 *
 * Don't use directly.
 *
 * @sa TLSData, TLSDataAccumulator templates
 */
class CV_EXPORTS TLSDataContainer
{
protected:
    TLSDataContainer();
    virtual ~TLSDataContainer();

    /// @deprecated use detachData() instead
    void  gatherData(std::vector<void*> &data) const;
    /// get TLS data and detach all data from threads (similar to cleanup() call)
    void  detachData(std::vector<void*>& data);

    void* getData() const;
    void  release();

protected:
    virtual void* createDataInstance() const = 0;
    virtual void  deleteDataInstance(void* pData) const = 0;

private:
    int key_;

    friend class cv::details::TlsStorage;  // core/src/system.cpp

public:
    void cleanup(); //!< Release created TLS data container objects. It is similar to release() call, but it keeps TLS container valid.

private:
    // Disable copy/assign (noncopyable pattern)
    TLSDataContainer(TLSDataContainer &) = delete;
    TLSDataContainer& operator =(const TLSDataContainer &) = delete;
};


/** @brief Simple TLS data class
 *
 * @sa TLSDataAccumulator
 */
template <typename T>
class TLSData : protected TLSDataContainer
{
public:
    inline TLSData() {}
    inline ~TLSData() { release(); }

    inline T* get() const   { return (T*)getData(); }  //!< Get data associated with key
    inline T& getRef() const { T* ptr = (T*)getData(); CV_DbgAssert(ptr); return *ptr; }  //!< Get data associated with key

    /// Release associated thread data
    inline void cleanup()
    {
        TLSDataContainer::cleanup();
    }

protected:
    /// Wrapper to allocate data by template
    virtual void* createDataInstance() const CV_OVERRIDE { return new T; }
    /// Wrapper to release data by template
    virtual void  deleteDataInstance(void* pData) const CV_OVERRIDE { delete (T*)pData; }
};


/// TLS data accumulator with gathering methods
template <typename T>
class TLSDataAccumulator : public TLSData<T>
{
    mutable cv::Mutex mutex;
    mutable std::vector<T*> dataFromTerminatedThreads;
    std::vector<T*> detachedData;
    bool cleanupMode;
public:
    TLSDataAccumulator() : cleanupMode(false) {}
    ~TLSDataAccumulator()
    {
        release();
    }

    /** @brief Get data from all threads
     * @deprecated replaced by detachData()
     *
     * Lifetime of vector data is valid until next detachData()/cleanup()/release() calls
     *
     * @param[out] data result buffer (should be empty)
     */
    void gather(std::vector<T*> &data) const
    {
        CV_Assert(cleanupMode == false);  // state is not valid
        CV_Assert(data.empty());
        {
            std::vector<void*> &dataVoid = reinterpret_cast<std::vector<void*>&>(data);
            TLSDataContainer::gatherData(dataVoid);
        }
        {
            AutoLock lock(mutex);
            data.reserve(data.size() + dataFromTerminatedThreads.size());
            for (typename std::vector<T*>::const_iterator i = dataFromTerminatedThreads.begin(); i != dataFromTerminatedThreads.end(); ++i)
            {
                data.push_back((T*)*i);
            }
        }
    }

    /** @brief Get and detach data from all threads
     *
     * Call cleanupDetachedData() when returned vector is not needed anymore.
     *
     * @return Vector with associated data. Content is preserved (including lifetime of attached data pointers) until next detachData()/cleanupDetachedData()/cleanup()/release() calls
     */
    std::vector<T*>& detachData()
    {
        CV_Assert(cleanupMode == false);  // state is not valid
        std::vector<void*> dataVoid;
        {
            TLSDataContainer::detachData(dataVoid);
        }
        {
            AutoLock lock(mutex);
            detachedData.reserve(dataVoid.size() + dataFromTerminatedThreads.size());
            for (typename std::vector<T*>::const_iterator i = dataFromTerminatedThreads.begin(); i != dataFromTerminatedThreads.end(); ++i)
            {
                detachedData.push_back((T*)*i);
            }
            dataFromTerminatedThreads.clear();
            for (typename std::vector<void*>::const_iterator i = dataVoid.begin(); i != dataVoid.end(); ++i)
            {
                detachedData.push_back((T*)(void*)*i);
            }
        }
        dataVoid.clear();
        return detachedData;
    }

    /// Release associated thread data returned by detachData() call
    void cleanupDetachedData()
    {
        AutoLock lock(mutex);
        cleanupMode = true;
        _cleanupDetachedData();
        cleanupMode = false;
    }

    /// Release associated thread data
    void cleanup()
    {
        cleanupMode = true;
        TLSDataContainer::cleanup();

        AutoLock lock(mutex);
        _cleanupDetachedData();
        _cleanupTerminatedData();
        cleanupMode = false;
    }

    /// Release associated thread data and free TLS key
    void release()
    {
        cleanupMode = true;
        TLSDataContainer::release();
        {
            AutoLock lock(mutex);
            _cleanupDetachedData();
            _cleanupTerminatedData();
        }
    }

protected:
    // synchronized
    void _cleanupDetachedData()
    {
        for (typename std::vector<T*>::iterator i = detachedData.begin(); i != detachedData.end(); ++i)
        {
            deleteDataInstance((T*)*i);
        }
        detachedData.clear();
    }

    // synchronized
    void _cleanupTerminatedData()
    {
        for (typename std::vector<T*>::iterator i = dataFromTerminatedThreads.begin(); i != dataFromTerminatedThreads.end(); ++i)
        {
            deleteDataInstance((T*)*i);
        }
        dataFromTerminatedThreads.clear();
    }

protected:
    virtual void* createDataInstance() const CV_OVERRIDE
    {
        // Note: we can collect all allocated data here, but this would require raced mutex locks
        return new T;
    }
    virtual void  deleteDataInstance(void* pData) const CV_OVERRIDE
    {
        if (cleanupMode)
        {
            delete (T*)pData;
        }
        else
        {
            AutoLock lock(mutex);
            dataFromTerminatedThreads.push_back((T*)pData);
        }
    }
};


//! @}

} // namespace

#endif // OPENCV_UTILS_TLS_HPP
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

- **CV_EXPORTS**: A class/struct defined in this file
- **TlsStorage**: A class/struct defined in this file
- **cv**: A class/struct defined in this file
- **TLSData**: A class/struct defined in this file
- **TLSDataAccumulator**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_UTILS_TLS_HPP()**: A function/method defined in this file
- **OPENCV_CORE_UTILITY_H()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `all`
- `threads`


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

