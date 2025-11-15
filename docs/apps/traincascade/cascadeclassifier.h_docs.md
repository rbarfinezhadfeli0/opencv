# Documentation for `apps/traincascade/cascadeclassifier.h`

## File Metadata

- **Full Path**: `apps/traincascade/cascadeclassifier.h`
- **File Name**: `cascadeclassifier.h`
- **File Size**: 3,973 bytes
- **File Type**: .h
- **Link to Source**: [apps/traincascade/cascadeclassifier.h](../../apps/traincascade/cascadeclassifier.h)

## Purpose and Role

This file is located in the `apps/traincascade` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#ifndef _OPENCV_CASCADECLASSIFIER_H_
#define _OPENCV_CASCADECLASSIFIER_H_

#include <ctime>
#include "traincascade_features.h"
#include "haarfeatures.h"
#include "lbpfeatures.h"
#include "HOGfeatures.h" //new
#include "boost.h"

#define CC_CASCADE_FILENAME "cascade.xml"
#define CC_PARAMS_FILENAME "params.xml"

#define CC_CASCADE_PARAMS "cascadeParams"
#define CC_STAGE_TYPE "stageType"
#define CC_FEATURE_TYPE "featureType"
#define CC_HEIGHT "height"
#define CC_WIDTH  "width"

#define CC_STAGE_NUM    "stageNum"
#define CC_STAGES       "stages"
#define CC_STAGE_PARAMS "stageParams"

#define CC_BOOST            "BOOST"
#define CC_BOOST_TYPE       "boostType"
#define CC_DISCRETE_BOOST   "DAB"
#define CC_REAL_BOOST       "RAB"
#define CC_LOGIT_BOOST      "LB"
#define CC_GENTLE_BOOST     "GAB"
#define CC_MINHITRATE       "minHitRate"
#define CC_MAXFALSEALARM    "maxFalseAlarm"
#define CC_TRIM_RATE        "weightTrimRate"
#define CC_MAX_DEPTH        "maxDepth"
#define CC_WEAK_COUNT       "maxWeakCount"
#define CC_STAGE_THRESHOLD  "stageThreshold"
#define CC_WEAK_CLASSIFIERS "weakClassifiers"
#define CC_INTERNAL_NODES   "internalNodes"
#define CC_LEAF_VALUES      "leafValues"

#define CC_FEATURES       FEATURES
#define CC_FEATURE_PARAMS "featureParams"
#define CC_MAX_CAT_COUNT  "maxCatCount"
#define CC_FEATURE_SIZE   "featSize"

#define CC_HAAR        "HAAR"
#define CC_MODE        "mode"
#define CC_MODE_BASIC  "BASIC"
#define CC_MODE_CORE   "CORE"
#define CC_MODE_ALL    "ALL"
#define CC_RECTS       "rects"
#define CC_TILTED      "tilted"

#define CC_LBP  "LBP"
#define CC_RECT "rect"

#define CC_HOG "HOG"

#ifdef _WIN32
#define TIME( arg ) (((double) clock()) / CLOCKS_PER_SEC)
#else
#define TIME( arg ) (time( arg ))
#endif

class CvCascadeParams : public CvParams
{
public:
    enum { BOOST = 0 };
    static const int defaultStageType = BOOST;
    static const int defaultFeatureType = CvFeatureParams::HAAR;

    CvCascadeParams();
    CvCascadeParams( int _stageType, int _featureType );
    void write( cv::FileStorage &fs ) const;
    bool read( const cv::FileNode &node );

    void printDefaults() const;
    void printAttrs() const;
    bool scanAttr( const std::string prmName, const std::string val );

    int stageType;
    int featureType;
    cv::Size winSize;
};

class CvCascadeClassifier
{
public:
    bool train( const std::string _cascadeDirName,
                const std::string _posFilename,
                const std::string _negFilename,
                int _numPos, int _numNeg,
                int _precalcValBufSize, int _precalcIdxBufSize,
                int _numStages,
                const CvCascadeParams& _cascadeParams,
                const CvFeatureParams& _featureParams,
                const CvCascadeBoostParams& _stageParams,
                bool baseFormatSave = false,
                double acceptanceRatioBreakValue = -1.0 );
private:
    int predict( int sampleIdx );
    void save( const std::string cascadeDirName, bool baseFormat = false );
    bool load( const std::string cascadeDirName );
    bool updateTrainingSet( double minimumAcceptanceRatio, double& acceptanceRatio );
    int fillPassedSamples( int first, int count, bool isPositive, double requiredAcceptanceRatio, int64& consumed );

    void writeParams( cv::FileStorage &fs ) const;
    void writeStages( cv::FileStorage &fs, const cv::Mat& featureMap ) const;
    void writeFeatures( cv::FileStorage &fs, const cv::Mat& featureMap ) const;
    bool readParams( const cv::FileNode &node );
    bool readStages( const cv::FileNode &node );

    void getUsedFeaturesIdxMap( cv::Mat& featureMap );

    CvCascadeParams cascadeParams;
    cv::Ptr<CvFeatureParams> featureParams;
    cv::Ptr<CvCascadeBoostParams> stageParams;

    cv::Ptr<CvFeatureEvaluator> featureEvaluator;
    std::vector< cv::Ptr<CvCascadeBoost> > stageClassifiers;
    CvCascadeImageReader imgReader;
    int numStages, curNumSamples;
    int numPos, numNeg;
};

#endif
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

- **CvCascadeParams**: A class/struct defined in this file
- **CvCascadeClassifier**: A class/struct defined in this file

### Functions and Methods

- **_WIN32()**: A function/method defined in this file
- **_OPENCV_CASCADECLASSIFIER_H_()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `traincascade_features.h`
- `boost.h`
- `haarfeatures.h`
- `HOGfeatures.h`
- `lbpfeatures.h`
- `ctime`


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

