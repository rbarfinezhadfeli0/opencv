# Documentation for `samples/cpp/tutorial_code/gapi/doc_snippets/api_ref_snippets.cpp`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/gapi/doc_snippets/api_ref_snippets.cpp`
- **File Name**: `api_ref_snippets.cpp`
- **File Size**: 7,677 bytes
- **File Type**: .cpp
- **Link to Source**: [samples/cpp/tutorial_code/gapi/doc_snippets/api_ref_snippets.cpp](../../../../../samples/cpp/tutorial_code/gapi/doc_snippets/api_ref_snippets.cpp)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/gapi/doc_snippets` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include <opencv2/videoio.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/gapi.hpp>
#include <opencv2/gapi/core.hpp>
#include <opencv2/gapi/imgproc.hpp>

#include <opencv2/gapi/s11n.hpp>
#include <opencv2/gapi/garg.hpp>
#include <opencv2/gapi/gcommon.hpp>

#include <opencv2/gapi/cpu/gcpukernel.hpp>

#include <opencv2/gapi/fluid/core.hpp>
#include <opencv2/gapi/fluid/imgproc.hpp>

static void gscalar_example()
{
    //! [gscalar_implicit]
    cv::GMat a;
    cv::GMat b = a + 1;
    //! [gscalar_implicit]
}

static void typed_example()
{
    const cv::Size sz(32, 32);
    cv::Mat
        in_mat1        (sz, CV_8UC1),
        in_mat2        (sz, CV_8UC1),
        out_mat_untyped(sz, CV_8UC1),
        out_mat_typed1 (sz, CV_8UC1),
        out_mat_typed2 (sz, CV_8UC1);
    cv::randu(in_mat1, cv::Scalar::all(0), cv::Scalar::all(255));
    cv::randu(in_mat2, cv::Scalar::all(0), cv::Scalar::all(255));

    //! [Untyped_Example]
    // Untyped G-API ///////////////////////////////////////////////////////////
    cv::GComputation cvtU([]()
    {
        cv::GMat in1, in2;
        cv::GMat out = cv::gapi::add(in1, in2);
        return cv::GComputation({in1, in2}, {out});
    });
    std::vector<cv::Mat> u_ins  = {in_mat1, in_mat2};
    std::vector<cv::Mat> u_outs = {out_mat_untyped};
    cvtU.apply(u_ins, u_outs);
    //! [Untyped_Example]

    //! [Typed_Example]
    // Typed G-API /////////////////////////////////////////////////////////////
    cv::GComputationT<cv::GMat (cv::GMat, cv::GMat)> cvtT([](cv::GMat m1, cv::GMat m2)
    {
        return m1+m2;
    });
    cvtT.apply(in_mat1, in_mat2, out_mat_typed1);

    auto cvtTC =  cvtT.compile(cv::descr_of(in_mat1), cv::descr_of(in_mat2));
    cvtTC(in_mat1, in_mat2, out_mat_typed2);
    //! [Typed_Example]
}

static void bind_serialization_example()
{
    // ! [bind after deserialization]
    cv::GCompiled compd;
    std::vector<char> bytes;
    auto graph = cv::gapi::deserialize<cv::GComputation>(bytes);
    auto meta = cv::gapi::deserialize<cv::GMetaArgs>(bytes);

    compd = graph.compile(std::move(meta), cv::compile_args());
    auto in_args  = cv::gapi::deserialize<cv::GRunArgs>(bytes);
    auto out_args = cv::gapi::deserialize<cv::GRunArgs>(bytes);
    compd(std::move(in_args), cv::gapi::bind(out_args));
    // ! [bind after deserialization]
}

static void bind_deserialization_example()
{
    // ! [bind before serialization]
    std::vector<cv::GRunArgP> graph_outs;
    cv::GRunArgs out_args;

    for (auto &&out : graph_outs) {
        out_args.emplace_back(cv::gapi::bind(out));
    }
    const auto sargsout = cv::gapi::serialize(out_args);
    // ! [bind before serialization]
}

struct SimpleCustomType {
    bool val;
    bool operator==(const SimpleCustomType& other) const {
        return val == other.val;
    }
};

struct SimpleCustomType2 {
    int val;
    std::string name;
    std::vector<float> vec;
    std::map<int, uint64_t> mmap;
    bool operator==(const SimpleCustomType2& other) const {
        return val == other.val && name == other.name &&
               vec == other.vec && mmap == other.mmap;
    }
};

// ! [S11N usage]
namespace cv {
namespace gapi {
namespace s11n {
namespace detail {
template<> struct S11N<SimpleCustomType> {
    static void serialize(IOStream &os, const SimpleCustomType &p) {
        os << p.val;
    }
    static SimpleCustomType deserialize(IIStream &is) {
        SimpleCustomType p;
        is >> p.val;
        return p;
    }
};

template<> struct S11N<SimpleCustomType2> {
    static void serialize(IOStream &os, const SimpleCustomType2 &p) {
        os << p.val << p.name << p.vec << p.mmap;
    }
    static SimpleCustomType2 deserialize(IIStream &is) {
        SimpleCustomType2 p;
        is >> p.val >> p.name >> p.vec >> p.mmap;
        return p;
    }
};
} // namespace detail
} // namespace s11n
} // namespace gapi
} // namespace cv
// ! [S11N usage]

namespace cv {
namespace detail {
template<> struct CompileArgTag<SimpleCustomType> {
    static const char* tag() {
        return "org.opencv.test.simple_custom_type";
    }
};

template<> struct CompileArgTag<SimpleCustomType2> {
    static const char* tag() {
        return "org.opencv.test.simple_custom_type_2";
    }
};
} // namespace detail
} // namespace cv

static void s11n_example()
{
    SimpleCustomType  customVar1 { false };
    SimpleCustomType2 customVar2 { 1248, "World", {1280, 720, 640, 480},
                                   { {5, 32434142342}, {7, 34242432} } };

    std::vector<char> sArgs = cv::gapi::serialize(
        cv::compile_args(customVar1, customVar2));

    cv::GCompileArgs dArgs = cv::gapi::deserialize<cv::GCompileArgs,
                                                   SimpleCustomType,
                                                   SimpleCustomType2>(sArgs);

    SimpleCustomType  dCustomVar1 = cv::gapi::getCompileArg<SimpleCustomType>(dArgs).value();
    SimpleCustomType2 dCustomVar2 = cv::gapi::getCompileArg<SimpleCustomType2>(dArgs).value();

    (void) dCustomVar1;
    (void) dCustomVar2;
}

G_TYPED_KERNEL(IAdd, <cv::GMat(cv::GMat)>, "test.custom.add") {
    static cv::GMatDesc outMeta(const cv::GMatDesc &in) { return in; }
};
G_TYPED_KERNEL(IFilter2D, <cv::GMat(cv::GMat)>, "test.custom.filter2d") {
    static cv::GMatDesc outMeta(const cv::GMatDesc &in) { return in; }
};
G_TYPED_KERNEL(IRGB2YUV, <cv::GMat(cv::GMat)>, "test.custom.add") {
    static cv::GMatDesc outMeta(const cv::GMatDesc &in) { return in; }
};
GAPI_OCV_KERNEL(CustomAdd,      IAdd)      { static void run(cv::Mat, cv::Mat &) {} };
GAPI_OCV_KERNEL(CustomFilter2D, IFilter2D) { static void run(cv::Mat, cv::Mat &) {} };
GAPI_OCV_KERNEL(CustomRGB2YUV,  IRGB2YUV)  { static void run(cv::Mat, cv::Mat &) {} };

int main(int argc, char *argv[])
{
    if (argc < 3)
        return -1;

    cv::Mat input = cv::imread(argv[1]);
    cv::Mat output;

    {
    //! [graph_def]
    cv::GMat in;
    cv::GMat gx = cv::gapi::Sobel(in, CV_32F, 1, 0);
    cv::GMat gy = cv::gapi::Sobel(in, CV_32F, 0, 1);
    cv::GMat g  = cv::gapi::sqrt(cv::gapi::mul(gx, gx) + cv::gapi::mul(gy, gy));
    cv::GMat out = cv::gapi::convertTo(g, CV_8U);
    //! [graph_def]

    //! [graph_decl_apply]
    //! [graph_cap_full]
    cv::GComputation sobelEdge(cv::GIn(in), cv::GOut(out));
    //! [graph_cap_full]
    sobelEdge.apply(input, output);
    //! [graph_decl_apply]

    //! [apply_with_param]
    cv::GKernelPackage kernels = cv::gapi::combine
        (cv::gapi::core::fluid::kernels(),
         cv::gapi::imgproc::fluid::kernels());
    sobelEdge.apply(input, output, cv::compile_args(kernels));
    //! [apply_with_param]

    //! [graph_cap_sub]
    cv::GComputation sobelEdgeSub(cv::GIn(gx, gy), cv::GOut(out));
    //! [graph_cap_sub]
    }
    //! [graph_gen]
    cv::GComputation sobelEdgeGen([](){
            cv::GMat in;
            cv::GMat gx = cv::gapi::Sobel(in, CV_32F, 1, 0);
            cv::GMat gy = cv::gapi::Sobel(in, CV_32F, 0, 1);
            cv::GMat g  = cv::gapi::sqrt(cv::gapi::mul(gx, gx) + cv::gapi::mul(gy, gy));
            cv::GMat out = cv::gapi::convertTo(g, CV_8U);
            return cv::GComputation(in, out);
        });
    //! [graph_gen]

    cv::imwrite(argv[2], output);

    //! [kernels_snippet]
    cv::GKernelPackage pkg = cv::gapi::kernels
        < CustomAdd
        , CustomFilter2D
        , CustomRGB2YUV
        >();
    //! [kernels_snippet]

    // Just call typed example with no input/output - avoid warnings about
    // unused functions
    typed_example();
    gscalar_example();
    bind_serialization_example();
    bind_deserialization_example();
    s11n_example();
    return 0;
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

- **S11N**: A class/struct defined in this file
- **SimpleCustomType2**: A class/struct defined in this file
- **SimpleCustomType**: A class/struct defined in this file
- **CompileArgTag**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/imgproc.hpp`
- `opencv2/gapi/s11n.hpp`
- `opencv2/gapi/fluid/core.hpp`
- `opencv2/gapi/cpu/gcpukernel.hpp`
- `opencv2/gapi/garg.hpp`
- `opencv2/gapi.hpp`
- `opencv2/gapi/gcommon.hpp`
- `opencv2/videoio.hpp`
- `opencv2/gapi/fluid/imgproc.hpp`
- `opencv2/highgui.hpp`
- `opencv2/gapi/core.hpp`


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

