# Documentation for `modules/flann/include/opencv2/flann/flann_base.hpp`

## File Metadata

- **Full Path**: `modules/flann/include/opencv2/flann/flann_base.hpp`
- **File Name**: `flann_base.hpp`
- **File Size**: 9,340 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/flann/include/opencv2/flann/flann_base.hpp](../../../../../modules/flann/include/opencv2/flann/flann_base.hpp)

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

#ifndef OPENCV_FLANN_BASE_HPP_
#define OPENCV_FLANN_BASE_HPP_

//! @cond IGNORED

#include <vector>
#include <cstdio>

#include "general.h"
#include "matrix.h"
#include "params.h"
#include "saving.h"

#include "all_indices.h"

namespace cvflann
{
class FILEScopeGuard {

public:
    explicit FILEScopeGuard(FILE* file) {
        file_ = file;
    };

    ~FILEScopeGuard() {
        fclose(file_);
    };

private:
    FILE* file_;
};


/**
 * Sets the log level used for all flann functions
 * @param level Verbosity level
 */
inline void log_verbosity(int level)
{
    if (level >= 0) {
        Logger::setLevel(level);
    }
}

/**
 * (Deprecated) Index parameters for creating a saved index.
 */
struct SavedIndexParams : public IndexParams
{
    SavedIndexParams(cv::String filename)
    {
        (* this)["algorithm"] = FLANN_INDEX_SAVED;
        (*this)["filename"] = filename;
    }
};

template<typename Distance>
NNIndex<Distance>* load_saved_index(const Matrix<typename Distance::ElementType>& dataset, const cv::String& filename, Distance distance)
{
    typedef typename Distance::ElementType ElementType;

    FILE* fin = fopen(filename.c_str(), "rb");
    if (fin == NULL) {
        return NULL;
    }
    FILEScopeGuard fscgd(fin);

    IndexHeader header = load_header(fin);
    if (header.data_type != Datatype<ElementType>::type()) {
        FLANN_THROW(cv::Error::StsError, "Datatype of saved index is different than of the one to be created.");
    }
    if ((size_t(header.rows) != dataset.rows)||(size_t(header.cols) != dataset.cols)) {
        FLANN_THROW(cv::Error::StsError, "The index saved belongs to a different dataset");
    }

    IndexParams params;
    params["algorithm"] = header.index_type;
    NNIndex<Distance>* nnIndex = create_index_by_type<Distance>(dataset, params, distance);
    nnIndex->loadIndex(fin);

    return nnIndex;
}


template<typename Distance>
class Index : public NNIndex<Distance>
{
public:
    typedef typename Distance::ElementType ElementType;
    typedef typename Distance::ResultType DistanceType;

    Index(const Matrix<ElementType>& features, const IndexParams& params, Distance distance = Distance() )
        :index_params_(params)
    {
        flann_algorithm_t index_type = get_param<flann_algorithm_t>(params,"algorithm");
        loaded_ = false;

        if (index_type == FLANN_INDEX_SAVED) {
            nnIndex_ = load_saved_index<Distance>(features, get_param<cv::String>(params,"filename"), distance);
            loaded_ = true;
        }
        else {
            nnIndex_ = create_index_by_type<Distance>(features, params, distance);
        }
    }

    ~Index()
    {
        delete nnIndex_;
    }

    /**
     * Builds the index.
     */
    void buildIndex() CV_OVERRIDE
    {
        if (!loaded_) {
            nnIndex_->buildIndex();
        }
    }

    void save(cv::String filename)
    {
        FILE* fout = fopen(filename.c_str(), "wb");
        if (fout == NULL) {
            FLANN_THROW(cv::Error::StsError, "Cannot open file");
        }
        save_header(fout, *nnIndex_);
        saveIndex(fout);
        fclose(fout);
    }

    /**
     * \brief Saves the index to a stream
     * \param stream The stream to save the index to
     */
    virtual void saveIndex(FILE* stream) CV_OVERRIDE
    {
        nnIndex_->saveIndex(stream);
    }

    /**
     * \brief Loads the index from a stream
     * \param stream The stream from which the index is loaded
     */
    virtual void loadIndex(FILE* stream) CV_OVERRIDE
    {
        nnIndex_->loadIndex(stream);
    }

    /**
     * \returns number of features in this index.
     */
    size_t veclen() const CV_OVERRIDE
    {
        return nnIndex_->veclen();
    }

    /**
     * \returns The dimensionality of the features in this index.
     */
    size_t size() const CV_OVERRIDE
    {
        return nnIndex_->size();
    }

    /**
     * \returns The index type (kdtree, kmeans,...)
     */
    flann_algorithm_t getType() const CV_OVERRIDE
    {
        return nnIndex_->getType();
    }

    /**
     * \returns The amount of memory (in bytes) used by the index.
     */
    virtual int usedMemory() const CV_OVERRIDE
    {
        return nnIndex_->usedMemory();
    }


    /**
     * \returns The index parameters
     */
    IndexParams getParameters() const CV_OVERRIDE
    {
        return nnIndex_->getParameters();
    }

    /**
     * \brief Perform k-nearest neighbor search
     * \param[in] queries The query points for which to find the nearest neighbors
     * \param[out] indices The indices of the nearest neighbors found
     * \param[out] dists Distances to the nearest neighbors found
     * \param[in] knn Number of nearest neighbors to return
     * \param[in] params Search parameters
     */
    void knnSearch(const Matrix<ElementType>& queries, Matrix<int>& indices, Matrix<DistanceType>& dists, int knn, const SearchParams& params) CV_OVERRIDE
    {
        nnIndex_->knnSearch(queries, indices, dists, knn, params);
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
    int radiusSearch(const Matrix<ElementType>& query, Matrix<int>& indices, Matrix<DistanceType>& dists, float radius, const SearchParams& params) CV_OVERRIDE
    {
        return nnIndex_->radiusSearch(query, indices, dists, radius, params);
    }

    /**
     * \brief Method that searches for nearest-neighbours
     */
    void findNeighbors(ResultSet<DistanceType>& result, const ElementType* vec, const SearchParams& searchParams) CV_OVERRIDE
    {
        nnIndex_->findNeighbors(result, vec, searchParams);
    }

    /**
     * \brief Returns actual index
     */
    CV_DEPRECATED NNIndex<Distance>* getIndex()
    {
        return nnIndex_;
    }

    /**
     * \brief Returns index parameters.
     * \deprecated use getParameters() instead.
     */
    CV_DEPRECATED  const IndexParams* getIndexParameters()
    {
        return &index_params_;
    }

private:
    /** Pointer to actual index class */
    NNIndex<Distance>* nnIndex_;
    /** Indices if the index was loaded from a file */
    bool loaded_;
    /** Parameters passed to the index */
    IndexParams index_params_;

    Index(const Index &); // copy disabled
    Index& operator=(const Index &); // assign disabled
};

/**
 * Performs a hierarchical clustering of the points passed as argument and then takes a cut in the
 * the clustering tree to return a flat clustering.
 * @param[in] points Points to be clustered
 * @param centers The computed cluster centres. Matrix should be preallocated and centers.rows is the
 *  number of clusters requested.
 * @param params Clustering parameters (The same as for cvflann::KMeansIndex)
 * @param d Distance to be used for clustering (eg: cvflann::L2)
 * @return number of clusters computed (can be different than clusters.rows and is the highest number
 * of the form (branching-1)*K+1 smaller than clusters.rows).
 */
template <typename Distance>
int hierarchicalClustering(const Matrix<typename Distance::ElementType>& points, Matrix<typename Distance::CentersType>& centers,
                           const KMeansIndexParams& params, Distance d = Distance())
{
    KMeansIndex<Distance> kmeans(points, params, d);
    kmeans.buildIndex();

    int clusterNum = kmeans.getClusterCenters(centers);
    return clusterNum;
}

}

//! @endcond

#endif /* OPENCV_FLANN_BASE_HPP_ */
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

- **Index**: A class/struct defined in this file
- **FILEScopeGuard**: A class/struct defined in this file
- **SavedIndexParams**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_FLANN_BASE_HPP_()**: A function/method defined in this file
- **typename()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `saving.h`
- `vector`
- `params.h`
- `general.h`
- `all_indices.h`
- `matrix.h`
- `cstdio`

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

