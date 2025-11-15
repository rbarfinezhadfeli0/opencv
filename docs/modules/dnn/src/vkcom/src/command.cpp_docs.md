# Documentation for `modules/dnn/src/vkcom/src/command.cpp`

## File Metadata

- **Full Path**: `modules/dnn/src/vkcom/src/command.cpp`
- **File Name**: `command.cpp`
- **File Size**: 6,009 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/dnn/src/vkcom/src/command.cpp](../../../../../modules/dnn/src/vkcom/src/command.cpp)

## Purpose and Role

This file is located in the `modules/dnn/src/vkcom/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

/*
The code has referenced MNN (https://github.com/alibaba/MNN/blob/2.4.0/source/backend/vulkan/component/VulkanCommandPool.cpp)
and adapted for OpenCV by Zihao Mu.
Below is the original copyright:
*/

//
//  VulkanCommandPool.cpp
//  MNN
//
//  Created by MNN on 2019/01/31.
//  Copyright © 2018, Alibaba Group Holding Limited
//

#include "../../precomp.hpp"
#include "internal.hpp"
#include "../include/command.hpp"

namespace cv { namespace dnn { namespace vkcom {

#ifdef HAVE_VULKAN

// *********************** CommandBuffer ********************
CommandBuffer::CommandBuffer(CommandPool* pool) : cmdPool(pool)
{
    CV_Assert(cmdPool);
    if (pool->bufferQueue.empty())
    {
        VkCommandBufferAllocateInfo cmdBufferCreateInfo {
                /* .sType              = */ VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO,
                /* .pNext              = */ nullptr,
                /* .commandPool        = */ cmdPool->get(),
                /* .level              = */ VK_COMMAND_BUFFER_LEVEL_PRIMARY,
                /* .commandBufferCount = */ 1,
        };
        vkAllocateCommandBuffers(kDevice, &cmdBufferCreateInfo, &cmdBuffer);
    }
    else
    {
        cmdBuffer = pool->bufferQueue.front();
        pool-> bufferQueue.pop();
    }
}

void CommandBuffer::barrierSource(VkBuffer source, size_t start, size_t size, BarrierType type) const
{
    VkBufferMemoryBarrier barrier;
    barrier.sType               = VK_STRUCTURE_TYPE_BUFFER_MEMORY_BARRIER;
    barrier.buffer              = source;
    barrier.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
    barrier.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
    barrier.offset              = start;
    barrier.pNext               = nullptr;
    barrier.size                = size;
    switch (type) {
        case READ_WRITE:
            barrier.srcAccessMask       = VK_ACCESS_SHADER_WRITE_BIT | VK_ACCESS_TRANSFER_WRITE_BIT;
            barrier.dstAccessMask       = VK_ACCESS_SHADER_READ_BIT | VK_ACCESS_TRANSFER_READ_BIT;
            break;
        case WRITE_WRITE:
            barrier.srcAccessMask       = VK_ACCESS_SHADER_WRITE_BIT | VK_ACCESS_TRANSFER_WRITE_BIT;
            barrier.dstAccessMask       = VK_ACCESS_SHADER_WRITE_BIT | VK_ACCESS_SHADER_WRITE_BIT;
            break;
        default:
            break;
    }
    vkCmdPipelineBarrier(cmdBuffer, VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT | VK_PIPELINE_STAGE_TRANSFER_BIT,
                         VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT | VK_PIPELINE_STAGE_TRANSFER_BIT, 0, 0, nullptr, 1,
                         &barrier, 0, nullptr);
}

void CommandBuffer::beginRecord(VkCommandBufferUsageFlags flag)
{
    cv::AutoLock lock(kContextMtx);
    VkCommandBufferBeginInfo cmdBufferBeginInfo{
            /* .sType            = */ VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO,
            /* .pNext            = */ nullptr,
            /* .flags            = */ flag,
            /* .pInheritanceInfo = */ nullptr,
    };
    vkResetCommandBuffer(cmdBuffer, 0);

    VK_CHECK_RESULT(vkBeginCommandBuffer(cmdBuffer, &cmdBufferBeginInfo));
}

void CommandBuffer::endRecord()
{
    VK_CHECK_RESULT(vkEndCommandBuffer(cmdBuffer));
}

CommandBuffer::~CommandBuffer()
{
    CV_Assert(cmdPool);
    if (needRelease)
    {
        vkFreeCommandBuffers(kDevice, cmdPool->get(), 1, &cmdBuffer);
    }
    else
    {
        cmdPool->bufferQueue.push(cmdBuffer);
    }
}

// *********************** CommandPool ********************
Ptr<CommandPool> CommandPool::create(const VkQueue &q, uint32_t _queueFamilyIndex)
{
    cv::AutoLock lock(kContextMtx);
    Ptr<CommandPool> cmdPoolInstance = Ptr<CommandPool>(new CommandPool(q, _queueFamilyIndex));

    return cmdPoolInstance;
}

CommandPool::CommandPool(const VkQueue& q, uint32_t _queueFamilyIndex) : queue(q), cmdPool(VK_NULL_HANDLE), queueFamilyIndex(_queueFamilyIndex)
{
    cv::AutoLock lock(kContextMtx);
    VkCommandPoolCreateInfo cmdPoolCreateInfo{
        /* .sType            = */ VK_STRUCTURE_TYPE_COMMAND_POOL_CREATE_INFO,
        /* .pNext            = */ nullptr,
        /* .flags            = */ VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT,
        /* .queueFamilyIndex = */ queueFamilyIndex,
    };
    vkCreateCommandPool(kDevice, &cmdPoolCreateInfo, nullptr, &cmdPool);
}

void CommandPool::reset()
{
    // reset all bufferQueue.
    while (!bufferQueue.empty())
    {
        auto cmdBuffer = bufferQueue.front();
        bufferQueue.pop();

        vkFreeCommandBuffers(kDevice, cmdPool, 1, &cmdBuffer);
    }
}

CommandPool::~CommandPool()
{
    while (!bufferQueue.empty())
    {
        auto cmdBuffer = bufferQueue.front();
        bufferQueue.pop();

        vkFreeCommandBuffers(kDevice, cmdPool, 1, &cmdBuffer);
    }
    vkDestroyCommandPool(kDevice, cmdPool, nullptr);
}

Ptr<CommandBuffer> CommandPool::allocBuffer()
{
    auto cmdBuffer = Ptr<CommandBuffer>(new CommandBuffer(this));
    cmdBuffer->needRelease = false;
    return cmdBuffer;
}

void CommandPool::submitAndWait(VkCommandBuffer& _buffer) const
{
    auto buffer = _buffer;
    Fence fence = Fence();
    VkFence fenceVk = fence.get();
    VkSubmitInfo submit_info = {
            /* .sType                = */ VK_STRUCTURE_TYPE_SUBMIT_INFO,
            /* .pNext                = */ nullptr,
            /* .waitSemaphoreCount   = */ 0,
            /* .pWaitSemaphores      = */ nullptr,
            /* .pWaitDstStageMask    = */ nullptr,
            /* .commandBufferCount   = */ 1,
            /* .pCommandBuffers      = */ &buffer,
            /* .signalSemaphoreCount = */ 0,
            /* .pSignalSemaphores    = */ nullptr};
    // need the queue class.
    VK_CHECK_RESULT(vkQueueSubmit(queue, 1, &submit_info, fenceVk));
    VK_CHECK_RESULT(fence.wait());
}

#endif // HAVE_VULKAN

}}} // namespace cv::dnn::vkcom```

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
- `internal.hpp`
- `../include/command.hpp`


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

