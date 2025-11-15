# Documentation for `docs/hal/ipp/src/minmax_ipp.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/hal/ipp/src/minmax_ipp.cpp_docs.md`
- **File Name**: `minmax_ipp.cpp_docs.md`
- **File Size**: 12,435 bytes
- **File Type**: .md
- **Link to Source**: [docs/hal/ipp/src/minmax_ipp.cpp_docs.md](../../../../docs/hal/ipp/src/minmax_ipp.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/hal/ipp/src` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `hal/ipp/src/minmax_ipp.cpp`

## File Metadata

- **Full Path**: `hal/ipp/src/minmax_ipp.cpp`
- **File Name**: `minmax_ipp.cpp`
- **File Size**: 9,376 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/ipp/src/minmax_ipp.cpp](../../../hal/ipp/src/minmax_ipp.cpp)

## Purpose and Role

This file is located in the `hal/ipp/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html

#include "ipp_hal_core.hpp"

#include <opencv2/core.hpp>
#include <opencv2/core/base.hpp>

#define IPP_DISABLE_MINMAXIDX_MANY_ROWS 1  // see Core_MinMaxIdx.rows_overflow test

static inline IppDataType ippiGetDataType(int depth)
{
    depth = CV_MAT_DEPTH(depth);
    return depth == CV_8U ? ipp8u :
    depth == CV_8S ? ipp8s :
    depth == CV_16U ? ipp16u :
    depth == CV_16S ? ipp16s :
    depth == CV_32S ? ipp32s :
    depth == CV_32F ? ipp32f :
    depth == CV_64F ? ipp64f :
    (IppDataType)-1;
}

#if IPP_VERSION_X100 >= 700


static IppStatus ipp_minMaxIndex_wrap(const void* pSrc, int srcStep, IppiSize size, IppDataType dataType,
    float* pMinVal, float* pMaxVal, IppiPoint* pMinIndex, IppiPoint* pMaxIndex, const Ipp8u*, int)
{
    switch(dataType)
    {
    case ipp8u:  return CV_INSTRUMENT_FUN_IPP(ippiMinMaxIndx_8u_C1R, (const Ipp8u*)pSrc, srcStep, size, pMinVal, pMaxVal, pMinIndex, pMaxIndex);
    case ipp16u: return CV_INSTRUMENT_FUN_IPP(ippiMinMaxIndx_16u_C1R, (const Ipp16u*)pSrc, srcStep, size, pMinVal, pMaxVal, pMinIndex, pMaxIndex);
    case ipp32f: return CV_INSTRUMENT_FUN_IPP(ippiMinMaxIndx_32f_C1R, (const Ipp32f*)pSrc, srcStep, size, pMinVal, pMaxVal, pMinIndex, pMaxIndex);
    default:     return ippStsDataTypeErr;
    }
}

static IppStatus ipp_minMaxIndexMask_wrap(const void* pSrc, int srcStep, IppiSize size, IppDataType dataType,
    float* pMinVal, float* pMaxVal, IppiPoint* pMinIndex, IppiPoint* pMaxIndex, const Ipp8u* pMask, int maskStep)
{
    switch(dataType)
    {
    case ipp8u:  return CV_INSTRUMENT_FUN_IPP(ippiMinMaxIndx_8u_C1MR, (const Ipp8u*)pSrc, srcStep, pMask, maskStep, size, pMinVal, pMaxVal, pMinIndex, pMaxIndex);
    case ipp16u: return CV_INSTRUMENT_FUN_IPP(ippiMinMaxIndx_16u_C1MR, (const Ipp16u*)pSrc, srcStep, pMask, maskStep, size, pMinVal, pMaxVal, pMinIndex, pMaxIndex);
    case ipp32f: return CV_INSTRUMENT_FUN_IPP(ippiMinMaxIndx_32f_C1MR, (const Ipp32f*)pSrc, srcStep, pMask, maskStep, size, pMinVal, pMaxVal, pMinIndex, pMaxIndex);
    default:     return ippStsDataTypeErr;
    }
}

static IppStatus ipp_minMax_wrap(const void* pSrc, int srcStep, IppiSize size, IppDataType dataType,
    float* pMinVal, float* pMaxVal, IppiPoint*, IppiPoint*, const Ipp8u*, int)
{
    IppStatus status;

    switch(dataType)
    {
#if IPP_VERSION_X100 > 201701 // wrong min values
    case ipp8u:
    {
        Ipp8u val[2];
        status = CV_INSTRUMENT_FUN_IPP(ippiMinMax_8u_C1R, (const Ipp8u*)pSrc, srcStep, size, &val[0], &val[1]);
        *pMinVal = val[0];
        *pMaxVal = val[1];
        return status;
    }
#endif
    case ipp16u:
    {
        Ipp16u val[2];
        status = CV_INSTRUMENT_FUN_IPP(ippiMinMax_16u_C1R, (const Ipp16u*)pSrc, srcStep, size, &val[0], &val[1]);
        *pMinVal = val[0];
        *pMaxVal = val[1];
        return status;
    }
    case ipp16s:
    {
        Ipp16s val[2];
        status = CV_INSTRUMENT_FUN_IPP(ippiMinMax_16s_C1R, (const Ipp16s*)pSrc, srcStep, size, &val[0], &val[1]);
        *pMinVal = val[0];
        *pMaxVal = val[1];
        return status;
    }
    case ipp32f: return CV_INSTRUMENT_FUN_IPP(ippiMinMax_32f_C1R, (const Ipp32f*)pSrc, srcStep, size, pMinVal, pMaxVal);
    default:     return ipp_minMaxIndex_wrap(pSrc, srcStep, size, dataType, pMinVal, pMaxVal, NULL, NULL, NULL, 0);
    }
}

static IppStatus ipp_minIdx_wrap(const void* pSrc, int srcStep, IppiSize size, IppDataType dataType,
    float* pMinVal, float*, IppiPoint* pMinIndex, IppiPoint*, const Ipp8u*, int)
{
    IppStatus status;

    switch(dataType)
    {
    case ipp8u:
    {
        Ipp8u val;
        status = CV_INSTRUMENT_FUN_IPP(ippiMinIndx_8u_C1R, (const Ipp8u*)pSrc, srcStep, size, &val, &pMinIndex->x, &pMinIndex->y);
        *pMinVal = val;
        return status;
    }
    case ipp16u:
    {
        Ipp16u val;
        status = CV_INSTRUMENT_FUN_IPP(ippiMinIndx_16u_C1R, (const Ipp16u*)pSrc, srcStep, size, &val, &pMinIndex->x, &pMinIndex->y);
        *pMinVal = val;
        return status;
    }
    case ipp16s:
    {
        Ipp16s val;
        status = CV_INSTRUMENT_FUN_IPP(ippiMinIndx_16s_C1R, (const Ipp16s*)pSrc, srcStep, size, &val, &pMinIndex->x, &pMinIndex->y);
        *pMinVal = val;
        return status;
    }
    case ipp32f: return CV_INSTRUMENT_FUN_IPP(ippiMinIndx_32f_C1R, (const Ipp32f*)pSrc, srcStep, size, pMinVal, &pMinIndex->x, &pMinIndex->y);
    default:     return ipp_minMaxIndex_wrap(pSrc, srcStep, size, dataType, pMinVal, NULL, pMinIndex, NULL, NULL, 0);
    }
}

static IppStatus ipp_maxIdx_wrap(const void* pSrc, int srcStep, IppiSize size, IppDataType dataType,
    float*, float* pMaxVal, IppiPoint*, IppiPoint* pMaxIndex, const Ipp8u*, int)
{
    IppStatus status;

    switch(dataType)
    {
    case ipp8u:
    {
        Ipp8u val;
        status = CV_INSTRUMENT_FUN_IPP(ippiMaxIndx_8u_C1R, (const Ipp8u*)pSrc, srcStep, size, &val, &pMaxIndex->x, &pMaxIndex->y);
        *pMaxVal = val;
        return status;
    }
    case ipp16u:
    {
        Ipp16u val;
        status = CV_INSTRUMENT_FUN_IPP(ippiMaxIndx_16u_C1R, (const Ipp16u*)pSrc, srcStep, size, &val, &pMaxIndex->x, &pMaxIndex->y);
        *pMaxVal = val;
        return status;
    }
    case ipp16s:
    {
        Ipp16s val;
        status = CV_INSTRUMENT_FUN_IPP(ippiMaxIndx_16s_C1R, (const Ipp16s*)pSrc, srcStep, size, &val, &pMaxIndex->x, &pMaxIndex->y);
        *pMaxVal = val;
        return status;
    }
    case ipp32f: return CV_INSTRUMENT_FUN_IPP(ippiMaxIndx_32f_C1R, (const Ipp32f*)pSrc, srcStep, size, pMaxVal, &pMaxIndex->x, &pMaxIndex->y);
    default:     return ipp_minMaxIndex_wrap(pSrc, srcStep, size, dataType, NULL, pMaxVal, NULL, pMaxIndex, NULL, 0);
    }
}

typedef IppStatus (*IppMinMaxSelector)(const void* pSrc, int srcStep, IppiSize size, IppDataType dataType,
    float* pMinVal, float* pMaxVal, IppiPoint* pMinIndex, IppiPoint* pMaxIndex, const Ipp8u* pMask, int maskStep);


int ipp_hal_minMaxIdxMaskStep(const uchar* src_data, size_t src_step, int width, int height, int depth,
                              double* _minVal, double* _maxVal, int* _minIdx, int* _maxIdx, uchar* mask, size_t mask_step)
{
#if IPP_VERSION_X100 < 201800
    // cv::minMaxIdx problem with NaN input
    // Disable 32F processing only
    if(depth == CV_32F && cv::ipp::getIppTopFeatures() == ippCPUID_SSE42)
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
#endif

#if IPP_VERSION_X100 < 201801
    // cv::minMaxIdx problem with index positions on AVX
    if(mask && _maxIdx && cv::ipp::getIppTopFeatures() != ippCPUID_SSE42)
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
#endif

    IppStatus   status;
    IppDataType dataType = ippiGetDataType(depth);
    float       minVal = 0;
    float       maxVal = 0;
    IppiPoint   minIdx = {-1, -1};
    IppiPoint   maxIdx = {-1, -1};

    float       *pMinVal = (_minVal || _minIdx)?&minVal:NULL;
    float       *pMaxVal = (_maxVal || _maxIdx)?&maxVal:NULL;
    IppiPoint   *pMinIdx = (_minIdx)?&minIdx:NULL;
    IppiPoint   *pMaxIdx = (_maxIdx)?&maxIdx:NULL;

    IppMinMaxSelector ippMinMaxFun = ipp_minMaxIndexMask_wrap;
    if(!mask)
    {
        if(_maxVal && _maxIdx && !_minVal && !_minIdx)
            ippMinMaxFun = ipp_maxIdx_wrap;
        else if(!_maxVal && !_maxIdx && _minVal && _minIdx)
            ippMinMaxFun = ipp_minIdx_wrap;
        else if(_maxVal && !_maxIdx && _minVal && !_minIdx)
            ippMinMaxFun = ipp_minMax_wrap;
        else if(!_maxVal && !_maxIdx && !_minVal && !_minIdx)
            return CV_HAL_ERROR_NOT_IMPLEMENTED;
        else
            ippMinMaxFun = ipp_minMaxIndex_wrap;
    }

    IppiSize size = { width, height };
#if defined(_WIN32) && !defined(_WIN64) && IPP_VERSION_X100 == 201900 && IPP_DISABLE_MINMAXIDX_MANY_ROWS
    if (size.height > 65536)
        return CV_HAL_ERROR_NOT_IMPLEMENTED;  // test: Core_MinMaxIdx.rows_overflow
#endif

    status = ippMinMaxFun(src_data, (int)src_step, size, dataType, pMinVal, pMaxVal, pMinIdx, pMaxIdx, (Ipp8u*)mask, (int)mask_step);
    if(status < 0)
        return CV_HAL_ERROR_NOT_IMPLEMENTED;
    if(_minVal)
        *_minVal = minVal;
    if(_maxVal)
        *_maxVal = maxVal;
    if(_minIdx)
    {
#if IPP_VERSION_X100 < 201801
        // Should be just ippStsNoOperation check, but there is a bug in the function so we need additional checks
        if(status == ippStsNoOperation && mask && !pMinIdx->x && !pMinIdx->y)
#else
        if(status == ippStsNoOperation)
#endif
        {
            _minIdx[0] = -1;
            _minIdx[1] = -1;
        }
        else
        {
            _minIdx[0] = minIdx.y;
            _minIdx[1] = minIdx.x;
        }
    }
    if(_maxIdx)
    {
#if IPP_VERSION_X100 < 201801
        // Should be just ippStsNoOperation check, but there is a bug in the function so we need additional checks
        if(status == ippStsNoOperation && mask && !pMaxIdx->x && !pMaxIdx->y)
#else
        if(status == ippStsNoOperation)
#endif
        {
            _maxIdx[0] = -1;
            _maxIdx[1] = -1;
        }
        else
        {
            _maxIdx[0] = maxIdx.y;
            _maxIdx[1] = maxIdx.x;
        }
    }

    return CV_HAL_ERROR_OK;
}

#endif // IPP_VERSION_X100 >= 700
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

### Functions and Methods

- **IppStatus()**: A function/method defined in this file
- **so()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`
- `opencv2/core/base.hpp`
- `ipp_hal_core.hpp`


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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

