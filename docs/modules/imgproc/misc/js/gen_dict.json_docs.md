# Documentation for `modules/imgproc/misc/js/gen_dict.json`

## File Metadata

- **Full Path**: `modules/imgproc/misc/js/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 3,344 bytes
- **File Type**: .json
- **Link to Source**: [modules/imgproc/misc/js/gen_dict.json](../../../../modules/imgproc/misc/js/gen_dict.json)

## Purpose and Role

This file is located in the `modules/imgproc/misc/js` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "whitelist":
    {
        "": [
            "adaptiveThreshold",
            "applyColorMap",
            "approxPolyDP",
            "approxPolyN",
            "arcLength",
            "arrowedLine",
            "bilateralFilter",
            "blendLinear",
            "blur",
            "boundingRect",
            "boxFilter",
            "calcBackProject",
            "calcHist",
            "Canny",
            "circle",
            "clipLine",
            "compareHist",
            "connectedComponents",
            "connectedComponentsWithStats",
            "contourArea",
            "convertMaps",
            "convexHull",
            "convexityDefects",
            "cornerHarris",
            "cornerMinEigenVal",
            "createCLAHE",
            "createHanningWindow",
            "createLineSegmentDetector",
            "cvtColor",
            "demosaicing",
            "dilate",
            "distanceTransform",
            "distanceTransformWithLabels",
            "divSpectrums",
            "drawContours",
            "drawMarker",
            "ellipse",
            "ellipse2Poly",
            "equalizeHist",
            "erode",
            "fillConvexPoly",
            "fillPoly",
            "filter2D",
            "findContours",
            "findContoursLinkRuns",
            "fitEllipse",
            "fitEllipseAMS",
            "fitEllipseDirect",
            "fitLine",
            "floodFill",
            "GaussianBlur",
            "getAffineTransform",
            "getFontScaleFromHeight",
            "getPerspectiveTransform",
            "getRectSubPix",
            "getRotationMatrix2D",
            "getStructuringElement",
            "goodFeaturesToTrack",
            "grabCut",
            "HoughCircles",
            "HoughLines",
            "HoughLinesP",
            "HuMoments",
            "integral",
            "integral2",
            "intersectConvexConvex",
            "invertAffineTransform",
            "isContourConvex",
            "Laplacian",
            "line",
            "matchShapes",
            "matchTemplate",
            "medianBlur",
            "minAreaRect",
            "minEnclosingCircle",
            "minEnclosingTriangle",
            "moments",
            "morphologyEx",
            "pointPolygonTest",
            "polylines",
            "preCornerDetect",
            "putText",
            "pyrDown",
            "pyrUp",
            "rectangle",
            "remap",
            "resize",
            "rotatedRectangleIntersection",
            "Scharr",
            "sepFilter2D",
            "Sobel",
            "spatialGradient",
            "sqrBoxFilter",
            "stackBlur",
            "threshold",
            "warpAffine",
            "warpPerspective",
            "warpPolar",
            "watershed"
        ],
        "CLAHE": ["apply", "collectGarbage", "getClipLimit", "getTilesGridSize", "setClipLimit", "setTilesGridSize"],
        "segmentation_IntelligentScissorsMB": [
            "IntelligentScissorsMB",
            "setWeights",
            "setGradientMagnitudeMaxLimit",
            "setEdgeFeatureZeroCrossingParameters",
            "setEdgeFeatureCannyParameters",
            "applyImage",
            "applyImageFeatures",
            "buildMap",
            "getContour"
        ]
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

