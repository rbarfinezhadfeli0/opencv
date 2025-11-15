# Documentation for `modules/features2d/misc/python/test/test_feature_homography.py`

## File Metadata

- **Full Path**: `modules/features2d/misc/python/test/test_feature_homography.py`
- **File Name**: `test_feature_homography.py`
- **File Size**: 5,603 bytes
- **File Type**: .py
- **Link to Source**: [modules/features2d/misc/python/test/test_feature_homography.py](../../../../../modules/features2d/misc/python/test/test_feature_homography.py)

## Purpose and Role

This file is located in the `modules/features2d/misc/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

'''
Feature homography
==================

Example of using features2d framework for interactive video homography matching.
ORB features and FLANN matcher are used. The actual tracking is implemented by
PlaneTracker class in plane_tracker.py
'''

# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv
import sys
PY3 = sys.version_info[0] == 3

if PY3:
    xrange = range

# local modules
from tst_scene_render import TestSceneRender

def intersectionRate(s1, s2):

    x1, y1, x2, y2 = s1
    s1 = np.array([[x1, y1], [x2,y1], [x2, y2], [x1, y2]])

    area, _intersection = cv.intersectConvexConvex(s1, np.array(s2))
    return 2 * area / (cv.contourArea(s1) + cv.contourArea(np.array(s2)))

from tests_common import NewOpenCVTests

class feature_homography_test(NewOpenCVTests):

    render = None
    tracker = None
    framesCounter = 0
    frame = None

    def test_feature_homography(self):

        self.render = TestSceneRender(self.get_sample('samples/data/graf1.png'),
            self.get_sample('samples/data/box.png'), noise = 0.5, speed = 0.5)
        self.frame = self.render.getNextFrame()
        self.tracker = PlaneTracker()
        self.tracker.clear()
        self.tracker.add_target(self.frame, self.render.getCurrentRect())

        while self.framesCounter < 100:
            self.framesCounter += 1
            tracked = self.tracker.track(self.frame)
            if len(tracked) > 0:
                tracked = tracked[0]
                self.assertGreater(intersectionRate(self.render.getCurrentRect(), np.int32(tracked.quad)), 0.6)
            else:
                self.assertEqual(0, 1, 'Tracking error')
            self.frame = self.render.getNextFrame()


# built-in modules
from collections import namedtuple

FLANN_INDEX_KDTREE = 1
FLANN_INDEX_LSH    = 6
flann_params= dict(algorithm = FLANN_INDEX_LSH,
                   table_number = 6, # 12
                   key_size = 12,     # 20
                   multi_probe_level = 1) #2

MIN_MATCH_COUNT = 10

'''
  image     - image to track
  rect      - tracked rectangle (x1, y1, x2, y2)
  keypoints - keypoints detected inside rect
  descrs    - their descriptors
  data      - some user-provided data
'''
PlanarTarget = namedtuple('PlaneTarget', 'image, rect, keypoints, descrs, data')

'''
  target - reference to PlanarTarget
  p0     - matched points coords in target image
  p1     - matched points coords in input frame
  H      - homography matrix from p0 to p1
  quad   - target boundary quad in input frame
'''
TrackedTarget = namedtuple('TrackedTarget', 'target, p0, p1, H, quad')

class PlaneTracker:
    def __init__(self):
        self.detector = cv.AKAZE_create(threshold = 0.003)
        self.matcher = cv.FlannBasedMatcher(flann_params, {})  # bug : need to pass empty dict (#1329)
        self.targets = []
        self.frame_points = []

    def add_target(self, image, rect, data=None):
        '''Add a new tracking target.'''
        x0, y0, x1, y1 = rect
        raw_points, raw_descrs = self.detect_features(image)
        points, descs = [], []
        for kp, desc in zip(raw_points, raw_descrs):
            x, y = kp.pt
            if x0 <= x <= x1 and y0 <= y <= y1:
                points.append(kp)
                descs.append(desc)
        descs = np.uint8(descs)
        self.matcher.add([descs])
        target = PlanarTarget(image = image, rect=rect, keypoints = points, descrs=descs, data=data)
        self.targets.append(target)

    def clear(self):
        '''Remove all targets'''
        self.targets = []
        self.matcher.clear()

    def track(self, frame):
        '''Returns a list of detected TrackedTarget objects'''
        self.frame_points, frame_descrs = self.detect_features(frame)
        if len(self.frame_points) < MIN_MATCH_COUNT:
            return []
        matches = self.matcher.knnMatch(frame_descrs, k = 2)
        matches = [m[0] for m in matches if len(m) == 2 and m[0].distance < m[1].distance * 0.75]
        if len(matches) < MIN_MATCH_COUNT:
            return []
        matches_by_id = [[] for _ in xrange(len(self.targets))]
        for m in matches:
            matches_by_id[m.imgIdx].append(m)
        tracked = []
        for imgIdx, matches in enumerate(matches_by_id):
            if len(matches) < MIN_MATCH_COUNT:
                continue
            target = self.targets[imgIdx]
            p0 = [target.keypoints[m.trainIdx].pt for m in matches]
            p1 = [self.frame_points[m.queryIdx].pt for m in matches]
            p0, p1 = np.float32((p0, p1))
            H, status = cv.findHomography(p0, p1, cv.RANSAC, 3.0)
            status = status.ravel() != 0
            if status.sum() < MIN_MATCH_COUNT:
                continue
            p0, p1 = p0[status], p1[status]

            x0, y0, x1, y1 = target.rect
            quad = np.float32([[x0, y0], [x1, y0], [x1, y1], [x0, y1]])
            quad = cv.perspectiveTransform(quad.reshape(1, -1, 2), H).reshape(-1, 2)

            track = TrackedTarget(target=target, p0=p0, p1=p1, H=H, quad=quad)
            tracked.append(track)
        tracked.sort(key = lambda t: len(t.p0), reverse=True)
        return tracked

    def detect_features(self, frame):
        '''detect_features(self, frame) -> keypoints, descrs'''
        keypoints, descrs = self.detector.detectAndCompute(frame, None)
        if descrs is None:  # detectAndCompute returns descs=None if no keypoints found
            descrs = []
        return keypoints, descrs


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

- **PlaneTracker**: A class/struct defined in this file
- **in**: A class/struct defined in this file
- **feature_homography_test**: A class/struct defined in this file

### Functions and Methods

- **track()**: A function/method defined in this file
- **intersectionRate()**: A function/method defined in this file
- **add_target()**: A function/method defined in this file
- **clear()**: A function/method defined in this file
- **detect_features()**: A function/method defined in this file
- **test_feature_homography()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `sys`
- `NewOpenCVTests`
- `TestSceneRender`
- `namedtuple`
- `p0`
- `print_function`
- `tst_scene_render`
- `numpy`
- `cv2`
- `__future__`
- `collections`


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

