# Documentation for `docs/samples/python/deconvolution.py_docs.md`

## File Metadata

- **Full Path**: `docs/samples/python/deconvolution.py_docs.md`
- **File Name**: `deconvolution.py_docs.md`
- **File Size**: 7,105 bytes
- **File Type**: .md
- **Link to Source**: [docs/samples/python/deconvolution.py_docs.md](../../../docs/samples/python/deconvolution.py_docs.md)

## Purpose and Role

This file is located in the `docs/samples/python` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

# Documentation for `samples/python/deconvolution.py`

## File Metadata

- **Full Path**: `samples/python/deconvolution.py`
- **File Name**: `deconvolution.py`
- **File Size**: 3,809 bytes
- **File Type**: .py
- **Link to Source**: [samples/python/deconvolution.py](../../samples/python/deconvolution.py)

## Purpose and Role

This file is located in the `samples/python` directory and serves as part of the OpenCV library infrastructure.

## Original Source Code

The following is the complete source code of this file:

```
#!/usr/bin/env python

'''
Wiener deconvolution.

Sample shows how DFT can be used to perform Weiner deconvolution [1]
of an image with user-defined point spread function (PSF)

Usage:
  deconvolution.py  [--circle]
      [--angle <degrees>]
      [--d <diameter>]
      [--snr <signal/noise ratio in db>]
      [<input image>]

  Use sliders to adjust PSF paramitiers.
  Keys:
    SPACE - switch btw linear/circular PSF
    ESC   - exit

Examples:
  deconvolution.py --angle 135 --d 22  licenseplate_motion.jpg
    (image source: http://www.topazlabs.com/infocus/_images/licenseplate_compare.jpg)

  deconvolution.py --angle 86 --d 31  text_motion.jpg
  deconvolution.py --circle --d 19  text_defocus.jpg
    (image source: compact digital photo camera, no artificial distortion)


[1] http://en.wikipedia.org/wiki/Wiener_deconvolution
'''

# Python 2/3 compatibility
from __future__ import print_function

import numpy as np
import cv2 as cv

# local module
from common import nothing


def blur_edge(img, d=31):
    h, w  = img.shape[:2]
    img_pad = cv.copyMakeBorder(img, d, d, d, d, cv.BORDER_WRAP)
    img_blur = cv.GaussianBlur(img_pad, (2*d+1, 2*d+1), -1)[d:-d,d:-d]
    y, x = np.indices((h, w))
    dist = np.dstack([x, w-x-1, y, h-y-1]).min(-1)
    w = np.minimum(np.float32(dist)/d, 1.0)
    return img*w + img_blur*(1-w)

def motion_kernel(angle, d, sz=65):
    kern = np.ones((1, d), np.float32)
    c, s = np.cos(angle), np.sin(angle)
    A = np.float32([[c, -s, 0], [s, c, 0]])
    sz2 = sz // 2
    A[:,2] = (sz2, sz2) - np.dot(A[:,:2], ((d-1)*0.5, 0))
    kern = cv.warpAffine(kern, A, (sz, sz), flags=cv.INTER_CUBIC)
    return kern

def defocus_kernel(d, sz=65):
    kern = np.zeros((sz, sz), np.uint8)
    cv.circle(kern, (sz, sz), d, 255, -1, cv.LINE_AA, shift=1)
    kern = np.float32(kern) / 255.0
    return kern


def main():
    import sys, getopt
    opts, args = getopt.getopt(sys.argv[1:], '', ['circle', 'angle=', 'd=', 'snr='])
    opts = dict(opts)
    try:
        fn = args[0]
    except:
        fn = 'licenseplate_motion.jpg'

    win = 'deconvolution'

    img = cv.imread(cv.samples.findFile(fn), cv.IMREAD_GRAYSCALE)
    if img is None:
        print('Failed to load file:', fn)
        sys.exit(1)

    img = np.float32(img)/255.0
    cv.imshow('input', img)

    img = blur_edge(img)
    IMG = cv.dft(img, flags=cv.DFT_COMPLEX_OUTPUT)

    defocus = '--circle' in opts

    def update(_):
        ang = np.deg2rad( cv.getTrackbarPos('angle', win) )
        d = cv.getTrackbarPos('d', win)
        noise = 10**(-0.1*cv.getTrackbarPos('SNR (db)', win))

        if defocus:
            psf = defocus_kernel(d)
        else:
            psf = motion_kernel(ang, d)
        cv.imshow('psf', psf)

        psf /= psf.sum()
        psf_pad = np.zeros_like(img)
        kh, kw = psf.shape
        psf_pad[:kh, :kw] = psf
        PSF = cv.dft(psf_pad, flags=cv.DFT_COMPLEX_OUTPUT, nonzeroRows = kh)
        PSF2 = (PSF**2).sum(-1)
        iPSF = PSF / (PSF2 + noise)[...,np.newaxis]
        RES = cv.mulSpectrums(IMG, iPSF, 0)
        res = cv.idft(RES, flags=cv.DFT_SCALE | cv.DFT_REAL_OUTPUT )
        res = np.roll(res, -kh//2, 0)
        res = np.roll(res, -kw//2, 1)
        cv.imshow(win, res)

    cv.namedWindow(win)
    cv.namedWindow('psf', 0)
    cv.createTrackbar('angle', win, int(opts.get('--angle', 135)), 180, update)
    cv.createTrackbar('d', win, int(opts.get('--d', 22)), 50, update)
    cv.createTrackbar('SNR (db)', win, int(opts.get('--snr', 25)), 50, update)
    update(None)

    while True:
        ch = cv.waitKey()
        if ch == 27:
            break
        if ch == ord(' '):
            defocus = not defocus
            update(None)

    print('Done')


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

- **blur_edge()**: A function/method defined in this file
- **motion_kernel()**: A function/method defined in this file
- **defocus_kernel()**: A function/method defined in this file
- **main()**: A function/method defined in this file
- **update()**: A function/method defined in this file
- **import()**: A function/method defined in this file


## Design and Architecture

This file is part of the larger OpenCV architecture. It contributes to the overall functionality by providing specific implementations and interfaces.

### Dependencies

**Python Imports:**
- `nothing`
- `sys`
- `print_function`
- `common`
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



## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

