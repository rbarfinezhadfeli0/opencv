# Documentation for `apps/pattern-tools/test_charuco_board.py`

## File Metadata

- **Full Path**: `apps/pattern-tools/test_charuco_board.py`
- **File Name**: `test_charuco_board.py`
- **File Size**: 6,553 bytes
- **File Type**: .py
- **Link to Source**: [apps/pattern-tools/test_charuco_board.py](../../apps/pattern-tools/test_charuco_board.py)

## Purpose and Role

This file is located in the `apps/pattern-tools` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
from __future__ import print_function

import os, tempfile, numpy as np

import sys
import cv2 as cv
from tests_common import NewOpenCVTests
import generate_pattern

class aruco_objdetect_test(NewOpenCVTests):

    def test_aruco_dicts(self):
        try:
            import cairosvg
        except:
            raise self.skipTest("cairosvg library was not found")
        else:
            cols = 3
            rows = 5
            square_size = 100
            aruco_type = [cv.aruco.DICT_4X4_1000, cv.aruco.DICT_5X5_1000, cv.aruco.DICT_6X6_1000,
                        cv.aruco.DICT_7X7_1000, cv.aruco.DICT_ARUCO_ORIGINAL, cv.aruco.DICT_APRILTAG_16h5,
                        cv.aruco.DICT_APRILTAG_25h9, cv.aruco.DICT_APRILTAG_36h10, cv.aruco.DICT_APRILTAG_36h11]
            aruco_type_str = ['DICT_4X4_1000','DICT_5X5_1000', 'DICT_6X6_1000',
                        'DICT_7X7_1000', 'DICT_ARUCO_ORIGINAL', 'DICT_APRILTAG_16h5',
                        'DICT_APRILTAG_25h9', 'DICT_APRILTAG_36h10', 'DICT_APRILTAG_36h11']
            marker_size = 0.8*square_size
            board_width = cols*square_size
            board_height = rows*square_size

            for aruco_type_i in range(len(aruco_type)):
                #draw desk using opencv
                aruco_dict = cv.aruco.getPredefinedDictionary(aruco_type[aruco_type_i])
                board = cv.aruco.CharucoBoard((cols, rows), square_size, marker_size, aruco_dict)
                charuco_detector = cv.aruco.CharucoDetector(board)
                from_cv_img = board.generateImage((cols*square_size, rows*square_size))

                #draw desk using svg
                fd1, filesvg = tempfile.mkstemp(prefix="out", suffix=".svg")
                os.close(fd1)
                fd2, filepng = tempfile.mkstemp(prefix="svg_marker", suffix=".png")
                os.close(fd2)

                try:
                    basedir = os.path.abspath(os.path.dirname(__file__))
                    pm = generate_pattern.PatternMaker(cols, rows, filesvg, "px", square_size, 0, board_width,
                                board_height, "charuco_checkboard", marker_size,
                                os.path.join(basedir, aruco_type_str[aruco_type_i]+'.json.gz'), 0)
                    pm.make_charuco_board()
                    pm.save()
                    cairosvg.svg2png(url=filesvg, write_to=filepng, background_color="white")
                    from_svg_img = cv.imread(filepng)
                    _charucoCorners, _charuco_ids_svg, marker_corners_svg, marker_ids_svg = charuco_detector.detectBoard(from_svg_img)
                    _charucoCorners, _charuco_ids_cv, marker_corners_cv, marker_ids_cv = charuco_detector.detectBoard(from_cv_img)
                    marker_corners_svg_map, marker_corners_cv_map = {}, {}
                    for i in range(len(marker_ids_svg)):
                        marker_corners_svg_map[int(marker_ids_svg[i][0])] = marker_corners_svg[i]
                    for i in range(len(marker_ids_cv)):
                        marker_corners_cv_map[int(marker_ids_cv[i][0])] = marker_corners_cv[i]

                    for key_svg in marker_corners_svg_map.keys():
                        marker_svg = marker_corners_svg_map[key_svg]
                        marker_cv = marker_corners_cv_map[key_svg]
                        np.testing.assert_allclose(marker_svg, marker_cv, 0.1, 0.1)
                finally:
                    if os.path.exists(filesvg):
                        os.remove(filesvg)
                    if os.path.exists(filepng):
                        os.remove(filepng)

    def test_aruco_marker_sizes(self):
        try:
            import cairosvg
        except:
            raise self.skipTest("cairosvg library was not found")
        else:
            cols = 3
            rows = 5
            square_size = 100
            aruco_type =  cv.aruco.DICT_5X5_1000
            aruco_type_str = 'DICT_5X5_1000'
            marker_sizes_rate = [0.25, 0.5, 0.75, 0.9]
            board_width = cols*square_size
            board_height = rows*square_size

            for marker_s_rate in marker_sizes_rate:
                marker_size = marker_s_rate*square_size
                #draw desk using opencv
                aruco_dict = cv.aruco.getPredefinedDictionary(aruco_type)
                board = cv.aruco.CharucoBoard((cols, rows), square_size, marker_size, aruco_dict)
                charuco_detector = cv.aruco.CharucoDetector(board)
                from_cv_img = board.generateImage((cols*square_size, rows*square_size))

                #draw desk using svg
                fd1, filesvg = tempfile.mkstemp(prefix="out", suffix=".svg")
                os.close(fd1)
                fd2, filepng = tempfile.mkstemp(prefix="svg_marker", suffix=".png")
                os.close(fd2)

                try:
                    basedir = os.path.abspath(os.path.dirname(__file__))
                    pm = generate_pattern.PatternMaker(cols, rows, filesvg, "px", square_size, 0, board_width,
                                board_height, "charuco_checkboard", marker_size, os.path.join(basedir, aruco_type_str+'.json.gz'), 0)
                    pm.make_charuco_board()
                    pm.save()
                    cairosvg.svg2png(url=filesvg, write_to=filepng, background_color="white")
                    from_svg_img = cv.imread(filepng)

                    #test
                    _charucoCorners, _charuco_ids_svg, marker_corners_svg, marker_ids_svg = charuco_detector.detectBoard(from_svg_img)
                    _charucoCorners, _charuco_ids_cv, marker_corners_cv, marker_ids_cv = charuco_detector.detectBoard(from_cv_img)
                    marker_corners_svg_map, marker_corners_cv_map = {}, {}
                    for i in range(len(marker_ids_svg)):
                        marker_corners_svg_map[int(marker_ids_svg[i][0])] = marker_corners_svg[i]
                    for i in range(len(marker_ids_cv)):
                        marker_corners_cv_map[int(marker_ids_cv[i][0])] = marker_corners_cv[i]

                    for key_svg in marker_corners_svg_map.keys():
                        marker_svg = marker_corners_svg_map[key_svg]
                        marker_cv = marker_corners_cv_map[key_svg]
                        np.testing.assert_allclose(marker_svg, marker_cv, 0.1, 0.1)
                finally:
                    if os.path.exists(filesvg):
                        os.remove(filesvg)
                    if os.path.exists(filepng):
                        os.remove(filepng)
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

- **aruco_objdetect_test**: A class/struct defined in this file

### Functions and Methods

- **test_aruco_marker_sizes()**: A function/method defined in this file
- **test_aruco_dicts()**: A function/method defined in this file
- **import()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `tests_common`
- `sys`
- `os`
- `NewOpenCVTests`
- `cairosvg`
- `print_function`
- `generate_pattern`
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

