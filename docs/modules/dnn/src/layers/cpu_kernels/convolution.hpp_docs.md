# Documentation for `modules/dnn/src/layers/cpu_kernels/convolution.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/cpu_kernels/convolution.hpp`
- **File Name**: `convolution.hpp`
- **File Size**: 5,103 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/layers/cpu_kernels/convolution.hpp](../../../../../modules/dnn/src/layers/cpu_kernels/convolution.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src/layers/cpu_kernels` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef OPENCV_FAST_CONVOLUTION_HPP
#define OPENCV_FAST_CONVOLUTION_HPP

#include "opencv2/core/hal/intrin.hpp"
#include "opencv2/dnn/all_layers.hpp"

#ifndef CONV_PRAM
#define CONV_PRAM
#if CV_NEON && CV_NEON_AARCH64  // 32 registers.
#define CONV_MR_FP32 4
#define CONV_NR_FP32 28

// The FP16 can only be supported by ARM64 and with FP16 FMA supported.
#if CV_FP16 && CV_TRY_NEON_FP16 // check FP16 FMA.
#define CONV_ARM_FP16 1
#endif

#ifdef CONV_ARM_FP16
// Currently, only ARM 64 support FP16.
#define CONV_MR_FP16 8
#define CONV_NR_FP16 24
#endif

#elif CV_NEON              // 16 registers.
#define CONV_MR_FP32 4
#define CONV_NR_FP32 12
#elif CV_WASM_SIMD
#define CONV_MR_FP32 4
#define CONV_NR_FP32 8
#else // SIMD 128, AVX or AVX2
#define CONV_MR_FP32 4
#define CONV_NR_FP32 24
#endif

enum {
    CONV_WINO_STEP=6,
    CONV_WINO_KSIZE=3,
    CONV_WINO_SIZE=CONV_WINO_STEP+CONV_WINO_KSIZE - 1, // 8
    CONV_WINO_AREA=CONV_WINO_SIZE*CONV_WINO_SIZE,
};

// NOTE that: CONV_TYPE_DEPTHWISE is for 3x3 depthwise conv, and others depthwise will be set as CONV_TYPE_DEPTHWISE_REMAIN.
enum { CONV_TYPE_GENERIC=0, CONV_TYPE_DEPTHWISE=1, CONV_TYPE_WINOGRAD3X3=2, CONV_TYPE_DEPTHWISE_REMAIN=3 };
enum { CONV_1D = 0, CONV_2D = 1, CONV_3D = 2 };

#endif

namespace cv {
namespace dnn {

struct FastConv
{
    int ngroups;
    int K, C, Hk, Wk, Dk;
    int stride_h, stride_w, stride_d;
    int dilation_h, dilation_w, dilation_d;
    int pad_top, pad_bottom, pad_left, pad_right, pad_front, pad_behind;

    std::vector<float> weightsBuf;     // For generic Conv 2D
    std::vector<float> weightsWinoBuf; // For Winograd F(6x6, 3x3).
    std::vector<float> biasBuf;
    float* getWeights();
    float* getWeightsWino();

    std::vector<hfloat> weightsBuf_FP16;
    std::vector<hfloat> weightsWinoBuf_FP16;
    hfloat* getWeightsFP16();
    hfloat* getWeightsWinoFP16();

    int conv_type;
    int conv_dim;  // Flag for conv1d, conv2d, or conv3d.
    bool useFP16 = false; // Only ARMv8 is supported.
#if CV_SIMD128
    bool useSIMD128 = true;
#else
    bool useSIMD128 = false;
#endif

#if CV_NEON
    bool useNEON = checkHardwareSupport(CPU_NEON);
#else
    bool useNEON = false;
#endif

    bool useAVX   = checkHardwareSupport(CPU_AVX);
    bool useAVX2  = checkHardwareSupport(CPU_AVX2);
    bool useRVV   = checkHardwareSupport(CPU_RVV);
};

// return a FastConv instance.
Ptr<FastConv> initFastConv(
        InputArray weightsMat,
        float* srcBias,
        int ngroups,
        int K, int C,
        const std::vector<size_t>& kernel_size,
        const std::vector<size_t>& strides,
        const std::vector<size_t>& dilations,
        const std::vector<size_t>& pads_begin,
        const std::vector<size_t>& pads_end,
        int conv_dim,
        const bool useFP16,
        bool useWinograd);

// It contains different computing branches, like winograd, 1x1 conv.
void runFastConv(InputArray _input, OutputArray _output, const Ptr<FastConv>& conv, int ntasks,
                   const Ptr<ActivationLayer>& actLayer, const std::vector<float>& reluslope, bool fusedAdd);

void runDepthwise(InputArray _input, OutputArray _output, const Ptr<FastConv>& conv, ActivationLayer* activ,
                  const std::vector<float>& reluslope, bool fusedAdd);

int runWinograd63(InputArray _input, InputArray _fusedAddMat, OutputArray _output, const Ptr<FastConv>& conv, int ntasks,
                  float minval, float maxval, ActivationLayer* activ, bool ifMinMaxAct);

// Work around of NEON, the following functions are only used internally.
namespace opt_NEON {
#if CV_NEON
void convBlock_F32(int np, const float* a, const float* b, float* c, int ldc, bool init_c, int width, const int convMR, const int convNR);

void convBlockMR1_F32(int np, const float* a, const float* b, float* c, const float bias, bool init_c,
                      const float minval, const float maxval, bool ifMinMaxAct, const int width, const int convNR);
#endif // CV_NEON
} // namespace opt_NEON.



// === Function tables
struct Winofunc
{
    void (*accum)(const uchar* inwptr, const uchar* wptr, uchar* outbuf, int Cg, int iblock, const int winoIblock, const int winoKblock, const int winoAtomF32, const int winoNatomF32);
    void (*BtXB_8x8)(const float* inptr, int inpstep, uchar* outptr, int Cg, const int winoIblock, const int winoAtomF32);
    void (*AtXA_8x8)(const uchar* inptr, int inpstep, float* bpptr, int bpstep, float* outptr, int outstep, float bias, float minval, float maxval, bool ifMinMaxAct);
    int iblock;
    int natom;
    int esz;

    bool isGood() const { return accum && BtXB_8x8 && AtXA_8x8 && iblock > 0 && natom > 0 && esz > 0; }
    static Winofunc empty() { return {0, 0, 0, 0, 0, 0}; }
};

// === wrapper calls (implemented in .dispatch.cpp)
Winofunc getWinofunc_F32();
Winofunc getWinofunc_F16();


} // namespace dnn
} // namespace cv

#endif //OPENCV_FAST_CONVOLUTION_HPP
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

- **Winofunc**: A class/struct defined in this file
- **FastConv**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_FAST_CONVOLUTION_HPP()**: A function/method defined in this file
- **empty()**: A function/method defined in this file
- **getWinofunc_F16()**: A function/method defined in this file
- **CONV_ARM_FP16()**: A function/method defined in this file
- **CONV_PRAM()**: A function/method defined in this file
- **getWinofunc_F32()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core/hal/intrin.hpp`
- `opencv2/dnn/all_layers.hpp`


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

