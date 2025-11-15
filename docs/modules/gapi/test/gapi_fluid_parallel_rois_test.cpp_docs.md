# Documentation for `modules/gapi/test/gapi_fluid_parallel_rois_test.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gapi_fluid_parallel_rois_test.cpp`
- **File Name**: `gapi_fluid_parallel_rois_test.cpp`
- **File Size**: 11,988 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gapi_fluid_parallel_rois_test.cpp](../../../modules/gapi/test/gapi_fluid_parallel_rois_test.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2019 Intel Corporation


#include "test_precomp.hpp"

#include "gapi_fluid_test_kernels.hpp"

namespace opencv_test
{

namespace {
    cv::Mat randomMat(cv::Size img_sz, int type = CV_8UC1, cv::Scalar mean   = cv::Scalar(127.0f), cv::Scalar stddev = cv::Scalar(40.f)){
        cv::Mat mat(img_sz, type);
        cv::randn(mat, mean, stddev);
        return mat;
    }

    cv::GFluidParallelOutputRois asGFluidParallelOutputRois(const std::vector<cv::Rect>& rois){
        cv::GFluidParallelOutputRois parallel_rois;
        for (auto const& roi : rois) {
            parallel_rois.parallel_rois.emplace_back(GFluidOutputRois{{roi}});
        }
        return parallel_rois;
    }

    void adjust_empty_roi(cv::Rect& roi, cv::Size size){
        if (roi.empty()) roi = cv::Rect{{0,0}, size};
    }

    cv::GCompileArgs combine(cv::GCompileArgs&& lhs, cv::GCompileArgs const& rhs){
        lhs.insert(lhs.end(), rhs.begin(), rhs.end());
        return std::move(lhs);
    }
}
using namespace cv::gapi_test_kernels;

//As GTest can not simultaneously parameterize test with both types and values - lets use type-erasure and virtual interfaces
//to use different computation pipelines
struct ComputationPair {
    void run_with_gapi(const cv::Mat& in_mat, cv::GCompileArgs const& compile_args, cv::Mat& out_mat){
        run_with_gapi_impl(in_mat, combine(cv::compile_args(fluidTestPackage), compile_args), out_mat);
    }
    void run_with_gapi(const cv::Mat& in_mat, cv::GFluidParallelOutputRois const& parallel_rois, cv::Mat& out_mat){
        run_with_gapi_impl(in_mat, cv::compile_args(fluidTestPackage, parallel_rois), out_mat);
    }

    virtual void run_with_ocv (const cv::Mat& in_mat, const std::vector<cv::Rect>& rois,                 cv::Mat& out_mat) = 0;

    virtual std::string name() const { return {}; }

    virtual ~ComputationPair ()  = default;

    friend std::ostream& operator<<(std::ostream& o, ComputationPair const* cp){
        std::string custom_name = cp->name();
        return o << (custom_name.empty() ? typeid(cp).name() : custom_name );
    }

private:
    virtual void run_with_gapi_impl(const cv::Mat& in_mat, cv::GCompileArgs const& comp_args, cv::Mat& out_mat) = 0;
};

struct Blur3x3CP  : ComputationPair{
    static constexpr int borderType = BORDER_REPLICATE;
    static constexpr int kernelSize = 3;

    std::string name() const override { return "Blur3x3"; }
    void run_with_gapi_impl(const cv::Mat& in_mat, cv::GCompileArgs const& comp_args, cv::Mat& out_mat_gapi) override {
        cv::GMat in;
        cv::GMat out = TBlur3x3::on(in, borderType, {});
        cv::GComputation c(cv::GIn(in), cv::GOut(out));

        // Run G-API
        auto cc = c.compile(cv::descr_of(in_mat), comp_args);
        cc(cv::gin(in_mat), cv::gout(out_mat_gapi));
    }

    void run_with_ocv(const cv::Mat& in_mat, const std::vector<cv::Rect>& rois, cv::Mat& out_mat_ocv) override {
        cv::Point anchor = {-1, -1};
        // Check with OpenCV
        for (auto roi : rois) {
            adjust_empty_roi(roi, in_mat.size());
            cv::blur(in_mat(roi), out_mat_ocv(roi), {kernelSize, kernelSize}, anchor, borderType);
        }
    }
};

struct AddCCP : ComputationPair{
    std::string name() const override { return "AddC"; }
    void run_with_gapi_impl(const cv::Mat& in_mat, cv::GCompileArgs const& comp_args, cv::Mat& out_mat_gapi) override {
        cv::GMat in;
        cv::GMat out = TAddCSimple::on(in, 1);
        cv::GComputation c(cv::GIn(in), cv::GOut(out));

        // Run G-API
        auto cc = c.compile(cv::descr_of(in_mat), comp_args);
        cc(cv::gin(in_mat), cv::gout(out_mat_gapi));
    }

    void run_with_ocv(const cv::Mat& in_mat, const std::vector<cv::Rect>& rois, cv::Mat& out_mat_ocv) override {
        // Check with OpenCV
        for (auto roi : rois) {
            adjust_empty_roi(roi, in_mat.size());
            out_mat_ocv(roi) = in_mat(roi) + 1u;
        }
    }
};

template<BorderTypes _borderType>
struct SequenceOfBlursCP : ComputationPair{
    BorderTypes borderType = _borderType;

    std::string name() const override { return "SequenceOfBlurs, border type: " + std::to_string(static_cast<int>(borderType)); }
    void run_with_gapi_impl(const cv::Mat& in_mat, cv::GCompileArgs const& comp_args, cv::Mat& out_mat) override {
        cv::Scalar borderValue(0);

        GMat in;
        auto mid = TBlur3x3::on(in,  borderType, borderValue);
        auto out = TBlur5x5::on(mid, borderType, borderValue);

        GComputation c(GIn(in), GOut(out));
        auto cc = c.compile(descr_of(in_mat), comp_args);
        cc(cv::gin(in_mat), cv::gout(out_mat));
    }
    void run_with_ocv(const cv::Mat& in_mat, const std::vector<cv::Rect>& rois,                 cv::Mat& out_mat) override {
        cv::Mat mid_mat_ocv = Mat::zeros(in_mat.size(), in_mat.type());
        cv::Point anchor = {-1, -1};

        for (auto roi : rois) {
            adjust_empty_roi(roi, in_mat.size());
            cv::blur(in_mat, mid_mat_ocv, {3,3}, anchor, borderType);
            cv::blur(mid_mat_ocv(roi), out_mat(roi), {5,5}, anchor, borderType);
        }
    }
};

struct TiledComputation : public TestWithParam <std::tuple<ComputationPair*, cv::Size, std::vector<cv::Rect>, decltype(cv::GFluidParallelFor::parallel_for)>> {};
TEST_P(TiledComputation, Test)
{
    ComputationPair*        cp;
    cv::Size                img_sz;
    std::vector<cv::Rect>   rois ;
    decltype(cv::GFluidParallelFor::parallel_for)        pfor;
    auto                    mat_type  =  CV_8UC1;

    std::tie(cp, img_sz, rois, pfor) = GetParam();

    cv::Mat in_mat       =      randomMat(img_sz, mat_type);
    cv::Mat out_mat_gapi = cv::Mat::zeros(img_sz, mat_type);
    cv::Mat out_mat_ocv  = cv::Mat::zeros(img_sz, mat_type);

    auto comp_args = combine(cv::compile_args(asGFluidParallelOutputRois(rois)), pfor ? cv::compile_args(cv::GFluidParallelFor{pfor}) : cv::GCompileArgs{});
    cp->run_with_gapi(in_mat, comp_args, out_mat_gapi);
    cp->run_with_ocv (in_mat, rois,      out_mat_ocv);

    EXPECT_EQ(0, cvtest::norm(out_mat_gapi, out_mat_ocv, NORM_INF))
            << "in_mat : \n"      << in_mat << std::endl
            << "diff matrix :\n " << (out_mat_gapi != out_mat_ocv) << std::endl
            << "out_mat_gapi: \n" << out_mat_gapi << std::endl
            << "out_mat_ocv:  \n" << out_mat_ocv << std::endl;;
}


namespace {
    //this is ugly but other variants (like using shared_ptr) are IMHO even more ugly :)
    template<typename T, typename... Arg>
    T* addr_of_static(Arg... arg) {
        static T obj(std::forward<Arg>(arg)...);
        return &obj;
    }
}

auto single_arg_computations = [](){
    return Values(  addr_of_static<Blur3x3CP>(),
                    addr_of_static<AddCCP>(),
                    addr_of_static<SequenceOfBlursCP<BORDER_CONSTANT>>(),
                    addr_of_static<SequenceOfBlursCP<BORDER_REPLICATE>>(),
                    addr_of_static<SequenceOfBlursCP<BORDER_REFLECT_101>>()
            );

};

auto tilesets_8x10 = [](){
    return  Values(std::vector<cv::Rect>{cv::Rect{}},
                   std::vector<cv::Rect>{cv::Rect{0,0,8,5}, cv::Rect{0,5,8,5}},
                   std::vector<cv::Rect>{cv::Rect{0,1,8,3}, cv::Rect{0,4,8,3}},
                   std::vector<cv::Rect>{cv::Rect{0,2,8,3}, cv::Rect{0,5,8,2}},
                   std::vector<cv::Rect>{cv::Rect{0,3,8,4}, cv::Rect{0,9,8,1}});
};

auto tilesets_20x15 = [](){
    return   Values(std::vector<cv::Rect>{cv::Rect{}},
                    std::vector<cv::Rect>{cv::Rect{{0,0},cv::Size{20,7}},
                                          cv::Rect{{0,7},cv::Size{20,8}}});
};

auto tilesets_320x240 = [](){
    return  Values(std::vector<cv::Rect>{cv::Rect{{0,0},   cv::Size{320,120}},
                                         cv::Rect{{0,120}, cv::Size{320,120}}},

                   std::vector<cv::Rect>{cv::Rect{{0,0},   cv::Size{320,120}},
                                         cv::Rect{{0,120}, cv::Size{320,120}}},

                   std::vector<cv::Rect>{cv::Rect{{0,0},  cv::Size{320,60}},
                                         cv::Rect{{0,60}, cv::Size{320,60}},
                                         cv::Rect{{0,120},cv::Size{320,120}}});
};

namespace{
    auto no_custom_pfor = decltype(cv::GFluidParallelFor::parallel_for){};
}

INSTANTIATE_TEST_CASE_P(FluidTiledSerial8x10, TiledComputation,
                        Combine(
                            single_arg_computations(),
                            Values(cv::Size(8, 10)),
                            tilesets_8x10(),
                            Values(no_custom_pfor))
);

INSTANTIATE_TEST_CASE_P(FluidTiledSerial20x15, TiledComputation,
                        Combine(
                            single_arg_computations(),
                            Values(cv::Size(20, 15)),
                            tilesets_20x15(),
                            Values(no_custom_pfor))
);

INSTANTIATE_TEST_CASE_P(FluidTiledSerial320x240, TiledComputation,
                        Combine(
                            single_arg_computations(),
                            Values(cv::Size(320, 240)),
                            tilesets_320x240(),
                            Values(no_custom_pfor))
);

//FIXME: add multiple outputs tests

TEST(FluidTiledParallelFor, basic)
{
    cv::Size                img_sz{8,20};
    auto                    mat_type  =  CV_8UC1;

    cv::GMat in;
    cv::GMat out = TAddCSimple::on(in, 1);
    cv::GComputation c(cv::GIn(in), cv::GOut(out));

    cv::Mat in_mat       =      randomMat(img_sz, mat_type);
    cv::Mat out_mat_gapi = cv::Mat::zeros(img_sz, mat_type);

    auto  parallel_rois = asGFluidParallelOutputRois( std::vector<cv::Rect>{cv::Rect{0,0,8,5}, cv::Rect{0,5,8,5}});

    std::size_t items_count = 0;
    auto pfor = [&items_count](std::size_t count, std::function<void(std::size_t)> ){
        items_count = count;
    };

    // Run G-API
    auto cc = c.compile(cv::descr_of(in_mat), cv::compile_args(fluidTestPackage, parallel_rois, GFluidParallelFor{pfor}));
    cc(cv::gin(in_mat), cv::gout(out_mat_gapi));
    ASSERT_EQ(parallel_rois.parallel_rois.size(), items_count);
}

namespace {
    auto serial_for = [](std::size_t count, std::function<void(std::size_t)> f){
        for (std::size_t i  = 0; i < count; ++i){
            f(i);
        }
    };

    auto cv_parallel_for = [](std::size_t count, std::function<void(std::size_t)> f){
        cv::parallel_for_(cv::Range(0, static_cast<int>(count)), [f](const cv::Range& r){
            for (auto i = r.start; i < r.end; ++i){
                f(i);
            }        });
    };
}

INSTANTIATE_TEST_CASE_P(FluidTiledParallel8x10, TiledComputation,
                        Combine(
                            single_arg_computations(),
                            Values(cv::Size(8, 10)),
                            tilesets_8x10(),
                            Values(serial_for, cv_parallel_for))
);
} // namespace opencv_test

//define custom printer for "parallel_for" test parameter
namespace std {
    void PrintTo(decltype(cv::GFluidParallelFor::parallel_for) const& f, std::ostream* o);
}

//separate declaration and definition are needed to please the compiler
void std::PrintTo(decltype(cv::GFluidParallelFor::parallel_for) const& f, std::ostream* o){
    if (f) {
        using namespace opencv_test;
        if      (f.target<decltype(serial_for)>()){
                    *o <<"serial_for";
        }
        else if (f.target<decltype(cv_parallel_for)>()){
            *o <<"cv_parallel_for";
        }
        else {
            *o <<"parallel_for of type: " << f.target_type().name();
        }
    }
    else
    {
        *o << "default parallel_for";
    }

}
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

- **ComputationPair**: A class/struct defined in this file
- **TiledComputation**: A class/struct defined in this file
- **Blur3x3CP**: A class/struct defined in this file
- **AddCCP**: A class/struct defined in this file
- **SequenceOfBlursCP**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `gapi_fluid_test_kernels.hpp`
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

