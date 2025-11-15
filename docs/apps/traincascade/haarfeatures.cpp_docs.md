# Documentation for `apps/traincascade/haarfeatures.cpp`

## File Metadata

- **Full Path**: `apps/traincascade/haarfeatures.cpp`
- **File Name**: `haarfeatures.cpp`
- **File Size**: 11,457 bytes
- **File Type**: .cpp
- **Link to Source**: [apps/traincascade/haarfeatures.cpp](../../apps/traincascade/haarfeatures.cpp)

## Purpose and Role

This file is located in the `apps/traincascade` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#include "opencv2/core.hpp"
#include "opencv2/imgproc.hpp"

#include "haarfeatures.h"
#include "cascadeclassifier.h"

using namespace std;
using namespace cv;

CvHaarFeatureParams::CvHaarFeatureParams() : mode(BASIC)
{
    name = HFP_NAME;
}

CvHaarFeatureParams::CvHaarFeatureParams( int _mode ) : mode( _mode )
{
    name = HFP_NAME;
}

void CvHaarFeatureParams::init( const CvFeatureParams& fp )
{
    CvFeatureParams::init( fp );
    mode = ((const CvHaarFeatureParams&)fp).mode;
}

void CvHaarFeatureParams::write( FileStorage &fs ) const
{
    CvFeatureParams::write( fs );
    string modeStr = mode == BASIC ? CC_MODE_BASIC :
                     mode == CORE ? CC_MODE_CORE :
                     mode == ALL ? CC_MODE_ALL : string();
    CV_Assert( !modeStr.empty() );
    fs << CC_MODE << modeStr;
}

bool CvHaarFeatureParams::read( const FileNode &node )
{
    if( !CvFeatureParams::read( node ) )
        return false;

    FileNode rnode = node[CC_MODE];
    if( !rnode.isString() )
        return false;
    string modeStr;
    rnode >> modeStr;
    mode = !modeStr.compare( CC_MODE_BASIC ) ? BASIC :
           !modeStr.compare( CC_MODE_CORE ) ? CORE :
           !modeStr.compare( CC_MODE_ALL ) ? ALL : -1;
    return (mode >= 0);
}

void CvHaarFeatureParams::printDefaults() const
{
    CvFeatureParams::printDefaults();
    cout << "  [-mode <" CC_MODE_BASIC << "(default) | "
            << CC_MODE_CORE <<" | " << CC_MODE_ALL << endl;
}

void CvHaarFeatureParams::printAttrs() const
{
    CvFeatureParams::printAttrs();
    string mode_str = mode == BASIC ? CC_MODE_BASIC :
                       mode == CORE ? CC_MODE_CORE :
                       mode == ALL ? CC_MODE_ALL : 0;
    cout << "mode: " <<  mode_str << endl;
}

bool CvHaarFeatureParams::scanAttr( const string prmName, const string val)
{
    if ( !CvFeatureParams::scanAttr( prmName, val ) )
    {
        if( !prmName.compare("-mode") )
        {
            mode = !val.compare( CC_MODE_CORE ) ? CORE :
                   !val.compare( CC_MODE_ALL ) ? ALL :
                   !val.compare( CC_MODE_BASIC ) ? BASIC : -1;
            if (mode == -1)
                return false;
        }
        return false;
    }
    return true;
}

//--------------------- HaarFeatureEvaluator ----------------

void CvHaarEvaluator::init(const CvFeatureParams *_featureParams,
                           int _maxSampleCount, Size _winSize )
{
    CV_Assert(_maxSampleCount > 0);
    int cols = (_winSize.width + 1) * (_winSize.height + 1);
    sum.create((int)_maxSampleCount, cols, CV_32SC1);
    tilted.create((int)_maxSampleCount, cols, CV_32SC1);
    normfactor.create(1, (int)_maxSampleCount, CV_32FC1);
    CvFeatureEvaluator::init( _featureParams, _maxSampleCount, _winSize );
}

void CvHaarEvaluator::setImage(const Mat& img, uchar clsLabel, int idx)
{
    CV_DbgAssert( !sum.empty() && !tilted.empty() && !normfactor.empty() );
    CvFeatureEvaluator::setImage( img, clsLabel, idx);
    Mat innSum(winSize.height + 1, winSize.width + 1, sum.type(), sum.ptr<int>((int)idx));
    Mat innSqSum;
    if (((const CvHaarFeatureParams*)featureParams)->mode == CvHaarFeatureParams::ALL)
    {
        Mat innTilted(winSize.height + 1, winSize.width + 1, tilted.type(), tilted.ptr<int>((int)idx));
        integral(img, innSum, innSqSum, innTilted);
    }
    else
        integral(img, innSum, innSqSum);
    normfactor.ptr<float>(0)[idx] = calcNormFactor( innSum, innSqSum );
}

void CvHaarEvaluator::writeFeatures( FileStorage &fs, const Mat& featureMap ) const
{
    _writeFeatures( features, fs, featureMap );
}

void CvHaarEvaluator::writeFeature(FileStorage &fs, int fi) const
{
    CV_DbgAssert( fi < (int)features.size() );
    features[fi].write(fs);
}

void CvHaarEvaluator::generateFeatures()
{
    int mode = ((const CvHaarFeatureParams*)((CvFeatureParams*)featureParams))->mode;
    int offset = winSize.width + 1;
    for( int x = 0; x < winSize.width; x++ )
    {
        for( int y = 0; y < winSize.height; y++ )
        {
            for( int dx = 1; dx <= winSize.width; dx++ )
            {
                for( int dy = 1; dy <= winSize.height; dy++ )
                {
                    // haar_x2
                    if ( (x+dx*2 <= winSize.width) && (y+dy <= winSize.height) )
                    {
                        features.push_back( Feature( offset, false,
                            x,    y, dx*2, dy, -1,
                            x+dx, y, dx  , dy, +2 ) );
                    }
                    // haar_y2
                    if ( (x+dx <= winSize.width) && (y+dy*2 <= winSize.height) )
                    {
                        features.push_back( Feature( offset, false,
                            x,    y, dx, dy*2, -1,
                            x, y+dy, dx, dy,   +2 ) );
                    }
                    // haar_x3
                    if ( (x+dx*3 <= winSize.width) && (y+dy <= winSize.height) )
                    {
                        features.push_back( Feature( offset, false,
                            x,    y, dx*3, dy, -1,
                            x+dx, y, dx  , dy, +2 ) );
                    }
                    // haar_y3
                    if ( (x+dx <= winSize.width) && (y+dy*3 <= winSize.height) )
                    {
                        features.push_back( Feature( offset, false,
                            x, y,    dx, dy*3, -1,
                            x, y+dy, dx, dy,   +2 ) );
                    }
                    if( mode != CvHaarFeatureParams::BASIC )
                    {
                        // haar_x4
                        if ( (x+dx*4 <= winSize.width) && (y+dy <= winSize.height) )
                        {
                            features.push_back( Feature( offset, false,
                                x,    y, dx*4, dy, -1,
                                x+dx, y, dx*2, dy, +2 ) );
                        }
                        // haar_y4
                        if ( (x+dx <= winSize.width ) && (y+dy*4 <= winSize.height) )
                        {
                            features.push_back( Feature( offset, false,
                                x, y,    dx, dy*4, -1,
                                x, y+dy, dx, dy*2, +2 ) );
                        }
                    }
                    // x2_y2
                    if ( (x+dx*2 <= winSize.width) && (y+dy*2 <= winSize.height) )
                    {
                        features.push_back( Feature( offset, false,
                            x,    y,    dx*2, dy*2, -1,
                            x,    y,    dx,   dy,   +2,
                            x+dx, y+dy, dx,   dy,   +2 ) );
                    }
                    if (mode != CvHaarFeatureParams::BASIC)
                    {
                        if ( (x+dx*3 <= winSize.width) && (y+dy*3 <= winSize.height) )
                        {
                            features.push_back( Feature( offset, false,
                                x   , y   , dx*3, dy*3, -1,
                                x+dx, y+dy, dx  , dy  , +9) );
                        }
                    }
                    if (mode == CvHaarFeatureParams::ALL)
                    {
                        // tilted haar_x2
                        if ( (x+2*dx <= winSize.width) && (y+2*dx+dy <= winSize.height) && (x-dy>= 0) )
                        {
                            features.push_back( Feature( offset, true,
                                x, y, dx*2, dy, -1,
                                x, y, dx,   dy, +2 ) );
                        }
                        // tilted haar_y2
                        if ( (x+dx <= winSize.width) && (y+dx+2*dy <= winSize.height) && (x-2*dy>= 0) )
                        {
                            features.push_back( Feature( offset, true,
                                x, y, dx, 2*dy, -1,
                                x, y, dx, dy,   +2 ) );
                        }
                        // tilted haar_x3
                        if ( (x+3*dx <= winSize.width) && (y+3*dx+dy <= winSize.height) && (x-dy>= 0) )
                        {
                            features.push_back( Feature( offset, true,
                                x,    y,    dx*3, dy, -1,
                                x+dx, y+dx, dx,   dy, +3 ) );
                        }
                        // tilted haar_y3
                        if ( (x+dx <= winSize.width) && (y+dx+3*dy <= winSize.height) && (x-3*dy>= 0) )
                        {
                            features.push_back( Feature( offset, true,
                                x,    y,    dx, 3*dy, -1,
                                x-dy, y+dy, dx, dy,   +3 ) );
                        }
                        // tilted haar_x4
                        if ( (x+4*dx <= winSize.width) && (y+4*dx+dy <= winSize.height) && (x-dy>= 0) )
                        {
                            features.push_back( Feature( offset, true,
                                x,    y,    dx*4, dy, -1,
                                x+dx, y+dx, dx*2, dy, +2 ) );
                        }
                        // tilted haar_y4
                        if ( (x+dx <= winSize.width) && (y+dx+4*dy <= winSize.height) && (x-4*dy>= 0) )
                        {
                            features.push_back( Feature( offset, true,
                                x,    y,    dx, 4*dy, -1,
                                x-dy, y+dy, dx, 2*dy, +2 ) );
                        }
                    }
                }
            }
        }
    }
    numFeatures = (int)features.size();
}

CvHaarEvaluator::Feature::Feature()
{
    tilted = false;
    rect[0].r = rect[1].r = rect[2].r = Rect(0,0,0,0);
    rect[0].weight = rect[1].weight = rect[2].weight = 0;
}

CvHaarEvaluator::Feature::Feature( int offset, bool _tilted,
                                          int x0, int y0, int w0, int h0, float wt0,
                                          int x1, int y1, int w1, int h1, float wt1,
                                          int x2, int y2, int w2, int h2, float wt2 )
{
    tilted = _tilted;

    rect[0].r.x = x0;
    rect[0].r.y = y0;
    rect[0].r.width  = w0;
    rect[0].r.height = h0;
    rect[0].weight   = wt0;

    rect[1].r.x = x1;
    rect[1].r.y = y1;
    rect[1].r.width  = w1;
    rect[1].r.height = h1;
    rect[1].weight   = wt1;

    rect[2].r.x = x2;
    rect[2].r.y = y2;
    rect[2].r.width  = w2;
    rect[2].r.height = h2;
    rect[2].weight   = wt2;

    if( !tilted )
    {
        for( int j = 0; j < CV_HAAR_FEATURE_MAX; j++ )
        {
            if( rect[j].weight == 0.0F )
                break;
            CV_SUM_OFFSETS( fastRect[j].p0, fastRect[j].p1, fastRect[j].p2, fastRect[j].p3, rect[j].r, offset )
        }
    }
    else
    {
        for( int j = 0; j < CV_HAAR_FEATURE_MAX; j++ )
        {
            if( rect[j].weight == 0.0F )
                break;
            CV_TILTED_OFFSETS( fastRect[j].p0, fastRect[j].p1, fastRect[j].p2, fastRect[j].p3, rect[j].r, offset )
        }
    }
}

void CvHaarEvaluator::Feature::write( FileStorage &fs ) const
{
    fs << CC_RECTS << "[";
    for( int ri = 0; ri < CV_HAAR_FEATURE_MAX && rect[ri].r.width != 0; ++ri )
    {
        fs << "[:" << rect[ri].r.x << rect[ri].r.y <<
            rect[ri].r.width << rect[ri].r.height << rect[ri].weight << "]";
    }
    fs << "]" << CC_TILTED << tilted;
}
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/core.hpp`
- `opencv2/imgproc.hpp`
- `cascadeclassifier.h`
- `haarfeatures.h`


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

