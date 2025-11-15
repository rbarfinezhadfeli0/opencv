# Documentation for `modules/ts/include/opencv2/ts/cuda_test.hpp`

## File Metadata

- **Full Path**: `modules/ts/include/opencv2/ts/cuda_test.hpp`
- **File Name**: `cuda_test.hpp`
- **File Size**: 15,338 bytes
- **File Type**: .hpp
- **Link to Source**: [modules/ts/include/opencv2/ts/cuda_test.hpp](../../../../../modules/ts/include/opencv2/ts/cuda_test.hpp)

## Purpose and Role

This file is located in the `modules/ts/include/opencv2/ts` directory and serves as part of the OpenCV library infrastructure.

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

#ifndef OPENCV_CUDA_TEST_UTILITY_HPP
#define OPENCV_CUDA_TEST_UTILITY_HPP

#include "opencv2/ts.hpp"

#include <stdexcept>
#include "opencv2/core/cuda.hpp"

namespace cvtest
{
    //////////////////////////////////////////////////////////////////////
    // random generators

    int randomInt(int minVal, int maxVal);
    double randomDouble(double minVal, double maxVal);
    cv::Size randomSize(int minVal, int maxVal);
    cv::Scalar randomScalar(double minVal, double maxVal);
    cv::Mat randomMat(cv::Size size, int type, double minVal = 0.0, double maxVal = 255.0);

    //////////////////////////////////////////////////////////////////////
    // GpuMat create

    cv::cuda::GpuMat createMat(cv::Size size, int type, bool useRoi = false);
    cv::cuda::GpuMat createMat(cv::Size size, int type, cv::Size& size0, cv::Point& ofs, bool useRoi = false);
    cv::cuda::GpuMat loadMat(const cv::Mat& m, bool useRoi = false);

    //////////////////////////////////////////////////////////////////////
    // Image load

    //! read image from testdata folder
    cv::Mat readImage(const std::string& fileName, int flags = cv::IMREAD_COLOR);

    //! read image from testdata folder and convert it to specified type
    cv::Mat readImageType(const std::string& fname, int type);

    //////////////////////////////////////////////////////////////////////
    // Gpu devices

    //! return true if device supports specified feature and gpu module was built with support the feature.
    bool supportFeature(const cv::cuda::DeviceInfo& info, cv::cuda::FeatureSet feature);

    class DeviceManager
    {
    public:
        static DeviceManager& instance();

        void load(int i);
        void loadAll();

        const std::vector<cv::cuda::DeviceInfo>& values() const { return devices_; }

    private:
        std::vector<cv::cuda::DeviceInfo> devices_;
    };

    #define ALL_DEVICES testing::ValuesIn(cvtest::DeviceManager::instance().values())

    //////////////////////////////////////////////////////////////////////
    // Additional assertion

    void minMaxLocGold(const cv::Mat& src, double* minVal_, double* maxVal_ = 0, cv::Point* minLoc_ = 0, cv::Point* maxLoc_ = 0, const cv::Mat& mask = cv::Mat());

    cv::Mat getMat(cv::InputArray arr);

    testing::AssertionResult assertMatNear(const char* expr1, const char* expr2, const char* eps_expr, cv::InputArray m1, cv::InputArray m2, double eps);

    #undef EXPECT_MAT_NEAR
    #define EXPECT_MAT_NEAR(m1, m2, eps) EXPECT_PRED_FORMAT3(cvtest::assertMatNear, m1, m2, eps)
    #define ASSERT_MAT_NEAR(m1, m2, eps) ASSERT_PRED_FORMAT3(cvtest::assertMatNear, m1, m2, eps)

    #define EXPECT_SCALAR_NEAR(s1, s2, eps) \
        { \
            EXPECT_NEAR(s1[0], s2[0], eps); \
            EXPECT_NEAR(s1[1], s2[1], eps); \
            EXPECT_NEAR(s1[2], s2[2], eps); \
            EXPECT_NEAR(s1[3], s2[3], eps); \
        }
    #define ASSERT_SCALAR_NEAR(s1, s2, eps) \
        { \
            ASSERT_NEAR(s1[0], s2[0], eps); \
            ASSERT_NEAR(s1[1], s2[1], eps); \
            ASSERT_NEAR(s1[2], s2[2], eps); \
            ASSERT_NEAR(s1[3], s2[3], eps); \
        }

    #define EXPECT_POINT2_NEAR(p1, p2, eps) \
        { \
            EXPECT_NEAR(p1.x, p2.x, eps); \
            EXPECT_NEAR(p1.y, p2.y, eps); \
        }
    #define ASSERT_POINT2_NEAR(p1, p2, eps) \
        { \
            ASSERT_NEAR(p1.x, p2.x, eps); \
            ASSERT_NEAR(p1.y, p2.y, eps); \
        }

    #define EXPECT_POINT3_NEAR(p1, p2, eps) \
        { \
            EXPECT_NEAR(p1.x, p2.x, eps); \
            EXPECT_NEAR(p1.y, p2.y, eps); \
            EXPECT_NEAR(p1.z, p2.z, eps); \
        }
    #define ASSERT_POINT3_NEAR(p1, p2, eps) \
        { \
            ASSERT_NEAR(p1.x, p2.x, eps); \
            ASSERT_NEAR(p1.y, p2.y, eps); \
            ASSERT_NEAR(p1.z, p2.z, eps); \
        }

    double checkSimilarity(cv::InputArray m1, cv::InputArray m2);

    #undef EXPECT_MAT_SIMILAR
    #define EXPECT_MAT_SIMILAR(mat1, mat2, eps) \
        { \
            ASSERT_EQ(mat1.type(), mat2.type()); \
            ASSERT_EQ(mat1.size(), mat2.size()); \
            EXPECT_LE(checkSimilarity(mat1, mat2), eps); \
        }
    #define ASSERT_MAT_SIMILAR(mat1, mat2, eps) \
        { \
            ASSERT_EQ(mat1.type(), mat2.type()); \
            ASSERT_EQ(mat1.size(), mat2.size()); \
            ASSERT_LE(checkSimilarity(mat1, mat2), eps); \
        }

    //////////////////////////////////////////////////////////////////////
    // Helper structs for value-parameterized tests

    #define CUDA_TEST_P(test_case_name, test_name) \
      class GTEST_TEST_CLASS_NAME_(test_case_name, test_name) \
          : public test_case_name { \
       public: \
        GTEST_TEST_CLASS_NAME_(test_case_name, test_name)() {} \
        virtual void TestBody(); \
       private: \
        void UnsafeTestBody(); \
        static int AddToRegistry() { \
          ::testing::UnitTest::GetInstance()->parameterized_test_registry(). \
              GetTestCasePatternHolder<test_case_name>(\
                  #test_case_name, \
                  ::testing::internal::CodeLocation(\
                      __FILE__, __LINE__))->AddTestPattern(\
                          #test_case_name, \
                          #test_name, \
                          new ::testing::internal::TestMetaFactory< \
                              GTEST_TEST_CLASS_NAME_(\
                                  test_case_name, test_name)>()); \
          return 0; \
        } \
        static int gtest_registering_dummy_ GTEST_ATTRIBUTE_UNUSED_; \
        GTEST_DISALLOW_COPY_AND_ASSIGN_(\
            GTEST_TEST_CLASS_NAME_(test_case_name, test_name)); \
      }; \
      int GTEST_TEST_CLASS_NAME_(test_case_name, \
                                 test_name)::gtest_registering_dummy_ = \
          GTEST_TEST_CLASS_NAME_(test_case_name, test_name)::AddToRegistry(); \
      void GTEST_TEST_CLASS_NAME_(test_case_name, test_name)::TestBody() \
      { \
        try \
        { \
          UnsafeTestBody(); \
        } \
        catch (const cvtest::details::SkipTestExceptionBase& e) \
        { \
            printf("[     SKIP ] %s\n", e.what()); \
            cv::cuda::resetDevice(); \
        } \
        catch (...) \
        { \
          cv::cuda::resetDevice(); \
          throw; \
        } \
      } \
      void GTEST_TEST_CLASS_NAME_(test_case_name, test_name)::UnsafeTestBody()

    #define DIFFERENT_SIZES testing::Values(cv::Size(128, 128), cv::Size(113, 113))

    #define DIFFERENT_SIZES_EXTRA testing::Values(cv::Size(13, 1), cv::Size(1, 13), cv::Size(128, 128), cv::Size(113, 113))

    // Depth

    using perf::MatDepth;

    #define ALL_DEPTH testing::Values(MatDepth(CV_8U), MatDepth(CV_8S), MatDepth(CV_16U), MatDepth(CV_16S), MatDepth(CV_32S), MatDepth(CV_32F), MatDepth(CV_64F))

    #define DEPTH_PAIRS testing::Values(std::make_pair(MatDepth(CV_8U), MatDepth(CV_8U)),   \
                                        std::make_pair(MatDepth(CV_8U), MatDepth(CV_16U)),  \
                                        std::make_pair(MatDepth(CV_8U), MatDepth(CV_16S)),  \
                                        std::make_pair(MatDepth(CV_8U), MatDepth(CV_32S)),  \
                                        std::make_pair(MatDepth(CV_8U), MatDepth(CV_32F)),  \
                                        std::make_pair(MatDepth(CV_8U), MatDepth(CV_64F)),  \
                                                                                            \
                                        std::make_pair(MatDepth(CV_16U), MatDepth(CV_16U)), \
                                        std::make_pair(MatDepth(CV_16U), MatDepth(CV_32S)), \
                                        std::make_pair(MatDepth(CV_16U), MatDepth(CV_32F)), \
                                        std::make_pair(MatDepth(CV_16U), MatDepth(CV_64F)), \
                                                                                            \
                                        std::make_pair(MatDepth(CV_16S), MatDepth(CV_16S)), \
                                        std::make_pair(MatDepth(CV_16S), MatDepth(CV_32S)), \
                                        std::make_pair(MatDepth(CV_16S), MatDepth(CV_32F)), \
                                        std::make_pair(MatDepth(CV_16S), MatDepth(CV_64F)), \
                                                                                            \
                                        std::make_pair(MatDepth(CV_32S), MatDepth(CV_32S)), \
                                        std::make_pair(MatDepth(CV_32S), MatDepth(CV_32F)), \
                                        std::make_pair(MatDepth(CV_32S), MatDepth(CV_64F)), \
                                                                                            \
                                        std::make_pair(MatDepth(CV_32F), MatDepth(CV_32F)), \
                                        std::make_pair(MatDepth(CV_32F), MatDepth(CV_64F)), \
                                                                                            \
                                        std::make_pair(MatDepth(CV_64F), MatDepth(CV_64F)))

    // Type

    using perf::MatType;

    //! return vector with types from specified range.
    std::vector<MatType> types(int depth_start, int depth_end, int cn_start, int cn_end);

    //! return vector with all types (depth: CV_8U-CV_64F, channels: 1-4).
    const std::vector<MatType>& all_types();

    #define ALL_TYPES testing::ValuesIn(all_types())
    #define TYPES(depth_start, depth_end, cn_start, cn_end) testing::ValuesIn(types(depth_start, depth_end, cn_start, cn_end))

    // ROI

    class UseRoi
    {
    public:
        inline UseRoi(bool val = false) : val_(val) {}

        inline operator bool() const { return val_; }

    private:
        bool val_;
    };

    void PrintTo(const UseRoi& useRoi, std::ostream* os);

    #define WHOLE_SUBMAT testing::Values(UseRoi(false), UseRoi(true))

    // Direct/Inverse

    class Inverse
    {
    public:
        inline Inverse(bool val = false) : val_(val) {}

        inline operator bool() const { return val_; }

    private:
        bool val_;
    };

    void PrintTo(const Inverse& useRoi, std::ostream* os);

    #define DIRECT_INVERSE testing::Values(Inverse(false), Inverse(true))

    // Param class

    #define IMPLEMENT_PARAM_CLASS(name, type) \
        class name \
        { \
        public: \
            name ( type arg = type ()) : val_(arg) {} \
            operator type () const {return val_;} \
        private: \
            type val_; \
        }; \
        inline void PrintTo( name param, std::ostream* os) \
        { \
            *os << #name <<  "(" << testing::PrintToString(static_cast< type >(param)) << ")"; \
        }

    IMPLEMENT_PARAM_CLASS(Channels, int)

    #define ALL_CHANNELS testing::Values(Channels(1), Channels(2), Channels(3), Channels(4))
    #define IMAGE_CHANNELS testing::Values(Channels(1), Channels(3), Channels(4))

    // Flags and enums

    CV_ENUM(NormCode, NORM_INF, NORM_L1, NORM_L2, NORM_TYPE_MASK, NORM_RELATIVE, NORM_MINMAX)

    CV_ENUM(Interpolation, INTER_NEAREST, INTER_LINEAR, INTER_CUBIC, INTER_AREA)

    CV_ENUM(BorderType, BORDER_REFLECT101, BORDER_REPLICATE, BORDER_CONSTANT, BORDER_REFLECT, BORDER_WRAP)
    #define ALL_BORDER_TYPES testing::Values(BorderType(cv::BORDER_REFLECT101), BorderType(cv::BORDER_REPLICATE), BorderType(cv::BORDER_CONSTANT), BorderType(cv::BORDER_REFLECT), BorderType(cv::BORDER_WRAP))

    CV_FLAGS(WarpFlags, INTER_NEAREST, INTER_LINEAR, INTER_CUBIC, WARP_INVERSE_MAP)

    //////////////////////////////////////////////////////////////////////
    // Features2D

    testing::AssertionResult assertKeyPointsEquals(const char* gold_expr, const char* actual_expr, std::vector<cv::KeyPoint>& gold, std::vector<cv::KeyPoint>& actual);

    #define ASSERT_KEYPOINTS_EQ(gold, actual) EXPECT_PRED_FORMAT2(assertKeyPointsEquals, gold, actual)

    int getMatchedPointsCount(std::vector<cv::KeyPoint>& gold, std::vector<cv::KeyPoint>& actual);
    int getMatchedPointsCount(const std::vector<cv::KeyPoint>& keypoints1, const std::vector<cv::KeyPoint>& keypoints2, const std::vector<cv::DMatch>& matches);

    //////////////////////////////////////////////////////////////////////
    // Other

    void dumpImage(const std::string& fileName, const cv::Mat& image);
    void showDiff(cv::InputArray gold, cv::InputArray actual, double eps);

    void parseCudaDeviceOptions(int argc, char **argv);
    void printCudaInfo();
}

namespace cv { namespace cuda
{
    void PrintTo(const DeviceInfo& info, std::ostream* os);
}}

#ifdef HAVE_CUDA

#define CV_TEST_INIT0_CUDA cvtest::parseCudaDeviceOptions(argc, argv), cvtest::printCudaInfo(), cv::setUseOptimized(false)

#define CV_CUDA_TEST_MAIN(resourcesubdir, ...) \
    CV_TEST_MAIN_EX(resourcesubdir, CUDA, __VA_ARGS__)

#else // HAVE_CUDA

#define CV_CUDA_TEST_MAIN(resourcesubdir) \
    int main() \
    { \
        printf("OpenCV was built without CUDA support\n"); \
        return 0; \
    }

#endif // HAVE_CUDA


#endif // OPENCV_CUDA_TEST_UTILITY_HPP
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

- **Inverse**: A class/struct defined in this file
- **name**: A class/struct defined in this file
- **GTEST_TEST_CLASS_NAME_**: A class/struct defined in this file
- **UseRoi**: A class/struct defined in this file
- **DeviceManager**: A class/struct defined in this file

### Functions and Methods

- **OPENCV_CUDA_TEST_UTILITY_HPP()**: A function/method defined in this file
- **EXPECT_MAT_NEAR()**: A function/method defined in this file
- **EXPECT_MAT_SIMILAR()**: A function/method defined in this file
- **HAVE_CUDA()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/ts.hpp`
- `stdexcept`
- `opencv2/core/cuda.hpp`

**Python Imports:**
- `testdata`
- `specified`
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

