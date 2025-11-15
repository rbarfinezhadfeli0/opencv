# Documentation for `modules/features2d/misc/objc/gen_dict.json`

## File Metadata

- **Full Path**: `modules/features2d/misc/objc/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 1,564 bytes
- **File Type**: .json
- **Link to Source**: [modules/features2d/misc/objc/gen_dict.json](../../../../modules/features2d/misc/objc/gen_dict.json)

## Purpose and Role

This file is located in the `modules/features2d/misc/objc` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "ManualFuncs" : {
        "SimpleBlobDetector": {
            "setParams": { "declaration" : [""], "implementation" : [""] },
            "getParams": { "declaration" : [""], "implementation" : [""] }
        }
    },
    "enum_fix" : {
        "FastFeatureDetector" : { "DetectorType": "FastDetectorType" },
        "AgastFeatureDetector" : { "DetectorType": "AgastDetectorType" }
    },
    "func_arg_fix" : {
        "Feature2D": {
            "(void)compute:(NSArray<Mat*>*)images keypoints:(NSMutableArray<NSMutableArray<KeyPoint*>*>*)keypoints descriptors:(NSMutableArray<Mat*>*)descriptors" : { "compute" : {"name" : "compute2"} },
            "(void)detect:(NSArray<Mat*>*)images keypoints:(NSMutableArray<NSMutableArray<KeyPoint*>*>*)keypoints masks:(NSArray<Mat*>*)masks" : { "detect" : {"name" : "detect2"} }
        },
        "DescriptorMatcher": {
            "(DescriptorMatcher*)create:(NSString*)descriptorMatcherType" : { "create" : {"name" : "create2"} }
        },
        "FlannBasedMatcher": {
            "FlannBasedMatcher": { "indexParams" : {"defval" : "cv::makePtr<cv::flann::KDTreeIndexParams>()"}, "searchParams" : {"defval" : "cv::makePtr<cv::flann::SearchParams>()"} }
        },
        "BFMatcher": {
            "BFMatcher" : { "normType" : {"ctype" : "NormTypes"} },
            "(BFMatcher*)create:(int)normType crossCheck:(BOOL)crossCheck" : { "create" : {"name" : "createBFMatcher"},
                                                                               "normType" : {"ctype" : "NormTypes"} }
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

