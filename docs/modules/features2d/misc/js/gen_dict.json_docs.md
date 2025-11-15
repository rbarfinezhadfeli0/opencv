# Documentation for `modules/features2d/misc/js/gen_dict.json`

## File Metadata

- **Full Path**: `modules/features2d/misc/js/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 2,136 bytes
- **File Type**: .json
- **Link to Source**: [modules/features2d/misc/js/gen_dict.json](../../../../modules/features2d/misc/js/gen_dict.json)

## Purpose and Role

This file is located in the `modules/features2d/misc/js` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "whitelist":
    {
        "Feature2D": ["detect", "compute", "detectAndCompute", "descriptorSize", "descriptorType", "defaultNorm", "empty", "getDefaultName"],
        "BRISK": ["create", "getDefaultName"],
        "ORB": ["create", "setMaxFeatures", "setScaleFactor", "setNLevels", "setEdgeThreshold", "setFastThreshold", "setFirstLevel", "setWTA_K", "setScoreType", "setPatchSize", "getFastThreshold", "getDefaultName"],
        "MSER": ["create", "detectRegions", "setDelta", "getDelta", "setMinArea", "getMinArea", "setMaxArea", "getMaxArea", "setPass2Only", "getPass2Only", "getDefaultName"],
        "FastFeatureDetector": ["create", "setThreshold", "getThreshold", "setNonmaxSuppression", "getNonmaxSuppression", "setType", "getType", "getDefaultName"],
        "AgastFeatureDetector": ["create", "setThreshold", "getThreshold", "setNonmaxSuppression", "getNonmaxSuppression", "setType", "getType", "getDefaultName"],
        "GFTTDetector": ["create", "setMaxFeatures", "getMaxFeatures", "setQualityLevel", "getQualityLevel", "setMinDistance", "getMinDistance", "setBlockSize", "getBlockSize", "setHarrisDetector", "getHarrisDetector", "setK", "getK", "getDefaultName"],
        "SimpleBlobDetector": ["create", "setParams", "getParams", "getDefaultName"],
        "SimpleBlobDetector_Params": [],
        "KAZE": ["create", "setExtended", "getExtended", "setUpright", "getUpright", "setThreshold", "getThreshold", "setNOctaves", "getNOctaves", "setNOctaveLayers", "getNOctaveLayers", "setDiffusivity", "getDiffusivity", "getDefaultName"],
        "AKAZE": ["create", "setDescriptorType", "getDescriptorType", "setDescriptorSize", "getDescriptorSize", "setDescriptorChannels", "getDescriptorChannels", "setThreshold", "getThreshold", "setNOctaves", "getNOctaves", "setNOctaveLayers", "getNOctaveLayers", "setDiffusivity", "getDiffusivity", "getDefaultName"],
        "DescriptorMatcher": ["add", "clear", "empty", "isMaskSupported", "train", "match", "knnMatch", "radiusMatch", "clone", "create"],
        "BFMatcher": ["isMaskSupported", "create"],
        "": ["drawKeypoints", "drawMatches", "drawMatchesKnn"]
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

