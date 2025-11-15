# Documentation for `modules/objdetect/test/test_barcode.cpp`

## File Metadata

- **Full Path**: `modules/objdetect/test/test_barcode.cpp`
- **File Name**: `test_barcode.cpp`
- **File Size**: 6,844 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/objdetect/test/test_barcode.cpp](../../../modules/objdetect/test/test_barcode.cpp)

## Purpose and Role

This file is located in the `modules/objdetect/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "test_precomp.hpp"
#include "opencv2/objdetect/barcode.hpp"
#include <set>

using namespace std;

namespace opencv_test{namespace{

typedef std::set<string> StringSet;

// Convert ';'-separated strings to a set
inline static StringSet toSet(const string &line)
{
    StringSet res;
    string::size_type it = 0, ti;
    while (true)
    {
        ti = line.find(';', it);
        if (ti == string::npos)
        {
            res.insert(string(line, it, line.size() - it));
            break;
        }
        res.insert(string(line, it, ti - it));
        it = ti + 1;
    }
    return res;
}

// Convert vector of strings to a set
inline static StringSet toSet(const vector<string> &lines)
{
    StringSet res;
    for (const string & line : lines)
        res.insert(line);
    return res;
}

// Get all keys of a map in a vector
template<typename T, typename V>
inline static vector<T> getKeys(const map<T, V> &m)
{
    vector<T> res;
    for (const auto & it : m)
        res.push_back(it.first);
    return res;
}

struct BarcodeResult
{
    string type;
    string data;
};

map<string, BarcodeResult> testResults {
    { "single/book.jpg", {"EAN_13", "9787115279460"} },
    { "single/bottle_1.jpg", {"EAN_13", "6922255451427"} },
    { "single/bottle_2.jpg", {"EAN_13", "6921168509256"} },
    { "multiple/4_barcodes.jpg", {"EAN_13;EAN_13;EAN_13;EAN_13", "9787564350840;9783319200064;9787118081473;9787122276124"} },
};

typedef testing::TestWithParam< string > BarcodeDetector_main;

TEST_P(BarcodeDetector_main, interface)
{
    const string fname = GetParam();
    const string image_path = findDataFile(string("barcode/") + fname);
    const StringSet expected_lines = toSet(testResults[fname].data);
    const StringSet expected_types = toSet(testResults[fname].type);
    const size_t expected_count = expected_lines.size(); // assume codes are unique
    // TODO: verify points location

    Mat img = imread(image_path);
    ASSERT_FALSE(img.empty()) << "Can't read image: " << image_path;

    barcode::BarcodeDetector det;
    vector<Point2f> points;
    vector<string> types;
    vector<string> lines;

    // common interface (single)
    {
        bool res = det.detect(img, points);
        ASSERT_TRUE(res);
        EXPECT_EQ(expected_count * 4, points.size());
    }

    {
        string res = det.decode(img, points);
        ASSERT_FALSE(res.empty());
        EXPECT_EQ(1u, expected_lines.count(res));
    }

    {
        string res = det.detectAndDecode(img, points);
        ASSERT_FALSE(res.empty());
        EXPECT_EQ(1u, expected_lines.count(res));
        EXPECT_EQ(4u, points.size());
    }

    // common interface (multi)
    {
        bool res = det.detectMulti(img, points);
        ASSERT_TRUE(res);
        EXPECT_EQ(expected_count * 4, points.size());
    }

    {
        bool res = det.decodeMulti(img, points, lines);
        ASSERT_TRUE(res);
        EXPECT_EQ(expected_lines, toSet(lines));
    }

    // specific interface
    {
        bool res = det.decodeWithType(img, points, lines, types);
        ASSERT_TRUE(res);
        EXPECT_EQ(expected_types, toSet(types));
        EXPECT_EQ(expected_lines, toSet(lines));
    }

    {
        bool res = det.detectAndDecodeWithType(img, lines, types, points);
        ASSERT_TRUE(res);
        EXPECT_EQ(expected_types, toSet(types));
        EXPECT_EQ(expected_lines, toSet(lines));
    }
}

INSTANTIATE_TEST_CASE_P(/**/, BarcodeDetector_main, testing::ValuesIn(getKeys(testResults)));

TEST(BarcodeDetector_base, invalid)
{
    auto bardet = barcode::BarcodeDetector();
    std::vector<Point> corners;
    vector<cv::String> decoded_info;
    Mat zero_image = Mat::zeros(256, 256, CV_8UC1);
    EXPECT_FALSE(bardet.detectMulti(zero_image, corners));
    corners = std::vector<Point>(4);
    EXPECT_ANY_THROW(bardet.decodeMulti(zero_image, corners, decoded_info));
}

struct ParamStruct
{
    double down_thresh;
    vector<float> scales;
    double grad_thresh;
    unsigned res_count;
};

inline static std::ostream &operator<<(std::ostream &out, const ParamStruct &p)
{
    out << "(" << p.down_thresh << ", ";
    for(float val : p.scales)
        out << val << ", ";
    out << p.grad_thresh << ")";
    return out;
}

ParamStruct param_list[] = {
    { 512, {0.01f, 0.03f, 0.06f, 0.08f}, 64, 4 }, // default values -> 4 codes
    { 512, {0.01f, 0.03f, 0.06f, 0.08f}, 1024, 2 },
    { 512, {0.01f, 0.03f, 0.06f, 0.08f}, 2048, 0 },
    { 128, {0.01f, 0.03f, 0.06f, 0.08f}, 64, 3 },
    { 64, {0.01f, 0.03f, 0.06f, 0.08f}, 64, 2 },
    { 128, {0.0000001f}, 64, 1 },
    { 128, {0.0000001f, 0.0001f}, 64, 1 },
    { 128, {0.0000001f, 0.1f}, 64, 1 },
    { 512, {0.1f}, 64, 0 },
};

typedef testing::TestWithParam<ParamStruct> BarcodeDetector_parameters_tune;

TEST_P(BarcodeDetector_parameters_tune, accuracy)
{
    const ParamStruct param = GetParam();

    const string fname = "multiple/4_barcodes.jpg";
    const string image_path = findDataFile(string("barcode/") + fname);

    const Mat img = imread(image_path);
    ASSERT_FALSE(img.empty()) << "Can't read image: " << image_path;

    auto bardet = barcode::BarcodeDetector();
    bardet.setDownsamplingThreshold(param.down_thresh);
    bardet.setDetectorScales(param.scales);
    bardet.setGradientThreshold(param.grad_thresh);
    vector<Point2f> points;
    bardet.detectMulti(img, points);
    EXPECT_EQ(points.size() / 4, param.res_count);
}

INSTANTIATE_TEST_CASE_P(/**/, BarcodeDetector_parameters_tune, testing::ValuesIn(param_list));

TEST(BarcodeDetector_parameters, regression)
{
    const double expected_dt = 1024, expected_gt = 256;
    const vector<float> expected_ds = {0.1f};
    vector<float> ds_value = {0.0f};

    auto bardet = barcode::BarcodeDetector();

    bardet.setDownsamplingThreshold(expected_dt).setDetectorScales(expected_ds).setGradientThreshold(expected_gt);

    double dt_value = bardet.getDownsamplingThreshold();
    bardet.getDetectorScales(ds_value);
    double gt_value = bardet.getGradientThreshold();

    EXPECT_EQ(expected_dt, dt_value);
    EXPECT_EQ(expected_ds, ds_value);
    EXPECT_EQ(expected_gt, gt_value);
}

TEST(BarcodeDetector_parameters, invalid)
{
    auto bardet = barcode::BarcodeDetector();

    EXPECT_ANY_THROW(bardet.setDownsamplingThreshold(-1));
    EXPECT_ANY_THROW(bardet.setDetectorScales(vector<float> {}));
    EXPECT_ANY_THROW(bardet.setDetectorScales(vector<float> {-1}));
    EXPECT_ANY_THROW(bardet.setDetectorScales(vector<float> {1.5}));
    EXPECT_ANY_THROW(bardet.setDetectorScales(vector<float> (17, 0.5)));
    EXPECT_ANY_THROW(bardet.setGradientThreshold(-0.1));
}

}} // opencv_test::<anonymous>::
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

- **BarcodeResult**: A class/struct defined in this file
- **ParamStruct**: A class/struct defined in this file

### Functions and Methods

- **std()**: A function/method defined in this file
- **testing()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `set`
- `opencv2/objdetect/barcode.hpp`
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

