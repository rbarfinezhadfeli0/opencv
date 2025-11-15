# Documentation for `docs/samples/python/background_subtractor_mask.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/python/background_subtractor_mask.py_docs.md`
- **File Name**: `background_subtractor_mask.py_docs.md`
- **File Size**: 5,253 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/python/background_subtractor_mask.py_docs.md](../../../docs/samples/python/background_subtractor_mask.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/python` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/python/background_subtractor_mask.py`

## File Metadata

- **Full Path**: `samples/python/background_subtractor_mask.py`
- **File Name**: `background_subtractor_mask.py`
- **File Size**: 2,155 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/background_subtractor_mask.py](../../samples/python/background_subtractor_mask.py)

## Purpose and Role

This file is located in the `samples/python` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```

'''
Showcases the use of background subtraction from a live video feed,
aswell as pass through of a known foreground parameter
'''

# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv

def main():
    cap = cv.VideoCapture(0)
    if not cap.isOpened:
        print("Capture source avaialable.")
        exit()

    # Create background subtractor
    mog2_bg_subtractor = cv.createBackgroundSubtractorMOG2(history=300, varThreshold=50, detectShadows=False)
    knn_bg_subtractor = cv.createBackgroundSubtractorKNN(history=300, detectShadows=False)

    frame_count = 0
    # Allows for a frame buffer for the mask to learn pre known foreground
    show_count = 10

    while True:
        ret, frame = cap.read()
        if not ret:
            break

        x = 100 + (frame_count % 10) * 3

        frame = cv.resize(frame, (640, 480))
        aKnownForegroundMask = np.zeros(frame.shape[:2], dtype=np.uint8)

        # Allow for models to "settle"/learn
        if frame_count > show_count:
            cv.rectangle(aKnownForegroundMask, (x,200), (x+50,300), 255, -1)
            cv.rectangle(aKnownForegroundMask, (540,180), (640,480), 255, -1)

        #MOG2 Subtraction
        mog2_with_mask = mog2_bg_subtractor.apply(frame,knownForegroundMask=aKnownForegroundMask)
        mog2_without_mask = mog2_bg_subtractor.apply(frame)

        #KNN Subtraction
        knn_with_mask = knn_bg_subtractor.apply(frame,knownForegroundMask=aKnownForegroundMask)
        knn_without_mask = knn_bg_subtractor.apply(frame)

        # Display the 3 parameter apply and the 4 parameter apply for both subtractors
        cv.imshow("MOG2 With a Foreground Mask", mog2_with_mask)
        cv.imshow("MOG2 Without a Foreground Mask", mog2_without_mask)
        cv.imshow("KNN With a Foreground Mask", knn_with_mask)
        cv.imshow("KNN Without a Foreground Mask", knn_without_mask)

        key = cv.waitKey(30)
        if key == 27:  # ESC
            break

        frame_count += 1

    cap.release()
    cv.destroyAllWindows()

if __name__ == '__main__':
    print(__doc__)
    main()
    cv.destroyAllWindows()
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

### Functions and Methods

- **main()**: A function/method defined in this file
- **import()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `print_function`
- `numpy`
- `cv2`
- `a`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

