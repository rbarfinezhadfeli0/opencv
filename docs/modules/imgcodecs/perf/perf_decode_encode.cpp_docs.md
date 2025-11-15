# Documentation for `modules/imgcodecs/perf/perf_decode_encode.cpp`

## File Metadata

- **Full Path**: `modules/imgcodecs/perf/perf_decode_encode.cpp`
- **File Name**: `perf_decode_encode.cpp`
- **File Size**: 7,160 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgcodecs/perf/perf_decode_encode.cpp](../../../modules/imgcodecs/perf/perf_decode_encode.cpp)

## Purpose and Role

This file is located in the `modules/imgcodecs/perf` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html

#include "perf_precomp.hpp"

namespace opencv_test
{

using namespace perf;

static Animation makeCirclesAnimation(Size size = Size(320, 240), int type = CV_8UC4, int nbits = 8, int frameCount = 40)
{
    struct AnimatedCircle {
        cv::Point2f pos;
        cv::Point2f velocity;
        float radius;
        float radius_speed;
        cv::Scalar color;
        cv::Scalar border_color;
    };

    const int numCircles = 80;
    const int maxval = (1 << nbits) - 1;

    cv::RNG rng = theRNG();
    std::vector<AnimatedCircle> circles;
    Animation animation;

    // Initialize animated circles
    for (int i = 0; i < numCircles; ++i) {
        AnimatedCircle c;
        c.pos = cv::Point2f(rng.uniform(0.f, (float)size.width),
            rng.uniform(0.f, (float)size.height));
        c.velocity = cv::Point2f(rng.uniform(-2.f, 2.f),
            rng.uniform(-2.f, 2.f));
        c.radius = rng.uniform(10.f, 40.f);
        c.radius_speed = rng.uniform(-0.5f, 0.5f);
        c.color = cv::Scalar(rng.uniform(0, maxval),
            rng.uniform(0, maxval),
            rng.uniform(0, maxval),
            rng.uniform(230, maxval));
        c.border_color = c.color;
        circles.push_back(c);
    }

    // Generate frames
    for (int frame = 0; frame < frameCount; ++frame) {
        cv::Mat img(size, type, cv::Scalar(20, 0, 10, 128));

        for (size_t i = 0; i < circles.size(); ++i) {
            AnimatedCircle& c = circles[i];

            // Update position
            c.pos += c.velocity;

            // Bounce on edges
            if (c.pos.x < 0 || c.pos.x > size.width) c.velocity.x *= -1;
            if (c.pos.y < 0 || c.pos.y > size.height) c.velocity.y *= -1;

            // Update radius
            c.radius += c.radius_speed;
            if (c.radius < 10.f || c.radius > 80.f) {
                c.radius_speed *= -1;
                c.radius = std::max(10.f, std::min(c.radius, 80.f));
            }

            c.color = c.color - Scalar(c.velocity.x, 0, c.velocity.y, rng.uniform(1, 4));

            // Draw
            cv::circle(img, c.pos, (int)c.radius, c.color, cv::FILLED, cv::LINE_AA);
            cv::circle(img, c.pos, (int)c.radius, c.border_color, 1, cv::LINE_AA);
        }

        animation.frames.push_back(img);
        animation.durations.push_back(20); // milliseconds
    }

    for (int i = (int)animation.frames.size() - 1; i >= 0; --i) {
        animation.frames.push_back(animation.frames[i].clone());
        animation.durations.push_back(15);
    }
    return animation;
}

typedef perf::TestBaseWithParam<std::string> Decode;
typedef perf::TestBaseWithParam<std::string> Encode;

const string exts[] = {
#ifdef HAVE_AVIF
    ".avif",
#endif
    ".bmp",
#ifdef HAVE_IMGCODEC_GIF
    ".gif",
#endif
#if (defined(HAVE_JASPER) && defined(OPENCV_IMGCODECS_ENABLE_JASPER_TESTS)) \
    || defined(HAVE_OPENJPEG)
    ".jp2",
#endif
#ifdef HAVE_JPEG
    ".jpg",
#endif
#ifdef HAVE_JPEGXL
    ".jxl",
#endif
#ifdef HAVE_PNG
    ".png",
#endif
#ifdef HAVE_IMGCODEC_PXM
    ".ppm",
#endif
#ifdef HAVE_IMGCODEC_SUNRASTER
    ".ras",
#endif
#ifdef HAVE_TIFF
    ".tiff",
#endif
#ifdef HAVE_WEBP
    ".webp",
#endif
};

const string exts_multi[] = {
#ifdef HAVE_AVIF
    ".avif",
#endif
#ifdef HAVE_IMGCODEC_GIF
    ".gif",
#endif
#ifdef HAVE_PNG
    ".png",
#endif
#ifdef HAVE_TIFF
    ".tiff",
#endif
#ifdef HAVE_WEBP
    ".webp",
#endif
};

const string exts_anim[] = {
#ifdef HAVE_AVIF
    ".avif",
#endif
#ifdef HAVE_IMGCODEC_GIF
    ".gif",
#endif
#ifdef HAVE_PNG
    ".png",
#endif
#ifdef HAVE_WEBP
    ".webp",
#endif
};

#ifdef HAVE_PNG

PERF_TEST_P(Decode, bgr, testing::ValuesIn(exts))
{
    String filename = getDataPath("perf/1920x1080.png");

    Mat src = imread(filename);
    EXPECT_FALSE(src.empty()) << "Cannot open test image perf/1920x1080.png";
    vector<uchar> buf;
    EXPECT_TRUE(imencode(GetParam(), src, buf));

    TEST_CYCLE() imdecode(buf, IMREAD_UNCHANGED);

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(Decode, rgb, testing::ValuesIn(exts))
{
    String filename = getDataPath("perf/1920x1080.png");

    Mat src = imread(filename);
    EXPECT_FALSE(src.empty()) << "Cannot open test image perf/1920x1080.png";
    vector<uchar> buf;
    EXPECT_TRUE(imencode(GetParam(), src, buf));

    TEST_CYCLE() imdecode(buf, IMREAD_COLOR_RGB);

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(Encode, bgr, testing::ValuesIn(exts))
{
    String filename = getDataPath("perf/1920x1080.png");

    Mat src = imread(filename);
    EXPECT_FALSE(src.empty()) << "Cannot open test image perf/1920x1080.png";
    vector<uchar> buf;

    TEST_CYCLE() imencode(GetParam(), src, buf);

    std::cout << "  Encoded buffer size: " << buf.size()
        << " bytes, Compression ratio: " << std::fixed << std::setprecision(2)
        << (static_cast<double>(buf.size()) / (src.total() * src.channels())) * 100.0 << "%" << std::endl;

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(Encode, multi, testing::ValuesIn(exts_multi))
{
    String filename = getDataPath("perf/1920x1080.png");
    vector<Mat> vec;
    EXPECT_TRUE(imreadmulti(filename, vec));
    vec.push_back(vec.back().clone());
    circle(vec.back(), Point(100, 100), 45, Scalar(0, 0, 255, 0), 2, LINE_AA);
    vector<uchar> buf;
    EXPECT_TRUE(imwrite("test" + GetParam(), vec));

    TEST_CYCLE() imencode(GetParam(), vec, buf);

    std::cout << "  Encoded buffer size: " << buf.size()
        << " bytes, Compression ratio: " << std::fixed << std::setprecision(2)
        << (static_cast<double>(buf.size()) / (vec[0].total() * vec[0].channels())) * 100.0 << "%" << std::endl;

    SANITY_CHECK_NOTHING();
}

#endif // HAVE_PNG

PERF_TEST_P(Encode, animation, testing::ValuesIn(exts_anim))
{
    Animation animation = makeCirclesAnimation();

    TEST_CYCLE()
    {
        vector<uchar> buf;
        EXPECT_TRUE(imencodeanimation(GetParam().c_str(), animation, buf));
    }

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(Encode, multi_page, testing::ValuesIn(exts_multi))
{
    Animation animation = makeCirclesAnimation();

    TEST_CYCLE()
    {
        vector<uchar> buf;
        EXPECT_TRUE(imencodemulti(GetParam().c_str(), animation.frames, buf));
    }

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(Decode, animation, testing::ValuesIn(exts_anim))
{
    Animation animation = makeCirclesAnimation();
    vector<uchar> buf;
    ASSERT_TRUE(imencodeanimation(GetParam().c_str(), animation, buf));

    TEST_CYCLE()
    {
        Animation tmp_animation;
        EXPECT_TRUE(imdecodeanimation(buf, tmp_animation));
    }

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(Decode, multi_page, testing::ValuesIn(exts_multi))
{
    Animation animation = makeCirclesAnimation();
    vector<uchar> buf;
    ASSERT_TRUE(imencodemulti(GetParam().c_str(), animation.frames, buf));

    TEST_CYCLE()
    {
        vector<Mat> tmp_frames;
        EXPECT_TRUE(imdecodemulti(buf, IMREAD_UNCHANGED, tmp_frames));
    }

    SANITY_CHECK_NOTHING();
}

} // namespace
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

- **AnimatedCircle**: A class/struct defined in this file

### Functions and Methods

- **HAVE_IMGCODEC_GIF()**: A function/method defined in this file
- **HAVE_JPEGXL()**: A function/method defined in this file
- **HAVE_JPEG()**: A function/method defined in this file
- **HAVE_PNG()**: A function/method defined in this file
- **perf()**: A function/method defined in this file
- **HAVE_AVIF()**: A function/method defined in this file
- **HAVE_IMGCODEC_SUNRASTER()**: A function/method defined in this file
- **HAVE_WEBP()**: A function/method defined in this file
- **HAVE_TIFF()**: A function/method defined in this file
- **HAVE_IMGCODEC_PXM()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `perf_precomp.hpp`


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

