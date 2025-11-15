# Documentation for `modules/imgcodecs/test/test_jpeg.cpp`

## File Metadata

- **Full Path**: `modules/imgcodecs/test/test_jpeg.cpp`
- **File Name**: `test_jpeg.cpp`
- **File Size**: 9,647 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgcodecs/test/test_jpeg.cpp](../../../modules/imgcodecs/test/test_jpeg.cpp)

## Purpose and Role

This file is located in the `modules/imgcodecs/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html
#include "test_precomp.hpp"

namespace opencv_test { namespace {

#ifdef HAVE_JPEG

extern "C" {
#include "jpeglib.h"
}

TEST(Imgcodecs_Jpeg, encode_empty)
{
    cv::Mat img;
    std::vector<uchar> jpegImg;
    ASSERT_THROW(cv::imencode(".jpg", img, jpegImg), cv::Exception);
}

TEST(Imgcodecs_Jpeg, encode_decode_progressive_jpeg)
{
    cvtest::TS& ts = *cvtest::TS::ptr();
    string input = string(ts.get_data_path()) + "../cv/shared/lena.png";
    cv::Mat img = cv::imread(input);
    ASSERT_FALSE(img.empty());

    std::vector<int> params;
    params.push_back(IMWRITE_JPEG_PROGRESSIVE);
    params.push_back(1);

    string output_progressive = cv::tempfile(".jpg");
    EXPECT_NO_THROW(cv::imwrite(output_progressive, img, params));
    cv::Mat img_jpg_progressive = cv::imread(output_progressive);

    string output_normal = cv::tempfile(".jpg");
    EXPECT_NO_THROW(cv::imwrite(output_normal, img));
    cv::Mat img_jpg_normal = cv::imread(output_normal);

    EXPECT_EQ(0, cvtest::norm(img_jpg_progressive, img_jpg_normal, NORM_INF));

    EXPECT_EQ(0, remove(output_progressive.c_str()));
    EXPECT_EQ(0, remove(output_normal.c_str()));
}

TEST(Imgcodecs_Jpeg, encode_decode_optimize_jpeg)
{
    cvtest::TS& ts = *cvtest::TS::ptr();
    string input = string(ts.get_data_path()) + "../cv/shared/lena.png";
    cv::Mat img = cv::imread(input);
    ASSERT_FALSE(img.empty());

    std::vector<int> params;
    params.push_back(IMWRITE_JPEG_OPTIMIZE);
    params.push_back(1);

    string output_optimized = cv::tempfile(".jpg");
    EXPECT_NO_THROW(cv::imwrite(output_optimized, img, params));
    cv::Mat img_jpg_optimized = cv::imread(output_optimized);

    string output_normal = cv::tempfile(".jpg");
    EXPECT_NO_THROW(cv::imwrite(output_normal, img));
    cv::Mat img_jpg_normal = cv::imread(output_normal);

    EXPECT_EQ(0, cvtest::norm(img_jpg_optimized, img_jpg_normal, NORM_INF));

    EXPECT_EQ(0, remove(output_optimized.c_str()));
    EXPECT_EQ(0, remove(output_normal.c_str()));
}

TEST(Imgcodecs_Jpeg, encode_decode_rst_jpeg)
{
    cvtest::TS& ts = *cvtest::TS::ptr();
    string input = string(ts.get_data_path()) + "../cv/shared/lena.png";
    cv::Mat img = cv::imread(input);
    ASSERT_FALSE(img.empty());

    std::vector<int> params;
    params.push_back(IMWRITE_JPEG_RST_INTERVAL);
    params.push_back(1);

    string output_rst = cv::tempfile(".jpg");
    EXPECT_NO_THROW(cv::imwrite(output_rst, img, params));
    cv::Mat img_jpg_rst = cv::imread(output_rst);

    string output_normal = cv::tempfile(".jpg");
    EXPECT_NO_THROW(cv::imwrite(output_normal, img));
    cv::Mat img_jpg_normal = cv::imread(output_normal);

    EXPECT_EQ(0, cvtest::norm(img_jpg_rst, img_jpg_normal, NORM_INF));

    EXPECT_EQ(0, remove(output_rst.c_str()));
    EXPECT_EQ(0, remove(output_normal.c_str()));
}

// See https://github.com/opencv/opencv/issues/25274
typedef testing::TestWithParam<int> Imgcodecs_Jpeg_decode_cmyk;
TEST_P(Imgcodecs_Jpeg_decode_cmyk, regression25274)
{
    const int imread_flag = GetParam();

    /*
     * "test_1_c4.jpg" is CMYK-JPEG.
     * $ convert test_1_c3.jpg -colorspace CMYK test_1_c4.jpg
     * $ identify test_1_c4.jpg
     * test_1_c4.jpg JPEG 480x640 480x640+0+0 8-bit CMYK 11240B 0.000u 0:00.000
     */

    cvtest::TS& ts = *cvtest::TS::ptr();

    string  rgb_filename  = string(ts.get_data_path()) + "readwrite/test_1_c3.jpg";
    cv::Mat rgb_img       = cv::imread(rgb_filename, imread_flag);
    ASSERT_FALSE(rgb_img.empty());

    string  cmyk_filename = string(ts.get_data_path()) + "readwrite/test_1_c4.jpg";
    cv::Mat cmyk_img      = cv::imread(cmyk_filename, imread_flag);
    ASSERT_FALSE(cmyk_img.empty());

    EXPECT_EQ(rgb_img.size(), cmyk_img.size());
    EXPECT_EQ(rgb_img.type(), cmyk_img.type());

    // Jpeg is lossy compression.
    // There may be small differences in decoding results by environments.
    // -> 255 * 1% = 2.55 .
    EXPECT_LE(cvtest::norm(rgb_img, cmyk_img, NORM_INF), 3); // norm() <= 3
}

INSTANTIATE_TEST_CASE_P( /* nothing */,
                        Imgcodecs_Jpeg_decode_cmyk,
                        testing::Values(cv::IMREAD_COLOR,
                                        cv::IMREAD_COLOR_RGB,
                                        cv::IMREAD_GRAYSCALE,
                                        cv::IMREAD_ANYCOLOR));

//==================================================================================================

static const uint32_t default_sampling_factor = static_cast<uint32_t>(0x221111);

static uint32_t test_jpeg_subsampling( const Mat src, const vector<int> param )
{
    vector<uint8_t> jpeg;

    if ( cv::imencode(".jpg", src, jpeg, param ) == false )
    {
        return 0;
    }

    if ( src.channels() != 3 )
    {
        return 0;
    }

    // Find SOF Marker(FFC0)
    int sof_offset = 0; // not found.
    int jpeg_size = static_cast<int>( jpeg.size() );
    for ( int i = 0 ; i < jpeg_size - 1; i++ )
    {
        if ( (jpeg[i] == 0xff ) && ( jpeg[i+1] == 0xC0 ) )
        {
            sof_offset = i;
            break;
        }
    }
    if ( sof_offset == 0 )
    {
        return 0;
    }

    // Extract Subsampling Factor from SOF.
    return ( jpeg[sof_offset + 0x0A + 3 * 0 + 1] << 16 ) +
           ( jpeg[sof_offset + 0x0A + 3 * 1 + 1] << 8  ) +
           ( jpeg[sof_offset + 0x0A + 3 * 2 + 1]       ) ;
}

TEST(Imgcodecs_Jpeg, encode_subsamplingfactor_default)
{
    vector<int> param;
    Mat src( 48, 64, CV_8UC3, cv::Scalar::all(0) );
    EXPECT_EQ( default_sampling_factor, test_jpeg_subsampling(src, param) );
}

TEST(Imgcodecs_Jpeg, encode_subsamplingfactor_usersetting_valid)
{
    Mat src( 48, 64, CV_8UC3, cv::Scalar::all(0) );
    const uint32_t sampling_factor_list[] = {
        IMWRITE_JPEG_SAMPLING_FACTOR_411,
        IMWRITE_JPEG_SAMPLING_FACTOR_420,
        IMWRITE_JPEG_SAMPLING_FACTOR_422,
        IMWRITE_JPEG_SAMPLING_FACTOR_440,
        IMWRITE_JPEG_SAMPLING_FACTOR_444,
    };
    const int sampling_factor_list_num = 5;

    for ( int i = 0 ; i < sampling_factor_list_num; i ++ )
    {
        vector<int> param;
        param.push_back( IMWRITE_JPEG_SAMPLING_FACTOR );
        param.push_back( sampling_factor_list[i] );
        EXPECT_EQ( sampling_factor_list[i], test_jpeg_subsampling(src, param) );
    }
}

TEST(Imgcodecs_Jpeg, encode_subsamplingfactor_usersetting_invalid)
{
    Mat src( 48, 64, CV_8UC3, cv::Scalar::all(0) );
    const uint32_t sampling_factor_list[] = { // Invalid list
        0x111112,
        0x000000,
        0x001111,
        0xFF1111,
        0x141111, // 1x4,1x1,1x1 - unknown
        0x241111, // 2x4,1x1,1x1 - unknown
        0x421111, // 4x2,1x1,1x1 - unknown
        0x441111, // 4x4,1x1,1x1 - 410(libjpeg cannot handle it)
    };
    const int sampling_factor_list_num = 8;

    for ( int i = 0 ; i < sampling_factor_list_num; i ++ )
    {
        vector<int> param;
        param.push_back( IMWRITE_JPEG_SAMPLING_FACTOR );
        param.push_back( sampling_factor_list[i] );
        EXPECT_EQ( default_sampling_factor, test_jpeg_subsampling(src, param) );
    }
}

//==================================================================================================
// See https://github.com/opencv/opencv/issues/25646
typedef testing::TestWithParam<std::tuple<int, int>> Imgcodecs_Jpeg_encode_withLumaChromaQuality;

TEST_P(Imgcodecs_Jpeg_encode_withLumaChromaQuality, basic)
{
    const int luma   = get<0>(GetParam());
    const int chroma = get<1>(GetParam());

    cvtest::TS& ts = *cvtest::TS::ptr();
    string fname = string(ts.get_data_path()) + "../cv/shared/lena.png";

    cv::Mat src = imread(fname, cv::IMREAD_COLOR);
    ASSERT_FALSE(src.empty());

    // Add imread RGB test
    cv::Mat src_rgb = imread(fname, cv::IMREAD_COLOR_RGB);
    ASSERT_FALSE(src_rgb.empty());

    cvtColor(src_rgb, src_rgb, COLOR_RGB2BGR);
    EXPECT_TRUE(cvtest::norm(src, src_rgb, NORM_INF) == 0);

    std::vector<uint8_t> jpegNormal;
    ASSERT_NO_THROW(cv::imencode(".jpg", src, jpegNormal));

    std::vector<int> param;
    param.push_back(IMWRITE_JPEG_LUMA_QUALITY);
    param.push_back(luma);
    param.push_back(IMWRITE_JPEG_CHROMA_QUALITY);
    param.push_back(chroma);

    std::vector<uint8_t> jpegCustom;
    ASSERT_NO_THROW(cv::imencode(".jpg", src, jpegCustom, param));

#if JPEG_LIB_VERSION >= 70
    // For jpeg7+, we can support IMWRITE_JPEG_LUMA_QUALITY and IMWRITE_JPEG_CHROMA_QUALITY.
    if( (luma == 95 /* Default Luma Quality */ ) && ( chroma == 95 /* Default Chroma Quality */))
    {
        EXPECT_EQ(jpegNormal, jpegCustom);
    }
    else
    {
        EXPECT_NE(jpegNormal, jpegCustom);
    }
#else
    // For jpeg6-, we cannot support IMWRITE_JPEG_LUMA/CHROMA_QUALITY because jpeg_default_qtables() is missing.
    // - IMWRITE_JPEG_LUMA_QUALITY updates internal parameter of IMWRITE_JPEG_QUALITY.
    // - IMWRITE_JPEG_CHROMA_QUALITY updates nothing.
    if( luma == 95 /* Default Jpeg Quality */ )
    {
        EXPECT_EQ(jpegNormal, jpegCustom);
    }
    else
    {
        EXPECT_NE(jpegNormal, jpegCustom);
    }
#endif
}

INSTANTIATE_TEST_CASE_P( /* nothing */,
                        Imgcodecs_Jpeg_encode_withLumaChromaQuality,
                        testing::Combine(
                            testing::Values(70, 95, 100),    // IMWRITE_JPEG_LUMA_QUALITY
                            testing::Values(70, 95, 100) )); // IMWRITE_JPEG_CHROMA_QUALITY

#endif // HAVE_JPEG

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

### Functions and Methods

- **testing()**: A function/method defined in this file
- **HAVE_JPEG()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `jpeglib.h`
- `test_precomp.hpp`

**Python Imports:**
- `SOF.`


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

