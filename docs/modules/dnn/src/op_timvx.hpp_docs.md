# Documentation for `modules/dnn/src/op_timvx.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/op_timvx.hpp`
- **File Name**: `op_timvx.hpp`
- **File Size**: 5,941 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/op_timvx.hpp](../../../modules/dnn/src/op_timvx.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2019-2021, Shenzhen Institute of Artificial Intelligence and
// Robotics for Society, all rights reserved.
// Third party copyrights are property of their respective owners.

#ifndef OPENCV_DNN_OP_TIMVX_HPP
#define OPENCV_DNN_OP_TIMVX_HPP

#include <opencv2/dnn/shape_utils.hpp>

// TimVX head file.
#ifdef HAVE_TIMVX
#include "tim/vx/context.h"
#include "tim/vx/graph.h"
#include "tim/vx/operation.h"
#include "tim/vx/ops.h"
#include "tim/vx/tensor.h"
#endif  // HAVE_TIMVX

namespace cv
{
namespace dnn
{
#ifdef HAVE_TIMVX

enum tvActivationType{
    tvActNotSupported = -1,
    tvActReLU,
    tvActReLU6,
    tvActTanH,
    tvActSwish,
    tvActMish,
    tvActSigmoid,
    tvActELU
};

// Data copied from/to Mat to/from Tensor. Change the shape of dst if
// needed to make it the same shape as src.
bool copyToTensor(Ptr<tim::vx::Tensor> &dst, const Mat &src);
bool copyToMat(const Mat &dst, Ptr<tim::vx::Tensor> &src);
tvActivationType getTimVXActType(String & actString);

// Convert Mat shape to TimVX TensorShape
tim::vx::ShapeType getShapeTypeFromMat(const Mat& mat, bool ifConst = false);

// if all value in weight
bool getQuantType(const std::vector<float>& scales, int numOutput = -1);

class TimVXInfo;
class TimVXGraph;
class TimVXBackendNode;
class TimVXBackendWrapper;

// Maintain the tvGraph and tvTensor List. For now, every tvGraph only have one output node, and each node
// in tvGraph has only one output too. It could be optimized in future.
// TODO: tvGraph supports multiple output node.
class TimVXGraph
{
public:
    TimVXGraph();
    ~TimVXGraph();
    std::shared_ptr<tim::vx::Operation> getOp(const int opIndex);

    // It will add tensorWrapper to wrapperList, and return index.
    // And add tensor Ptr to tensorList.
    int addWrapper(Ptr<TimVXBackendWrapper>& tensorWrapper);

    void forward();

    // Add new op to opList, and return the index.
    int addOp(const std::shared_ptr<tim::vx::Operation>& op);

    // If tensor existed in tensorList, return the tensorIndex, otherwise return -1.
    int getTensorIndex(const std::shared_ptr<tim::vx::Tensor>& tensor);

    Ptr<TimVXBackendWrapper> getWrapper(int wrapperIndex);

    std::shared_ptr<tim::vx::Graph> graph;
    bool isCompiled; // Every tvGraph can only be compiled once.

private:
    std::shared_ptr<tim::vx::Context> context;
    std::vector<int> inputWrappersIndex;
    std::vector<int> outputWrappersIndex;
    std::vector<Ptr<TimVXBackendWrapper> > wrapperList;
    std::vector<std::shared_ptr<tim::vx::Tensor> > tensorList;
    std::vector<std::shared_ptr<tim::vx::Operation> > opList;
};

class TimVXBackendNode : public BackendNode
{
public:
    TimVXBackendNode(const Ptr<TimVXGraph>& tvGraph);
    TimVXBackendNode(const Ptr<TimVXGraph>& tvGraph, const std::shared_ptr<tim::vx::Operation>& op);
    TimVXBackendNode(const Ptr<TimVXGraph>& tvGraph, std::shared_ptr<tim::vx::Operation>& op,
                    std::vector<int>& inputsIndex, std::vector<int>& outpusIndex);

    void setInputTensor();
    bool opBinding();

    // flag for marking OutputNode of tvGraph this node is the last node in this TimVX Graph.
    bool isLast;
    int opIndex;

    // index of tensor and wrapper.
    std::vector<int> inputIndexList;
    std::vector<int> outputIndexList;
    Ptr<TimVXGraph> tvGraph;
};

class TimVXBackendWrapper : public BackendWrapper
{
public:
    TimVXBackendWrapper();
    TimVXBackendWrapper(Mat& m);
    TimVXBackendWrapper(const Ptr<BackendWrapper>& baseBuffer, Mat& m);
    TimVXBackendWrapper(std::shared_ptr<tim::vx::Tensor>& tensor);

    // Create Output Tensor
    void createTensor(std::shared_ptr<tim::vx::Graph>& graph, tim::vx::TensorAttribute tensorAttribute);
    void createTensor(std::shared_ptr<tim::vx::Graph>& graph, tim::vx::TensorAttribute tensorAttribute,
                                                  Ptr<tim::vx::Quantization>& tvQuant);
    std::shared_ptr<tim::vx::Tensor> getTensor();
    Mat getMat();

    // The Output tensor in TimVX doesn't have HostMat, The shape can only be given.
    void setTensorShape(const tim::vx::ShapeType & matShape);
    int getTensorIndex();
    Ptr<tim::vx::Quantization> getTensorQuantization();
    tim::vx::TensorAttribute getTensorAttr();
    bool isTensor();

    // Data Copy, CPU <==> NPU
    virtual void copyToHost() CV_OVERRIDE;
    virtual void setHostDirty() CV_OVERRIDE;
    void setDeviceDirty();
    void copyToDevice();

private:
    tim::vx::DataType tensorType;
    bool deviceDirty;
    bool hostDirty;
    int tensorIndex;  // index of tensorList in specific TimVXGraph.
    bool isTensor_;
    Mat host;

    tim::vx::ShapeType tensorShape;
    std::shared_ptr<tim::vx::Tensor> tensor;
    tim::vx::TensorAttribute tensorAttr;
};

// Contain all created tvGraphList, used in every
class TimVXInfo{
public:
    TimVXInfo();
    ~TimVXInfo();

    // Find the right graph Index set as graphIndex, if cannot find, return empty ptr.
    Ptr<TimVXGraph> getGraph();
    bool findGraphIndex(const std::vector<Ptr<BackendWrapper> > &inputsWrapper, int& graphIndex);
    void setTmpGraphIndex(int graphIndex);
    bool isConflict(int layerId, int graphIndex);

    // create a TimVXGraph, add it to tvGraphList, and return the index in tvGraphList.
    int createGraph();

    // graphConflictIndex[layerIndex] saves conflict graph index, which should be excluded
    std::vector<std::vector<int> > graphConflictMap;

private:
    int getTmpGraphIndex();
    std::vector<Ptr<TimVXGraph> > tvGraphList;
    int graphIndex;

};

#endif

void forwardTimVX(std::vector<Ptr<BackendWrapper> > &outputs, const Ptr<BackendNode>& node);
bool haveTimVX();
} // namespace dnn
} // namespace cv

#endif // OPENCV_DNN_OP_TIMVX_HPP```

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

- **TimVXGraph**: A class/struct defined in this file
- **TimVXBackendWrapper**: A class/struct defined in this file
- **TimVXInfo**: A class/struct defined in this file
- **TimVXBackendNode**: A class/struct defined in this file

### Functions and Methods

- **HAVE_TIMVX()**: A function/method defined in this file
- **OPENCV_DNN_OP_TIMVX_HPP()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `tim/vx/tensor.h`
- `tim/vx/ops.h`
- `tim/vx/context.h`
- `opencv2/dnn/shape_utils.hpp`
- `tim/vx/operation.h`
- `tim/vx/graph.h`

**Python Imports:**
- `Tensor.`


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

