# Documentation for `modules/gapi/src/backends/render/ft_render.cpp`

## File Metadata

- **Full Path**: `modules/gapi/src/backends/render/ft_render.cpp`
- **File Name**: `ft_render.cpp`
- **File Size**: 8,611 bytes
- **File Type**: .cpp
- **Link to Source**: [modules/gapi/src/backends/render/ft_render.cpp](../../../../../modules/gapi/src/backends/render/ft_render.cpp)

## Purpose and Role

This file is located in the `modules/gapi/src/backends/render` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
// This file is part of OpenCV project.
// It is subject to the license terms in the LICENSE file found in the top-level directory
// of this distribution and at http://opencv.org/license.html.
//
// Copyright (C) 2019 Intel Corporation

#include "precomp.hpp"
#include "ft_render.hpp"

#ifdef HAVE_FREETYPE

#include "ft_render_priv.hpp"

#include <opencv2/gapi/util/throw.hpp>
#include <opencv2/gapi/own/assert.hpp>

cv::gapi::wip::draw::FTTextRender::Priv::Priv(const std::string& path)
{
    if (FT_Init_FreeType(&m_library) != 0)
    {
        cv::util::throw_error(std::runtime_error("Failed to initialize FT"));
    }

    if (FT_New_Face(m_library, path.c_str(), 0, &m_face))
    {
        FT_Done_FreeType(m_library);
        cv::util::throw_error(std::runtime_error("Failed to create a font face"));
    }
}

cv::Size cv::gapi::wip::draw::FTTextRender::Priv::getTextSize(const std::wstring& text, int fh, int* baseline)
{
    //
    //
    //
    //   ^                                                           diff between size and advance(2)
    //   |       ______________               width        width    |<->|
    //   |      |      **      |            |<------>| <------------|--->
    //   |      |     *  *     |            |________| |____________|___|________
    //   | left |    *    *    |       left |* * * * | |   * * * * *|   |     ^   ^
    //   |<---->|   ** ** **   |     <----->|*      *| |       *    |   |  t  |   |
    //   |      |  *        *  |     |      |*      *| |       *    |   |  o  | h |
    //   |      | *          * |     |      |* * * * | |       *   (1)  |  p  | e |  baseline
    //   O------|*------------*|-----O----- |*-------|-|----O--*----O---|-----*-i-|------------>
    //   |      |______________|     |      |*       | |*   |  *    |   |     ^ g |
    //   |      |              |     |      |*       | |*   |  *    |   |  b  | h |
    //   |      |    width     |     |      |*       | |*   |  *    |   |  o  | t |
    //   |      |<------------>|     |      |*       | | * *|*      |   |  t  |   |
    //   |                           |      |________| |____|_______|___|_____|___*
    //   |         advance           |       advance   |    |advance| (advance maybe less than width)
    //   <---------------------------><----------------|----><------>
    //                                                 |left| (left maybe is negative)
    //                                                 |<-->|
    //
    //
    //   O                                       - The pen position for any time
    //
    //   left (m_face->glyph->bitmap_left)       - The horizontal distance from the current pen position to the glyph's left bbox edge.
    //
    //   advance (m_face->glyph->advance.x >> 6) - The horizontal distance to increment (for left-to-right writing)
    //                                              or decrement (for right-to-left writing) the pen position after a
    //                                              glyph has been rendered when processing text
    //
    //   width (bitmap->width)                   - The width of glyph
    //
    //
    //   Algorithm to compute size of the text bounding box:
    //
    //   1) Go through all symbols and shift pen position and save glyph parameters (left, advance, width)
    //      If left + pen position < 0 set left to 0. For example it's maybe happened
    //      if we print first letter 'J' or any other letter with negative 'left'
    //      We want to render glyph in pen position + left, so we must't allow it to be negative
    //
    //   2) If width == 0 we must to skip this symbol and don't save parameters for him.
    //      For example width == 0 for space sometimes
    //
    //   3) Also we compute max top and max bottom it's required for compute baseline
    //
    //   3) At the end we'll get the pen position for the symbol next to the last.
    //      See (1) on picture.
    //
    //   4) As we can see the last pen position is isn't horizontal size yet.
    //      We need to check if the glyph goes beyond the last position of the pen
    //      To do this we can:
    //      a) Return to the previous position -advance
    //      b) Shift on left value +left
    //      c) Shift on width of the last glyph
    //
    //      Compare result position with pen position and choose max
    //
    //      We can compute diff and check if diff > 0 pen.x += diff.
    //      See (2) on picture.
    //
    //  5) Return size. Complete!!!
    //
    // See also about freetype glyph metrics:
    // https://www.freetype.org/freetype2/docs/glyphs/glyphs-3.html

    GAPI_Assert(!FT_Set_Pixel_Sizes(m_face, fh, fh) &&
                "Failed to set pixel size");

    cv::Point pen(0, 0);

    int max_bot      = 0;
    int max_top      = 0;
    int last_advance = 0;
    int last_width   = 0;
    int last_left    = 0;

    for (const auto& wc : text)
    {
        GAPI_Assert(!FT_Load_Char(m_face, wc, FT_LOAD_RENDER) &&
                    "Failed to load char");

        FT_Bitmap *bitmap = &(m_face->glyph->bitmap);

        int left    = m_face->glyph->bitmap_left;
        int advance = (m_face->glyph->advance.x >> 6);
        int width   = bitmap->width;

        // NB: Read (1) paragraph of algorithm description
        if (pen.x + left < 0)
        {
            left = 0;
        }

        int bot = (m_face->glyph->metrics.height - m_face->glyph->metrics.horiBearingY) >> 6;
        max_bot = std::max(max_bot, bot);
        max_top = std::max(max_top, m_face->glyph->bitmap_top);

        // NB: Read (2) paragraph of algorithm description
        if (width != 0)
        {
            last_width = width;
            last_advance = advance;
            last_left    = left;
        }

        pen.x += advance;
    }

    // NB: Read (4) paragraph of algorithm description
    int diff = (last_width + last_left) - last_advance;
    pen.x += (diff > 0) ? diff : 0;

    if (baseline)
    {
        *baseline = max_bot;
    }

    return {pen.x, max_bot + max_top};
}

void cv::gapi::wip::draw::FTTextRender::Priv::putText(cv::Mat& mat,
                                                       const std::wstring& text,
                                                       const cv::Point& org,
                                                       int fh)
{
    GAPI_Assert(!FT_Set_Pixel_Sizes(m_face, fh, fh) &&
                "Failed to set pixel size");

    cv::Point pen = org;
    for (const auto& wc : text)
    {
        GAPI_Assert(!FT_Load_Char(m_face, wc, FT_LOAD_RENDER) &&
                    "Failed to load char");
        FT_Bitmap *bitmap = &(m_face->glyph->bitmap);

        // FIXME: Skip glyph, if size is 0
        if (bitmap->rows == 0 || bitmap->width == 0) {
            continue;
        }

        cv::Mat glyph(bitmap->rows, bitmap->width, CV_8UC1, bitmap->buffer, bitmap->pitch);

        int left    = m_face->glyph->bitmap_left;
        int top     = m_face->glyph->bitmap_top;
        int advance = (m_face->glyph->advance.x >> 6);

        if (pen.x + left < 0)
        {
            left = 0;
        }

        cv::Rect rect(pen.x + left, org.y - top, glyph.cols, glyph.rows);

        auto roi = mat(rect);
        roi += glyph;
        pen.x += advance;
    }
}

cv::gapi::wip::draw::FTTextRender::Priv::~Priv()
{
    FT_Done_Face(m_face);
    FT_Done_FreeType(m_library);
}

cv::gapi::wip::draw::FTTextRender::FTTextRender(const std::string& path)
    : m_priv(new Priv(path))
{
}

cv::Size cv::gapi::wip::draw::FTTextRender::getTextSize(const std::wstring& text,
                                                         int fh,
                                                         int* baseline)
{
    return m_priv->getTextSize(text, fh, baseline);
}

void cv::gapi::wip::draw::FTTextRender::putText(cv::Mat& mat,
                                                 const std::wstring& text,
                                                 const cv::Point& org,
                                                 int fh)
{
    m_priv->putText(mat, text, org, fh);
}

#else

cv::Size cv::gapi::wip::draw::FTTextRender::getTextSize(const std::wstring&, int, int*)
{
    cv::util::throw_error(std::runtime_error("Freetype not found"));
}

void cv::gapi::wip::draw::FTTextRender::putText(cv::Mat&, const std::wstring&, const cv::Point&, int)
{
    cv::util::throw_error(std::runtime_error("Freetype not found"));
}

cv::gapi::wip::draw::FTTextRender::FTTextRender(const std::string&)
{
    cv::util::throw_error(std::runtime_error("Freetype not found"));
}

#endif // HAVE_FREETYPE
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

- **HAVE_FREETYPE()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**C++ Includes:**
- `ft_render.hpp`
- `opencv2/gapi/own/assert.hpp`
- `opencv2/gapi/util/throw.hpp`
- `precomp.hpp`
- `ft_render_priv.hpp`

**Python Imports:**
- `the`


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

