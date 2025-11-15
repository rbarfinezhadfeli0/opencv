# Documentation for `modules/dnn/perf/perf_convolution1d.cpp`

## File Metadata

- **Full Path**: `modules/dnn/perf/perf_convolution1d.cpp`
- **File Name**: `perf_convolution1d.cpp`
- **File Size**: 5,265 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/perf/perf_convolution1d.cpp](../../../modules/dnn/perf/perf_convolution1d.cpp)

## Purpose and Role

This file is located in the `modules/dnn/perf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "perf_precomp.hpp"
#include <opencv2/dnn/shape_utils.hpp>

namespace opencv_test {

struct Conv1DParam_t {
    int kernel;
    struct BlobShape { int dims[3]; } shapeIn;
    int outCN;
    int groups;
    int stride;
    int dilation;
    int pad[2];
    const char* padMode;
    bool hasBias;
    double declared_flops;
};
// Details: #12142
static const Conv1DParam_t testConvolution1DConfigs[] = {
        {3, {{1, 6, 10}}, 6, 1, 1, 1, {0, 0}, "VALID", true, 1776.},
        {3, {{1, 2, 19}}, 2, 2, 2, 1, {1, 1}, "", true, 260.},
        {3, {{1, 2, 25}}, 2, 2, 1, 1, {2, 2}, "SAME", false, 650.},
};

struct Conv1DParamID
{
    enum {
        CONV_0 = 0,
        CONV_LAST = sizeof(testConvolution1DConfigs) / sizeof(testConvolution1DConfigs[0])
    };
    int val_;
    Conv1DParamID(int val = 0) : val_(val) {}
    operator int() const { return val_; }
    static ::testing::internal::ParamGenerator<Conv1DParamID> all()
    {
        enum { NUM = (int)CONV_LAST };
        Conv1DParamID v_[NUM]; for (int i = 0; i < NUM; ++i) { v_[i] = Conv1DParamID(i); } // reduce generated code size
        return ::testing::ValuesIn(v_, v_ + NUM);
    }
};
static inline void PrintTo(const Conv1DParamID& v, std::ostream* os)
{
    CV_Assert((int)v >= 0); CV_Assert((int)v < Conv1DParamID::CONV_LAST);
    const Conv1DParam_t& p = testConvolution1DConfigs[(int)v];

    *os << "GFLOPS=" << cv::format("%.3f", p.declared_flops * 1e-9)
        << ", K=[" << p.kernel << "]"
        << ", IN={" << p.shapeIn.dims[0] << ", " << p.shapeIn.dims[1] << ", " << p.shapeIn.dims[2] << "}"
        << ", OCN=" << p.outCN;
    if (p.groups > 1)
        *os << ", G=" << p.groups;
    if (p.stride != 1)
        *os << ", S=" << p.stride;
    if (p.dilation != 1)
        *os << ", D="  << p.dilation;
    if (p.pad[0] != 0 && p.pad[1] != 0 )
        *os << ", P=(" << p.pad[0] << ", " << p.pad[1] << ")";
    if (!((std::string)p.padMode).empty())
        *os << ", PM=" << ((std::string)p.padMode);
    if (p.hasBias)
        *os << ", BIAS";
}


typedef tuple<Conv1DParamID, tuple<Backend, Target> > Conv1DTestParam_t;
typedef TestBaseWithParam<Conv1DTestParam_t> Conv1D;

PERF_TEST_P_(Conv1D, conv1d)
{
    int test_id = (int)get<0>(GetParam());
    ASSERT_GE(test_id, 0); ASSERT_LT(test_id, Conv1DParamID::CONV_LAST);
    const Conv1DParam_t& params = testConvolution1DConfigs[test_id];
    double declared_flops = params.declared_flops;

    DictValue kernel   = DictValue::arrayInt(&params.kernel, 1);
    DictValue stride   = DictValue::arrayInt(&params.stride, 1);
    DictValue pad      = DictValue::arrayInt(&params.pad[0], 2);
    DictValue dilation = DictValue::arrayInt(&params.dilation, 1);

    MatShape inputShape = MatShape(params.shapeIn.dims, params.shapeIn.dims + 3);
    int outChannels = params.outCN;
    int groups = params.groups;
    std::string padMode(params.padMode);

    bool hasBias = params.hasBias;
    Backend backendId = get<0>(get<1>(GetParam()));
    Target targetId = get<1>(get<1>(GetParam()));

    if (targetId != DNN_TARGET_CPU)
        throw SkipTestException("Only CPU is supported");

    int inChannels = inputShape[1];

    int sz[] = {outChannels, inChannels / groups, params.kernel};
    Mat weights(3, &sz[0], CV_32F);
    randu(weights, -1.0f, 1.0f);

    LayerParams lp;
    lp.set("kernel_size", kernel);
    lp.set("pad", pad);
    if (!padMode.empty())
        lp.set("pad_mode", padMode);

    lp.set("stride", stride);
    lp.set("dilation", dilation);
    lp.set("num_output", outChannels);
    lp.set("group", groups);
    lp.set("bias_term", hasBias);
    lp.type = "Convolution";
    lp.name = "testLayer";
    lp.blobs.push_back(weights);

    if (hasBias)
    {
        Mat bias(1, outChannels, CV_32F);
        randu(bias, -1.0f, 1.0f);
        lp.blobs.push_back(bias);
    }

    int inpSz[] = {1, inChannels, inputShape[2]};
    Mat input(3, &inpSz[0], CV_32F);
    randu(input, -1.0f, 1.0f);

    Net net;
    net.addLayerToPrev(lp.name, lp.type, lp);

    net.setInput(input);
    net.setPreferableBackend(backendId);
    net.setPreferableTarget(targetId);

    // warmup
    Mat output = net.forward();

    MatShape netInputShape = shape(input);
    size_t weightsMemory = 0, blobsMemory = 0;
    net.getMemoryConsumption(netInputShape, weightsMemory, blobsMemory);
    int64 flops = net.getFLOPS(netInputShape);
    CV_Assert(flops > 0);

    std::cout
    << "IN=" << divUp(input.total() * input.elemSize(), 1u<<10) << " Kb " << netInputShape
    << "    OUT=" << divUp(output.total() * output.elemSize(), 1u<<10) << " Kb " << shape(output)
    << "    Weights(parameters): " << divUp(weightsMemory, 1u<<10) << " Kb"
    << "    MFLOPS=" << flops * 1e-6 << std::endl;

    TEST_CYCLE()
    {
        Mat res = net.forward();
    }
    EXPECT_NEAR(flops, declared_flops, declared_flops * 1e-6);
    SANITY_CHECK_NOTHING();
}

INSTANTIATE_TEST_CASE_P(/**/, Conv1D, Combine(
        Conv1DParamID::all(),
        dnnBackendsAndTargets(false, false)  // defined in ../test/test_common.hpp
));

} // namespace
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

- **Conv1DParamID**: A class/struct defined in this file
- **BlobShape**: A class/struct defined in this file
- **Conv1DParam_t**: A class/struct defined in this file

### Functions and Methods

- **tuple()**: A function/method defined in this file
- **TestBaseWithParam()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/dnn/shape_utils.hpp`
- `perf_precomp.hpp`


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

