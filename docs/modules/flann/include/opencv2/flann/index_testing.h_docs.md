# Documentation for `modules/flann/include/opencv2/flann/index_testing.h`

## File Metadata

- **Full Path**: `modules/flann/include/opencv2/flann/index_testing.h`
- **File Name**: `index_testing.h`
- **File Size**: 10,800 bytes
- **File Type**: .h
- **Link to Source**: [modules/flann/include/opencv2/flann/index_testing.h](../../../../../modules/flann/include/opencv2/flann/index_testing.h)

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

#ifndef OPENCV_FLANN_INDEX_TESTING_H_
#define OPENCV_FLANN_INDEX_TESTING_H_

//! @cond IGNORED

#include <cstring>
#include <cmath>

#include "matrix.h"
#include "nn_index.h"
#include "result_set.h"
#include "logger.h"
#include "timer.h"


namespace cvflann
{

inline int countCorrectMatches(int* neighbors, int* groundTruth, int n)
{
    int count = 0;
    for (int i=0; i<n; ++i) {
        for (int k=0; k<n; ++k) {
            if (neighbors[i]==groundTruth[k]) {
                count++;
                break;
            }
        }
    }
    return count;
}


template <typename Distance>
typename Distance::ResultType computeDistanceRaport(const Matrix<typename Distance::ElementType>& inputData, typename Distance::ElementType* target,
                                                    int* neighbors, int* groundTruth, int veclen, int n, const Distance& distance)
{
    typedef typename Distance::ResultType DistanceType;

    DistanceType ret = 0;
    for (int i=0; i<n; ++i) {
        DistanceType den = distance(inputData[groundTruth[i]], target, veclen);
        DistanceType num = distance(inputData[neighbors[i]], target, veclen);

        if ((den==0)&&(num==0)) {
            ret += 1;
        }
        else {
            ret += num/den;
        }
    }

    return ret;
}

template <typename Distance>
float search_with_ground_truth(NNIndex<Distance>& index, const Matrix<typename Distance::ElementType>& inputData,
                               const Matrix<typename Distance::ElementType>& testData, const Matrix<int>& matches, int nn, int checks,
                               float& time, typename Distance::ResultType& dist, const Distance& distance, int skipMatches)
{
    typedef typename Distance::ResultType DistanceType;

    if (matches.cols<size_t(nn)) {
        Logger::info("matches.cols=%d, nn=%d\n",matches.cols,nn);

        FLANN_THROW(cv::Error::StsError, "Ground truth is not computed for as many neighbors as requested");
    }

    KNNResultSet<DistanceType> resultSet(nn+skipMatches);
    SearchParams searchParams(checks);

    std::vector<int> indices(nn+skipMatches);
    std::vector<DistanceType> dists(nn+skipMatches);
    int* neighbors = &indices[skipMatches];

    int correct = 0;
    DistanceType distR = 0;
    StartStopTimer t;
    int repeats = 0;
    while (t.value<0.2) {
        repeats++;
        t.start();
        correct = 0;
        distR = 0;
        for (size_t i = 0; i < testData.rows; i++) {
            resultSet.init(&indices[0], &dists[0]);
            index.findNeighbors(resultSet, testData[i], searchParams);

            correct += countCorrectMatches(neighbors,matches[i], nn);
            distR += computeDistanceRaport<Distance>(inputData, testData[i], neighbors, matches[i], (int)testData.cols, nn, distance);
        }
        t.stop();
    }
    time = float(t.value/repeats);

    float precicion = (float)correct/(nn*testData.rows);

    dist = distR/(testData.rows*nn);

    Logger::info("%8d %10.4g %10.5g %10.5g %10.5g\n",
                 checks, precicion, time, 1000.0 * time / testData.rows, dist);

    return precicion;
}


template <typename Distance>
float test_index_checks(NNIndex<Distance>& index, const Matrix<typename Distance::ElementType>& inputData,
                        const Matrix<typename Distance::ElementType>& testData, const Matrix<int>& matches,
                        int checks, float& precision, const Distance& distance, int nn = 1, int skipMatches = 0)
{
    typedef typename Distance::ResultType DistanceType;

    Logger::info("  Nodes  Precision(%)   Time(s)   Time/vec(ms)  Mean dist\n");
    Logger::info("---------------------------------------------------------\n");

    float time = 0;
    DistanceType dist = 0;
    precision = search_with_ground_truth(index, inputData, testData, matches, nn, checks, time, dist, distance, skipMatches);

    return time;
}

template <typename Distance>
float test_index_precision(NNIndex<Distance>& index, const Matrix<typename Distance::ElementType>& inputData,
                           const Matrix<typename Distance::ElementType>& testData, const Matrix<int>& matches,
                           float precision, int& checks, const Distance& distance, int nn = 1, int skipMatches = 0)
{
    typedef typename Distance::ResultType DistanceType;
    const float SEARCH_EPS = 0.001f;

    Logger::info("  Nodes  Precision(%)   Time(s)   Time/vec(ms)  Mean dist\n");
    Logger::info("---------------------------------------------------------\n");

    int c2 = 1;
    float p2;
    int c1 = 1;
    //float p1;
    float time;
    DistanceType dist;

    p2 = search_with_ground_truth(index, inputData, testData, matches, nn, c2, time, dist, distance, skipMatches);

    if (p2>precision) {
        Logger::info("Got as close as I can\n");
        checks = c2;
        return time;
    }

    while (p2<precision) {
        c1 = c2;
        //p1 = p2;
        c2 *=2;
        p2 = search_with_ground_truth(index, inputData, testData, matches, nn, c2, time, dist, distance, skipMatches);
    }

    int cx;
    float realPrecision;
    if (fabs(p2-precision)>SEARCH_EPS) {
        Logger::info("Start linear estimation\n");
        // after we got to values in the vecinity of the desired precision
        // use linear approximation get a better estimation

        cx = (c1+c2)/2;
        realPrecision = search_with_ground_truth(index, inputData, testData, matches, nn, cx, time, dist, distance, skipMatches);
        while (fabs(realPrecision-precision)>SEARCH_EPS) {

            if (realPrecision<precision) {
                c1 = cx;
            }
            else {
                c2 = cx;
            }
            cx = (c1+c2)/2;
            if (cx==c1) {
                Logger::info("Got as close as I can\n");
                break;
            }
            realPrecision = search_with_ground_truth(index, inputData, testData, matches, nn, cx, time, dist, distance, skipMatches);
        }

        c2 = cx;
        p2 = realPrecision;

    }
    else {
        Logger::info("No need for linear estimation\n");
        cx = c2;
        realPrecision = p2;
    }

    checks = cx;
    return time;
}


template <typename Distance>
void test_index_precisions(NNIndex<Distance>& index, const Matrix<typename Distance::ElementType>& inputData,
                           const Matrix<typename Distance::ElementType>& testData, const Matrix<int>& matches,
                           float* precisions, int precisions_length, const Distance& distance, int nn = 1, int skipMatches = 0, float maxTime = 0)
{
    typedef typename Distance::ResultType DistanceType;

    const float SEARCH_EPS = 0.001;

    // make sure precisions array is sorted
    std::sort(precisions, precisions+precisions_length);

    int pindex = 0;
    float precision = precisions[pindex];

    Logger::info("  Nodes  Precision(%)   Time(s)   Time/vec(ms)  Mean dist\n");
    Logger::info("---------------------------------------------------------\n");

    int c2 = 1;
    float p2;

    int c1 = 1;

    float time;
    DistanceType dist;

    p2 = search_with_ground_truth(index, inputData, testData, matches, nn, c2, time, dist, distance, skipMatches);

    // if precision for 1 run down the tree is already
    // better then some of the requested precisions, then
    // skip those
    while (precisions[pindex]<p2 && pindex<precisions_length) {
        pindex++;
    }

    if (pindex==precisions_length) {
        Logger::info("Got as close as I can\n");
        return;
    }

    for (int i=pindex; i<precisions_length; ++i) {

        precision = precisions[i];
        while (p2<precision) {
            c1 = c2;
            c2 *=2;
            p2 = search_with_ground_truth(index, inputData, testData, matches, nn, c2, time, dist, distance, skipMatches);
            if ((maxTime> 0)&&(time > maxTime)&&(p2<precision)) return;
        }

        int cx;
        float realPrecision;
        if (fabs(p2-precision)>SEARCH_EPS) {
            Logger::info("Start linear estimation\n");
            // after we got to values in the vecinity of the desired precision
            // use linear approximation get a better estimation

            cx = (c1+c2)/2;
            realPrecision = search_with_ground_truth(index, inputData, testData, matches, nn, cx, time, dist, distance, skipMatches);
            while (fabs(realPrecision-precision)>SEARCH_EPS) {

                if (realPrecision<precision) {
                    c1 = cx;
                }
                else {
                    c2 = cx;
                }
                cx = (c1+c2)/2;
                if (cx==c1) {
                    Logger::info("Got as close as I can\n");
                    break;
                }
                realPrecision = search_with_ground_truth(index, inputData, testData, matches, nn, cx, time, dist, distance, skipMatches);
            }

            c2 = cx;
            p2 = realPrecision;

        }
        else {
            Logger::info("No need for linear estimation\n");
            cx = c2;
            realPrecision = p2;
        }

    }
}

}

//! @endcond

#endif //OPENCV_FLANN_INDEX_TESTING_H_
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

### Functions and Methods

- **typename()**: A function/method defined in this file
- **OPENCV_FLANN_INDEX_TESTING_H_()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cmath`
- `logger.h`
- `cstring`
- `nn_index.h`
- `timer.h`
- `result_set.h`
- `matrix.h`


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

