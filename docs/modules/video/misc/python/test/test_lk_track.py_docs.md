# Documentation for `modules/video/misc/python/test/test_lk_track.py`

## File Metadata

- **Full Path**: `modules/video/misc/python/test/test_lk_track.py`
- **File Name**: `test_lk_track.py`
- **File Size**: 3,923 bytes
- **File Type**: .py
- **Link to Source**: [modules/video/misc/python/test/test_lk_track.py](../../../../../modules/video/misc/python/test/test_lk_track.py)

## Purpose and Role

This file is located in the `modules/video/misc/python/test` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

'''
Lucas-Kanade tracker
====================

Lucas-Kanade sparse optical flow demo. Uses goodFeaturesToTrack
for track initialization and back-tracking for match verification
between frames.
'''

# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv

#local modules
from tst_scene_render import TestSceneRender
from tests_common import NewOpenCVTests, intersectionRate, isPointInRect

lk_params = dict( winSize  = (15, 15),
                  maxLevel = 2,
                  criteria = (cv.TERM_CRITERIA_EPS | cv.TERM_CRITERIA_COUNT, 10, 0.03))

feature_params = dict( maxCorners = 500,
                       qualityLevel = 0.3,
                       minDistance = 7,
                       blockSize = 7 )

def getRectFromPoints(points):

    distances = []
    for point in points:
        distances.append(cv.norm(point, cv.NORM_L2))

    x0, y0 = points[np.argmin(distances)]
    x1, y1 = points[np.argmax(distances)]

    return np.array([x0, y0, x1, y1])


class lk_track_test(NewOpenCVTests):

    track_len = 10
    detect_interval = 5
    tracks = []
    frame_idx = 0
    render = None

    def test_lk_track(self):

        self.render = TestSceneRender(self.get_sample('samples/data/graf1.png'), self.get_sample('samples/data/box.png'))
        self.runTracker()

    def runTracker(self):
        foregroundPointsNum = 0

        while True:
            frame = self.render.getNextFrame()
            frame_gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)

            if len(self.tracks) > 0:
                img0, img1 = self.prev_gray, frame_gray
                p0 = np.float32([tr[-1][0] for tr in self.tracks]).reshape(-1, 1, 2)
                p1, _st, _err = cv.calcOpticalFlowPyrLK(img0, img1, p0, None, **lk_params)
                p0r, _st, _err = cv.calcOpticalFlowPyrLK(img1, img0, p1, None, **lk_params)
                d = abs(p0-p0r).reshape(-1, 2).max(-1)
                good = d < 1
                new_tracks = []
                for tr, (x, y), good_flag in zip(self.tracks, p1.reshape(-1, 2), good):
                    if not good_flag:
                        continue
                    tr.append([(x, y), self.frame_idx])
                    if len(tr) > self.track_len:
                        del tr[0]
                    new_tracks.append(tr)
                self.tracks = new_tracks

            if self.frame_idx % self.detect_interval == 0:
                goodTracksCount = 0
                for tr in self.tracks:
                    oldRect = self.render.getRectInTime(self.render.timeStep * tr[0][1])
                    newRect = self.render.getRectInTime(self.render.timeStep * tr[-1][1])
                    if isPointInRect(tr[0][0], oldRect) and isPointInRect(tr[-1][0], newRect):
                        goodTracksCount += 1

                if self.frame_idx == self.detect_interval:
                    foregroundPointsNum = goodTracksCount

                fgIndex = float(foregroundPointsNum) / (foregroundPointsNum + 1)
                fgRate = float(goodTracksCount) / (len(self.tracks) + 1)

                if self.frame_idx > 0:
                    self.assertGreater(fgIndex, 0.9)
                    self.assertGreater(fgRate, 0.2)

                mask = np.zeros_like(frame_gray)
                mask[:] = 255
                for x, y in [np.int32(tr[-1][0]) for tr in self.tracks]:
                    cv.circle(mask, (x, y), 5, 0, -1)
                p = cv.goodFeaturesToTrack(frame_gray, mask = mask, **feature_params)
                if p is not None:
                    for x, y in np.float32(p).reshape(-1, 2):
                        self.tracks.append([[(x, y), self.frame_idx]])

            self.frame_idx += 1
            self.prev_gray = frame_gray

            if self.frame_idx > 300:
                break


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

- **lk_track_test**: A class/struct defined in this file

### Functions and Methods

- **getRectFromPoints()**: A function/method defined in this file
- **test_lk_track()**: A function/method defined in this file
- **runTracker()**: A function/method defined in this file
- **import()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `NewOpenCVTests`
- `TestSceneRender`
- `print_function`
- `tst_scene_render`
- `numpy`
- `cv2`
- `__future__`


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

