# Documentation for `modules/imgcodecs/perf/perf_png.cpp`

## File Metadata

- **Full Path**: `modules/imgcodecs/perf/perf_png.cpp`
- **File Name**: `perf_png.cpp`
- **File Size**: 3,072 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgcodecs/perf/perf_png.cpp](../../../modules/imgcodecs/perf/perf_png.cpp)

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

#if defined(HAVE_PNG) || defined(HAVE_SPNG)

using namespace perf;

CV_ENUM(PNGStrategy, IMWRITE_PNG_STRATEGY_DEFAULT, IMWRITE_PNG_STRATEGY_FILTERED, IMWRITE_PNG_STRATEGY_HUFFMAN_ONLY, IMWRITE_PNG_STRATEGY_RLE, IMWRITE_PNG_STRATEGY_FIXED);
CV_ENUM(PNGFilters, IMWRITE_PNG_FILTER_NONE, IMWRITE_PNG_FILTER_SUB, IMWRITE_PNG_FILTER_UP, IMWRITE_PNG_FILTER_AVG, IMWRITE_PNG_FILTER_PAETH, IMWRITE_PNG_FAST_FILTERS, IMWRITE_PNG_ALL_FILTERS);

typedef perf::TestBaseWithParam<testing::tuple<PNGStrategy, PNGFilters, int>> PNG;

PERF_TEST(PNG, decode)
{
    String filename = getDataPath("perf/2560x1600.png");

    FILE *f = fopen(filename.c_str(), "rb");
    fseek(f, 0, SEEK_END);
    long len = ftell(f);
    fseek(f, 0, SEEK_SET);
    vector<uchar> file_buf((size_t)len);
    EXPECT_EQ(len, (long)fread(&file_buf[0], 1, (size_t)len, f));
    fclose(f); f = NULL;

    TEST_CYCLE() imdecode(file_buf, IMREAD_UNCHANGED);

    SANITY_CHECK_NOTHING();
}

PERF_TEST(PNG, decode_rgb)
{
    String filename = getDataPath("perf/2560x1600.png");

    FILE *f = fopen(filename.c_str(), "rb");
    fseek(f, 0, SEEK_END);
    long len = ftell(f);
    fseek(f, 0, SEEK_SET);
    vector<uchar> file_buf((size_t)len);
    EXPECT_EQ(len, (long)fread(&file_buf[0], 1, (size_t)len, f));
    fclose(f); f = NULL;

    TEST_CYCLE() imdecode(file_buf, IMREAD_COLOR_RGB);

    SANITY_CHECK_NOTHING();
}

PERF_TEST(PNG, encode)
{
    String filename = getDataPath("perf/2560x1600.png");
    cv::Mat src = imread(filename);

    vector<uchar> buf;
    TEST_CYCLE() imencode(".png", src, buf);

    SANITY_CHECK_NOTHING();
}

PERF_TEST_P(PNG, params,
    testing::Combine(
        testing::Values(IMWRITE_PNG_STRATEGY_DEFAULT, IMWRITE_PNG_STRATEGY_FILTERED, IMWRITE_PNG_STRATEGY_HUFFMAN_ONLY, IMWRITE_PNG_STRATEGY_RLE, IMWRITE_PNG_STRATEGY_FIXED),
        testing::Values(IMWRITE_PNG_FILTER_NONE, IMWRITE_PNG_FILTER_SUB, IMWRITE_PNG_FILTER_UP, IMWRITE_PNG_FILTER_AVG, IMWRITE_PNG_FILTER_PAETH, IMWRITE_PNG_FAST_FILTERS, IMWRITE_PNG_ALL_FILTERS),
        testing::Values(1, 6)))
{
    String filename = getDataPath("perf/1920x1080.png");
    const int strategy = get<0>(GetParam());
    const int filter = get<1>(GetParam());
    const int level = get<2>(GetParam());

    Mat src = imread(filename);
    EXPECT_FALSE(src.empty()) << "Cannot open test image perf/1920x1080.png";
    vector<uchar> buf;

    TEST_CYCLE() imencode(".png", src, buf, { IMWRITE_PNG_COMPRESSION, level, IMWRITE_PNG_STRATEGY, strategy, IMWRITE_PNG_FILTER, filter });

    std::cout << "  Encoded buffer size: " << buf.size()
        << " bytes, Compression ratio: " << std::fixed << std::setprecision(2)
        << (static_cast<double>(buf.size()) / (src.total() * src.channels())) * 100.0 << "%" << std::endl;

    SANITY_CHECK_NOTHING();
}

#endif // HAVE_PNG

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

### Functions and Methods

- **perf()**: A function/method defined in this file


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

