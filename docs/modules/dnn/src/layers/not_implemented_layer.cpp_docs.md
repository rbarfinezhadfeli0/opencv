# Documentation for `modules/dnn/src/layers/not_implemented_layer.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/layers/not_implemented_layer.cpp`
- **File Name**: `not_implemented_layer.cpp`
- **File Size**: 5,706 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/layers/not_implemented_layer.cpp](../../../../modules/dnn/src/layers/not_implemented_layer.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/layers` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "../precomp.hpp"
#include "../dnn_common.hpp"

namespace cv { namespace dnn {
CV__DNN_INLINE_NS_BEGIN

inline namespace detail {

class NotImplementedImpl CV_FINAL : public NotImplemented
{
public:
    NotImplementedImpl(const LayerParams& params)
    {
        setParamsFrom(params);
        CV_Assert(params.has("type"));
        std::stringstream ss;
        ss << "Node for layer '" << params.name << "' of type '" << params.get("type") << "' wasn't initialized.";
        msg = ss.str();
    }

    CV_DEPRECATED_EXTERNAL
    virtual void finalize(const std::vector<Mat*> &input, std::vector<Mat> &output) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual void finalize(InputArrayOfArrays inputs, OutputArrayOfArrays outputs) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    CV_DEPRECATED_EXTERNAL
    virtual void forward(std::vector<Mat*> &input, std::vector<Mat> &output, std::vector<Mat> &internals) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual void forward(InputArrayOfArrays inputs_arr, OutputArrayOfArrays outputs_arr, OutputArrayOfArrays internals_arr) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    void forward_fallback(InputArrayOfArrays inputs, OutputArrayOfArrays outputs, OutputArrayOfArrays internals)
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    CV_DEPRECATED_EXTERNAL
    void finalize(const std::vector<Mat> &inputs, CV_OUT std::vector<Mat> &outputs)
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    CV_DEPRECATED std::vector<Mat> finalize(const std::vector<Mat> &inputs)
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    CV_DEPRECATED void run(const std::vector<Mat> &inputs,
                           CV_OUT std::vector<Mat> &outputs,
                           CV_IN_OUT std::vector<Mat> &internals)
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual int inputNameToIndex(String inputName) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual int outputNameToIndex(const String& outputName) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual bool supportBackend(int backendId) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual Ptr<BackendNode> initHalide(const std::vector<Ptr<BackendWrapper> > &inputs) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual Ptr<BackendNode> initNgraph(const std::vector<Ptr<BackendWrapper> > &inputs,
                                        const std::vector<Ptr<BackendNode> >& nodes) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual Ptr<BackendNode> initVkCom(const std::vector<Ptr<BackendWrapper> > &inputs,
                                       std::vector<Ptr<BackendWrapper> > &outputs) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual Ptr<BackendNode> initCUDA(
            void *context,
            const std::vector<Ptr<BackendWrapper>>& inputs,
            const std::vector<Ptr<BackendWrapper>>& outputs
    ) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual void applyHalideScheduler(Ptr<BackendNode>& node,
                                      const std::vector<Mat*> &inputs,
                                      const std::vector<Mat> &outputs,
                                      int targetId) const CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual Ptr<BackendNode> tryAttach(const Ptr<BackendNode>& node) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual bool setActivation(const Ptr<ActivationLayer>& layer) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual bool tryFuse(Ptr<Layer>& top) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual void getScaleShift(Mat& scale, Mat& shift) const CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual void unsetAttached() CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual bool getMemoryShapes(const std::vector<MatShape> &inputs,
                                 const int requiredOutputs,
                                 std::vector<MatShape> &outputs,
                                 std::vector<MatShape> &internals) const CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual int64 getFLOPS(const std::vector<MatShape> &inputs,
                           const std::vector<MatShape> &outputs) const CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

    virtual bool updateMemoryShapes(const std::vector<MatShape> &inputs) CV_OVERRIDE
    {
        CV_Error(Error::StsNotImplemented, msg);
    }

private:
    std::string msg;
};

Ptr<Layer> NotImplemented::create(const LayerParams& params)
{
    return makePtr<NotImplementedImpl>(params);
}

Ptr<Layer> notImplementedRegisterer(LayerParams &params)
{
    return detail::NotImplemented::create(params);
}

void NotImplemented::Register()
{
    LayerFactory::registerLayer("NotImplemented", detail::notImplementedRegisterer);
}

void NotImplemented::unRegister()
{
    LayerFactory::unregisterLayer("NotImplemented");
}

} // namespace detail

CV__DNN_INLINE_NS_END
}}  // namespace cv::dnn
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

- **NotImplementedImpl**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../dnn_common.hpp`
- `../precomp.hpp`


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

