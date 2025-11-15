# Documentation for `modules/photo/misc/js/gen_dict.json`

## File Metadata

- **Full Path**: `modules/photo/misc/js/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 1,684 bytes
- **File Type**: .json
- **Link to Source**: [modules/photo/misc/js/gen_dict.json](../../../../modules/photo/misc/js/gen_dict.json)

## Purpose and Role

This file is located in the `modules/photo/misc/js` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "whitelist":
    {
        "": [
            "createAlignMTB", "createCalibrateDebevec", "createCalibrateRobertson",
                    "createMergeDebevec", "createMergeMertens", "createMergeRobertson",
                    "createTonemapDrago", "createTonemapMantiuk", "createTonemapReinhard", "inpaint"],
            "CalibrateCRF": ["process"],
            "AlignExposures": ["process"],
            "AlignMTB" : ["calculateShift", "shiftMat", "computeBitmaps", "getMaxBits", "setMaxBits",
                            "getExcludeRange", "setExcludeRange", "getCut", "setCut"],
            "CalibrateDebevec" : ["getLambda", "setLambda", "getSamples", "setSamples", "getRandom", "setRandom"],
            "CalibrateRobertson" : ["getMaxIter", "setMaxIter", "getThreshold", "setThreshold", "getRadiance"],
            "MergeExposures" : ["process"],
            "MergeDebevec" : ["process"],
            "MergeMertens" : ["process", "getContrastWeight", "setContrastWeight", "getSaturationWeight",
                                "setSaturationWeight", "getExposureWeight", "setExposureWeight"],
            "MergeRobertson" : ["process"],
            "Tonemap" : ["process" , "getGamma", "setGamma"],
            "TonemapDrago" : ["getSaturation", "setSaturation", "getBias", "setBias",
                                "getSigmaColor", "setSigmaColor", "getSigmaSpace","setSigmaSpace"],
            "TonemapMantiuk" : ["getScale", "setScale", "getSaturation", "setSaturation"],
            "TonemapReinhard" : ["getIntensity", "setIntensity", "getLightAdaptation", "setLightAdaptation",
                                    "getColorAdaptation", "setColorAdaptation"]
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

