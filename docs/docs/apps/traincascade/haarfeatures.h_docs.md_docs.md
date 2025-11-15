# Documentation for `docs/apps/traincascade/haarfeatures.h_docs.md`

## File Metadata

- **Full Path**: `docs/apps/traincascade/haarfeatures.h_docs.md`
- **File Name**: `haarfeatures.h_docs.md`
- **File Size**: 6,232 bytes
- **File Type**: .md
- **Link to Source**: [docs/apps/traincascade/haarfeatures.h_docs.md](../../../docs/apps/traincascade/haarfeatures.h_docs.md)

## Purpose and Role

This file is located in the `docs/apps/traincascade` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `apps/traincascade/haarfeatures.h`

## File Metadata

- **Full Path**: `apps/traincascade/haarfeatures.h`
- **File Name**: `haarfeatures.h`
- **File Size**: 3,019 bytes
- **File Type**: .h
- **Link to Source**: [apps/traincascade/haarfeatures.h](../../apps/traincascade/haarfeatures.h)

## Purpose and Role

This file is located in the `apps/traincascade` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#ifndef _OPENCV_HAARFEATURES_H_
#define _OPENCV_HAARFEATURES_H_

#include "traincascade_features.h"

#define CV_HAAR_FEATURE_MAX      3

#define HFP_NAME "haarFeatureParams"
class CvHaarFeatureParams : public CvFeatureParams
{
public:
    enum { BASIC = 0, CORE = 1, ALL = 2 };
     /* 0 - BASIC = Viola
     *  1 - CORE  = All upright
     *  2 - ALL   = All features */

    CvHaarFeatureParams();
    CvHaarFeatureParams( int _mode );

    virtual void init( const CvFeatureParams& fp );
    virtual void write( cv::FileStorage &fs ) const;
    virtual bool read( const cv::FileNode &node );

    virtual void printDefaults() const;
    virtual void printAttrs() const;
    virtual bool scanAttr( const std::string prm, const std::string val);

    int mode;
};

class CvHaarEvaluator : public CvFeatureEvaluator
{
public:
    virtual void init(const CvFeatureParams *_featureParams,
        int _maxSampleCount, cv::Size _winSize );
    virtual void setImage(const cv::Mat& img, uchar clsLabel, int idx);
    virtual float operator()(int featureIdx, int sampleIdx) const;
    virtual void writeFeatures( cv::FileStorage &fs, const cv::Mat& featureMap ) const;
    void writeFeature( cv::FileStorage &fs, int fi ) const; // for old file fornat
protected:
    virtual void generateFeatures();

    class Feature
    {
    public:
        Feature();
        Feature( int offset, bool _tilted,
            int x0, int y0, int w0, int h0, float wt0,
            int x1, int y1, int w1, int h1, float wt1,
            int x2 = 0, int y2 = 0, int w2 = 0, int h2 = 0, float wt2 = 0.0F );
        float calc( const cv::Mat &sum, const cv::Mat &tilted, size_t y) const;
        void write( cv::FileStorage &fs ) const;

        bool  tilted;
        struct
        {
            cv::Rect r;
            float weight;
        } rect[CV_HAAR_FEATURE_MAX];

        struct
        {
            int p0, p1, p2, p3;
        } fastRect[CV_HAAR_FEATURE_MAX];
    };

    std::vector<Feature> features;
    cv::Mat  sum;         /* sum images (each row represents image) */
    cv::Mat  tilted;      /* tilted sum images (each row represents image) */
    cv::Mat  normfactor;  /* normalization factor */
};

inline float CvHaarEvaluator::operator()(int featureIdx, int sampleIdx) const
{
    float nf = normfactor.at<float>(0, sampleIdx);
    return !nf ? 0.0f : (features[featureIdx].calc( sum, tilted, sampleIdx)/nf);
}

inline float CvHaarEvaluator::Feature::calc( const cv::Mat &_sum, const cv::Mat &_tilted, size_t y) const
{
    const int* img = tilted ? _tilted.ptr<int>((int)y) : _sum.ptr<int>((int)y);
    float ret = rect[0].weight * (img[fastRect[0].p0] - img[fastRect[0].p1] - img[fastRect[0].p2] + img[fastRect[0].p3] ) +
        rect[1].weight * (img[fastRect[1].p0] - img[fastRect[1].p1] - img[fastRect[1].p2] + img[fastRect[1].p3] );
    if( rect[2].weight != 0.0f )
        ret += rect[2].weight * (img[fastRect[2].p0] - img[fastRect[2].p1] - img[fastRect[2].p2] + img[fastRect[2].p3] );
    return ret;
}

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

- **CvHaarEvaluator**: A class/struct defined in this file
- **CvHaarFeatureParams**: A class/struct defined in this file
- **Feature**: A class/struct defined in this file

### Functions and Methods

- **_OPENCV_HAARFEATURES_H_()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `traincascade_features.h`


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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

