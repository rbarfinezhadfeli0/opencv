# Documentation for `docs/samples/cpp/tutorial_code/objectDetection/detector_params.yml_docs.md`

## File Metadata

- **Full Path**: `docs/samples/cpp/tutorial_code/objectDetection/detector_params.yml_docs.md`
- **File Name**: `detector_params.yml_docs.md`
- **File Size**: 2,035 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/cpp/tutorial_code/objectDetection/detector_params.yml_docs.md](../../../../../docs/samples/cpp/tutorial_code/objectDetection/detector_params.yml_docs.md)

## Purpose and Role

This file is located in the `docs/samples/cpp/tutorial_code/objectDetection` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/cpp/tutorial_code/objectDetection/detector_params.yml`

## File Metadata

- **Full Path**: `samples/cpp/tutorial_code/objectDetection/detector_params.yml`
- **File Name**: `detector_params.yml`
- **File Size**: 921 bytes
- **File Type**: .yml
- **Link to Source**: [samples/cpp/tutorial_code/objectDetection/detector_params.yml](../../../../samples/cpp/tutorial_code/objectDetection/detector_params.yml)

## Purpose and Role

This file is located in the `samples/cpp/tutorial_code/objectDetection` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
%YAML:1.0
adaptiveThreshWinSizeMin: 3
adaptiveThreshWinSizeMax: 23
adaptiveThreshWinSizeStep: 10
adaptiveThreshWinSize: 21
adaptiveThreshConstant: 7
minMarkerPerimeterRate: 0.03
maxMarkerPerimeterRate: 4.0
polygonalApproxAccuracyRate: 0.05
minCornerDistanceRate: 0.05
minDistanceToBorder: 3
minMarkerDistance: 10.0
minMarkerDistanceRate: 0.05
cornerRefinementMethod: 0
cornerRefinementWinSize: 5
cornerRefinementMaxIterations: 30
cornerRefinementMinAccuracy: 0.1
markerBorderBits: 1
perspectiveRemovePixelPerCell: 8
perspectiveRemoveIgnoredMarginPerCell: 0.13
maxErroneousBitsInBorderRate: 0.04
minOtsuStdDev: 5.0
errorCorrectionRate: 0.6

# new aruco 3 functionality
useAruco3Detection: 0
minSideLengthCanonicalImg: 32 # 16, 32, 64 --> tau_c from the paper
minMarkerLengthRatioOriginalImg: 0.02 # range [0,0.2] --> tau_i from the paper
cameraMotionSpeed: 0.1 # range [0,1) --> tau_s from the paper
useGlobalThreshold: 0

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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

