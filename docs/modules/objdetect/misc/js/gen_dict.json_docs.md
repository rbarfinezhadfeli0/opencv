# Documentation for `modules/objdetect/misc/js/gen_dict.json`

## File Metadata

- **Full Path**: `modules/objdetect/misc/js/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 2,602 bytes
- **File Type**: .json
- **Link to Source**: [modules/objdetect/misc/js/gen_dict.json](../../../../modules/objdetect/misc/js/gen_dict.json)

## Purpose and Role

This file is located in the `modules/objdetect/misc/js` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "whitelist":
    {
        "": ["groupRectangles", "getPredefinedDictionary", "extendDictionary", "drawDetectedMarkers", "generateImageMarker", "drawDetectedCornersCharuco", "drawDetectedDiamonds"],
        "HOGDescriptor": ["load", "HOGDescriptor", "getDefaultPeopleDetector", "getDaimlerPeopleDetector", "setSVMDetector", "detectMultiScale"],
        "CascadeClassifier": ["load", "detectMultiScale2", "CascadeClassifier", "detectMultiScale3", "empty", "detectMultiScale"],
        "GraphicalCodeDetector": ["decode", "detect", "detectAndDecode", "detectMulti", "decodeMulti", "detectAndDecodeMulti"],
        "QRCodeDetector": ["QRCodeDetector", "decode", "detect", "detectAndDecode", "detectMulti", "decodeMulti", "detectAndDecodeMulti", "decodeCurved", "detectAndDecodeCurved", "setEpsX", "setEpsY"],
        "aruco_PredefinedDictionaryType": [],
        "aruco_Dictionary": ["Dictionary", "getDistanceToId", "generateImageMarker", "getByteListFromBits", "getBitsFromByteList"],
        "aruco_Board": ["Board", "matchImagePoints", "generateImage"],
        "aruco_GridBoard": ["GridBoard", "generateImage", "getGridSize", "getMarkerLength", "getMarkerSeparation", "matchImagePoints"],
        "aruco_CharucoParameters": ["CharucoParameters"],
        "aruco_CharucoBoard": ["CharucoBoard", "generateImage", "getChessboardCorners", "getNearestMarkerCorners", "checkCharucoCornersCollinear", "matchImagePoints", "getLegacyPattern", "setLegacyPattern"],
        "aruco_DetectorParameters": ["DetectorParameters"],
        "aruco_RefineParameters": ["RefineParameters"],
        "aruco_ArucoDetector": ["ArucoDetector", "detectMarkers", "refineDetectedMarkers", "setDictionary", "setDetectorParameters", "setRefineParameters"],
        "aruco_CharucoDetector": ["CharucoDetector", "setBoard", "setCharucoParameters", "setDetectorParameters", "setRefineParameters", "detectBoard", "detectDiamonds"],
        "QRCodeDetectorAruco_Params": ["Params"],
        "QRCodeDetectorAruco": ["QRCodeDetectorAruco", "decode", "detect", "detectAndDecode", "detectMulti", "decodeMulti", "detectAndDecodeMulti", "setDetectorParameters", "setArucoParameters"],
        "barcode_BarcodeDetector": ["BarcodeDetector", "decode", "detect", "detectAndDecode", "detectMulti", "decodeMulti", "detectAndDecodeMulti", "decodeWithType", "detectAndDecodeWithType"],
        "FaceDetectorYN": ["setInputSize", "getInputSize", "setScoreThreshold", "getScoreThreshold", "setNMSThreshold", "getNMSThreshold", "setTopK", "getTopK", "detect", "create"]
    },
    "namespace_prefix_override":
    {
        "aruco": ""
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

