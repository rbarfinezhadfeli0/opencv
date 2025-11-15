# Documentation for `modules/videoio/misc/objc/gen_dict.json`

## File Metadata

- **Full Path**: `modules/videoio/misc/objc/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 852 bytes
- **File Type**: .json
- **Link to Source**: [modules/videoio/misc/objc/gen_dict.json](../../../../modules/videoio/misc/objc/gen_dict.json)

## Purpose and Role

This file is located in the `modules/videoio/misc/objc` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "AdditionalImports" : {
        "Videoio" :
            [ "\"videoio/registry.hpp\"" ]
    },
    "ManualFuncs" : {
        "VideoCapture" : {
            "release"         : {"declaration" : [""], "implementation" : [""] }
        },
        "VideoWriter" : {
            "release"         : {"declaration" : [""], "implementation" : [""] }
        }
    },
    "func_arg_fix" : {
        "VideoCapture" : {
            "(BOOL)open:(int)index apiPreference:(int)apiPreference" : { "open" : {"name" : "openWithIndex"} },
            "(BOOL)open:(int)index apiPreference:(int)apiPreference params:(IntVector*)params" : { "open" : {"name" : "openWithIndexAndParameters"} },
            "(BOOL)open:(IStreamReader*)source apiPreference:(int)apiPreference params:(IntVector*)params" : { "open" : {"name" : "openWithStreamReader"} }
        }
    }
}

```

## Purpose

This configuration file is used to control build settings, dependencies, or runtime behavior of the OpenCV library.

## Key Settings

Configuration files in OpenCV typically control:
- Build system configuration (CMake)
- Compiler flags and options
- Feature enablement/disablement
- Path specifications
- Version information
- Dependency management

## Usage

This file is processed during the build configuration phase or at runtime to customize OpenCV behavior.

