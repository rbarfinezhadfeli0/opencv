# Documentation for `modules/flann/include/opencv2/flann/nn_index.h`

## File Metadata

- **Full Path**: `modules/flann/include/opencv2/flann/nn_index.h`
- **File Name**: `nn_index.h`
- **File Size**: 6,089 bytes
- **File Type**: .h
- **Link to Source**: [modules/flann/include/opencv2/flann/nn_index.h](../../../../../modules/flann/include/opencv2/flann/nn_index.h)

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

#ifndef OPENCV_FLANN_NNINDEX_H
#define OPENCV_FLANN_NNINDEX_H

#include "matrix.h"
#include "result_set.h"
#include "params.h"

//! @cond IGNORED

namespace cvflann
{

/**
 * Nearest-neighbour index base class
 */
template <typename Distance>
class NNIndex
{
    typedef typename Distance::ElementType ElementType;
    typedef typename Distance::ResultType DistanceType;

public:

    virtual ~NNIndex() {}

    /**
     * \brief Builds the index
     */
    virtual void buildIndex() = 0;

    /**
     * \brief Perform k-nearest neighbor search
     * \param[in] queries The query points for which to find the nearest neighbors
     * \param[out] indices The indices of the nearest neighbors found
     * \param[out] dists Distances to the nearest neighbors found
     * \param[in] knn Number of nearest neighbors to return
     * \param[in] params Search parameters
     */
    virtual void knnSearch(const Matrix<ElementType>& queries, Matrix<int>& indices, Matrix<DistanceType>& dists, int knn, const SearchParams& params)
    {
        CV_Assert(queries.cols == veclen());
        CV_Assert(indices.rows >= queries.rows);
        CV_Assert(dists.rows >= queries.rows);
        CV_Assert(int(indices.cols) >= knn);
        CV_Assert(int(dists.cols) >= knn);

#if 0
        KNNResultSet<DistanceType> resultSet(knn);
        for (size_t i = 0; i < queries.rows; i++) {
            resultSet.init(indices[i], dists[i]);
            findNeighbors(resultSet, queries[i], params);
        }
#else
        KNNUniqueResultSet<DistanceType> resultSet(knn);
        for (size_t i = 0; i < queries.rows; i++) {
            resultSet.clear();
            findNeighbors(resultSet, queries[i], params);
            if (get_param(params,"sorted",true)) resultSet.sortAndCopy(indices[i], dists[i], knn);
            else resultSet.copy(indices[i], dists[i], knn);
        }
#endif
    }

    /**
     * \brief Perform radius search
     * \param[in] query The query point
     * \param[out] indices The indinces of the neighbors found within the given radius
     * \param[out] dists The distances to the nearest neighbors found
     * \param[in] radius The radius used for search
     * \param[in] params Search parameters
     * \returns Number of neighbors found
     */
    virtual int radiusSearch(const Matrix<ElementType>& query, Matrix<int>& indices, Matrix<DistanceType>& dists, float radius, const SearchParams& params)
    {
        if (query.rows != 1) {
            fprintf(stderr, "I can only search one feature at a time for range search\n");
            return -1;
        }
        CV_Assert(query.cols == veclen());
        CV_Assert(indices.cols == dists.cols);

        int n = 0;
        int* indices_ptr = NULL;
        DistanceType* dists_ptr = NULL;
        if (indices.cols > 0) {
            n = (int)indices.cols;
            indices_ptr = indices[0];
            dists_ptr = dists[0];
        }

        RadiusUniqueResultSet<DistanceType> resultSet((DistanceType)radius);
        resultSet.clear();
        findNeighbors(resultSet, query[0], params);
        if (n>0) {
            if (get_param(params,"sorted",true)) resultSet.sortAndCopy(indices_ptr, dists_ptr, n);
            else resultSet.copy(indices_ptr, dists_ptr, n);
        }

        return (int)resultSet.size();
    }

    /**
     * \brief Saves the index to a stream
     * \param stream The stream to save the index to
     */
    virtual void saveIndex(FILE* stream) = 0;

    /**
     * \brief Loads the index from a stream
     * \param stream The stream from which the index is loaded
     */
    virtual void loadIndex(FILE* stream) = 0;

    /**
     * \returns number of features in this index.
     */
    virtual size_t size() const = 0;

    /**
     * \returns The dimensionality of the features in this index.
     */
    virtual size_t veclen() const = 0;

    /**
     * \returns The amount of memory (in bytes) used by the index.
     */
    virtual int usedMemory() const = 0;

    /**
     * \returns The index type (kdtree, kmeans,...)
     */
    virtual flann_algorithm_t getType() const = 0;

    /**
     * \returns The index parameters
     */
    virtual IndexParams getParameters() const = 0;


    /**
     * \brief Method that searches for nearest-neighbours
     */
    virtual void findNeighbors(ResultSet<DistanceType>& result, const ElementType* vec, const SearchParams& searchParams) = 0;
};

}

//! @endcond

#endif //OPENCV_FLANN_NNINDEX_H
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

- **NNIndex**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_FLANN_NNINDEX_H()**: A function/method defined in this file
- **typename()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `params.h`
- `result_set.h`
- `matrix.h`

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

