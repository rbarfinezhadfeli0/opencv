# Documentation for `modules/objdetect/src/barcode_decoder/common/hybrid_binarizer.cpp`

## File Metadata

- **Full Path**: `modules/objdetect/src/barcode_decoder/common/hybrid_binarizer.cpp`
- **File Name**: `hybrid_binarizer.cpp`
- **File Size**: 7,260 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/objdetect/src/barcode_decoder/common/hybrid_binarizer.cpp](../../../../../modules/objdetect/src/barcode_decoder/common/hybrid_binarizer.cpp)

## Purpose and Role

This file is located in the `modules/objdetect/src/barcode_decoder/common` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
// Modified from ZXing. Copyright ZXing authors.
// Licensed under the Apache License, Version 2.0 (the "License").

#include "../../precomp.hpp"
#include "hybrid_binarizer.hpp"

namespace cv {
namespace barcode {


#define CLAMP(x, x1, x2) x < (x1) ? (x1) : ((x) > (x2) ? (x2) : (x))

// This class uses 5x5 blocks to compute local luminance, where each block is 8x8 pixels.
// So this is the smallest dimension in each axis we can accept.
constexpr static int BLOCK_SIZE_POWER = 3;
constexpr static int BLOCK_SIZE = 1 << BLOCK_SIZE_POWER; // ...0100...00
constexpr static int BLOCK_SIZE_MASK = BLOCK_SIZE - 1;   // ...0011...11
constexpr static int MINIMUM_DIMENSION = BLOCK_SIZE * 5;
constexpr static int MIN_DYNAMIC_RANGE = 24;

void
calculateThresholdForBlock(const std::vector<uchar> &luminances, int sub_width, int sub_height, int width, int height,
                           const Mat &black_points, Mat &dst)
{
    int maxYOffset = height - BLOCK_SIZE;
    int maxXOffset = width - BLOCK_SIZE;
    for (int y = 0; y < sub_height; y++)
    {
        int yoffset = y << BLOCK_SIZE_POWER;
        if (yoffset > maxYOffset)
        {
            yoffset = maxYOffset;
        }
        int top = CLAMP(y, 2, sub_height - 3);
        for (int x = 0; x < sub_width; x++)
        {
            int xoffset = x << BLOCK_SIZE_POWER;
            if (xoffset > maxXOffset)
            {
                xoffset = maxXOffset;
            }
            int left = CLAMP(x, 2, sub_width - 3);
            int sum = 0;
            const auto *black_row = black_points.ptr<uchar>(top - 2);
            for (int z = 0; z <= 4; z++)
            {
                sum += black_row[left - 2] + black_row[left - 1] + black_row[left] + black_row[left + 1] +
                       black_row[left + 2];
                black_row += black_points.cols;
            }
            int average = sum / 25;
            int temp_y = 0;

            auto *ptr = dst.ptr<uchar>(yoffset, xoffset);
            for (int offset = yoffset * width + xoffset; temp_y < 8; offset += width)
            {
                for (int temp_x = 0; temp_x < 8; ++temp_x)
                {
                    *(ptr + temp_x) = (luminances[offset + temp_x] & 255) <= average ? 0 : 255;
                }
                ++temp_y;
                ptr += width;
            }
        }
    }

}

Mat calculateBlackPoints(std::vector<uchar> luminances, int sub_width, int sub_height, int width, int height)
{
    int maxYOffset = height - BLOCK_SIZE;
    int maxXOffset = width - BLOCK_SIZE;
    Mat black_points(Size(sub_width, sub_height), CV_8UC1);
    for (int y = 0; y < sub_height; y++)
    {
        int yoffset = y << BLOCK_SIZE_POWER;
        if (yoffset > maxYOffset)
        {
            yoffset = maxYOffset;
        }
        for (int x = 0; x < sub_width; x++)
        {
            int xoffset = x << BLOCK_SIZE_POWER;
            if (xoffset > maxXOffset)
            {
                xoffset = maxXOffset;
            }
            int sum = 0;
            int min = 0xFF;
            int max = 0;
            for (int yy = 0, offset = yoffset * width + xoffset; yy < BLOCK_SIZE; yy++, offset += width)
            {
                for (int xx = 0; xx < BLOCK_SIZE; xx++)
                {
                    int pixel = luminances[offset + xx] & 0xFF;
                    sum += pixel;
                    // still looking for good contrast
                    if (pixel < min)
                    {
                        min = pixel;
                    }
                    if (pixel > max)
                    {
                        max = pixel;
                    }
                }
                // short-circuit min/max tests once dynamic range is met
                if (max - min > MIN_DYNAMIC_RANGE)
                {
                    // finish the rest of the rows quickly
                    for (yy++, offset += width; yy < BLOCK_SIZE; yy++, offset += width)
                    {
                        for (int xx = 0; xx < BLOCK_SIZE; xx++)
                        {
                            sum += luminances[offset + xx] & 0xFF;
                        }
                    }
                }
            }

            // The default estimate is the average of the values in the block.
            int average = sum >> (BLOCK_SIZE_POWER * 2);
            if (max - min <= MIN_DYNAMIC_RANGE)
            {
                // If variation within the block is low, assume this is a block with only light or only
                // dark pixels. In that case we do not want to use the average, as it would divide this
                // low contrast area into black and white pixels, essentially creating data out of noise.
                //
                // The default assumption is that the block is light/background. Since no estimate for
                // the level of dark pixels exists locally, use half the min for the block.
                average = min / 2;

                if (y > 0 && x > 0)
                {
                    // Correct the "white background" assumption for blocks that have neighbors by comparing
                    // the pixels in this block to the previously calculated black points. This is based on
                    // the fact that dark barcode symbology is always surrounded by some amount of light
                    // background for which reasonable black point estimates were made. The bp estimated at
                    // the boundaries is used for the interior.

                    // The (min < bp) is arbitrary but works better than other heuristics that were tried.
                    int averageNeighborBlackPoint =
                            (black_points.at<uchar>(y - 1, x) + (2 * black_points.at<uchar>(y, x - 1)) +
                             black_points.at<uchar>(y - 1, x - 1)) / 4;
                    if (min < averageNeighborBlackPoint)
                    {
                        average = averageNeighborBlackPoint;
                    }
                }
            }
            black_points.at<uchar>(y, x) = (uchar) average;
        }
    }
    return black_points;

}


void hybridBinarization(const Mat &src, Mat &dst)
{
    int width = src.cols;
    int height = src.rows;

    if (width >= MINIMUM_DIMENSION && height >= MINIMUM_DIMENSION)
    {
        std::vector<uchar> luminances(src.begin<uchar>(), src.end<uchar>());

        int sub_width = width >> BLOCK_SIZE_POWER;
        if ((width & BLOCK_SIZE_MASK) != 0)
        {
            sub_width++;
        }

        int sub_height = height >> BLOCK_SIZE_POWER;
        if ((height & BLOCK_SIZE_MASK) != 0)
        {
            sub_height++;
        }

        Mat black_points = calculateBlackPoints(luminances, sub_width, sub_height, width, height);

        dst.create(src.size(), src.type());
        calculateThresholdForBlock(luminances, sub_width, sub_height, width, height, black_points, dst);
    }
    else
    {
        threshold(src, dst, 155, 255, THRESH_OTSU + THRESH_BINARY);
    }

}
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

- **uses**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `../../precomp.hpp`
- `hybrid_binarizer.hpp`

**Python Imports:**
- `ZXing.`


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

