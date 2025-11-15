# Documentation for `docs/samples/semihosting/include/raw_pixels.hpp.in_docs.md`

## File Metadata

- **Full Path**: `docs/samples/semihosting/include/raw_pixels.hpp.in_docs.md`
- **File Name**: `raw_pixels.hpp.in_docs.md`
- **File Size**: 885 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/semihosting/include/raw_pixels.hpp.in_docs.md](../../../../docs/samples/semihosting/include/raw_pixels.hpp.in_docs.md)

## Purpose and Role

This file is located in the `docs/samples/semihosting/include` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/semihosting/include/raw_pixels.hpp.in`

## File Metadata

- **Full Path**: `samples/semihosting/include/raw_pixels.hpp.in`
- **File Name**: `raw_pixels.hpp.in`
- **File Size**: 263 bytes
- **File Type**: .in
- **Link to Source**: [samples/semihosting/include/raw_pixels.hpp.in](../../../samples/semihosting/include/raw_pixels.hpp.in)

## Purpose and Role

This file is located in the `samples/semihosting/include` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
#ifndef RAW_PIXELS_HPP
#define RAW_PIXELS_HP
#include <cstdint>

#cmakedefine RAW_PIXEL_VALUES @RAW_PIXEL_VALUES@
#cmakedefine RAW_PIXELS_SIZE @RAW_PIXELS_SIZE@

static std::uint32_t raw_pixels[RAW_PIXELS_SIZE] = {
    RAW_PIXEL_VALUES
};
#endif //RAW_PIXELS_HPP

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

