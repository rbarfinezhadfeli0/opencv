# Documentation for `docs/doc/tutorials/features2d/feature_description/feature_description.markdown_docs.md`

## File Metadata

- **Full Path**: `docs/doc/tutorials/features2d/feature_description/feature_description.markdown_docs.md`
- **File Name**: `feature_description.markdown_docs.md`
- **File Size**: 2,875 bytes
- **File Type**: .md
- **Link to Source**: [docs/doc/tutorials/features2d/feature_description/feature_description.markdown_docs.md](../../../../../docs/doc/tutorials/features2d/feature_description/feature_description.markdown_docs.md)

## Purpose and Role

This file is located in the `docs/doc/tutorials/features2d/feature_description` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `doc/tutorials/features2d/feature_description/feature_description.markdown`

## File Metadata

- **Full Path**: `doc/tutorials/features2d/feature_description/feature_description.markdown`
- **File Name**: `feature_description.markdown`
- **File Size**: 2,102 bytes
- **File Type**: .markdown
- **Link to Source**: [doc/tutorials/features2d/feature_description/feature_description.markdown](../../../../doc/tutorials/features2d/feature_description/feature_description.markdown)

## Purpose and Role

This file is located in the `doc/tutorials/features2d/feature_description` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
Feature Description {#tutorial_feature_description}
===================

@tableofcontents

@prev_tutorial{tutorial_feature_detection}
@next_tutorial{tutorial_feature_flann_matcher}

|    |    |
| -: | :- |
| Original author | Ana Huamán |
| Compatibility | OpenCV >= 3.0 |

Goal
----

In this tutorial you will learn how to:

-   Use the @ref cv::DescriptorExtractor interface in order to find the feature vector correspondent
    to the keypoints. Specifically:
    -   Use cv::xfeatures2d::SURF and its function cv::xfeatures2d::SURF::compute to perform the
        required calculations.
    -   Use a @ref cv::DescriptorMatcher to match the features vector
    -   Use the function @ref cv::drawMatches to draw the detected matches.

\warning You need the <a href="https://github.com/opencv/opencv_contrib">OpenCV contrib modules</a> to be able to use the SURF features
(alternatives are ORB, KAZE, ... features).

Theory
------

Code
----

@add_toggle_cpp
This tutorial code's is shown lines below. You can also download it from
[here](https://github.com/opencv/opencv/tree/4.x/samples/cpp/tutorial_code/features2D/feature_description/SURF_matching_Demo.cpp)
@include samples/cpp/tutorial_code/features2D/feature_description/SURF_matching_Demo.cpp
@end_toggle

@add_toggle_java
This tutorial code's is shown lines below. You can also download it from
[here](https://github.com/opencv/opencv/tree/4.x/samples/java/tutorial_code/features2D/feature_description/SURFMatchingDemo.java)
@include samples/java/tutorial_code/features2D/feature_description/SURFMatchingDemo.java
@end_toggle

@add_toggle_python
This tutorial code's is shown lines below. You can also download it from
[here](https://github.com/opencv/opencv/tree/4.x/samples/python/tutorial_code/features2D/feature_description/SURF_matching_Demo.py)
@include samples/python/tutorial_code/features2D/feature_description/SURF_matching_Demo.py
@end_toggle

Explanation
-----------

Result
------

Here is the result after applying the BruteForce matcher between the two original images:

![](images/Feature_Description_BruteForce_Result.jpg)

```

## General Information

This file is part of the OpenCV repository infrastructure.



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

