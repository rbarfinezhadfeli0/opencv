# Documentation for `samples/wp8/OpenCVXaml/OpenCVXaml/LocalizedStrings.cs`

## File Metadata

- **Full Path**: `samples/wp8/OpenCVXaml/OpenCVXaml/LocalizedStrings.cs`
- **File Name**: `LocalizedStrings.cs`
- **File Size**: 350 bytes
- **File Type**: .cs
- **Link to Source**: [samples/wp8/OpenCVXaml/OpenCVXaml/LocalizedStrings.cs](../../../../samples/wp8/OpenCVXaml/OpenCVXaml/LocalizedStrings.cs)

## Purpose and Role

This file is located in the `samples/wp8/OpenCVXaml/OpenCVXaml` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
﻿using OpenCVXaml.Resources;

namespace OpenCVXaml
{
    /// <summary>
    /// Provides access to string resources.
    /// </summary>
    public class LocalizedStrings
    {
        private static AppResources _localizedResources = new AppResources();

        public AppResources LocalizedResources { get { return _localizedResources; } }
    }
}
```

## General Information

This file is part of the OpenCV repository infrastructure.

