# Documentation for `modules/dnn/src/op_webnn.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/op_webnn.hpp`
- **File Name**: `op_webnn.hpp`
- **File Size**: 4,468 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/op_webnn.hpp](../../../modules/dnn/src/op_webnn.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef __OPENCV_DNN_OP_WEBNN_HPP__
#define __OPENCV_DNN_OP_WEBNN_HPP__

#include "opencv2/core/cvdef.h"
#include "opencv2/core/cvstd.hpp"
#include "opencv2/dnn.hpp"

#ifdef HAVE_WEBNN

#include <webnn/webnn_cpp.h>
#include <webnn/webnn.h>
#ifdef __EMSCRIPTEN__
#include <emscripten.h>
#include <emscripten/html5.h>
#include <emscripten/html5_webnn.h>
#else
#include <webnn/webnn_proc.h>
#include <webnn_native/WebnnNative.h>
#endif

#include <unordered_map>
#include <unordered_set>

#endif  // HAVE_WEBNN

namespace cv { namespace dnn {

constexpr bool haveWebnn() {
#ifdef HAVE_WEBNN
        return true;
#else
        return false;
#endif
}

#ifdef HAVE_WEBNN

class WebnnBackendNode;
class WebnnBackendWrapper;

namespace webnn {
inline std::vector<int32_t> getShape(const Mat& mat)
{
    std::vector<int32_t> result(mat.dims);
    for (int i = 0; i < mat.dims; i++)
        result[i] = (int32_t)mat.size[i];
    return result;
}

ml::Operand BuildConstant(const ml::GraphBuilder& builder,
                              const std::vector<int32_t>& dimensions,
                              const void* value,
                              size_t size,
                              ml::OperandType type);

struct Pool2dOptions {
    public:
        std::vector<int32_t> windowDimensions;
        std::vector<int32_t> padding;
        std::vector<int32_t> strides;
        std::vector<int32_t> dilations;
        ml::AutoPad autoPad = ml::AutoPad::Explicit;
        ml::InputOperandLayout layout = ml::InputOperandLayout::Nchw;

        const ml::Pool2dOptions* AsPtr() {
            if (!windowDimensions.empty()) {
                mOptions.windowDimensionsCount = windowDimensions.size();
                mOptions.windowDimensions = windowDimensions.data();
            }
            if (!padding.empty()) {
                mOptions.paddingCount = padding.size();
                mOptions.padding = padding.data();
            }
            if (!strides.empty()) {
                mOptions.stridesCount = strides.size();
                mOptions.strides = strides.data();
            }
            if (!dilations.empty()) {
                mOptions.dilationsCount = dilations.size();
                mOptions.dilations = dilations.data();
            }
            mOptions.layout = layout;
            mOptions.autoPad = autoPad;
            return &mOptions;
        }

    private:
        ml::Pool2dOptions mOptions;
    };
}

class WebnnNet
{
public:
    WebnnNet();

    void addOutput(const std::string& name);

    bool isInitialized();
    void init(Target targetId);

    void forward(const std::vector<Ptr<BackendWrapper> >& outBlobsWrappers, bool isAsync);

    std::vector<ml::Operand> setInputs(const std::vector<cv::Mat>& inputs, const std::vector<std::string>& names);

    void setUnconnectedNodes(Ptr<WebnnBackendNode>& node);
    void addBlobs(const std::vector<cv::Ptr<BackendWrapper> >& ptrs);

    void createNet(Target targetId);
    // void setNodePtr(std::shared_ptr<ov::Node>* ptr);

    void reset();

    ml::GraphBuilder builder;
    ml::Context context;
    ml::Graph graph;

    std::unordered_map<std::string, cv::Ptr<WebnnBackendWrapper>> allBlobs;

    bool hasNetOwner;
    std::string device_name;
    bool isInit = false;

    std::vector<std::string> requestedOutputs;

    std::vector<std::string> inputNames;
    std::vector<std::string> outputNames;
    ml::NamedOperands namedOperands;
};

class WebnnBackendNode : public BackendNode
{
public:
    WebnnBackendNode(ml::Operand&& operand);
    WebnnBackendNode(ml::Operand& operand);

    std::string name;
    ml::Operand operand;
    Ptr<WebnnNet> net;
};

class WebnnBackendWrapper : public BackendWrapper
{
public:
    WebnnBackendWrapper(int targetId, Mat& m);
    ~WebnnBackendWrapper();

    virtual void copyToHost() CV_OVERRIDE;
    virtual void setHostDirty() CV_OVERRIDE;

    std::string name;
    Mat* host;
    std::unique_ptr<char> buffer;
    size_t size;
    std::vector<int32_t> dimensions;
    ml::OperandDescriptor descriptor;
};

#endif  // HAVE_WebNN

void forwardWebnn(const std::vector<Ptr<BackendWrapper> >& outBlobsWrappers,
                   Ptr<BackendNode>& node, bool isAsync);

}}  // namespace cv::dnn


#endif  // __OPENCV_DNN_OP_WEBNN_HPP__
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

- **Pool2dOptions**: A class/struct defined in this file
- **WebnnBackendWrapper**: A class/struct defined in this file
- **WebnnNet**: A class/struct defined in this file
- **WebnnBackendNode**: A class/struct defined in this file

### Functions and Methods

- **__EMSCRIPTEN__()**: A function/method defined in this file
- **HAVE_WEBNN()**: A function/method defined in this file
- **__OPENCV_DNN_OP_WEBNN_HPP__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `emscripten/html5_webnn.h`
- `webnn/webnn_cpp.h`
- `webnn_native/WebnnNative.h`
- `unordered_map`
- `webnn/webnn_proc.h`
- `emscripten.h`
- `webnn/webnn.h`
- `opencv2/core/cvstd.hpp`
- `unordered_set`
- `emscripten/html5.h`
- `opencv2/dnn.hpp`
- `opencv2/core/cvdef.h`


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

