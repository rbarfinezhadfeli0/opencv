# Documentation for `modules/core/perf/opencl/perf_matop.cpp`

## File Metadata

- **Full Path**: `modules/core/perf/opencl/perf_matop.cpp`
- **File Name**: `perf_matop.cpp`
- **File Size**: 12,151 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/perf/opencl/perf_matop.cpp](../../../../modules/core/perf/opencl/perf_matop.cpp)

## Purpose and Role

This file is located in the `modules/core/perf/opencl` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

// Copyright (C) 2014, Advanced Micro Devices, Inc., all rights reserved.
// Third party copyrights are property of their respective owners.

#include "../perf_precomp.hpp"
#include "opencv2/ts/ocl_perf.hpp"

#ifdef HAVE_OPENCL

namespace opencv_test {
namespace ocl {

///////////// SetTo ////////////////////////

typedef Size_MatType SetToFixture;

OCL_PERF_TEST_P(SetToFixture, SetTo,
                ::testing::Combine(OCL_TEST_SIZES, OCL_TEST_TYPES))
{
    const Size_MatType_t params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params);
    const Scalar s = Scalar::all(17);

    checkDeviceMaxMemoryAllocSize(srcSize, type);

    UMat src(srcSize, type);
    declare.in(src, WARMUP_RNG).out(src);

    OCL_TEST_CYCLE() src.setTo(s);

    SANITY_CHECK(src);
}

///////////// SetTo with mask ////////////////////////

typedef Size_MatType SetToFixture;

OCL_PERF_TEST_P(SetToFixture, SetToWithMask,
                ::testing::Combine(OCL_TEST_SIZES, OCL_TEST_TYPES))
{
    const Size_MatType_t params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params);
    const Scalar s = Scalar::all(17);

    checkDeviceMaxMemoryAllocSize(srcSize, type);

    UMat src(srcSize, type), mask(srcSize, CV_8UC1);
    declare.in(src, mask, WARMUP_RNG).out(src);

    OCL_TEST_CYCLE() src.setTo(s, mask);

    SANITY_CHECK(src);
}

///////////// ConvertTo ////////////////////////

typedef Size_MatType ConvertToFixture;

OCL_PERF_TEST_P(ConvertToFixture, ConvertTo,
                ::testing::Combine(OCL_TEST_SIZES, OCL_TEST_TYPES))
{
    const Size_MatType_t params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params), ddepth = CV_MAT_DEPTH(type) == CV_8U ? CV_32F : CV_8U,
        cn = CV_MAT_CN(type), dtype = CV_MAKE_TYPE(ddepth, cn);

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(srcSize, dtype);

    UMat src(srcSize, type), dst(srcSize, dtype);
    declare.in(src, WARMUP_RNG).out(dst);

    OCL_TEST_CYCLE() src.convertTo(dst, dtype);

    SANITY_CHECK(dst);
}


//#define RUN_CONVERTFP16
static Size convertFP16_srcSize(4000, 4000);

OCL_PERF_TEST(Core, ConvertFP32FP16MatMat)
{
    const Size srcSize = convertFP16_srcSize;
    const int type = CV_32F;
    const int dtype = CV_16F;

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(srcSize, dtype);

    Mat src(srcSize, type);
    Mat dst(srcSize, dtype);
    declare.in(src, WARMUP_RNG).out(dst);

#ifdef RUN_CONVERTFP16
    OCL_TEST_CYCLE() convertFp16(src, dst);
#else
    OCL_TEST_CYCLE() src.convertTo(dst, dtype);
#endif

    SANITY_CHECK_NOTHING();
}

OCL_PERF_TEST(Core, ConvertFP32FP16MatUMat)
{
    const Size srcSize = convertFP16_srcSize;
    const int type = CV_32F;
    const int dtype = CV_16F;

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(srcSize, dtype);

    Mat src(srcSize, type);
    UMat dst(srcSize, dtype);
    declare.in(src, WARMUP_RNG).out(dst);

#ifdef RUN_CONVERTFP16
    OCL_TEST_CYCLE() convertFp16(src, dst);
#else
    OCL_TEST_CYCLE() src.convertTo(dst, dtype);
#endif

    SANITY_CHECK_NOTHING();
}

OCL_PERF_TEST(Core, ConvertFP32FP16UMatMat)
{
    const Size srcSize = convertFP16_srcSize;
    const int type = CV_32F;
    const int dtype = CV_16F;

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(srcSize, dtype);

    UMat src(srcSize, type);
    Mat dst(srcSize, dtype);
    declare.in(src, WARMUP_RNG).out(dst);

#ifdef RUN_CONVERTFP16
    OCL_TEST_CYCLE() convertFp16(src, dst);
#else
    OCL_TEST_CYCLE() src.convertTo(dst, dtype);
#endif

    SANITY_CHECK_NOTHING();
}

OCL_PERF_TEST(Core, ConvertFP32FP16UMatUMat)
{
    const Size srcSize = convertFP16_srcSize;
    const int type = CV_32F;
    const int dtype = CV_16F;

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(srcSize, dtype);

    UMat src(srcSize, type);
    UMat dst(srcSize, dtype);
    declare.in(src, WARMUP_RNG).out(dst);

#ifdef RUN_CONVERTFP16
    OCL_TEST_CYCLE() convertFp16(src, dst);
#else
    OCL_TEST_CYCLE() src.convertTo(dst, dtype);
#endif

    SANITY_CHECK_NOTHING();
}

OCL_PERF_TEST(Core, ConvertFP16FP32MatMat)
{
    const Size srcSize = convertFP16_srcSize;
    const int type = CV_16F;
    const int dtype = CV_32F;

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(srcSize, dtype);

    Mat src(srcSize, type);
    Mat dst(srcSize, dtype);
    declare.in(src, WARMUP_RNG).out(dst);

#ifdef RUN_CONVERTFP16
    OCL_TEST_CYCLE() convertFp16(src, dst);
#else
    OCL_TEST_CYCLE() src.convertTo(dst, dtype);
#endif

    SANITY_CHECK_NOTHING();
}

OCL_PERF_TEST(Core, ConvertFP16FP32MatUMat)
{
    const Size srcSize = convertFP16_srcSize;
    const int type = CV_16F;
    const int dtype = CV_32F;

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(srcSize, dtype);

    Mat src(srcSize, type);
    UMat dst(srcSize, dtype);
    declare.in(src, WARMUP_RNG).out(dst);

#ifdef RUN_CONVERTFP16
    OCL_TEST_CYCLE() convertFp16(src, dst);
#else
    OCL_TEST_CYCLE() src.convertTo(dst, dtype);
#endif

    SANITY_CHECK_NOTHING();
}

OCL_PERF_TEST(Core, ConvertFP16FP32UMatMat)
{
    const Size srcSize = convertFP16_srcSize;
    const int type = CV_16F;
    const int dtype = CV_32F;

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(srcSize, dtype);

    UMat src(srcSize, type);
    Mat dst(srcSize, dtype);
    declare.in(src, WARMUP_RNG).out(dst);

#ifdef RUN_CONVERTFP16
    OCL_TEST_CYCLE() convertFp16(src, dst);
#else
    OCL_TEST_CYCLE() src.convertTo(dst, dtype);
#endif

    SANITY_CHECK_NOTHING();
}

OCL_PERF_TEST(Core, ConvertFP16FP32UMatUMat)
{
    const Size srcSize = convertFP16_srcSize;
    const int type = CV_16F;
    const int dtype = CV_32F;

    checkDeviceMaxMemoryAllocSize(srcSize, type);
    checkDeviceMaxMemoryAllocSize(srcSize, dtype);

    UMat src(srcSize, type);
    UMat dst(srcSize, dtype);
    declare.in(src, WARMUP_RNG).out(dst);

#ifdef RUN_CONVERTFP16
    OCL_TEST_CYCLE() convertFp16(src, dst);
#else
    OCL_TEST_CYCLE() src.convertTo(dst, dtype);
#endif

    SANITY_CHECK_NOTHING();
}


///////////// CopyTo ////////////////////////

typedef Size_MatType CopyToFixture;

OCL_PERF_TEST_P(CopyToFixture, CopyTo,
                ::testing::Combine(OCL_TEST_SIZES, OCL_TEST_TYPES))
{
    const Size_MatType_t params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params);

    checkDeviceMaxMemoryAllocSize(srcSize, type);

    UMat src(srcSize, type), dst(srcSize, type);
    declare.in(src, WARMUP_RNG).out(dst);

    OCL_TEST_CYCLE() src.copyTo(dst);

    SANITY_CHECK(dst);
}

///////////// CopyTo with mask ////////////////////////

typedef Size_MatType CopyToFixture;

OCL_PERF_TEST_P(CopyToFixture, CopyToWithMask,
                ::testing::Combine(OCL_TEST_SIZES, OCL_TEST_TYPES))
{
    const Size_MatType_t params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params);

    checkDeviceMaxMemoryAllocSize(srcSize, type);

    UMat src(srcSize, type), dst(srcSize, type), mask(srcSize, CV_8UC1);
    declare.in(src, mask, WARMUP_RNG).out(dst);

    OCL_TEST_CYCLE() src.copyTo(dst, mask);

    SANITY_CHECK(dst);
}

OCL_PERF_TEST_P(CopyToFixture, CopyToWithMaskUninit,
                ::testing::Combine(OCL_PERF_ENUM(OCL_SIZE_1, OCL_SIZE_2, OCL_SIZE_3), OCL_TEST_TYPES))
{
    const Size_MatType_t params = GetParam();
    const Size srcSize = get<0>(params);
    const int type = get<1>(params);

    checkDeviceMaxMemoryAllocSize(srcSize, type);

    UMat src(srcSize, type), dst, mask(srcSize, CV_8UC1);
    declare.in(src, mask, WARMUP_RNG);

    for ( ;  next(); )
    {
        dst.release();
        startTimer();
        src.copyTo(dst, mask);
        cvtest::ocl::perf::safeFinish();
        stopTimer();
    }

    SANITY_CHECK(dst);
}



enum ROIType {
    ROI_FULL,
    ROI_2_RECT,
    ROI_2_TOP,  // contiguous memory block
    ROI_2_LEFT,
    ROI_4,
    ROI_16,
};
static Rect getROI(enum ROIType t, const Size& sz)
{
    switch (t)
    {
        case ROI_FULL: return Rect(0, 0, sz.width, sz.height);
        case ROI_2_RECT: return Rect(0, 0, sz.width * 71 / 100, sz.height * 71 / 100);  // 71 = sqrt(1/2) * 100
        case ROI_2_TOP: return Rect(0, 0, sz.width, sz.height / 2);  // 71 = sqrt(1/2) * 100
        case ROI_2_LEFT: return Rect(0, 0, sz.width / 2, sz.height);  // 71 = sqrt(1/2) * 100
        case ROI_4: return Rect(0, 0, sz.width / 2, sz.height / 2);
        case ROI_16: return Rect(0, 0, sz.width / 4, sz.height / 4);
    }
    CV_Assert(false);
}

typedef TestBaseWithParam< tuple<cv::Size, MatType, ROIType> > OpenCLBuffer;

static inline void PrintTo(const tuple<cv::Size, MatType, enum ROIType>& v, std::ostream* os)
{
    *os << "(" << get<0>(v) << ", " << typeToString(get<1>(v)) << ", ";
    enum ROIType roiType = get<2>(v);
    if (roiType == ROI_FULL)
        *os << "ROI_100_FULL";
    else if (roiType == ROI_2_RECT)
        *os << "ROI_050_RECT_HALF";
    else if (roiType == ROI_2_TOP)
        *os << "ROI_050_TOP_HALF";
    else if (roiType == ROI_2_LEFT)
        *os << "ROI_050_LEFT_HALF";
    else if (roiType == ROI_4)
        *os << "ROI_025_1/4";
    else
        *os << "ROI_012_1/16";
    *os << ")";
}

PERF_TEST_P_(OpenCLBuffer, cpu_write)
{
    const Size srcSize = get<0>(GetParam());
    const int type = get<1>(GetParam());
    const Rect roi = getROI(get<2>(GetParam()), srcSize);

    checkDeviceMaxMemoryAllocSize(srcSize, type);

    UMat src(srcSize, type);
    declare.in(src(roi), WARMUP_NONE);

    OCL_TEST_CYCLE()
    {
        Mat m = src(roi).getMat(ACCESS_WRITE);
        m.setTo(Scalar(1, 2, 3, 4));
    }

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P_(OpenCLBuffer, cpu_read)
{
    const Size srcSize = get<0>(GetParam());
    const int type = get<1>(GetParam());
    const Rect roi = getROI(get<2>(GetParam()), srcSize);

    checkDeviceMaxMemoryAllocSize(srcSize, type);

    UMat src(srcSize, type, Scalar(1, 2, 3, 4));
    declare.in(src(roi), WARMUP_NONE);

    OCL_TEST_CYCLE()
    {
        unsigned counter = 0;
        Mat m = src(roi).getMat(ACCESS_READ);
        for (int y = 0; y < m.rows; y++)
        {
            uchar* ptr = m.ptr(y);
            size_t width_bytes = m.cols * m.elemSize();
            for (size_t x_bytes = 0; x_bytes < width_bytes; x_bytes++)
                counter += (unsigned)(ptr[x_bytes]);
        }
        (void)counter; // To avoid -Wunused-but-set-variable
    }

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P_(OpenCLBuffer, cpu_update)
{
    const Size srcSize = get<0>(GetParam());
    const int type = get<1>(GetParam());
    const Rect roi = getROI(get<2>(GetParam()), srcSize);

    checkDeviceMaxMemoryAllocSize(srcSize, type);

    UMat src(srcSize, type, Scalar(1, 2, 3, 4));
    declare.in(src(roi), WARMUP_NONE);

    OCL_TEST_CYCLE()
    {
        Mat m = src(roi).getMat(ACCESS_READ | ACCESS_WRITE);
        for (int y = 0; y < m.rows; y++)
        {
            uchar* ptr = m.ptr(y);
            size_t width_bytes = m.cols * m.elemSize();
            for (size_t x_bytes = 0; x_bytes < width_bytes; x_bytes++)
                ptr[x_bytes] += 1;
        }
    }

    SANITY_CHECK_NOTHING();
}

INSTANTIATE_TEST_CASE_P(/*FULL*/, OpenCLBuffer,
    testing::Combine(
        testing::Values(szVGA, sz720p, sz1080p, sz2160p),
        testing::Values(CV_8UC1, CV_8UC2, CV_8UC3, CV_8UC4),
        testing::Values(ROI_FULL)
    )
);

INSTANTIATE_TEST_CASE_P(ROI, OpenCLBuffer,
    testing::Combine(
        testing::Values(sz1080p, sz2160p),
        testing::Values(CV_8UC1),
        testing::Values(ROI_16, ROI_4, ROI_2_RECT, ROI_2_LEFT, ROI_2_TOP, ROI_FULL)
    )
);


} } // namespace opencv_test::ocl

#endif // HAVE_OPENCL
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

- **RUN_CONVERTFP16()**: A function/method defined in this file
- **HAVE_OPENCL()**: A function/method defined in this file
- **TestBaseWithParam()**: A function/method defined in this file
- **Size_MatType()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/ts/ocl_perf.hpp`
- `../perf_precomp.hpp`


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

