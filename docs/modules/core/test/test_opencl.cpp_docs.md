# Documentation for `modules/core/test/test_opencl.cpp`

## File Metadata

- **Full Path**: `modules/core/test/test_opencl.cpp`
- **File Name**: `test_opencl.cpp`
- **File Size**: 6,204 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/core/test/test_opencl.cpp](../../../modules/core/test/test_opencl.cpp)

## Purpose and Role

This file is located in the `modules/core/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "test_precomp.hpp"
#include "opencv2/ts/ocl_test.hpp"

namespace opencv_test {
namespace ocl {

static
testing::internal::ParamGenerator<std::string> getOpenCLTestConfigurations()
{
    if (!cv::ocl::useOpenCL())
    {
        return testing::ValuesIn(std::vector<std::string>());
    }

    std::vector<std::string> configurations = {
        ":GPU:0",
        ":GPU:1",
        ":CPU:0",
    };
    return testing::ValuesIn(configurations);
}


static void executeUMatCall(bool requireOpenCL = true)
{
    UMat a(100, 100, CV_8UC1, Scalar::all(0));
    UMat b;
    cv::add(a, Scalar::all(1), b);
    Mat b_cpu = b.getMat(ACCESS_READ);
    EXPECT_EQ(0, cv::norm(b_cpu - 1, NORM_INF));

    if (requireOpenCL)
    {
        EXPECT_TRUE(cv::ocl::useOpenCL());
    }
}

TEST(OCL_Context, createFromDevice)
{
    bool useOCL = cv::ocl::useOpenCL();

    OpenCLExecutionContext ctx = OpenCLExecutionContext::getCurrent();

    if (!useOCL)
    {
        ASSERT_TRUE(ctx.empty());  // Other tests should not broke global state
        throw SkipTestException("OpenCL is not available / disabled");
    }

    ASSERT_FALSE(ctx.empty());

    ocl::Device device = ctx.getDevice();
    ASSERT_FALSE(device.empty());

    ocl::Context context = ocl::Context::fromDevice(device);
    ocl::Context context2 = ocl::Context::fromDevice(device);

    EXPECT_TRUE(context.getImpl() == context2.getImpl()) << "Broken cache for OpenCL context (device)";
}

TEST(OCL_OpenCLExecutionContextDefault, basic)
{
    bool useOCL = cv::ocl::useOpenCL();

    OpenCLExecutionContext ctx = OpenCLExecutionContext::getCurrent();

    if (!useOCL)
    {
        ASSERT_TRUE(ctx.empty());  // Other tests should not broke global state
        throw SkipTestException("OpenCL is not available / disabled");
    }

    ASSERT_FALSE(ctx.empty());

    ocl::Context context = ctx.getContext();
    ocl::Context context2 = ocl::Context::getDefault();
    EXPECT_TRUE(context.getImpl() == context2.getImpl());

    ocl::Device device = ctx.getDevice();
    ocl::Device device2 = ocl::Device::getDefault();
    EXPECT_TRUE(device.getImpl() == device2.getImpl());

    ocl::Queue queue = ctx.getQueue();
    ocl::Queue queue2 = ocl::Queue::getDefault();
    EXPECT_TRUE(queue.getImpl() == queue2.getImpl());
}

TEST(OCL_OpenCLExecutionContextDefault, createAndBind)
{
    bool useOCL = cv::ocl::useOpenCL();

    OpenCLExecutionContext ctx = OpenCLExecutionContext::getCurrent();

    if (!useOCL)
    {
        ASSERT_TRUE(ctx.empty());  // Other tests should not broke global state
        throw SkipTestException("OpenCL is not available / disabled");
    }

    ASSERT_FALSE(ctx.empty());

    ocl::Context context = ctx.getContext();
    ocl::Device device = ctx.getDevice();

    OpenCLExecutionContext ctx2 = OpenCLExecutionContext::create(context, device);
    ASSERT_FALSE(ctx2.empty());

    try
    {
        ctx2.bind();
        executeUMatCall();
        ctx.bind();
        executeUMatCall();
    }
    catch (...)
    {
        ctx.bind();  // restore
        throw;
    }
}

typedef testing::TestWithParam<std::string> OCL_OpenCLExecutionContext_P;

TEST_P(OCL_OpenCLExecutionContext_P, multipleBindAndExecute)
{
    bool useOCL = cv::ocl::useOpenCL();

    OpenCLExecutionContext ctx = OpenCLExecutionContext::getCurrent();

    if (!useOCL)
    {
        ASSERT_TRUE(ctx.empty());  // Other tests should not broke global state
        throw SkipTestException("OpenCL is not available / disabled");
    }

    ASSERT_FALSE(ctx.empty());

    std::string opencl_device = GetParam();
    ocl::Context context = ocl::Context::create(opencl_device);
    if (context.empty())
    {
        throw SkipTestException(std::string("OpenCL device is not available: '") + opencl_device + "'");
    }

    ocl::Device device = context.device(0);

    OpenCLExecutionContext ctx2 = OpenCLExecutionContext::create(context, device);
    ASSERT_FALSE(ctx2.empty());

    try
    {
        std::cout << "ctx2..." << std::endl;
        ctx2.bind();
        executeUMatCall();
        std::cout << "ctx..." << std::endl;
        ctx.bind();
        executeUMatCall();
    }
    catch (...)
    {
        ctx.bind();  // restore
        throw;
    }
}

TEST_P(OCL_OpenCLExecutionContext_P, ScopeTest)
{
    bool useOCL = cv::ocl::useOpenCL();

    OpenCLExecutionContext ctx = OpenCLExecutionContext::getCurrent();

    if (!useOCL)
    {
        ASSERT_TRUE(ctx.empty());  // Other tests should not broke global state
        throw SkipTestException("OpenCL is not available / disabled");
    }

    ASSERT_FALSE(ctx.empty());

    std::string opencl_device = GetParam();
    ocl::Context context = ocl::Context::create(opencl_device);
    if (context.empty())
    {
        throw SkipTestException(std::string("OpenCL device is not available: '") + opencl_device + "'");
    }

    ocl::Device device = context.device(0);

    OpenCLExecutionContext ctx2 = OpenCLExecutionContext::create(context, device);
    ASSERT_FALSE(ctx2.empty());

    try
    {
        OpenCLExecutionContextScope ctx_scope(ctx2);
        executeUMatCall();
    }
    catch (...)
    {
        ctx.bind();  // restore
        throw;
    }

    executeUMatCall();
}

INSTANTIATE_TEST_CASE_P(/*nothing*/, OCL_OpenCLExecutionContext_P, getOpenCLTestConfigurations());


typedef testing::TestWithParam<UMatUsageFlags> UsageFlagsFixture;
OCL_TEST_P(UsageFlagsFixture, UsageFlagsRetained)
{
    if (!cv::ocl::useOpenCL())
    {
        throw SkipTestException("OpenCL is not available / disabled");
    }

    const UMatUsageFlags usage = GetParam();
    cv::UMat flip_in(10, 10, CV_32F, usage);
    cv::UMat flip_out(usage);
    cv::flip(flip_in, flip_out, 1);
    cv::ocl::finish();

    ASSERT_EQ(usage, flip_in.usageFlags);
    ASSERT_EQ(usage, flip_out.usageFlags);
}

INSTANTIATE_TEST_CASE_P(
    /*nothing*/,
    UsageFlagsFixture,
    testing::Values(USAGE_DEFAULT, USAGE_ALLOCATE_HOST_MEMORY, USAGE_ALLOCATE_DEVICE_MEMORY)
);


} } // namespace opencv_test::ocl
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

- **testing()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/ts/ocl_test.hpp`
- `test_precomp.hpp`


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

