# Documentation for `modules/highgui/test/test_gui.cpp`

## File Metadata

- **Full Path**: `modules/highgui/test/test_gui.cpp`
- **File Name**: `test_gui.cpp`
- **File Size**: 9,084 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/highgui/test/test_gui.cpp](../../../modules/highgui/test/test_gui.cpp)

## Purpose and Role

This file is located in the `modules/highgui/test` directory and serves as part of the OpenCV library infrastructure.

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
// Copyright (C) 2000-2008, Intel Corporation, all rights reserved.
// Copyright (C) 2009, Willow Garage Inc., all rights reserved.
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
//   * The name of the copyright holders may not be used to endorse or promote products
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

#include "test_precomp.hpp"

namespace opencv_test { namespace {

inline void verify_size(const std::string &nm, const cv::Mat &img)
{
    EXPECT_NO_THROW(imshow(nm, img));
    EXPECT_EQ(-1, waitKey(200));

    // see https://github.com/opencv/opencv/issues/25550
    // Wayland backend is not supported getWindowImageRect().
    string framework;
    EXPECT_NO_THROW(framework = currentUIFramework());
    if(framework == "WAYLAND")
    {
       return;
    }

    Rect rc;
    EXPECT_NO_THROW(rc = getWindowImageRect(nm));
    EXPECT_EQ(rc.size(), img.size());
}

#if (!defined(ENABLE_PLUGINS) \
        && !defined HAVE_GTK \
        && !defined HAVE_QT \
        && !defined HAVE_WIN32UI \
        && !defined HAVE_COCOA \
        && !defined HAVE_WAYLAND \
    )
TEST(Highgui_GUI, DISABLED_regression)
#else
TEST(Highgui_GUI, regression)
#endif
{
    const std::string window_name("opencv_highgui_test_window");
    const cv::Size image_size(800, 600);

    EXPECT_NO_THROW(destroyAllWindows());
    ASSERT_NO_THROW(namedWindow(window_name));
    const vector<int> channels = {1, 3, 4};
    const vector<int> depths = {CV_8U, CV_8S, CV_16U, CV_16S, CV_32F, CV_64F};
    for(int cn : channels)
    {
        SCOPED_TRACE(cn);
        for(int depth : depths)
        {
            SCOPED_TRACE(depth);
            double min_val = 0.;
            double max_val = 256.;
            switch(depth)
            {
            case CV_8S:
                min_val = static_cast<double>(-0x7F);
                max_val = static_cast<double>(0x7F + 1);
                break;
            case CV_16S:
                min_val = static_cast<double>(-0x7FFF);
                max_val = static_cast<double>(0x7FFF + 1);
                break;
            case CV_16U:
                max_val = static_cast<double>(0xFFFF + 1);
                break;
            case CV_32F:
            case CV_64F:
                max_val = 1.0;
                break;
            }
            Mat m = cvtest::randomMat(TS::ptr()->get_rng(), image_size, CV_MAKE_TYPE(depth, cn), min_val, max_val, false);
            verify_size(window_name, m);

            Mat bgr(image_size, CV_MAKE_TYPE(depth, cn));
            int b_g = image_size.width / 3, g_r = b_g * 2;
            if (cn > 1)
            {
                bgr.colRange(0, b_g).setTo(cv::Scalar(max_val, min_val, min_val));
                bgr.colRange(b_g, g_r).setTo(cv::Scalar(min_val, max_val, min_val));
                bgr.colRange(g_r, image_size.width).setTo(cv::Scalar(min_val, min_val, max_val));
            }
            else
            {
                bgr.colRange(0, b_g).setTo(cv::Scalar::all(min_val));
                bgr.colRange(b_g, g_r).setTo(cv::Scalar::all((min_val + max_val) / 2));
                bgr.colRange(g_r, image_size.width).setTo(cv::Scalar::all(max_val));
            }
            verify_size(window_name, bgr);
        }
    }
    EXPECT_NO_THROW(destroyAllWindows());
}

//==================================================================================================

static void Foo(int, void* counter)
{
    if (counter)
    {
        int *counter_int = static_cast<int*>(counter);
        (*counter_int)++;
    }
}

#if (!defined(ENABLE_PLUGINS) \
        && !defined HAVE_GTK \
        && !defined HAVE_QT \
        && !defined HAVE_WIN32UI \
        && !defined HAVE_WAYLAND \
    ) \
    || defined(__APPLE__)  /* test fails on Mac (cocoa) */ \
    || defined HAVE_FRAMEBUFFER /* trackbar is not supported */
TEST(Highgui_GUI, DISABLED_trackbar_unsafe)
#else
TEST(Highgui_GUI, trackbar_unsafe)
#endif
{
    int value = 50;
    int callback_count = 0;
    const std::string window_name("trackbar_test_window");
    const std::string trackbar_name("trackbar");

    EXPECT_NO_THROW(destroyAllWindows());
    ASSERT_NO_THROW(namedWindow(window_name));
    EXPECT_EQ((int)1, createTrackbar(trackbar_name, window_name, &value, 100, Foo, &callback_count));
    EXPECT_EQ(value, getTrackbarPos(trackbar_name, window_name));
    EXPECT_GE(callback_count, 0);
    EXPECT_LE(callback_count, 1);
    int callback_count_base = callback_count;
    EXPECT_NO_THROW(setTrackbarPos(trackbar_name, window_name, 90));
    EXPECT_EQ(callback_count_base + 1, callback_count);
    EXPECT_EQ(90, value);
    EXPECT_EQ(90, getTrackbarPos(trackbar_name, window_name));
    EXPECT_NO_THROW(destroyAllWindows());
}

static
void testTrackbarCallback(int pos, void* param)
{
    CV_Assert(param);
    int* status = (int*)param;
    status[0] = pos;
    status[1]++;
}

#if (!defined(ENABLE_PLUGINS) \
        && !defined HAVE_GTK \
        && !defined HAVE_QT \
        && !defined HAVE_WIN32UI \
        && !defined HAVE_WAYLAND \
    ) \
    || defined(__APPLE__) /* test fails on Mac (cocoa) */ \
    || defined HAVE_FRAMEBUFFER /* trackbar is not supported */
TEST(Highgui_GUI, DISABLED_trackbar)
#else
TEST(Highgui_GUI, trackbar)
#endif
{
    int status[2] = {-1, 0};  // pos, counter
    const std::string window_name("trackbar_test_window");
    const std::string trackbar_name("trackbar");

    EXPECT_NO_THROW(destroyAllWindows());
    ASSERT_NO_THROW(namedWindow(window_name));
    EXPECT_EQ((int)1, createTrackbar(trackbar_name, window_name, NULL, 100, testTrackbarCallback, status));
    EXPECT_EQ(0, getTrackbarPos(trackbar_name, window_name));
    int callback_count = status[1];
    EXPECT_GE(callback_count, 0);
    EXPECT_LE(callback_count, 1);
    int callback_count_base = callback_count;
    EXPECT_NO_THROW(setTrackbarPos(trackbar_name, window_name, 90));
    callback_count = status[1];
    EXPECT_EQ(callback_count_base + 1, callback_count);
    int value = status[0];
    EXPECT_EQ(90, value);
    EXPECT_EQ(90, getTrackbarPos(trackbar_name, window_name));
    EXPECT_NO_THROW(destroyAllWindows());
}

// See https://github.com/opencv/opencv/issues/25560
#if (!defined(ENABLE_PLUGINS) \
        && !defined HAVE_GTK \
        && !defined HAVE_QT \
        && !defined HAVE_WIN32UI \
        && !defined HAVE_WAYLAND)
TEST(Highgui_GUI, DISABLED_small_width_image)
#else
TEST(Highgui_GUI, small_width_image)
#endif
{
    const std::string window_name("trackbar_test_window");
    cv::Mat src(1,1,CV_8UC3,cv::Scalar(0));
    EXPECT_NO_THROW(destroyAllWindows());
    ASSERT_NO_THROW(namedWindow(window_name));
    ASSERT_NO_THROW(imshow(window_name, src));
    EXPECT_NO_THROW(waitKey(10));
    EXPECT_NO_THROW(destroyAllWindows());
}

TEST(Highgui_GUI, currentUIFramework)
{
    auto framework = currentUIFramework();
    std::cout << "UI framework: \"" << framework << "\"" << std::endl;
#if (!defined(ENABLE_PLUGINS) \
        && !defined HAVE_GTK \
        && !defined HAVE_QT \
        && !defined HAVE_WIN32UI \
        && !defined HAVE_COCOA \
        && !defined HAVE_WAYLAND \
    )
    EXPECT_TRUE(framework.empty());
#elif !defined(ENABLE_PLUGINS)
    EXPECT_GT(framework.size(), 0);  // builtin backends
#endif
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


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `test_precomp.hpp`

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

