# Documentation for `modules/gapi/test/gapi_gcomputation_tests.cpp`

## File Metadata

- **Full Path**: `modules/gapi/test/gapi_gcomputation_tests.cpp`
- **File Name**: `gapi_gcomputation_tests.cpp`
- **File Size**: 6,353 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/test/gapi_gcomputation_tests.cpp](../../../modules/gapi/test/gapi_gcomputation_tests.cpp)

## Purpose and Role

This file is located in the `modules/gapi/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2018 Intel Corporation


#include "test_precomp.hpp"

#include <opencv2/gapi/s11n.hpp>

#include <opencv2/gapi/cpu/gcpukernel.hpp>
#include <ade/util/zip_range.hpp>

namespace opencv_test
{

  namespace
  {
      G_TYPED_KERNEL(CustomResize, <cv::GMat(cv::GMat, cv::Size, double, double, int)>, "org.opencv.customk.resize")
      {
          static cv::GMatDesc outMeta(cv::GMatDesc in, cv::Size sz, double fx, double fy, int) {
              if (sz.width != 0 && sz.height != 0)
              {
                  return in.withSize(sz);
              }
              else
              {
                  GAPI_Assert(fx != 0. && fy != 0.);
                  return in.withSize
                    (cv::Size(static_cast<int>(std::round(in.size.width  * fx)),
                                         static_cast<int>(std::round(in.size.height * fy))));
              }
          }
      };

      GAPI_OCV_KERNEL(CustomResizeImpl, CustomResize)
      {
          static void run(const cv::Mat& in, cv::Size sz, double fx, double fy, int interp, cv::Mat &out)
          {
              cv::resize(in, out, sz, fx, fy, interp);
          }
      };

      struct GComputationApplyTest: public ::testing::Test
      {
          cv::GMat in;
          cv::Mat  in_mat;
          cv::Mat  out_mat;
          cv::GComputation m_c;

          GComputationApplyTest() : in_mat(300, 300, CV_8UC1),
                                    m_c(cv::GIn(in), cv::GOut(CustomResize::on(in, cv::Size(100, 100),
                                                                               0.0, 0.0, cv::INTER_LINEAR)))
          {
          }
      };

      struct GComputationVectorMatsAsOutput: public ::testing::Test
      {
          cv::Mat  in_mat;
          cv::GComputation m_c;
          std::vector<cv::Mat> ref_mats;

          GComputationVectorMatsAsOutput() : in_mat(300, 300, CV_8UC3),
          m_c([&](){
                      cv::GMat in;
                      cv::GMat out[3];
                      std::tie(out[0], out[1], out[2]) = cv::gapi::split3(in);
                      return cv::GComputation({in}, {out[0], out[1], out[2]});
                  })
          {
              cv::randu(in_mat, cv::Scalar::all(0), cv::Scalar::all(255));
              cv::split(in_mat, ref_mats);
          }

          void run(std::vector<cv::Mat>& out_mats)
          {
              m_c.apply({in_mat}, out_mats);
          }

          void check(const std::vector<cv::Mat>& out_mats)
          {
              for (const auto it : ade::util::zip(ref_mats, out_mats))
              {
                  const auto& ref_mat = std::get<0>(it);
                  const auto& out_mat = std::get<1>(it);

                  EXPECT_EQ(0, cvtest::norm(ref_mat, out_mat, NORM_INF));
              }
          }
      };

      struct GComputationPythonApplyTest: public ::testing::Test
      {
          cv::Size sz;
          MatType type;
          cv::Mat in_mat1, in_mat2, out_mat_ocv;
          cv::GComputation m_c;

          GComputationPythonApplyTest() : sz(cv::Size(300,300)), type(CV_8UC1),
          in_mat1(sz, type), in_mat2(sz, type), out_mat_ocv(sz, type),
          m_c([&](){
                  cv::GMat in1, in2;
                  cv::GMat out = in1 + in2;
                  return cv::GComputation(cv::GIn(in1, in2), cv::GOut(out));
                  })
          {
              cv::randu(in_mat1, cv::Scalar::all(0), cv::Scalar::all(255));
              cv::randu(in_mat2, cv::Scalar::all(0), cv::Scalar::all(255));
              out_mat_ocv = in_mat1 + in_mat2;
          }
      };
  }

  TEST_F(GComputationPythonApplyTest, WithoutSerialization)
  {
      auto output = m_c.apply(cv::detail::ExtractArgsCallback{[this](const cv::GTypesInfo& info)
                                  {
                                      GAPI_Assert(info[0].shape == cv::GShape::GMAT);
                                      GAPI_Assert(info[1].shape == cv::GShape::GMAT);
                                      return cv::GRunArgs{in_mat1, in_mat2};
                                  }
                              });

      EXPECT_EQ(1u, output.size());

      const auto& out_mat_gapi = cv::util::get<cv::Mat>(output[0]);
      EXPECT_EQ(0, cvtest::norm(out_mat_ocv, out_mat_gapi, NORM_INF));
  }

  TEST_F(GComputationPythonApplyTest, WithSerialization)
  {
      auto p = cv::gapi::serialize(m_c);
      auto c = cv::gapi::deserialize<cv::GComputation>(p);

      auto output = c.apply(cv::detail::ExtractArgsCallback{[this](const cv::GTypesInfo& info)
                                  {
                                      GAPI_Assert(info[0].shape == cv::GShape::GMAT);
                                      GAPI_Assert(info[1].shape == cv::GShape::GMAT);
                                      return cv::GRunArgs{in_mat1, in_mat2};
                                  }
                              });

      EXPECT_EQ(1u, output.size());

      const auto& out_mat_gapi = cv::util::get<cv::Mat>(output[0]);
      EXPECT_EQ(0, cvtest::norm(out_mat_ocv, out_mat_gapi, NORM_INF));
  }

  TEST_F(GComputationApplyTest, ThrowDontPassCustomKernel)
  {
      EXPECT_THROW(m_c.apply(in_mat, out_mat), std::logic_error);
  }

  TEST_F(GComputationApplyTest, NoThrowPassCustomKernel)
  {
      const auto pkg = cv::gapi::kernels<CustomResizeImpl>();

      ASSERT_NO_THROW(m_c.apply(in_mat, out_mat, cv::compile_args(pkg)));
  }

  TEST_F(GComputationVectorMatsAsOutput, OutputAllocated)
  {
      std::vector<cv::Mat> out_mats(3);
      for (auto& out_mat : out_mats)
      {
          out_mat.create(in_mat.size(), CV_8UC1);
      }

      run(out_mats);
      check(out_mats);
  }

  TEST_F(GComputationVectorMatsAsOutput, OutputNotAllocated)
  {
      std::vector<cv::Mat> out_mats(3);

      run(out_mats);
      check(out_mats);
  }

  TEST_F(GComputationVectorMatsAsOutput, OutputAllocatedWithInvalidMeta)
  {
      std::vector<cv::Mat> out_mats(3);

      for (auto& out_mat : out_mats)
      {
          out_mat.create(in_mat.size() / 2, CV_8UC1);
      }

      run(out_mats);
      check(out_mats);
  }

} // namespace opencv_test
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

- **GComputationVectorMatsAsOutput**: A class/struct defined in this file
- **GComputationPythonApplyTest**: A class/struct defined in this file
- **GComputationApplyTest**: A class/struct defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `opencv2/gapi/cpu/gcpukernel.hpp`
- `ade/util/zip_range.hpp`
- `opencv2/gapi/s11n.hpp`
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

