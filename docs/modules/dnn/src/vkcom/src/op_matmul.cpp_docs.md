# Documentation for `modules/dnn/src/vkcom/src/op_matmul.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/vkcom/src/op_matmul.cpp`
- **File Name**: `op_matmul.cpp`
- **File Size**: 3,820 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/vkcom/src/op_matmul.cpp](../../../../../modules/dnn/src/vkcom/src/op_matmul.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/vkcom/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "../../precomp.hpp"
#include "internal.hpp"
#include "../include/op_matmul.hpp"

namespace cv { namespace dnn { namespace vkcom {

#ifdef HAVE_VULKAN

#define KSTRIP_LEN 32
#define BLOCK_SIZE 64

OpMatMul::OpMatMul(std::vector<Mat>& matBlobs, const int _M, const int _K, const int _N) : M(_M), K(_K), N(_N)
{
    // Convert Weight to GPU Tensor.
    type_ = kOpTypeMatMul;
    CV_Assert(matBlobs.empty() || matBlobs.size() == 1);

    if (matBlobs.size() == 1)
    {
        Tensor weightTensor;
        CV_Assert(matBlobs[0].isContinuous() && matBlobs[0].type() == CV_32F);
        std::vector<int> matShape = shape(matBlobs[0]);
        weightTensor.reshape((const char*)matBlobs[0].data, matShape); // This code will copy the src data from Mat to VkBuffer.

        weightTensorPtr = makePtr<Tensor>(weightTensor);
    }
}

void OpMatMul::firstForward()
{
    if (!firstForwardFinsh)
    {
        config.local_size_x = BLOCK_SIZE;
        config.local_size_y = BLOCK_SIZE;
        config.local_size_z = 1;

        computeGroupCount();
        firstForwardFinsh = true;
    }
    else
        return;
}

bool OpMatMul::forward(std::vector<Tensor>& ins, std::vector<Tensor>& outs)
{
    CV_Assert((ins.size() == 1 || ins.size() == 2) && outs.size() == 1);
    Shape inputShape = ins[0].getShape();
    Shape outputShape = outs[0].getShape();
    CV_Assert(inputShape.size() == outputShape.size());

    CV_Assert(inputShape.size() == 2 || inputShape.size() == 4);

    if (inputShape.size() == 2)
    {
        batch = 0;
        Hi = inputShape[0];
        Wi = inputShape[1];

        H0 = outputShape[0];
        W0 = outputShape[1];
    }
    else if (inputShape.size() == 4)
    {
        batch = inputShape[kShapeIdxBatch];
        Hi = inputShape[kShapeIdxHeight];
        Wi = inputShape[kShapeIdxWidth];

        H0 = outputShape[kShapeIdxHeight];
        W0 = outputShape[kShapeIdxWidth];
    }

    firstForward();

    int KStrip = K/KSTRIP_LEN;
    int KStripRemain = K - KStrip * KSTRIP_LEN;
    std::vector<int> param = {M, K, N, KStrip, KStripRemain};

    std::vector<int> shape = {(int)param.size()};
    Tensor paramTensor = Tensor(reinterpret_cast<const char *>(param.data()), shape, kFormatInt32, VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT);

    std::string key = "gemm_spv";
    destTypes = {
            VK_DESCRIPTOR_TYPE_STORAGE_BUFFER, // input
            VK_DESCRIPTOR_TYPE_STORAGE_BUFFER, // weight
            VK_DESCRIPTOR_TYPE_STORAGE_BUFFER, // out
            VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER  // param
    };

    Ptr<Pipeline> pipeline = pipelineFactoryPtr->getPipeline(key, destTypes);
    Ptr<Descriptor> desSet = pipeline->createSet();
    Ptr<CommandBuffer> cmdBuffer = cmdPoolPtr->allocBuffer();

    VkCommandBuffer cmdBufferReal = cmdBuffer->get();
    desSet->writeTensor(ins[0], 0);

    if (weightTensorPtr)
        desSet->writeTensor(*weightTensorPtr, 1);
    else
    {
        CV_Assert(ins.size() == 2);
        desSet->writeTensor(ins[1], 1);
    }

    desSet->writeTensor(outs[0], 2);
    desSet->writeTensor(paramTensor, 3); // TODO change the parameter from pushconstance to buffer.

    cmdBuffer->beginRecord();
    pipeline->bind(cmdBufferReal, desSet->get());
    vkCmdDispatch(cmdBufferReal, group_x_, group_y_, group_z_);
    cmdBuffer->endRecord();

    cmdPoolPtr->submitAndWait(cmdBufferReal);

    return true;
}

bool OpMatMul::computeGroupCount()
{
    group_x_ = alignSize(M, BLOCK_SIZE) / BLOCK_SIZE;
    group_y_ = alignSize(N, BLOCK_SIZE) / BLOCK_SIZE;
    group_z_ = 1;

    return true;
}

#endif // HAVE_VULKAN

}}} // namespace cv::dnn::vkcom
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

- **HAVE_VULKAN()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../../precomp.hpp`
- `../include/op_matmul.hpp`
- `internal.hpp`

**Python Imports:**
- `Mat`
- `pushconstance`


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

