# Documentation for `modules/gapi/test/internal/gapi_int_executor_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/internal/gapi_int_executor_tests.cpp`
- **File Name**: `gapi_int_executor_tests.cpp`
- **File Size**: 11,475 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/internal/gapi_int_executor_tests.cpp](../../../../modules/gapi/test/internal/gapi_int_executor_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test/internal` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018-2022 Intel Corporation


#include "../test_precomp.hpp"
#include "../gapi_mock_kernels.hpp"

#include <opencv2/gapi/core.hpp>

namespace opencv_test
{

namespace
{

class GMockExecutable final: public cv::gimpl::GIslandExecutable
{
    virtual inline bool canReshape() const override {
        return m_priv->m_can_reshape;
    }

    virtual void reshape(ade::Graph&, const GCompileArgs&) override
    {
        m_priv->m_reshape_counter++;
    }
    virtual void handleNewStream() override {  }
    virtual void run(std::vector<InObj>&&, std::vector<OutObj>&&) override { }
    virtual bool allocatesOutputs() const override
    {
        return true;
    }

    virtual cv::RMat allocate(const cv::GMatDesc&) const override
    {
        m_priv->m_allocate_counter++;
        return cv::RMat();
    }

    // NB: GMockBackendImpl creates new unique_ptr<GMockExecutable>
    // on every compile call. Need to share counters between instances in order
    // to validate it in tests.
    struct Priv
    {
        bool m_can_reshape;
        int  m_reshape_counter;
        int  m_allocate_counter;
    };

    std::shared_ptr<Priv> m_priv;

public:
    GMockExecutable(bool can_reshape = true)
        : m_priv(new Priv{can_reshape, 0, 0})
    {
    }

    void setReshape(bool can_reshape) { m_priv->m_can_reshape = can_reshape; }

    int getReshapeCounter()  const { return m_priv->m_reshape_counter;  }
    int getAllocateCounter() const { return m_priv->m_allocate_counter; }
};

class GMockBackendImpl final: public cv::gapi::GBackend::Priv
{
    virtual void unpackKernel(ade::Graph            &,
                              const ade::NodeHandle &,
                              const cv::GKernelImpl &) override { }

    virtual EPtr compile(const ade::Graph &,
                         const cv::GCompileArgs &,
                         const std::vector<ade::NodeHandle> &) const override
    {
        ++m_compile_counter;
        return EPtr{new GMockExecutable(m_exec)};
    }

    mutable int     m_compile_counter = 0;
    GMockExecutable m_exec;

    virtual bool controlsMerge() const override {
        return true;
    }

    virtual bool allowsMerge(const cv::gimpl::GIslandModel::Graph &,
                             const ade::NodeHandle &,
                             const ade::NodeHandle &,
                             const ade::NodeHandle &) const override {
        return false;
    }

public:
    GMockBackendImpl(const GMockExecutable& exec) : m_exec(exec) { }
    int getCompileCounter() const { return m_compile_counter; }
};

class GMockFunctor : public gapi::cpu::GOCVFunctor
{
public:
    GMockFunctor(cv::gapi::GBackend backend,
                 const char* id,
                 const Meta &meta,
                 const Impl& impl)
        : gapi::cpu::GOCVFunctor(id, meta, impl), m_backend(backend)
    {
    }

    cv::gapi::GBackend backend() const override { return m_backend; }

private:
    cv::gapi::GBackend m_backend;
};

template<typename K, typename Callable>
GMockFunctor mock_kernel(const cv::gapi::GBackend& backend, Callable c)
{
    using P = cv::detail::OCVCallHelper<Callable, typename K::InArgs, typename K::OutArgs>;
    return GMockFunctor{ backend
                       , K::id()
                       , &K::getOutMeta
                       , std::bind(&P::callFunctor, std::placeholders::_1, c)
                       };
}

void dummyFooImpl(const cv::Mat&, cv::Mat&)                 { }
void dummyBarImpl(const cv::Mat&, const cv::Mat&, cv::Mat&) { }

struct GExecutorReshapeTest: public ::testing::Test
{
    GExecutorReshapeTest()
        : comp([](){
                cv::GMat in;
                cv::GMat out = I::Bar::on(I::Foo::on(in), in);
                return cv::GComputation(in, out);
          })
    {
        backend_impl1 = std::make_shared<GMockBackendImpl>(island1);
        backend1      = cv::gapi::GBackend{backend_impl1};
        backend_impl2 = std::make_shared<GMockBackendImpl>(island2);
        backend2      = cv::gapi::GBackend{backend_impl2};
        auto kernel1  = mock_kernel<I::Foo>(backend1, dummyFooImpl);
        auto kernel2  = mock_kernel<I::Bar>(backend2, dummyBarImpl);
        pkg           = cv::gapi::kernels(kernel1, kernel2);
        in_mat1       = cv::Mat::eye(32, 32, CV_8UC1);
        in_mat2       = cv::Mat::eye(64, 64, CV_8UC1);
    }

    cv::GComputation                  comp;
    GMockExecutable                   island1;
    std::shared_ptr<GMockBackendImpl> backend_impl1;
    cv::gapi::GBackend                backend1;
    GMockExecutable                   island2;
    std::shared_ptr<GMockBackendImpl> backend_impl2;
    cv::gapi::GBackend                backend2;
    cv::GKernelPackage                pkg;
    cv::Mat                           in_mat1, in_mat2, out_mat;
};

} // anonymous namespace

// FIXME: avoid code duplication
// The below graph and config is taken from ComplexIslands test suite
TEST(GExecutor, SmokeTest)
{
    cv::GMat    in[2];
    cv::GMat    tmp[4];
    cv::GScalar scl;
    cv::GMat    out[2];

    tmp[0] = cv::gapi::bitwise_not(cv::gapi::bitwise_not(in[0]));
    tmp[1] = cv::gapi::boxFilter(in[1], -1, cv::Size(3,3));
    tmp[2] = tmp[0] + tmp[1]; // FIXME: handle tmp[2] = tmp[0]+tmp[2] typo
    scl    = cv::gapi::sum(tmp[1]);
    tmp[3] = cv::gapi::medianBlur(tmp[1], 3);
    out[0] = tmp[2] + scl;
    out[1] = cv::gapi::boxFilter(tmp[3], -1, cv::Size(3,3));

    //       isl0                                         #internal1
    //       ...........................                  .........
    // (in1) -> NotNot ->(tmp0) --> Add ---------> (tmp2) --> AddC -------> (out1)
    //       :.....................^...:                  :..^....:
    //                             :                         :
    //                             :                         :
    //      #internal0             :                         :
    //        .....................:.........                :
    // (in2) -> Blur -> (tmp1) ----'--> Sum ----> (scl0) ----'
    //        :..........:..................:                  isl1
    //                   :           ..............................
    //                   `------------> Median -> (tmp3) --> Blur -------> (out2)
    //                               :............................:

    cv::gapi::island("isl0", cv::GIn(in[0], tmp[1]),  cv::GOut(tmp[2]));
    cv::gapi::island("isl1", cv::GIn(tmp[1]), cv::GOut(out[1]));

    cv::Mat in_mat1 = cv::Mat::eye(32, 32, CV_8UC1);
    cv::Mat in_mat2 = cv::Mat::eye(32, 32, CV_8UC1);
    cv::Mat out_gapi[2];

    // Run G-API:
    cv::GComputation(cv::GIn(in[0],   in[1]),    cv::GOut(out[0],      out[1]))
              .apply(cv::gin(in_mat1, in_mat2),  cv::gout(out_gapi[0], out_gapi[1]));

    // Run OpenCV
    cv::Mat out_ocv[2];
    {
        cv::Mat    ocv_tmp0;
        cv::Mat    ocv_tmp1;
        cv::Mat    ocv_tmp2;
        cv::Mat    ocv_tmp3;
        cv::Scalar ocv_scl;

        ocv_tmp0 = in_mat1; // skip !(!)
        cv::boxFilter(in_mat2, ocv_tmp1, -1, cv::Size(3,3));
        ocv_tmp2 = ocv_tmp0 + ocv_tmp1;
        ocv_scl  = cv::sum(ocv_tmp1);
        cv::medianBlur(ocv_tmp1, ocv_tmp3, 3);
        out_ocv[0] = ocv_tmp2 + ocv_scl;
        cv::boxFilter(ocv_tmp3, out_ocv[1], -1, cv::Size(3,3));
    }

    EXPECT_EQ(0, cvtest::norm(out_gapi[0], out_ocv[0], NORM_INF));
    EXPECT_EQ(0, cvtest::norm(out_gapi[1], out_ocv[1], NORM_INF));

    // FIXME: check that GIslandModel has more than 1 island (e.g. fusion
    // with breakdown worked)
}

TEST_F(GExecutorReshapeTest, ReshapeInsteadOfRecompile)
{
    // NB: Initial state
    EXPECT_EQ(0, backend_impl1->getCompileCounter());
    EXPECT_EQ(0, backend_impl2->getCompileCounter());
    EXPECT_EQ(0, island1.getReshapeCounter());
    EXPECT_EQ(0, island2.getReshapeCounter());

    // NB: First compilation.
    comp.apply(cv::gin(in_mat1), cv::gout(out_mat), cv::compile_args(pkg));
    EXPECT_EQ(1, backend_impl1->getCompileCounter());
    EXPECT_EQ(1, backend_impl2->getCompileCounter());
    EXPECT_EQ(0, island1.getReshapeCounter());
    EXPECT_EQ(0, island2.getReshapeCounter());

    // NB: GMockBackendImpl implements "reshape" method,
    // so it won't be recompiled if the meta is changed.
    comp.apply(cv::gin(in_mat2), cv::gout(out_mat), cv::compile_args(pkg));
    EXPECT_EQ(1, backend_impl1->getCompileCounter());
    EXPECT_EQ(1, backend_impl2->getCompileCounter());
    EXPECT_EQ(1, island1.getReshapeCounter());
    EXPECT_EQ(1, island2.getReshapeCounter());
}

TEST_F(GExecutorReshapeTest, OneBackendNotReshapable)
{
    // NB: Make first island not reshapable
    island1.setReshape(false);

    // NB: Initial state
    EXPECT_EQ(0, backend_impl1->getCompileCounter());
    EXPECT_EQ(0, island1.getReshapeCounter());
    EXPECT_EQ(0, backend_impl2->getCompileCounter());
    EXPECT_EQ(0, island2.getReshapeCounter());

    // NB: First compilation.
    comp.apply(cv::gin(in_mat1), cv::gout(out_mat), cv::compile_args(pkg));
    EXPECT_EQ(1, backend_impl1->getCompileCounter());
    EXPECT_EQ(1, backend_impl2->getCompileCounter());
    EXPECT_EQ(0, island1.getReshapeCounter());
    EXPECT_EQ(0, island2.getReshapeCounter());

    // NB: Since one of islands isn't reshapable
    // the entire graph isn't reshapable as well.
    comp.apply(cv::gin(in_mat2), cv::gout(out_mat), cv::compile_args(pkg));
    EXPECT_EQ(2, backend_impl1->getCompileCounter());
    EXPECT_EQ(2, backend_impl2->getCompileCounter());
    EXPECT_EQ(0, island1.getReshapeCounter());
    EXPECT_EQ(0, island2.getReshapeCounter());
}

TEST_F(GExecutorReshapeTest, ReshapeCallAllocate)
{
    // NB: Initial state
    EXPECT_EQ(0, island1.getAllocateCounter());
    EXPECT_EQ(0, island1.getReshapeCounter());

    // NB: First compilation.
    comp.apply(cv::gin(in_mat1), cv::gout(out_mat), cv::compile_args(pkg));
    EXPECT_EQ(1, island1.getAllocateCounter());
    EXPECT_EQ(0, island1.getReshapeCounter());

    // NB: The entire graph is reshapable, so it won't be recompiled, but reshaped.
    // Check that reshape call "allocate" to reallocate buffers.
    comp.apply(cv::gin(in_mat2), cv::gout(out_mat), cv::compile_args(pkg));
    EXPECT_EQ(2, island1.getAllocateCounter());
    EXPECT_EQ(1, island1.getReshapeCounter());
}

TEST_F(GExecutorReshapeTest, CPUBackendIsReshapable)
{
    comp = cv::GComputation([](){
        cv::GMat in;
        cv::GMat foo = I::Foo::on(in);
        cv::GMat out = cv::gapi::bitwise_not(cv::gapi::bitwise_not(in));
        return cv::GComputation(cv::GIn(in), cv::GOut(foo, out));
    });
    // NB: Initial state
    EXPECT_EQ(0, island1.getReshapeCounter());

    // NB: First compilation.
    cv::Mat out_mat2;
    comp.apply(cv::gin(in_mat1), cv::gout(out_mat, out_mat2), cv::compile_args(pkg));
    EXPECT_EQ(0, island1.getReshapeCounter());

    // NB: The entire graph is reshapable, so it won't be recompiled, but reshaped.
    comp.apply(cv::gin(in_mat2), cv::gout(out_mat, out_mat2), cv::compile_args(pkg));
    EXPECT_EQ(1, island1.getReshapeCounter());
    EXPECT_EQ(0, cvtest::norm(out_mat2, in_mat2, NORM_INF));
}

// FIXME: Add explicit tests on GMat/GScalar/GArray<T> being connectors
// between executed islands

} // namespace opencv_test
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

- **GMockBackendImpl**: A class/struct defined in this file
- **GMockFunctor**: A class/struct defined in this file
- **GMockExecutable**: A class/struct defined in this file
- **GExecutorReshapeTest**: A class/struct defined in this file
- **Priv**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../gapi_mock_kernels.hpp`
- `../test_precomp.hpp`
- `opencv2/gapi/core.hpp`

**Python Imports:**
- `ComplexIslands`


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

