# Documentation for `hal/riscv-rvv/src/imgproc/moments.cpp`

## File Metadata

- **Full Path**: `hal/riscv-rvv/src/imgproc/moments.cpp`
- **File Name**: `moments.cpp`
- **File Size**: 8,702 bytes
- **File Type**: .cpp
- **Link to Source**: [hal/riscv-rvv/src/imgproc/moments.cpp](../../../../hal/riscv-rvv/src/imgproc/moments.cpp)

## Purpose and Role

This file is located in the `hal/riscv-rvv/src/imgproc` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

// Copyright (C) 2025, Institute of Software, Chinese Academy of Sciences.

#include "rvv_hal.hpp"

namespace cv { namespace rvv_hal { namespace imgproc {

#if CV_HAL_RVV_1P0_ENABLED

namespace {

class MomentsInvoker : public ParallelLoopBody
{
public:
    template<typename... Args>
    MomentsInvoker(std::function<int(int, int, Args...)> _func, Args&&... args)
    {
        func = std::bind(_func, std::placeholders::_1, std::placeholders::_2, std::forward<Args>(args)...);
    }

    virtual void operator()(const Range& range) const override
    {
        func(range.start, range.end);
    }

private:
    std::function<int(int, int)> func;
};

template<typename... Args>
static inline int invoke(int width, int height, std::function<int(int, int, Args...)> func, Args&&... args)
{
    cv::parallel_for_(Range(1, height), MomentsInvoker(func, std::forward<Args>(args)...), static_cast<double>((width - 1) * height) / (1 << 10));
    return func(0, 1, std::forward<Args>(args)...);
}

template<typename helper> struct rvv;
template<> struct rvv<RVV_U32M2>
{
    static inline vuint8mf2_t vid(size_t a) { return __riscv_vid_v_u8mf2(a); }
    static inline RVV_U32M2::VecType vcvt(vuint8mf2_t a, size_t b) { return __riscv_vzext_vf4(a, b); }
};
template<> struct rvv<RVV_U32M4>
{
    static inline vuint8m1_t vid(size_t a) { return __riscv_vid_v_u8m1(a); }
    static inline RVV_U32M4::VecType vcvt(vuint8m1_t a, size_t b) { return __riscv_vzext_vf4(a, b); }
};
template<> struct rvv<RVV_I32M2>
{
    static inline vuint8mf2_t vid(size_t a) { return __riscv_vid_v_u8mf2(a); }
    static inline RVV_I32M2::VecType vcvt(vuint8mf2_t a, size_t b) { return RVV_I32M2::reinterpret(__riscv_vzext_vf4(a, b)); }
};
template<> struct rvv<RVV_F64M4>
{
    static inline vuint8mf2_t vid(size_t a) { return __riscv_vid_v_u8mf2(a); }
    static inline RVV_F64M4::VecType vcvt(vuint8mf2_t a, size_t b) { return __riscv_vfcvt_f(__riscv_vzext_vf8(a, b), b); }
};

constexpr int TILE_SIZE = 32;

template<bool binary, typename T, typename helperT, typename helperWT, typename helperMT>
static inline int imageMoments(int start, int end, const uchar* src_data, size_t src_step, int full_width, int full_height, double* m, std::mutex* mt)
{
    double mm[10] = {0};
    for (int yy = start; yy < end; yy++)
    {
        const int y = yy * TILE_SIZE;
        const int height = std::min(TILE_SIZE, full_height - y);
        for (int x = 0; x < full_width; x += TILE_SIZE)
        {
            const int width = std::min(TILE_SIZE, full_width - x);
            double mom[10] = {0};

            for (int i = 0; i < height; i++)
            {
                auto id = rvv<helperWT>::vid(helperT::setvlmax());
                auto v0 = helperWT::vmv(0, helperWT::setvlmax());
                auto v1 = helperWT::vmv(0, helperWT::setvlmax());
                auto v2 = helperWT::vmv(0, helperWT::setvlmax());
                auto v3 = helperMT::vmv(0, helperMT::setvlmax());

                int vl;
                for (int j = 0; j < width; j += vl)
                {
                    vl = helperT::setvl(width - j);
                    typename helperWT::VecType p;
                    if (binary)
                    {
                        auto src = RVV_SameLen<T, helperT>::vload(reinterpret_cast<const T*>(src_data + (i + y) * src_step) + j + x, vl);
                        p = __riscv_vmerge(helperWT::vmv(0, vl), helperWT::vmv(255, vl), RVV_SameLen<T, helperT>::vmne(src, 0, vl), vl);
                    }
                    else
                    {
                        p = helperWT::cast(helperT::vload(reinterpret_cast<const typename helperT::ElemType*>(src_data + (i + y) * src_step) + j + x, vl), vl);
                    }
                    auto xx = rvv<helperWT>::vcvt(id, vl);
                    auto xp = helperWT::vmul(xx, p, vl);
                    v0 = helperWT::vadd_tu(v0, v0, p, vl);
                    v1 = helperWT::vadd_tu(v1, v1, xp, vl);
                    auto xxp = helperWT::vmul(xx, xp, vl);
                    v2 = helperWT::vadd_tu(v2, v2, xxp, vl);
                    v3 = helperMT::vadd_tu(v3, v3, helperMT::vmul(helperMT::cast(xx, vl), helperMT::cast(xxp, vl), vl), vl);
                    id = __riscv_vadd(id, vl, vl);
                }

                auto x0 = RVV_BaseType<helperWT>::vmv_x(helperWT::vredsum(v0, RVV_BaseType<helperWT>::vmv_s(0, RVV_BaseType<helperWT>::setvlmax()), helperWT::setvlmax()));
                auto x1 = RVV_BaseType<helperWT>::vmv_x(helperWT::vredsum(v1, RVV_BaseType<helperWT>::vmv_s(0, RVV_BaseType<helperWT>::setvlmax()), helperWT::setvlmax()));
                auto x2 = RVV_BaseType<helperWT>::vmv_x(helperWT::vredsum(v2, RVV_BaseType<helperWT>::vmv_s(0, RVV_BaseType<helperWT>::setvlmax()), helperWT::setvlmax()));
                auto x3 = RVV_BaseType<helperMT>::vmv_x(helperMT::vredsum(v3, RVV_BaseType<helperMT>::vmv_s(0, RVV_BaseType<helperMT>::setvlmax()), helperMT::setvlmax()));
                typename helperWT::ElemType py = i * x0, sy = i*i;

                mom[9] += static_cast<typename helperMT::ElemType>(py) * sy;
                mom[8] += static_cast<typename helperMT::ElemType>(x1) * sy;
                mom[7] += static_cast<typename helperMT::ElemType>(x2) * i;
                mom[6] += x3;
                mom[5] += x0 * sy;
                mom[4] += x1 * i;
                mom[3] += x2;
                mom[2] += py;
                mom[1] += x1;
                mom[0] += x0;
            }

            if (binary)
            {
                mom[0] /= 255, mom[1] /= 255, mom[2] /= 255, mom[3] /= 255, mom[4] /= 255;
                mom[5] /= 255, mom[6] /= 255, mom[7] /= 255, mom[8] /= 255, mom[9] /= 255;
            }
            double xm = x * mom[0], ym = y * mom[0];
            mm[0] += mom[0];
            mm[1] += mom[1] + xm;
            mm[2] += mom[2] + ym;
            mm[3] += mom[3] + x * (mom[1] * 2 + xm);
            mm[4] += mom[4] + x * (mom[2] + ym) + y * mom[1];
            mm[5] += mom[5] + y * (mom[2] * 2 + ym);
            mm[6] += mom[6] + x * (3. * mom[3] + x * (3. * mom[1] + xm));
            mm[7] += mom[7] + x * (2 * (mom[4] + y * mom[1]) + x * (mom[2] + ym)) + y * mom[3];
            mm[8] += mom[8] + y * (2 * (mom[4] + x * mom[2]) + y * (mom[1] + xm)) + x * mom[5];
            mm[9] += mom[9] + y * (3. * mom[5] + y * (3. * mom[2] + ym));
        }
    }

    std::lock_guard<std::mutex> lk(*mt);
    for (int i = 0; i < 10; i++)
        m[i] += mm[i];
    return CV_HAL_ERROR_OK;
}

} // anonymous

// the algorithm is copied from imgproc/src/moments.cpp,
// in the function cv::Moments cv::moments
int imageMoments(const uchar* src_data, size_t src_step, int src_type, int width, int height, bool binary, double m[10])
{
    if (src_type != CV_16UC1 && src_type != CV_16SC1 && src_type != CV_32FC1 && src_type != CV_64FC1)
        return CV_HAL_ERROR_NOT_IMPLEMENTED;

    std::fill(m, m + 10, 0);
    const int cnt = (height + TILE_SIZE - 1) / TILE_SIZE;
    std::mutex mt;
    switch (static_cast<int>(binary)*100 + src_type)
    {
    case CV_16UC1:
        return invoke(width, cnt, {imageMoments<false, ushort, RVV_U16M1, RVV_U32M2, RVV_U64M4>}, src_data, src_step, width, height, m, &mt);
    case CV_16SC1:
        return invoke(width, cnt, {imageMoments<false, short, RVV_I16M1, RVV_I32M2, RVV_I64M4>}, src_data, src_step, width, height, m, &mt);
    case CV_32FC1:
        return invoke(width, cnt, {imageMoments<false, float, RVV_F32M2, RVV_F64M4, RVV_F64M4>}, src_data, src_step, width, height, m, &mt);
    case CV_64FC1:
        return invoke(width, cnt, {imageMoments<false, double, RVV_F64M4, RVV_F64M4, RVV_F64M4>}, src_data, src_step, width, height, m, &mt);
    case 100 + CV_16UC1:
        return invoke(width, cnt, {imageMoments<true, ushort, RVV_U8M1, RVV_U32M4, RVV_U32M4>}, src_data, src_step, width, height, m, &mt);
    case 100 + CV_16SC1:
        return invoke(width, cnt, {imageMoments<true, short, RVV_U8M1, RVV_U32M4, RVV_U32M4>}, src_data, src_step, width, height, m, &mt);
    case 100 + CV_32FC1:
        return invoke(width, cnt, {imageMoments<true, float, RVV_U8M1, RVV_U32M4, RVV_U32M4>}, src_data, src_step, width, height, m, &mt);
    case 100 + CV_64FC1:
        return invoke(width, cnt, {imageMoments<true, double, RVV_U8M1, RVV_U32M4, RVV_U32M4>}, src_data, src_step, width, height, m, &mt);
    }

    return CV_HAL_ERROR_NOT_IMPLEMENTED;
}

#endif // CV_HAL_RVV_1P0_ENABLED

}}} // cv::rvv_hal::imgproc
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

- **MomentsInvoker**: A class/struct defined in this file
- **rvv**: A class/struct defined in this file

### Functions and Methods

- **cv()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `rvv_hal.hpp`

**Python Imports:**
- `imgproc`


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

