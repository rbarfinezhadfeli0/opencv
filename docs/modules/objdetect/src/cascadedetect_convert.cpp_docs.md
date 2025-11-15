# Documentation for `modules/objdetect/src/cascadedetect_convert.cpp`

## File Metadata

- **Full Path**: `modules/objdetect/src/cascadedetect_convert.cpp`
- **File Name**: `cascadedetect_convert.cpp`
- **File Size**: 9,069 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/objdetect/src/cascadedetect_convert.cpp](../../../modules/objdetect/src/cascadedetect_convert.cpp)

## Purpose and Role

This file is located in the `modules/objdetect/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
/*M///////////////////////////////////////////////////////////////////////////////////////
//
//  IMPORTANT: READ BEFORE DOWNLOADING, COPYING, INSTALLING OR USING.
//
//  By downloading, copying, installing or using the software you agree to this license.
//  If you do not agree to this license, do not download, install,
//  copy or use the software.
//
//
//                           License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2013, Itseez Inc, all rights reserved.
// Third party copyrights are property of their respective owners.
//
// Redistribution and use in source and binary forms, with or without modification,
// are permitted provided that the following conditions are met:
//
//   * Redistribution's of source code must retain the above copyright notice,
//     this list of conditions and the following disclaimer.
//
//   * Redistribution's in binary form must reproduce the above copyright notice,
//     this list of conditions and the following disclaimer in the documentation
//     and/or other materials provided with the distribution.
//
//   * The name of Intel Corporation may not be used to endorse or promote products
//     derived from this software without specific prior written permission.
//
// This software is provided by the copyright holders and contributors "as is" and
// any express or implied warranties, including, but not limited to, the implied
// warranties of merchantability and fitness for a particular purpose are disclaimed.
// In no event shall the Intel Corporation or contributors be liable for any direct,
// indirect, incidental, special, exemplary, or consequential damages
// (including, but not limited to, procurement of substitute goods or services;
// loss of use, data, or profits; or business interruption) however caused
// and on any theory of liability, whether in contract, strict liability,
// or tort (including negligence or otherwise) arising in any way out of
// the use of this software, even if advised of the possibility of such damage.
//
//M*/

/* Haar features calculation */

#include "precomp.hpp"
#include "cascadedetect.hpp"
#include <stdio.h>

namespace cv
{

/* field names */

#define ICV_HAAR_SIZE_NAME            "size"
#define ICV_HAAR_STAGES_NAME          "stages"
#define ICV_HAAR_TREES_NAME           "trees"
#define ICV_HAAR_FEATURE_NAME         "feature"
#define ICV_HAAR_RECTS_NAME           "rects"
#define ICV_HAAR_TILTED_NAME          "tilted"
#define ICV_HAAR_THRESHOLD_NAME       "threshold"
#define ICV_HAAR_LEFT_NODE_NAME       "left_node"
#define ICV_HAAR_LEFT_VAL_NAME        "left_val"
#define ICV_HAAR_RIGHT_NODE_NAME      "right_node"
#define ICV_HAAR_RIGHT_VAL_NAME       "right_val"
#define ICV_HAAR_STAGE_THRESHOLD_NAME "stage_threshold"
#define ICV_HAAR_PARENT_NAME          "parent"
#define ICV_HAAR_NEXT_NAME            "next"

namespace haar_cvt
{

struct HaarFeature
{
    enum { RECT_NUM = 3 };

    HaarFeature()
    {
        tilted = false;
        for( int i = 0; i < RECT_NUM; i++ )
        {
            rect[i].r = Rect(0,0,0,0);
            rect[i].weight = 0.f;
        }
    }
    bool tilted;
    struct
    {
        Rect r;
        float weight;
    } rect[RECT_NUM];
};

struct HaarClassifierNode
{
    HaarClassifierNode()
    {
        f = left = right = 0;
        threshold = 0.f;
    }
    int f, left, right;
    float threshold;
};

struct HaarClassifier
{
    std::vector<HaarClassifierNode> nodes;
    std::vector<float> leaves;
};

struct HaarStageClassifier
{
    HaarStageClassifier() : threshold(0) {}

    double threshold;
    std::vector<HaarClassifier> weaks;
};

bool convert(const FileNode& oldroot, FileStorage& newfs)
{
    FileNode sznode = oldroot[ICV_HAAR_SIZE_NAME];
    if( sznode.empty() )
        return false;
    Size cascadesize;
    cascadesize.width = (int)sznode[0];
    cascadesize.height = (int)sznode[1];
    std::vector<HaarFeature> features;

    int i, j, k, n;

    FileNode stages_seq = oldroot[ICV_HAAR_STAGES_NAME];
    int nstages = (int)stages_seq.size();
    std::vector<HaarStageClassifier> stages(nstages);

    for( i = 0; i < nstages; i++ )
    {
        FileNode stagenode = stages_seq[i];
        HaarStageClassifier& stage = stages[i];
        stage.threshold = (double)stagenode[ICV_HAAR_STAGE_THRESHOLD_NAME];
        FileNode weaks_seq = stagenode[ICV_HAAR_TREES_NAME];
        int nweaks = (int)weaks_seq.size();
        stage.weaks.resize(nweaks);

        for( j = 0; j < nweaks; j++ )
        {
            HaarClassifier& weak = stage.weaks[j];
            FileNode weaknode = weaks_seq[j];
            int nnodes = (int)weaknode.size();

            for( n = 0; n < nnodes; n++ )
            {
                FileNode nnode = weaknode[n];
                FileNode fnode = nnode[ICV_HAAR_FEATURE_NAME];
                HaarFeature f;
                HaarClassifierNode node;
                node.f = (int)features.size();
                f.tilted = (int)fnode[ICV_HAAR_TILTED_NAME] != 0;
                FileNode rects_seq = fnode[ICV_HAAR_RECTS_NAME];
                int nrects = (int)rects_seq.size();

                for( k = 0; k < nrects; k++ )
                {
                    FileNode rnode = rects_seq[k];
                    f.rect[k].r.x = (int)rnode[0];
                    f.rect[k].r.y = (int)rnode[1];
                    f.rect[k].r.width = (int)rnode[2];
                    f.rect[k].r.height = (int)rnode[3];
                    f.rect[k].weight = (float)rnode[4];
                }
                features.push_back(f);
                node.threshold = nnode[ICV_HAAR_THRESHOLD_NAME];
                FileNode leftValNode = nnode[ICV_HAAR_LEFT_VAL_NAME];
                if( !leftValNode.empty() )
                {
                    node.left = -(int)weak.leaves.size();
                    weak.leaves.push_back((float)leftValNode);
                }
                else
                {
                    node.left = (int)nnode[ICV_HAAR_LEFT_NODE_NAME];
                }
                FileNode rightValNode = nnode[ICV_HAAR_RIGHT_VAL_NAME];
                if( !rightValNode.empty() )
                {
                    node.right = -(int)weak.leaves.size();
                    weak.leaves.push_back((float)rightValNode);
                }
                else
                {
                    node.right = (int)nnode[ICV_HAAR_RIGHT_NODE_NAME];
                }
                weak.nodes.push_back(node);
            }
        }
    }

    int maxWeakCount = 0, nfeatures = (int)features.size();
    for( i = 0; i < nstages; i++ )
        maxWeakCount = std::max(maxWeakCount, (int)stages[i].weaks.size());

    newfs << "cascade" << "{:opencv-cascade-classifier"
    << "stageType" << "BOOST"
    << "featureType" << "HAAR"
    << "width" << cascadesize.width
    << "height" << cascadesize.height
    << "stageParams" << "{"
        << "maxWeakCount" << (int)maxWeakCount
    << "}"
    << "featureParams" << "{"
        << "maxCatCount" << 0
    << "}"
    << "stageNum" << (int)nstages
    << "stages" << "[";

    for( i = 0; i < nstages; i++ )
    {
        int nweaks = (int)stages[i].weaks.size();
        newfs << "{" << "maxWeakCount" << (int)nweaks
            << "stageThreshold" << stages[i].threshold
            << "weakClassifiers" << "[";
        for( j = 0; j < nweaks; j++ )
        {
            const HaarClassifier& c = stages[i].weaks[j];
            newfs << "{" << "internalNodes" << "[:";
            int nnodes = (int)c.nodes.size(), nleaves = (int)c.leaves.size();
            for( k = 0; k < nnodes; k++ )
                newfs << c.nodes[k].left << c.nodes[k].right
                    << c.nodes[k].f << c.nodes[k].threshold;
            newfs << "]" << "leafValues" << "[:";
            for( k = 0; k < nleaves; k++ )
                newfs << c.leaves[k];
            newfs << "]" << "}";
        }
        newfs << "]" << "}";
    }

    newfs << "]"
        << "features" << "[";

    for( i = 0; i < nfeatures; i++ )
    {
        const HaarFeature& f = features[i];
        newfs << "{" << "rects" << "[";
        for( j = 0; j < HaarFeature::RECT_NUM; j++ )
        {
            if( j >= 2 && fabs(f.rect[j].weight) < FLT_EPSILON )
                break;
            newfs << "[:" << f.rect[j].r.x << f.rect[j].r.y <<
                f.rect[j].r.width << f.rect[j].r.height << f.rect[j].weight << "]";
        }
        newfs << "]";
        if( f.tilted )
            newfs << "tilted" << 1;
        newfs << "}";
    }

    newfs << "]" << "}";
    return true;
}

}

bool CascadeClassifier::convert(const String& oldcascade, const String& newcascade)
{
    FileStorage oldfs(oldcascade, FileStorage::READ);
    FileStorage newfs(newcascade, FileStorage::WRITE);
    if( !oldfs.isOpened() || !newfs.isOpened() )
        return false;
    FileNode oldroot = oldfs.getFirstTopLevelNode();

    bool ok = haar_cvt::convert(oldroot, newfs);
    if( !ok && newcascade.size() > 0 )
        remove(newcascade.c_str());
    return ok;
}

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

### Classes and Structures

- **HaarFeature**: A class/struct defined in this file
- **HaarClassifier**: A class/struct defined in this file
- **HaarStageClassifier**: A class/struct defined in this file
- **HaarClassifierNode**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `cascadedetect.hpp`
- `precomp.hpp`
- `stdio.h`

**Python Imports:**
- `this`


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

