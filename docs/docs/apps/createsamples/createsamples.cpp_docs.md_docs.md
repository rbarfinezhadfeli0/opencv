# Documentation for `docs/apps/createsamples/createsamples.cpp_docs.md`

## File Metadata

- **Full Path**: `docs/apps/createsamples/createsamples.cpp_docs.md`
- **File Name**: `createsamples.cpp_docs.md`
- **File Size**: 12,011 bytes
- **File Type**: .md
- **Link to Source**: [docs/apps/createsamples/createsamples.cpp_docs.md](../../../docs/apps/createsamples/createsamples.cpp_docs.md)

## Purpose and Role

This file is located in the `docs/apps/createsamples` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `apps/createsamples/createsamples.cpp`

## File Metadata

- **Full Path**: `apps/createsamples/createsamples.cpp`
- **File Name**: `createsamples.cpp`
- **File Size**: 8,968 bytes
- **File Type**: .cpp
- **Link to Source**: [apps/createsamples/createsamples.cpp](../../apps/createsamples/createsamples.cpp)

## Purpose and Role

This file is located in the `apps/createsamples` directory and serves as part of the OpenCV library infrastructure.

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
//                        Intel License Agreement
//                For Open Source Computer Vision Library
//
// Copyright (C) 2000, Intel Corporation, all rights reserved.
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

/*
 * createsamples.cpp
 *
 * Create test/training samples
 */

#include "opencv2/core.hpp"
#include "utility.hpp"
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <cmath>

using namespace std;

int main( int argc, char* argv[] )
{
    int i = 0;
    char* nullname   = (char*)"(NULL)";
    char* vecname    = NULL; /* .vec file name */
    char* infoname   = NULL; /* file name with marked up image descriptions */
    char* imagename  = NULL; /* single sample image */
    char* bgfilename = NULL; /* background */
    int num = 1000;
    int bgcolor = 0;
    int bgthreshold = 80;
    int invert = 0;
    int maxintensitydev = 40;
    double maxxangle = 1.1;
    double maxyangle = 1.1;
    double maxzangle = 0.5;
    int showsamples = 0;
    /* the samples are adjusted to this scale in the sample preview window */
    double scale = 4.0;
    int width  = 24;
    int height = 24;
    double maxscale = -1.0;
    int rngseed = 12345;

    if( argc == 1 )
    {
        printf( "Usage: %s\n  [-info <collection_file_name>]\n"
                "  [-img <image_file_name>]\n"
                "  [-vec <vec_file_name>]\n"
                "  [-bg <background_file_name>]\n  [-num <number_of_samples = %d>]\n"
                "  [-bgcolor <background_color = %d>]\n"
                "  [-inv] [-randinv] [-bgthresh <background_color_threshold = %d>]\n"
                "  [-maxidev <max_intensity_deviation = %d>]\n"
                "  [-maxxangle <max_x_rotation_angle = %f>]\n"
                "  [-maxyangle <max_y_rotation_angle = %f>]\n"
                "  [-maxzangle <max_z_rotation_angle = %f>]\n"
                "  [-show [<scale = %f>]]\n"
                "  [-w <sample_width = %d>]\n  [-h <sample_height = %d>]\n"
                "  [-maxscale <max sample scale = %f>]\n"
                "  [-rngseed <rng seed = %d>]\n",
                argv[0], num, bgcolor, bgthreshold, maxintensitydev,
                maxxangle, maxyangle, maxzangle, scale, width, height, maxscale, rngseed );

        return 0;
    }

    for( i = 1; i < argc; ++i )
    {
        if( !strcmp( argv[i], "-info" ) )
        {
            infoname = argv[++i];
        }
        else if( !strcmp( argv[i], "-img" ) )
        {
            imagename = argv[++i];
        }
        else if( !strcmp( argv[i], "-vec" ) )
        {
            vecname = argv[++i];
        }
        else if( !strcmp( argv[i], "-bg" ) )
        {
            bgfilename = argv[++i];
        }
        else if( !strcmp( argv[i], "-num" ) )
        {
            num = atoi( argv[++i] );
        }
        else if( !strcmp( argv[i], "-bgcolor" ) )
        {
            bgcolor = atoi( argv[++i] );
        }
        else if( !strcmp( argv[i], "-bgthresh" ) )
        {
            bgthreshold = atoi( argv[++i] );
        }
        else if( !strcmp( argv[i], "-inv" ) )
        {
            invert = 1;
        }
        else if( !strcmp( argv[i], "-randinv" ) )
        {
            invert = CV_RANDOM_INVERT;
        }
        else if( !strcmp( argv[i], "-maxidev" ) )
        {
            maxintensitydev = atoi( argv[++i] );
        }
        else if( !strcmp( argv[i], "-maxxangle" ) )
        {
            maxxangle = atof( argv[++i] );
        }
        else if( !strcmp( argv[i], "-maxyangle" ) )
        {
            maxyangle = atof( argv[++i] );
        }
        else if( !strcmp( argv[i], "-maxzangle" ) )
        {
            maxzangle = atof( argv[++i] );
        }
        else if( !strcmp( argv[i], "-show" ) )
        {
            showsamples = 1;
            if( i+1 < argc && strlen( argv[i+1] ) > 0 && argv[i+1][0] != '-' )
            {
                double d;
                d = strtod( argv[i+1], 0 );
                if( d != -HUGE_VAL && d != HUGE_VAL && d > 0 ) scale = d;
                ++i;
            }
        }
        else if( !strcmp( argv[i], "-w" ) )
        {
            width = atoi( argv[++i] );
        }
        else if( !strcmp( argv[i], "-h" ) )
        {
            height = atoi( argv[++i] );
        }
        else if( !strcmp( argv[i], "-maxscale" ) )
        {
            maxscale = atof( argv[++i] );
        }
        else if (!strcmp(argv[i], "-rngseed"))
        {
            rngseed = atoi(argv[++i]);
        }
    }

    cv::setRNGSeed( rngseed );

    printf( "Info file name: %s\n", ((infoname == NULL) ?   nullname : infoname ) );
    printf( "Img file name: %s\n",  ((imagename == NULL) ?  nullname : imagename ) );
    printf( "Vec file name: %s\n",  ((vecname == NULL) ?    nullname : vecname ) );
    printf( "BG  file name: %s\n",  ((bgfilename == NULL) ? nullname : bgfilename ) );
    printf( "Num: %d\n", num );
    printf( "BG color: %d\n", bgcolor );
    printf( "BG threshold: %d\n", bgthreshold );
    printf( "Invert: %s\n", (invert == CV_RANDOM_INVERT) ? "RANDOM"
                                : ( (invert) ? "TRUE" : "FALSE" ) );
    printf( "Max intensity deviation: %d\n", maxintensitydev );
    printf( "Max x angle: %g\n", maxxangle );
    printf( "Max y angle: %g\n", maxyangle );
    printf( "Max z angle: %g\n", maxzangle );
    printf( "Show samples: %s\n", (showsamples) ? "TRUE" : "FALSE" );
    if( showsamples )
    {
        printf( "Scale: %g\n", scale );
    }
    printf( "Width: %d\n", width );
    printf( "Height: %d\n", height );
    printf( "Max Scale: %g\n", maxscale );
    printf( "RNG Seed: %d\n", rngseed );

    /* determine action */
    if( imagename && vecname )
    {
        printf( "Create training samples from single image applying distortions...\n" );

        cvCreateTrainingSamples( vecname, imagename, bgcolor, bgthreshold, bgfilename,
                                 num, invert, maxintensitydev,
                                 maxxangle, maxyangle, maxzangle,
                                 showsamples, width, height );

        printf( "Done\n" );
    }
    else if( imagename && bgfilename && infoname )
    {
        printf( "Create test samples from single image applying distortions...\n" );

        cvCreateTestSamples( infoname, imagename, bgcolor, bgthreshold, bgfilename, num,
            invert, maxintensitydev,
            maxxangle, maxyangle, maxzangle, showsamples, width, height, maxscale);

        printf( "Done\n" );
    }
    else if( infoname && vecname )
    {
        int total;

        printf( "Create training samples from images collection...\n" );

        total = cvCreateTrainingSamplesFromInfo( infoname, vecname, num, showsamples,
                                                 width, height );

        printf( "Done. Created %d samples\n", total );
    }
    else if( vecname )
    {
        printf( "View samples from vec file (press ESC to exit)...\n" );

        cvShowVecSamples( vecname, width, height, scale );

        printf( "Done\n" );
    }
    else
    {
        printf( "Nothing to do\n" );
    }

    return 0;
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
- `cmath`
- `utility.hpp`
- `cstring`
- `opencv2/core.hpp`
- `cstdio`
- `cstdlib`

**Python Imports:**
- `single`
- `vec`
- `images`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

