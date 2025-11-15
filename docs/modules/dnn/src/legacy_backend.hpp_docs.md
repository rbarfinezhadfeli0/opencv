# Documentation for `modules/dnn/src/legacy_backend.hpp`

## File Metadata

- **Full Path**: `modules/dnn/src/legacy_backend.hpp`
- **File Name**: `legacy_backend.hpp`
- **File Size**: 10,744 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/dnn/src/legacy_backend.hpp](../../../modules/dnn/src/legacy_backend.hpp)

## Purpose and Role

This file is located in the `modules/dnn/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#ifndef __OPENCV_DNN_SRC_LEGACY_BACKEND_HPP__
#define __OPENCV_DNN_SRC_LEGACY_BACKEND_HPP__

#include "layer_internals.hpp"  // LayerPin LayerData DataLayer

namespace cv { namespace dnn {
CV__DNN_INLINE_NS_BEGIN
inline namespace detail {


#ifdef HAVE_OPENCL
class OpenCLBackendWrapper : public BackendWrapper
{
public:
    OpenCLBackendWrapper(Mat& m)
        : BackendWrapper(DNN_BACKEND_OPENCV, DNN_TARGET_OPENCL)
    {
        m.copyTo(umat);
        host = &m;
        hostDirty = false;
    }

    OpenCLBackendWrapper(const Ptr<BackendWrapper>& baseBuffer, Mat& m)
        : BackendWrapper(DNN_BACKEND_OPENCV, DNN_TARGET_OPENCL)
    {
        Ptr<OpenCLBackendWrapper> base = baseBuffer.dynamicCast<OpenCLBackendWrapper>();
        CV_Assert(!base.empty());

        host = &m;

        int shape[] = { 1, (int)base->umat.total() };
        umat = base->umat.reshape(1, 2, &shape[0])
                       .colRange(0, host->total())
                       .reshape(1, host->dims, &host->size[0]);
        hostDirty = false;
    }

    static Ptr<BackendWrapper> create(Mat& m)
    {
        return Ptr<BackendWrapper>(new OpenCLBackendWrapper(m));
    }

    static Ptr<BackendWrapper> create(const Ptr<BackendWrapper>& baseBuffer, Mat& m)
    {
        return Ptr<BackendWrapper>(new OpenCLBackendWrapper(baseBuffer, m));
    }

    static std::vector<UMat> getUMatVector(const std::vector<Ptr<BackendWrapper>>& wrappers)
    {
        const int numWrappers = wrappers.size();
        std::vector<UMat> mats(wrappers.size());
        for (int i = 0; i < numWrappers; ++i)
        {
            Ptr<OpenCLBackendWrapper> umatWrapper = wrappers[i].dynamicCast<OpenCLBackendWrapper>();
            CV_Assert(!umatWrapper.empty());
            umatWrapper->copyToDevice();
            mats[i] = umatWrapper->umat;
        }
        return mats;
    }

    // Replaces all umats in wrappers to specific ones.
    static void update(const std::vector<Ptr<BackendWrapper>>& wrappers,
            const std::vector<UMat>& umats)
    {
        CV_Assert(wrappers.size() == umats.size());
        for (int i = 0, n = umats.size(); i < n; ++i)
        {
            Ptr<OpenCLBackendWrapper> umatWrapper = wrappers[i].dynamicCast<OpenCLBackendWrapper>();
            CV_Assert(!umatWrapper.empty());
            umatWrapper->umat = umats[i];
        }
    }

    ~OpenCLBackendWrapper() {}

    // Copies data from device to a host memory.
    virtual void copyToHost() CV_OVERRIDE
    {
        umat.copyTo(*host);
    }

    virtual void setHostDirty() CV_OVERRIDE
    {
        hostDirty = true;
    };

    void copyToDevice()
    {
        if (hostDirty)
        {
            host->copyTo(umat);
            hostDirty = false;
        }
    }

private:
    UMat umat;
    Mat* host;
    bool hostDirty;
};  // OpenCLBackendWrapper
#endif  // HAVE_OPENCL


struct BlobManager
{
public:
    // Increase references counter to layer output.
    void addReference(const LayerPin& lp)
    {
        std::map<LayerPin, int>::iterator it = refCounter.find(lp);
        if (it == refCounter.end())
            refCounter[lp] = 1;
        else
            it->second += 1;
    }

    void addReferences(const std::vector<LayerPin>& pins)
    {
        for (int i = 0; i < pins.size(); i++)
        {
            addReference(pins[i]);
        }
    }

    // Returns number of references to allocated memory that used in specific
    // layer blob.
    int numReferences(const LayerPin& lp)
    {
        std::map<LayerPin, LayerPin>::const_iterator mapIt = reuseMap.find(lp);
        CV_Assert(mapIt != reuseMap.end());
        LayerPin memHost = mapIt->second;

        std::map<LayerPin, int>::const_iterator refIt = refCounter.find(memHost);
        CV_Assert(refIt != refCounter.end());
        return refIt->second;
    }

    // Reuse data allocated in <host> inside the <user> blob.
    void reuse(const LayerPin& host, const LayerPin& user)
    {
        CV_Assert(reuseMap.find(user) == reuseMap.end());
        CV_Assert(reuseMap.find(host) != reuseMap.end());
        LayerPin memHost = reuseMap[host];
        reuseMap[user] = memHost;
        if (refCounter.find(memHost) != refCounter.end())
        {
            std::map<LayerPin, int>::iterator userRefIt = refCounter.find(user);
            if (userRefIt != refCounter.end())
            {
                refCounter[memHost] += userRefIt->second;
                refCounter.erase(userRefIt);
            }
            else
                refCounter[memHost] += 1;
        }
    }

    // Decrease references counter to allocated memory inside specific blob.
    void releaseReference(const LayerPin& lp)
    {
        std::map<LayerPin, LayerPin>::const_iterator mapIt = reuseMap.find(lp);
        CV_Assert(mapIt != reuseMap.end());

        std::map<LayerPin, int>::iterator refIt = refCounter.find(mapIt->second);
        CV_Assert(refIt != refCounter.end());
        CV_Assert(refIt->second > 0);
        refIt->second -= 1;
    }

    void releaseReferences(const std::vector<LayerPin>& pins)
    {
        for (int i = 0; i < pins.size(); i++)
        {
            releaseReference(pins[i]);
        }
    }

    void reuseOrCreate(const MatShape& shape, const LayerPin& lp, Mat& dst, const int& dtype)
    {
        if (!getParam_DNN_DISABLE_MEMORY_OPTIMIZATIONS())
        {
            Mat bestBlob;
            LayerPin bestBlobPin;

            std::map<LayerPin, Mat>::const_iterator hostIt;
            std::map<LayerPin, int>::const_iterator refIt;

            const int targetTotal = total(shape);
            int bestBlobTotal = INT_MAX;

            for (hostIt = memHosts.begin(); hostIt != memHosts.end(); ++hostIt)
            {
                refIt = refCounter.find(hostIt->first);
                // Use only blobs that had references before because if not,
                // it might be used as output.
                if (refIt != refCounter.end() && refIt->second == 0)
                {
                    const Mat& unusedBlob = hostIt->second;
                    if (unusedBlob.total() >= targetTotal && unusedBlob.total() < bestBlobTotal && unusedBlob.type() == dtype)
                    {
                        bestBlobPin = hostIt->first;
                        bestBlob = unusedBlob;
                        bestBlobTotal = unusedBlob.total();
                    }
                }
            }
            if (!bestBlob.empty())
            {
                reuse(bestBlobPin, lp);
                dst = bestBlob.reshape(1, 1).colRange(0, targetTotal).reshape(1, shape);
                dst.dims = shape.size();
                return;
            }
        }

        {
            // if dst already has been allocated with total(shape) elements,
            // it won't be recreated and pointer of dst.data remains the same.
            dst.create(shape, dtype);
            addHost(lp, dst);
        }
    }

    void allocateBlobsForLayer(LayerData& ld, const LayerShapes& layerShapes,
            std::vector<LayerPin>& pinsForInternalBlobs)
    {
        CV_TRACE_FUNCTION();

        pinsForInternalBlobs.clear();

        std::vector<Mat>&outputBlobs = ld.outputBlobs,
        &internalBlobs = ld.internals;

        const ShapesVec &outShapes = layerShapes.out,
                        internalShapes = layerShapes.internal;

        outputBlobs.resize(std::max((size_t)1, outShapes.size()));  // layer produce at least one output blob
        internalBlobs.resize(internalShapes.size());

        CV_Assert(ld.requiredOutputs.size() <= outShapes.size());

        // Check that layer could work in-place.
        bool inPlace = false;
        if (layerShapes.supportInPlace)
        {
            if (ld.inputBlobs.size() == 1)
            {
                // Get number of references to the input memory.
                int numRef = numReferences(ld.inputBlobsId[0]);
                // If current layer is one and only customer of this blob.
                inPlace = numRef == 1;
            }
        }

        ShapesVec shapes(outShapes);
        shapes.insert(shapes.end(), internalShapes.begin(), internalShapes.end());
        std::vector<Mat*> blobs;
        for (int i = 0; i < outputBlobs.size(); i++)
        {
            blobs.push_back(&outputBlobs[i]);
        }

        for (int i = 0; i < internalBlobs.size(); i++)
        {
            blobs.push_back(&internalBlobs[i]);
            if (total(internalShapes[i]))
            {
                pinsForInternalBlobs.push_back(LayerPin(ld.id, ld.outputBlobs.size() + i));
            }
        }

        addReferences(pinsForInternalBlobs);

        std::map<int, std::vector<int>> idxSizes;
        for (int i = 0; i < shapes.size(); i++)
        {
            idxSizes[total(shapes[i])].push_back(i);
        }

        std::map<int, std::vector<int>>::reverse_iterator it;
        for (it = idxSizes.rbegin(); it != idxSizes.rend(); it++)
        {
            for (int j = 0; j < it->second.size(); j++)
            {
                int index = it->second[j];
                if (total(shapes[index]))
                {
                    LayerPin blobPin(ld.id, index);
                    if (index < outShapes.size() && inPlace)
                    {
                        CV_Assert(ld.inputBlobs[0]->total() == total(shapes[index]));
                        ld.outputBlobs[index] = ld.inputBlobs[0]->reshape(1, shapes[index]);
                        reuse(ld.inputBlobsId[0], blobPin);
                    }
                    else
                        reuseOrCreate(shapes[index], blobPin, *blobs[index], ld.dtype);
                }
            }
        }
    }

    // Clear internal state. Calls before an every reallocation.
    void reset()
    {
        CV_TRACE_FUNCTION();

        refCounter.clear();
        reuseMap.clear();
        memHosts.clear();
    }

private:
    // Register allocated memory.
    void addHost(const LayerPin& lp, const Mat& mat)
    {
        CV_Assert(memHosts.find(lp) == memHosts.end());
        reuseMap[lp] = lp;
        memHosts[lp] = mat;
    }

    std::map<LayerPin, int> refCounter;
    // Maps pin to origin blob (for whom memory was allocated firstly).
    // For origin blobs key == value.
    std::map<LayerPin, LayerPin> reuseMap;
    std::map<LayerPin, Mat> memHosts;
};  // BlobManager


Ptr<BackendWrapper> wrapMat(int backendId, int targetId, cv::Mat& m);


}  // namespace detail
CV__DNN_INLINE_NS_END
}}  // namespace cv::dnn
#endif  // __OPENCV_DNN_SRC_LEGACY_BACKEND_HPP__
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

- **OpenCLBackendWrapper**: A class/struct defined in this file
- **BlobManager**: A class/struct defined in this file

### Functions and Methods

- **HAVE_OPENCL()**: A function/method defined in this file
- **__OPENCV_DNN_SRC_LEGACY_BACKEND_HPP__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `layer_internals.hpp`

**Python Imports:**
- `device`


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

