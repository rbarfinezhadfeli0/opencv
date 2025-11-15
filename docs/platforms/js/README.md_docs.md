# Documentation for `platforms/js/README.md`

## File Metadata

- **Full Path**: `platforms/js/README.md`
- **File Name**: `README.md`
- **File Size**: 540 bytes
- **File Type**: .md
- **Link to Source**: [platforms/js/README.md](../../platforms/js/README.md)

## Purpose and Role

This file is located in the `platforms/js` directory and serves as part of the OpenCV library infrastructure.

## Documentation Content

Building OpenCV.js by Emscripten
====================

[Download and install Emscripten](https://emscripten.org/docs/getting_started/downloads.html).

Execute `build_js.py` script:
```
emcmake python <opencv_src_dir>/platforms/js/build_js.py <build_dir>
```

If everything is fine, a few minutes later you will get `<build_dir>/bin/opencv.js`. You can add this into your web pages.

Find out more build options by `-h` switch.

For detailed build tutorial, check out `<opencv_src_dir>/doc/js_tutorials/js_setup/js_setup/js_setup.markdown`.


## Documentation Purpose

This file provides documentation, guides, or README information for users and developers of OpenCV.

