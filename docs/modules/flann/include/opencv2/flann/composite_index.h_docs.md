# Documentation for `modules/flann/include/opencv2/flann/composite_index.h`

## File Metadata

- **Full Path**: `modules/flann/include/opencv2/flann/composite_index.h`
- **File Name**: `composite_index.h`
- **File Size**: 6,094 bytes
- **File Type**: .h
- **Link to Source**: [modules/flann/include/opencv2/flann/composite_index.h](../../../../../modules/flann/include/opencv2/flann/composite_index.h)

## Purpose and Role

This file is located in the `modules/flann/include/opencv2/flann` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/***********************************************************************
 * Software License Agreement (BSD License)
 *
 * Copyright 2008-2009  Marius Muja (mariusm@cs.ubc.ca). All rights reserved.
 * Copyright 2008-2009  David G. Lowe (lowe@cs.ubc.ca). All rights reserved.
 *
 * THE BSD LICENSE
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 *
 * 1. Redistributions of source code must retain the above copyright
 *    notice, this list of conditions and the following disclaimer.
 * 2. Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 * THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS OR
 * IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES
 * OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED.
 * IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY DIRECT, INDIRECT,
 * INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT
 * NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
 * DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
 * THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
 * (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
 * THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *************************************************************************/

#ifndef OPENCV_FLANN_COMPOSITE_INDEX_H_
#define OPENCV_FLANN_COMPOSITE_INDEX_H_

//! @cond IGNORED

#include "nn_index.h"
#include "kdtree_index.h"
#include "kmeans_index.h"

namespace cvflann
{

/**
 * Index parameters for the CompositeIndex.
 */
struct CompositeIndexParams : public IndexParams
{
    CompositeIndexParams(int trees = 4, int branching = 32, int iterations = 11,
                         flann_centers_init_t centers_init = FLANN_CENTERS_RANDOM, float cb_index = 0.2f )
    {
        (*this)["algorithm"] = FLANN_INDEX_KMEANS;
        // number of randomized trees to use (for kdtree)
        (*this)["trees"] = trees;
        // branching factor
        (*this)["branching"] = branching;
        // max iterations to perform in one kmeans clustering (kmeans tree)
        (*this)["iterations"] = iterations;
        // algorithm used for picking the initial cluster centers for kmeans tree
        (*this)["centers_init"] = centers_init;
        // cluster boundary index. Used when searching the kmeans tree
        (*this)["cb_index"] = cb_index;
    }
};


/**
 * This index builds a kd-tree index and a k-means index and performs nearest
 * neighbour search both indexes. This gives a slight boost in search performance
 * as some of the neighbours that are missed by one index are found by the other.
 */
template <typename Distance>
class CompositeIndex : public NNIndex<Distance>
{
public:
    typedef typename Distance::ElementType ElementType;
    typedef typename Distance::ResultType DistanceType;

    /**
     * Index constructor
     * @param inputData dataset containing the points to index
     * @param params Index parameters
     * @param d Distance functor
     */
    CompositeIndex(const Matrix<ElementType>& inputData, const IndexParams& params = CompositeIndexParams(),
                   Distance d = Distance()) : index_params_(params)
    {
        kdtree_index_ = new KDTreeIndex<Distance>(inputData, params, d);
        kmeans_index_ = new KMeansIndex<Distance>(inputData, params, d);

    }

    CompositeIndex(const CompositeIndex&);
    CompositeIndex& operator=(const CompositeIndex&);

    virtual ~CompositeIndex()
    {
        delete kdtree_index_;
        delete kmeans_index_;
    }

    /**
     * @return The index type
     */
    flann_algorithm_t getType() const CV_OVERRIDE
    {
        return FLANN_INDEX_COMPOSITE;
    }

    /**
     * @return Size of the index
     */
    size_t size() const CV_OVERRIDE
    {
        return kdtree_index_->size();
    }

    /**
     * \returns The dimensionality of the features in this index.
     */
    size_t veclen() const CV_OVERRIDE
    {
        return kdtree_index_->veclen();
    }

    /**
     * \returns The amount of memory (in bytes) used by the index.
     */
    int usedMemory() const CV_OVERRIDE
    {
        return kmeans_index_->usedMemory() + kdtree_index_->usedMemory();
    }

    /**
     * \brief Builds the index
     */
    void buildIndex() CV_OVERRIDE
    {
        Logger::info("Building kmeans tree...\n");
        kmeans_index_->buildIndex();
        Logger::info("Building kdtree tree...\n");
        kdtree_index_->buildIndex();
    }

    /**
     * \brief Saves the index to a stream
     * \param stream The stream to save the index to
     */
    void saveIndex(FILE* stream) CV_OVERRIDE
    {
        kmeans_index_->saveIndex(stream);
        kdtree_index_->saveIndex(stream);
    }

    /**
     * \brief Loads the index from a stream
     * \param stream The stream from which the index is loaded
     */
    void loadIndex(FILE* stream) CV_OVERRIDE
    {
        kmeans_index_->loadIndex(stream);
        kdtree_index_->loadIndex(stream);
    }

    /**
     * \returns The index parameters
     */
    IndexParams getParameters() const CV_OVERRIDE
    {
        return index_params_;
    }

    /**
     * \brief Method that searches for nearest-neighbours
     */
    void findNeighbors(ResultSet<DistanceType>& result, const ElementType* vec, const SearchParams& searchParams) CV_OVERRIDE
    {
        kmeans_index_->findNeighbors(result, vec, searchParams);
        kdtree_index_->findNeighbors(result, vec, searchParams);
    }

private:
    /** The k-means index */
    KMeansIndex<Distance>* kmeans_index_;

    /** The kd-tree index */
    KDTreeIndex<Distance>* kdtree_index_;

    /** The index parameters */
    const IndexParams index_params_;
};

}

//! @endcond

#endif //OPENCV_FLANN_COMPOSITE_INDEX_H_
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

- **CompositeIndex**: A class/struct defined in this file
- **CompositeIndexParams**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_FLANN_COMPOSITE_INDEX_H_()**: A function/method defined in this file
- **typename()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `kmeans_index.h`
- `nn_index.h`
- `kdtree_index.h`

**Python Imports:**
- `a`
- `which`


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

