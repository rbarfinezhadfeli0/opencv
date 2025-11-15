# Documentation for `modules/imgcodecs/src/grfmt_pfm.cpp`

## File Metadata

- **Full Path**: `modules/imgcodecs/src/grfmt_pfm.cpp`
- **File Name**: `grfmt_pfm.cpp`
- **File Size**: 6,382 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/imgcodecs/src/grfmt_pfm.cpp](../../../modules/imgcodecs/src/grfmt_pfm.cpp)

## Purpose and Role

This file is located in the `modules/imgcodecs/src` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.

#include "precomp.hpp"
#include "utils.hpp"
#include "grfmt_pfm.hpp"
#include <iostream>

#ifdef HAVE_IMGCODEC_PFM

namespace {

static_assert(sizeof(float) == 4, "float must be 32 bit.");


bool is_byte_order_swapped(double scale)
{
  // ".pfm" format file specifies that:
  // positive scale means big endianness;
  // negative scale means little endianness.

  #ifdef WORDS_BIGENDIAN
    return scale < 0.0;
  #else
    return scale >= 0.0;
  #endif
}

void swap_endianness(uint32_t& ui)
{
  static const uint32_t A(0x000000ffU);
  static const uint32_t B(0x0000ff00U);
  static const uint32_t C(0x00ff0000U);
  static const uint32_t D(0xff000000U);

  ui = ( (ui & A) << 24 )
     | ( (ui & B) <<  8 )
     | ( (ui & C) >>  8 )
     | ( (ui & D) >> 24 );
}

template<typename T> T atoT(const std::string& s);
template<> int atoT<int>(const std::string& s) { return std::atoi(s.c_str()); }
template<> double atoT<double>(const std::string& s) { return std::atof(s.c_str()); }

template<typename T>
T read_number(cv::RLByteStream& strm)
{
  // should be enough to take string representation of any number
  const size_t buffer_size = 2048;

  std::vector<char> buffer(buffer_size, 0);
  for (size_t i = 0; i < buffer_size; ++i) {
    const int intc = strm.getByte();
    CV_Assert(intc >= -128 && intc < 128);
    char c = static_cast<char>(intc);
    if (std::isspace(c)) {
      break;
    }
    buffer[i] = c;
  }
  const std::string str(buffer.begin(), buffer.end());
  return atoT<T>(str);
}

template<typename T> bool write_anything(cv::WLByteStream& strm, const T& t)
{
  std::ostringstream ss;
  ss << t;
  return strm.putBytes(ss.str().c_str(), static_cast<int>(ss.str().size()));
}

}

namespace cv {

PFMDecoder::~PFMDecoder()
{
}

PFMDecoder::PFMDecoder() : m_scale_factor(0), m_swap_byte_order(false)
{
  m_buf_supported = true;
}

bool PFMDecoder::readHeader()
{
  if (!m_buf.empty())
    m_strm.open(m_buf);
  else
    m_strm.open(m_filename);

  if( !m_strm.isOpened()) return false;

  if (m_strm.getByte() != 'P') {
    CV_Error(Error::StsError, "Unexpected file type (expected P)");
  }

  switch (m_strm.getByte()) {
  case 'f':
    m_type = CV_32FC1;
    break;
  case 'F':
    m_type = CV_32FC3;
    break;
  default:
    CV_Error(Error::StsError, "Unexpected file type (expected `f` or `F`)");
  }

  if ('\n' != m_strm.getByte()) {
    CV_Error(Error::StsError, "Unexpected header format (expected line break)");
  }


  m_width = read_number<int>(m_strm);
  m_height = read_number<int>(m_strm);
  m_scale_factor = read_number<double>(m_strm);
  m_swap_byte_order = is_byte_order_swapped(m_scale_factor);

  return true;
}

bool PFMDecoder::readData(Mat& mat)
{
  if (!m_strm.isOpened()) {
    CV_Error(Error::StsError, "Unexpected status in data stream");
  }

  Mat buffer(mat.size(), m_type);
  for (int y = m_height - 1; y >= 0; --y) {
    m_strm.getBytes(buffer.ptr(y), static_cast<int>(m_width * buffer.elemSize()));
    if (is_byte_order_swapped(m_scale_factor)) {
      for (int i = 0; i < m_width * buffer.channels(); ++i) {
        static_assert( sizeof(uint32_t) == sizeof(float),
                       "uint32_t and float must have same size." );
        swap_endianness(buffer.ptr<uint32_t>(y)[i]);
      }
    }
  }

  if (buffer.channels() == 3 && !m_use_rgb) {
    cv::cvtColor(buffer, buffer, cv::COLOR_BGR2RGB);
  }

  CV_Assert(fabs(m_scale_factor) > 0.0f);
  buffer *= 1.f / fabs(m_scale_factor);

  buffer.convertTo(mat, mat.type());

  return true;
}

size_t PFMDecoder::signatureLength() const
{
    return 3;
}

bool PFMDecoder::checkSignature( const String& signature ) const
{
    return signature.size() >= 3
        && signature[0] == 'P'
        && ( signature[1] == 'f' || signature[1] == 'F' )
        && isspace(signature[2]);
}

void PFMDecoder::close()
{
  // noop
}

//////////////////////////////////////////////////////////////////////////////////////////

PFMEncoder::PFMEncoder()
{
  m_description = "Portable image format - float (*.pfm)";
  m_buf_supported = true;
}

PFMEncoder::~PFMEncoder()
{
}

bool PFMEncoder::isFormatSupported(int depth) const
{
  // any depth will be converted into 32-bit float.
  CV_UNUSED(depth);
  return true;
}

bool PFMEncoder::write(const Mat& img, const std::vector<int>& params)
{
  CV_UNUSED(params);

  WLByteStream strm;
  if (m_buf) {
    if (!strm.open(*m_buf)) {
      return false;
    } else {
      m_buf->reserve(alignSize(256 + sizeof(float) * img.channels() * img.total(), 256));
    }
  } else if (!strm.open(m_filename)) {
    return false;
  }

  Mat float_img;
  CHECK_WRITE(strm.putByte('P'));
  switch (img.channels()) {
  case 1:
    CHECK_WRITE(strm.putByte('f'));
    img.convertTo(float_img, CV_32FC1);
    break;
  case 3:
    CHECK_WRITE(strm.putByte('F'));
    img.convertTo(float_img, CV_32FC3);
    break;
  default:
    CV_Error(Error::StsBadArg, "Expected 1 or 3 channel image.");
  }
  CHECK_WRITE(strm.putByte('\n'));


  CHECK_WRITE(write_anything(strm, float_img.cols));
  CHECK_WRITE(strm.putByte(' '));
  CHECK_WRITE(write_anything(strm, float_img.rows));
  CHECK_WRITE(strm.putByte('\n'));
#ifdef WORDS_BIGENDIAN
  CHECK_WRITE(write_anything(strm, 1.0));
#else
  CHECK_WRITE(write_anything(strm, -1.0));
#endif

  CHECK_WRITE(strm.putByte('\n'));

  // Comments are not officially supported in this file format.
  // write_anything(strm, "# Generated by OpenCV " CV_VERSION "\n");

  for (int y = float_img.rows - 1; y >= 0; --y)
  {
    if (float_img.channels() == 3) {
      const float* bgr_row = float_img.ptr<float>(y);
      size_t row_size = float_img.cols * float_img.channels();
      std::vector<float> rgb_row(row_size);
      for (int x = 0; x < float_img.cols; ++x) {
        rgb_row[x*3+0] = bgr_row[x*3+2];
        rgb_row[x*3+1] = bgr_row[x*3+1];
        rgb_row[x*3+2] = bgr_row[x*3+0];
      }
      CHECK_WRITE(strm.putBytes( reinterpret_cast<const uchar*>(rgb_row.data()),
                     static_cast<int>(sizeof(float) * row_size) ));
    } else if (float_img.channels() == 1) {
      CHECK_WRITE(strm.putBytes(float_img.ptr(y), sizeof(float) * float_img.cols));
    }
  }
  return true;
}

}

#endif // HAVE_IMGCODEC_PFM
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

- **WORDS_BIGENDIAN()**: A function/method defined in this file
- **HAVE_IMGCODEC_PFM()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `iostream`
- `precomp.hpp`
- `utils.hpp`
- `grfmt_pfm.hpp`


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

