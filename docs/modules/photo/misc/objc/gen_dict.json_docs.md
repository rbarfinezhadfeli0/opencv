# Documentation for `modules/photo/misc/objc/gen_dict.json`

## File Metadata

- **Full Path**: `modules/photo/misc/objc/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 612 bytes
- **File Type**: .json
- **Link to Source**: [modules/photo/misc/objc/gen_dict.json](../../../../modules/photo/misc/objc/gen_dict.json)

## Purpose and Role

This file is located in the `modules/photo/misc/objc` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "func_arg_fix" : {
        "Photo": {
            "(void)fastNlMeansDenoising:(Mat*)src dst:(Mat*)dst h:(FloatVector*)h templateWindowSize:(int)templateWindowSize searchWindowSize:(int)searchWindowSize normType:(int)normType" : { "h" : { "name" : "hVector" } },
            "(void)fastNlMeansDenoisingMulti:(NSArray<Mat*>*)srcImgs dst:(Mat*)dst imgToDenoiseIndex:(int)imgToDenoiseIndex temporalWindowSize:(int)temporalWindowSize h:(FloatVector*)h templateWindowSize:(int)templateWindowSize searchWindowSize:(int)searchWindowSize normType:(int)normType" : { "h" : { "name" : "hVector" } }
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

