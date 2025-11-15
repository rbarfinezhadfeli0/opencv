# Documentation for `modules/stitching/misc/python/test/test_stitching.py`

## File Metadata

- **Full Path**: `modules/stitching/misc/python/test/test_stitching.py`
- **File Name**: `test_stitching.py`
- **File Size**: 6,525 bytes
- **File Type**: .py
- **Link to Source**: [modules/stitching/misc/python/test/test_stitching.py](../../../../../modules/stitching/misc/python/test/test_stitching.py)

## Purpose and Role

This file is located in the `modules/stitching/misc/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python
import cv2 as cv
import numpy as np

from tests_common import NewOpenCVTests

class stitching_test(NewOpenCVTests):

    def test_simple(self):

        img1 = self.get_sample('stitching/a1.png')
        img2 = self.get_sample('stitching/a2.png')

        stitcher = cv.Stitcher.create(cv.Stitcher_PANORAMA)
        (_result, pano) = stitcher.stitch((img1, img2))

        #cv.imshow("pano", pano)
        #cv.waitKey()

        self.assertAlmostEqual(pano.shape[0], 685, delta=100, msg="rows: %r" % list(pano.shape))
        self.assertAlmostEqual(pano.shape[1], 1025, delta=100, msg="cols: %r" % list(pano.shape))


class stitching_detail_test(NewOpenCVTests):

    def test_simple(self):
        img = self.get_sample('stitching/a1.png')
        finder= cv.ORB.create()
        imgFea = cv.detail.computeImageFeatures2(finder,img)
        self.assertIsNotNone(imgFea)

        # Added Test for PR #21180
        self.assertIsNotNone(imgFea.keypoints)

        matcher = cv.detail_BestOf2NearestMatcher(False, 0.3)
        self.assertIsNotNone(matcher)
        matcher = cv.detail_AffineBestOf2NearestMatcher(False, False, 0.3)
        self.assertIsNotNone(matcher)
        matcher = cv.detail_BestOf2NearestRangeMatcher(2, False, 0.3)
        self.assertIsNotNone(matcher)
        estimator = cv.detail_AffineBasedEstimator()
        self.assertIsNotNone(estimator)
        estimator = cv.detail_HomographyBasedEstimator()
        self.assertIsNotNone(estimator)

        adjuster = cv.detail_BundleAdjusterReproj()
        self.assertIsNotNone(adjuster)
        adjuster = cv.detail_BundleAdjusterRay()
        self.assertIsNotNone(adjuster)
        adjuster = cv.detail_BundleAdjusterAffinePartial()
        self.assertIsNotNone(adjuster)
        adjuster = cv.detail_NoBundleAdjuster()
        self.assertIsNotNone(adjuster)

        compensator=cv.detail.ExposureCompensator_createDefault(cv.detail.ExposureCompensator_NO)
        self.assertIsNotNone(compensator)
        compensator=cv.detail.ExposureCompensator_createDefault(cv.detail.ExposureCompensator_GAIN)
        self.assertIsNotNone(compensator)
        compensator=cv.detail.ExposureCompensator_createDefault(cv.detail.ExposureCompensator_GAIN_BLOCKS)
        self.assertIsNotNone(compensator)

        seam_finder = cv.detail.SeamFinder_createDefault(cv.detail.SeamFinder_NO)
        self.assertIsNotNone(seam_finder)
        seam_finder = cv.detail.SeamFinder_createDefault(cv.detail.SeamFinder_NO)
        self.assertIsNotNone(seam_finder)
        seam_finder = cv.detail.SeamFinder_createDefault(cv.detail.SeamFinder_VORONOI_SEAM)
        self.assertIsNotNone(seam_finder)

        seam_finder = cv.detail_GraphCutSeamFinder("COST_COLOR")
        self.assertIsNotNone(seam_finder)
        seam_finder = cv.detail_GraphCutSeamFinder("COST_COLOR_GRAD")
        self.assertIsNotNone(seam_finder)
        seam_finder = cv.detail_DpSeamFinder("COLOR")
        self.assertIsNotNone(seam_finder)
        seam_finder = cv.detail_DpSeamFinder("COLOR_GRAD")
        self.assertIsNotNone(seam_finder)

        blender = cv.detail.Blender_createDefault(cv.detail.Blender_NO)
        self.assertIsNotNone(blender)
        blender = cv.detail.Blender_createDefault(cv.detail.Blender_FEATHER)
        self.assertIsNotNone(blender)
        blender = cv.detail.Blender_createDefault(cv.detail.Blender_MULTI_BAND)
        self.assertIsNotNone(blender)

        timelapser = cv.detail.Timelapser_createDefault(cv.detail.Timelapser_AS_IS);
        self.assertIsNotNone(timelapser)
        timelapser = cv.detail.Timelapser_createDefault(cv.detail.Timelapser_CROP);
        self.assertIsNotNone(timelapser)


class stitching_compose_panorama_test_no_args(NewOpenCVTests):

    def test_simple(self):

        img1 = self.get_sample('stitching/a1.png')
        img2 = self.get_sample('stitching/a2.png')

        stitcher = cv.Stitcher.create(cv.Stitcher_PANORAMA)

        stitcher.estimateTransform((img1, img2))

        result, _ = stitcher.composePanorama()

        assert result == 0


class stitching_compose_panorama_args(NewOpenCVTests):

    def test_simple(self):

        img1 = self.get_sample('stitching/a1.png')
        img2 = self.get_sample('stitching/a2.png')

        stitcher = cv.Stitcher.create(cv.Stitcher_PANORAMA)

        stitcher.estimateTransform((img1, img2))
        result, _ = stitcher.composePanorama((img1, img2))

        assert result == 0


class stitching_matches_info_test(NewOpenCVTests):

    def test_simple(self):
        finder = cv.ORB.create()
        img1 = self.get_sample('stitching/a1.png')
        img2 = self.get_sample('stitching/a2.png')

        img_feat1 = cv.detail.computeImageFeatures2(finder, img1)
        img_feat2 = cv.detail.computeImageFeatures2(finder, img2)

        matcher = cv.detail.BestOf2NearestMatcher_create()
        matches_info = matcher.apply(img_feat1, img_feat2)

        self.assertIsNotNone(matches_info.matches)
        self.assertIsNotNone(matches_info.inliers_mask)

class stitching_range_matcher_test(NewOpenCVTests):

    def test_simple(self):
        images = [
            self.get_sample('stitching/a1.png'),
            self.get_sample('stitching/a2.png'),
            self.get_sample('stitching/a3.png')
        ]

        orb = cv.ORB_create()

        features = [cv.detail.computeImageFeatures2(orb, img) for img in images]

        matcher = cv.detail_BestOf2NearestRangeMatcher(range_width=1)
        matches = matcher.apply2(features)

        # matches[1] is image 0 and image 1, should have non-zero confidence
        self.assertNotEqual(matches[1].confidence, 0)

        # matches[2] is image 0 and image 2, should have zero confidence due to range_width=1
        self.assertEqual(matches[2].confidence, 0)


class stitching_seam_finder_graph_cuts(NewOpenCVTests):

    def test_simple(self):
        images = [
            self.get_sample('stitching/a1.png'),
            self.get_sample('stitching/a2.png'),
            self.get_sample('stitching/a3.png')
        ]

        images = [cv.resize(img, [100, 100]) for img in images]

        finder = cv.detail_GraphCutSeamFinder('COST_COLOR_GRAD')
        masks = [cv.UMat(255 * np.ones((img.shape[0], img.shape[1]), np.uint8)) for img in images]
        images_f = [img.astype(np.float32) for img in images]
        masks_warped = finder.find(images_f, [(0, 0), (75, 0), (150, 0)], masks)

        self.assertIsNotNone(masks_warped)


if __name__ == '__main__':
    NewOpenCVTests.bootstrap()
```

## High-Level Overview

This is a Python file that may contain scripts, bindings, or utilities.

**Key Characteristics:**
- May provide Python bindings to C++ code
- Could be a utility script for build/test automation
- Might implement examples or tutorials
- Uses Python idioms and standard library


## Detailed Walkthrough

This section provides an in-depth examination of the code structure, logic, and implementation details.

### Classes and Structures

- **stitching_compose_panorama_args**: A class/struct defined in this file
- **stitching_detail_test**: A class/struct defined in this file
- **stitching_matches_info_test**: A class/struct defined in this file
- **stitching_test**: A class/struct defined in this file
- **stitching_range_matcher_test**: A class/struct defined in this file
- **stitching_compose_panorama_test_no_args**: A class/struct defined in this file
- **stitching_seam_finder_graph_cuts**: A class/struct defined in this file

### Functions and Methods

- **test_simple()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `numpy`
- `cv2`
- `NewOpenCVTests`


### Architectural Role

This file operates within the OpenCV module system, interfacing with other components through well-defined APIs and data structures.

## Performance and Complexity

### Computational Complexity

The algorithms and data structures in this file have various complexity characteristics depending on the operations performed.

### Memory Considerations

Memory usage patterns depend on the specific functionality implemented, including stack allocations, heap allocations, and resource management strategies.

### Performance Optimization

OpenCV employs various optimization techniques including:
- SIMD vectorization where applicable
- Multi-threading support
- Hardware acceleration (CUDA, OpenCL, etc.)
- Efficient memory access patterns

## Security and Safety Considerations

### Potential Vulnerabilities

Code that processes external data should be carefully reviewed for:
- Buffer overflow vulnerabilities
- Integer overflow/underflow
- Input validation issues
- Resource exhaustion attacks

### Safety Measures

OpenCV includes various safety mechanisms:
- Bounds checking in debug builds
- Exception handling
- Resource management (RAII in C++)
- Input sanitization

## Testing and Usage

### How to Use This File

This file is typically used as part of the larger OpenCV library and is not intended to be used in isolation.

### Testing Approach

Testing should cover:
- Unit tests for individual functions
- Integration tests for component interactions
- Performance benchmarks
- Edge case validation

## Related Files

This file is related to other files in the same module and may interact with files in other modules.

