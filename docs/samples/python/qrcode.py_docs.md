# Documentation for `samples/python/qrcode.py`

## File Metadata

- **Full Path**: `samples/python/qrcode.py`
- **File Name**: `qrcode.py`
- **File Size**: 8,828 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/qrcode.py](../../samples/python/qrcode.py)

## Purpose and Role

This file is located in the `samples/python` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

'''
This program detects the QR-codes using OpenCV Library.

Usage:
   qrcode.py
'''


# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv

import argparse
import sys

PY3 = sys.version_info[0] == 3
if PY3:
    xrange = range


class QrSample:
    def __init__(self, args):
        self.fname = ''
        self.fext = ''
        self.fsaveid = 0
        self.input = args.input
        self.detect = args.detect
        self.out = args.out
        self.multi = args.multi
        self.saveDetections = args.save_detections
        self.saveAll = args.save_all
        self.arucoBased = args.aruco_based

    def getQRModeString(self):
        msg1 = "multi " if self.multi else ""
        msg2 = "detector" if self.detect else "decoder"
        msg = "QR {:s}{:s}".format(msg1, msg2)
        return msg

    def drawFPS(self, result, fps):
        message = '{:.2f} FPS({:s})'.format(fps, self.getQRModeString())
        cv.putText(result, message, (20, 20), 1,
                   cv.FONT_HERSHEY_DUPLEX, (0, 0, 255))

    def drawQRCodeContours(self, image, cnt):
        if cnt.size != 0:
            rows, cols, _ = image.shape
            show_radius = 2.813 * ((rows / cols) if rows > cols else (cols / rows))
            contour_radius = show_radius * 0.4
            cv.drawContours(image, [cnt], 0, (0, 255, 0), int(round(contour_radius)))
            tpl = cnt.reshape((-1, 2))
            for x in tuple(tpl.tolist()):
                color = (255, 0, 0)
                cv.circle(image, tuple(x), int(round(contour_radius)), color, -1)

    def drawQRCodeResults(self, result, points, decode_info, fps):
        n = len(points)
        if isinstance(decode_info, str):
            decode_info = [decode_info]
        if n > 0:
            for i in range(n):
                cnt = np.array(points[i]).reshape((-1, 1, 2)).astype(np.int32)
                self.drawQRCodeContours(result, cnt)
                msg = 'QR[{:d}]@{} : '.format(i, *(cnt.reshape(1, -1).tolist()))
                print(msg, end="")
                if len(decode_info) > i:
                    if decode_info[i]:
                        print("'", decode_info[i], "'")
                    else:
                        print("Can't decode QR code")
                else:
                    print("Decode information is not available (disabled)")
        else:
            print("QRCode not detected!")
        self.drawFPS(result, fps)

    def runQR(self, qrCode, inputimg):
        if not self.multi:
            if not self.detect:
                decode_info, points, _ = qrCode.detectAndDecode(inputimg)
                dec_info = decode_info
            else:
                _, points = qrCode.detect(inputimg)
                dec_info = []
        else:
            if not self.detect:
                _, decode_info, points, _ = qrCode.detectAndDecodeMulti(
                    inputimg)
                dec_info = decode_info
            else:
                _, points = qrCode.detectMulti(inputimg)
                dec_info = []
        if points is None:
            points = []
        return points, dec_info

    def DetectQRFrmImage(self, inputfile):
        inputimg = cv.imread(inputfile, cv.IMREAD_COLOR)
        if inputimg is None:
            print('ERROR: Can not read image: {}'.format(inputfile))
            return
        print('Run {:s} on image [{:d}x{:d}]'.format(
            self.getQRModeString(), inputimg.shape[1], inputimg.shape[0]))

        if self.arucoBased:
            qrCode = cv.QRCodeDetectorAruco()
        else:
            qrCode = cv.QRCodeDetector()

        count = 10
        timer = cv.TickMeter()
        for _ in range(count):
            timer.start()
            points, decode_info = self.runQR(qrCode, inputimg)
            timer.stop()
        fps = count / timer.getTimeSec()
        print('FPS: {}'.format(fps))
        result = inputimg
        self.drawQRCodeResults(result, points, decode_info, fps)
        cv.imshow("QR", result)
        cv.waitKey(1)
        if self.out != '':
            outfile = self.fname + self.fext
            print("Saving Result: {}".format(outfile))
            cv.imwrite(outfile, result)

        print("Press any key to exit ...")
        cv.waitKey(0)
        print("Exit")

    def processQRCodeDetection(self, qrcode, frame):
        if len(frame.shape) == 2:
            result = cv.cvtColor(frame, cv.COLOR_GRAY2BGR)
        else:
            result = frame
        print('Run {:s} on video frame [{:d}x{:d}]'.format(
            self.getQRModeString(), frame.shape[1], frame.shape[0]))
        timer = cv.TickMeter()
        timer.start()
        points, decode_info = self.runQR(qrcode, frame)
        timer.stop()

        fps = 1 / timer.getTimeSec()
        self.drawQRCodeResults(result, points, decode_info, fps)
        return fps, result, points

    def DetectQRFrmCamera(self):
        cap = cv.VideoCapture(0)
        if not cap.isOpened():
            print("Cannot open the camera")
            return
        print("Press 'm' to switch between detectAndDecode and detectAndDecodeMulti")
        print("Press 'd' to switch between decoder and detector")
        print("Press ' ' (space) to save result into images")
        print("Press 'ESC' to exit")

        if self.arucoBased:
            qrcode = cv.QRCodeDetectorAruco()
        else:
            qrcode = cv.QRCodeDetector()

        while True:
            ret, frame = cap.read()
            if not ret:
                print("End of video stream")
                break
            forcesave = self.saveAll
            result = frame
            try:
                fps, result, corners = self.processQRCodeDetection(qrcode, frame)
                print('FPS: {:.2f}'.format(fps))
                forcesave |= self.saveDetections and (len(corners) != 0)
            except cv.error as e:
                print("Error exception: ", e)
                forcesave = True
            cv.imshow("QR code", result)
            code = cv.waitKey(1)
            if code < 0 and (not forcesave):
                continue
            if code == ord(' ') or forcesave:
                fsuffix = '-{:05d}'.format(self.fsaveid)
                self.fsaveid += 1
                fname_in = self.fname + fsuffix + "_input.png"
                print("Saving QR code detection result: '{}' ...".format(fname_in))
                cv.imwrite(fname_in, frame)
                print("Saved")
            if code == ord('m'):
                self.multi = not self.multi
                msg = 'Switching QR code mode ==> {:s}'.format(
                    "detectAndDecodeMulti" if self.multi else "detectAndDecode")
                print(msg)
            if code == ord('d'):
                self.detect = not self.detect
                msg = 'Switching QR code mode ==> {:s}'.format(
                    "detect" if self.detect else "decode")
                print(msg)
            if code == 27:
                print("'ESC' is pressed. Exiting...")
                break
        print("Exit.")


def main():
    parser = argparse.ArgumentParser(
        description='This program detects the QR-codes input images using OpenCV Library.')
    parser.add_argument(
        '-i',
        '--input',
        help="input image path (for example, 'opencv_extra/testdata/cv/qrcode/multiple/*_qrcodes.png)",
        default="",
        metavar="")
    parser.add_argument(
        '--aruco_based',
        help="use aruco-based detector",
        action='store_true')
    parser.add_argument(
        '-d',
        '--detect',
        help="detect QR code only (skip decoding) (default: False)",
        action='store_true')
    parser.add_argument(
        '-m',
        '--multi',
        help="enable multiple qr-codes detection",
        action='store_true')
    parser.add_argument(
        '-o',
        '--out',
        help="path to result file (default: qr_code.png)",
        default="qr_code.png",
        metavar="")
    parser.add_argument(
        '--save_detections',
        help="save all QR detections (video mode only)",
        action='store_true')
    parser.add_argument(
        '--save_all',
        help="save all processed frames (video mode only)",
        action='store_true')
    args = parser.parse_args()
    qrinst = QrSample(args)
    if args.out != '':
        index = args.out.rfind('.')
        if index != -1:
            qrinst.fname = args.out[:index]
            qrinst.fext = args.out[index:]
        else:
            qrinst.fname = args.out
            qrinst.fext = ".png"
    if args.input != '':
        qrinst.DetectQRFrmImage(args.input)
    else:
        qrinst.DetectQRFrmCamera()


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

### Classes and Structures

- **QrSample**: A class/struct defined in this file

### Functions and Methods

- **DetectQRFrmImage()**: A function/method defined in this file
- **drawQRCodeResults()**: A function/method defined in this file
- **drawFPS()**: A function/method defined in this file
- **drawQRCodeContours()**: A function/method defined in this file
- **getQRModeString()**: A function/method defined in this file
- **DetectQRFrmCamera()**: A function/method defined in this file
- **processQRCodeDetection()**: A function/method defined in this file
- **main()**: A function/method defined in this file
- **runQR()**: A function/method defined in this file
- **import()**: A function/method defined in this file
- **__init__()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `sys`
- `print_function`
- `numpy`
- `cv2`
- `argparse`
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

