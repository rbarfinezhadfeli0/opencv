# Documentation for `modules/calib3d/misc/java/gen_dict.json`

## File Metadata

- **Full Path**: `modules/calib3d/misc/java/gen_dict.json`
- **File Name**: `gen_dict.json`
- **File Size**: 1,684 bytes
- **File Type**: .json
- **Link to Source**: [modules/calib3d/misc/java/gen_dict.json](../../../../modules/calib3d/misc/java/gen_dict.json)

## Purpose and Role

This file is located in the `modules/calib3d/misc/java` directory and serves as part of the OpenCV library infrastructure.

## Configuration File Content

```
{
    "class_ignore_list": [
        "CirclesGridFinderParameters"
    ],
    "namespaces_dict": {
        "cv.fisheye": "fisheye"
    },
    "func_arg_fix" : {
        "findFundamentalMat"  : { "points1" : {"ctype" : "vector_Point2f"},
                                  "points2" : {"ctype" : "vector_Point2f"} },
        "cornerSubPix" : { "corners" : {"ctype" : "vector_Point2f"} },
        "findHomography" : { "srcPoints" : {"ctype" : "vector_Point2f"},
                             "dstPoints" : {"ctype" : "vector_Point2f"} },
        "solvePnP" : { "objectPoints" : {"ctype" : "vector_Point3f"},
                       "imagePoints"  : {"ctype" : "vector_Point2f"},
                       "distCoeffs"   : {"ctype" : "vector_double" } },
        "solvePnPRansac" : { "objectPoints" : {"ctype" : "vector_Point3f"},
                             "imagePoints"  : {"ctype" : "vector_Point2f"},
                             "distCoeffs"   : {"ctype" : "vector_double" } },
        "undistortPoints" : { "src" : {"ctype" : "vector_Point2f"},
                              "dst" : {"ctype" : "vector_Point2f"} },
        "projectPoints" : { "objectPoints" : {"ctype" : "vector_Point3f"},
                            "imagePoints"  : {"ctype" : "vector_Point2f"},
                            "distCoeffs"   : {"ctype" : "vector_double" } },
        "initCameraMatrix2D" : { "objectPoints" : {"ctype" : "vector_vector_Point3f"},
                                 "imagePoints"  : {"ctype" : "vector_vector_Point2f"} },
        "findChessboardCorners" : { "corners" : {"ctype" : "vector_Point2f"} },
        "drawChessboardCorners" : { "corners" : {"ctype" : "vector_Point2f"} }
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

