# Documentation for `samples/winrt/ImageManipulations/MediaExtensions/OcvTransform/OcvImageManipulations.idl`

## File Metadata

- **Full Path**: `samples/winrt/ImageManipulations/MediaExtensions/OcvTransform/OcvImageManipulations.idl`
- **File Name**: `OcvImageManipulations.idl`
- **File Size**: 206 bytes
- **File Type**: .idl
- **Link to Source**: [samples/winrt/ImageManipulations/MediaExtensions/OcvTransform/OcvImageManipulations.idl](../../../../../samples/winrt/ImageManipulations/MediaExtensions/OcvTransform/OcvImageManipulations.idl)

## Purpose and Role

This file is located in the `samples/winrt/ImageManipulations/MediaExtensions/OcvTransform` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
import "Windows.Media.idl";

#include <sdkddkver.h>

namespace OcvTransform
{
    [version(NTDDI_WIN8)]
    runtimeclass OcvImageManipulations
    {
        interface Windows.Media.IMediaExtension;
    }
}

```

## General Information

This file is part of the OpenCV repository infrastructure.

