# Documentation for `modules/ml/test/test_svmsgd.cpp`

## File Metadata

- **Full Path**: `modules/ml/test/test_svmsgd.cpp`
- **File Name**: `test_svmsgd.cpp`
- **File Size**: 5,364 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/ml/test/test_svmsgd.cpp](../../../modules/ml/test/test_svmsgd.cpp)

## Purpose and Role

This file is located in the `modules/ml/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "test_precomp.hpp"

namespace opencv_test { namespace {

static const int TEST_VALUE_LIMIT = 500;
enum
{
    UNIFORM_SAME_SCALE,
    UNIFORM_DIFFERENT_SCALES
};

CV_ENUM(SVMSGD_TYPE, UNIFORM_SAME_SCALE, UNIFORM_DIFFERENT_SCALES)

typedef std::vector< std::pair<float,float> > BorderList;

static void makeData(RNG &rng, int samplesCount, const Mat &weights, float shift, const BorderList & borders, Mat &samples, Mat & responses)
{
    int featureCount = weights.cols;
    samples.create(samplesCount, featureCount, CV_32FC1);
    for (int featureIndex = 0; featureIndex < featureCount; featureIndex++)
        rng.fill(samples.col(featureIndex), RNG::UNIFORM, borders[featureIndex].first, borders[featureIndex].second);
    responses.create(samplesCount, 1, CV_32FC1);
    for (int i = 0 ; i < samplesCount; i++)
    {
        double res = samples.row(i).dot(weights) + shift;
        responses.at<float>(i) = res > 0 ? 1.f : -1.f;
    }
}

//==================================================================================================

typedef tuple<SVMSGD_TYPE, int, double> ML_SVMSGD_Param;
typedef testing::TestWithParam<ML_SVMSGD_Param> ML_SVMSGD_Params;

TEST_P(ML_SVMSGD_Params, scale_and_features)
{
    const int type = get<0>(GetParam());
    const int featureCount = get<1>(GetParam());
    const double precision = get<2>(GetParam());

    RNG &rng = cv::theRNG();

    Mat_<float> weights(1, featureCount);
    rng.fill(weights, RNG::UNIFORM, -1, 1);
    const float shift = static_cast<float>(rng.uniform(-featureCount, featureCount));

    BorderList borders;
    float lowerLimit = -TEST_VALUE_LIMIT;
    float upperLimit = TEST_VALUE_LIMIT;
    if (type == UNIFORM_SAME_SCALE)
    {
        for (int featureIndex = 0; featureIndex < featureCount; featureIndex++)
            borders.push_back(std::pair<float,float>(lowerLimit, upperLimit));
    }
    else if (type == UNIFORM_DIFFERENT_SCALES)
    {
        for (int featureIndex = 0; featureIndex < featureCount; featureIndex++)
        {
            int crit = rng.uniform(0, 2);
            if (crit > 0)
                borders.push_back(std::pair<float,float>(lowerLimit, upperLimit));
            else
                borders.push_back(std::pair<float,float>(lowerLimit/1000, upperLimit/1000));
        }
    }
    ASSERT_FALSE(borders.empty());

    Mat trainSamples;
    Mat trainResponses;
    int trainSamplesCount = 10000;
    makeData(rng, trainSamplesCount, weights, shift, borders, trainSamples, trainResponses);
    ASSERT_EQ(trainResponses.type(), CV_32FC1);

    Mat testSamples;
    Mat testResponses;
    int testSamplesCount = 100000;
    makeData(rng, testSamplesCount, weights, shift, borders, testSamples, testResponses);
    ASSERT_EQ(testResponses.type(), CV_32FC1);

    Ptr<TrainData> data = TrainData::create(trainSamples, cv::ml::ROW_SAMPLE, trainResponses);
    ASSERT_TRUE(data);

    cv::Ptr<SVMSGD> svmsgd = SVMSGD::create();
    ASSERT_TRUE(svmsgd);

    svmsgd->train(data);

    Mat responses;
    svmsgd->predict(testSamples, responses);
    ASSERT_EQ(responses.type(), CV_32FC1);
    ASSERT_EQ(responses.rows, testSamplesCount);

    int errCount = 0;
    for (int i = 0; i < testSamplesCount; i++)
        if (responses.at<float>(i) * testResponses.at<float>(i) < 0)
            errCount++;
    float err = (float)errCount / testSamplesCount;
    EXPECT_LE(err, precision);
}

ML_SVMSGD_Param params_list[] = {
    ML_SVMSGD_Param(UNIFORM_SAME_SCALE, 2, 0.01),
    ML_SVMSGD_Param(UNIFORM_SAME_SCALE, 5, 0.01),
    ML_SVMSGD_Param(UNIFORM_SAME_SCALE, 100, 0.02),
    ML_SVMSGD_Param(UNIFORM_DIFFERENT_SCALES, 2, 0.01),
    ML_SVMSGD_Param(UNIFORM_DIFFERENT_SCALES, 5, 0.01),
    ML_SVMSGD_Param(UNIFORM_DIFFERENT_SCALES, 100, 0.01),
};

INSTANTIATE_TEST_CASE_P(/**/, ML_SVMSGD_Params, testing::ValuesIn(params_list));

//==================================================================================================

TEST(ML_SVMSGD, twoPoints)
{
    Mat samples(2, 2, CV_32FC1);
    samples.at<float>(0,0) = 0;
    samples.at<float>(0,1) = 0;
    samples.at<float>(1,0) = 1000;
    samples.at<float>(1,1) = 1;

    Mat responses(2, 1, CV_32FC1);
    responses.at<float>(0) = -1;
    responses.at<float>(1) = 1;

    cv::Ptr<TrainData> trainData = TrainData::create(samples, cv::ml::ROW_SAMPLE, responses);

    Mat realWeights(1, 2, CV_32FC1);
    realWeights.at<float>(0) = 1000;
    realWeights.at<float>(1) = 1;

    float realShift = -500000.5;

    float normRealWeights = static_cast<float>(cv::norm(realWeights)); // TODO cvtest
    realWeights /= normRealWeights;
    realShift /= normRealWeights;

    cv::Ptr<SVMSGD> svmsgd = SVMSGD::create();
    svmsgd->setOptimalParameters();
    svmsgd->train( trainData );

    Mat foundWeights = svmsgd->getWeights();
    float foundShift = svmsgd->getShift();

    float normFoundWeights = static_cast<float>(cv::norm(foundWeights)); // TODO cvtest
    foundWeights /= normFoundWeights;
    foundShift /= normFoundWeights;
    EXPECT_LE(cv::norm(Mat(foundWeights - realWeights)), 0.001); // TODO cvtest
    EXPECT_LE(std::abs((foundShift - realShift) / realShift), 0.05);
}

}} // namespace
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

- **std()**: A function/method defined in this file
- **testing()**: A function/method defined in this file
- **tuple()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `test_precomp.hpp`


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

