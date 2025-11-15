# Documentation for `modules/python/package/template/config.py.in`

## File Metadata

- **Full Path**: `modules/python/package/template/config.py.in`
- **File Name**: `config.py.in`
- **File Size**: 82 bytes
- **File Type**: .in
- **Link to Source**: [modules/python/package/template/config.py.in](../../../../modules/python/package/template/config.py.in)

## Purpose and Role

This file is located in the `modules/python/package/template` directory and serves as part of the OpenCV library infrastructure.

## File Content

```
import os

BINARIES_PATHS = [
    @CMAKE_PYTHON_BINARIES_PATH@
] + BINARIES_PATHS

```

## General Information

This file is part of the OpenCV repository infrastructure.

