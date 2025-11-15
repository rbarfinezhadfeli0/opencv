# Documentation for `docs/doc/tutorials/features2d/feature_homography/feature_homography.markdown_docs.md`

## File Metadata

- **Full Path**: `docs/doc/tutorials/features2d/feature_homography/feature_homography.markdown_docs.md`
- **File Name**: `feature_homography.markdown_docs.md`
- **File Size**: 2,889 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/tutorials/features2d/feature_homography/feature_homography.markdown_docs.md](../../../../../docs/doc/tutorials/features2d/feature_homography/feature_homography.markdown_docs.md)

## Purpose and Role

This file is located in the `docs/doc/tutorials/features2d/feature_homography` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/tutorials/features2d/feature_homography/feature_homography.markdown`

## File Metadata

- **Full Path**: `doc/tutorials/features2d/feature_homography/feature_homography.markdown`
- **File Name**: `feature_homography.markdown`
- **File Size**: 2,126 bytes
- **File Type**: .markdown
- **Link to Source**: [doc/tutorials/features2d/feature_homography/feature_homography.markdown](../../../../doc/tutorials/features2d/feature_homography/feature_homography.markdown)

## Purpose and Role

This file is located in the `doc/tutorials/features2d/feature_homography` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
Features2D + Homography to find a known object {#tutorial_feature_homography}
==============================================

@tableofcontents

@prev_tutorial{tutorial_feature_flann_matcher}
@next_tutorial{tutorial_detection_of_planar_objects}

|    |    |
| -: | :- |
| Original author | Ana Huamán |
| Compatibility | OpenCV >= 3.0 |

Goal
----

In this tutorial you will learn how to:

-   Use the function @ref cv::findHomography to find the transform between matched keypoints.
-   Use the function @ref cv::perspectiveTransform to map the points.

\warning You need the <a href="https://github.com/opencv/opencv_contrib">OpenCV contrib modules</a> to be able to use the SURF features
(alternatives are ORB, KAZE, ... features).

Theory
------

Code
----

@add_toggle_cpp
This tutorial code's is shown lines below. You can also download it from
[here](https://github.com/opencv/opencv/tree/4.x/samples/cpp/tutorial_code/features2D/feature_homography/SURF_FLANN_matching_homography_Demo.cpp)
@include samples/cpp/tutorial_code/features2D/feature_homography/SURF_FLANN_matching_homography_Demo.cpp
@end_toggle

@add_toggle_java
This tutorial code's is shown lines below. You can also download it from
[here](https://github.com/opencv/opencv/tree/4.x/samples/java/tutorial_code/features2D/feature_homography/SURFFLANNMatchingHomographyDemo.java)
@include samples/java/tutorial_code/features2D/feature_homography/SURFFLANNMatchingHomographyDemo.java
@end_toggle

@add_toggle_python
This tutorial code's is shown lines below. You can also download it from
[here](https://github.com/opencv/opencv/tree/4.x/samples/python/tutorial_code/features2D/feature_homography/SURF_FLANN_matching_homography_Demo.py)
@include samples/python/tutorial_code/features2D/feature_homography/SURF_FLANN_matching_homography_Demo.py
@end_toggle

Explanation
-----------

Result
------

-   And here is the result for the detected object (highlighted in green). Note that since the homography is estimated with a RANSAC approach,
    detected false matches will not impact the homography calculation.

    ![](images/Feature_Homography_Result.jpg)

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

